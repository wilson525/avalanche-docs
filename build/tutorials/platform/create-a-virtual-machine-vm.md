# 创建虚拟机 \(VM\)

_下面的代码稍微有点过时。一些方法、接口和实现与本教程稍有不同。但由于当前代码非常相似，并且本教程在演示Avalanche的虚拟机模型如何工作方面依然有效，因此我们将不再赘述。_

## 简介

Avalanche的核心特征之一是创建新的自定义区块链，这些区块链由[虚拟机 \(VMs\)](../../../learn/platform-overview/#virtual-machines)


在本教程中，我们将创建一个非常简单的虚拟机。虚拟机定义的区块链是一个时间戳服务器。区块链中的每个区块都包含创建时的时间戳和一个32字节的数据段\(关键信息\)。每个区块的时间戳都在其上一个区块的时间戳之后。

这样的服务器非常有用，因为它可以用来证明一份数据在创建区块链时就已经存在。假如您有一本书的手稿，并且您希望将来能够证明这份手稿在今天就已经存在。您可以在区块链中添加一个区块，区块的关键信息是您手稿的哈希值。将来，您就可以通过展示区块的关键信息中有您手稿的哈希值来证明这份手稿在今天就已经存在\(这是因为要找到哈希值的原像是不可能的\）。

在讨论虚拟机的实现之前，我们先来看一下虚拟机为与平台的Avalanche共识引擎兼容而必须实现的接口。我们将展示并解释代码片段中的所有代码。如果您想查看完整代码，而非代码片段，可参阅我们的 [GitHub 库](https://github.com/ava-labs/avalanchego/tree/master/vms/timestampvm)

##  `snowman.VM` 接口

为了就线性区块链 \(相对于DAG区块链而言\), 达成共识，Avalanche采用Snowman共识协议。为了与Snowman兼容，定义区块链的虚拟机必须实现`snowman.VM` 接口，我们在[`github.com/ava-labs/avalanchego/blob/master/snow/engine/snowman/block/vm.go`](https://github.com/ava-labs/avalanchego/blob/master/snow/engine/snowman/block/vm.go)的声明中包含了这个接口。

接口很多，但是不用担心，我们将对每种方法进行解释并观看一个实现示例。您没必要了解每一个细节。

```cpp
// ChainVM defines the methods a Virtual Machine must implement to use the Snowman consensus engine.
//
// A Snowman VM defines the state contained in a linear blockchain,
// the state transition functions that modify the blockchain's state,
// the API exposed by the blockchain, as well as other aspects of the blockchain.
type ChainVM interface {
    // Initialize an instance of the blockchain defined by this VM.
    // [ctx]: Run-time context and metadata about the blockchain.
    //     [ctx.networkID]: The ID of the network this blockchain exists on.
    //     [ctx.chainID]: The unique ID of this blockchain.
    //     [ctx.Log]: Used to log messages
    //     [ctx.NodeID]: The ID of this node.
    // [db]: The database the blockchain persists data to.
    // [genesisBytes]: The byte representation of the genesis state of this blockchain.
    //                 If this VM were an account-based payments system, for example
    //                 `genesisBytes` would probably be a genesis
    //                 transaction that gives coins to some accounts, and this
    //                 transaction would be in the genesis block.
    // [toEngine]: The channel used to send messages to the consensus engine.
    // [fxs]: Feature extensions that attach to this VM.
    // In this release, we do not document feature extensions. You can ignore them.
    Initialize(
        ctx *snow.Context,
        db database.Database,
        genesisBytes []byte,
        toEngine chan<- Message,
        fxs []*Fx,
    ) error

    // Shutdown this blockchain.
    Shutdown()

    // Creates the HTTP handlers for this blockchain's API
    // and specifies the endpoint where they handle traffic.
    //
    // Each handler handles traffic to a specific endpoint.
    // Each endpoint begins with:
    // [Node's address]:[Node's HTTP port]/ext/bc/[blockchain ID]
    // A handler may handle traffic at an *extension* of the above endpoint.
    //
    // The method returns a mapping from an extension to the HTTP handler at that extension.
    //
    // For example, if this VM implements an account-based payments system,
    // CreateHandlers might return this map:
    // "accounts" --> [handler for API calls that pertain to accounts]
    // "transactions" --> [handler for API calls that pertain to transactions]
    //
    // The accounts handler would have endpoint [Node's address]:[Node's HTTP port]/ext/bc/[blockchain ID]/accounts
    // The trasnsactions handler would have endpoint [Node's address]:[Node's HTTP port]/ext/bc/[blockchain ID]/trasnsactions
    //
    // If a handler is mapped to by the empty string, it has no extension.
    // It handles traffic at [Node's address]:[Node's HTTP port]/ext/bc/[blockchain ID]
    CreateHandlers() map[string]*HTTPHandler

    // Attempt to create a new block from pending data in the blockchain's mempool.
    //
    // If there is no new block to be created, returns an error.
    BuildBlock() (snowman.Block, error)

    // Attempt to create a block from its byte representation.
    ParseBlock([]byte) (snowman.Block, error)

    // Attempt to fetch a block by its ID.
    //
    // If the block does not exist, returns an error.
    GetBlock(ids.ID) (snowman.Block, error)

    // Set the preferred block to the one with the specified ID.
    // New blocks will be built atop the preferred block.
    //
    // This should always be a block that has no children known to consensus.
    SetPreference(ids.ID)

    // LastAccepted returns the ID of the last accepted block.
    //
    // If no blocks have been accepted yet, should return the genesis block's ID.
    LastAccepted() ids.ID
}
```

## The snowman.Block 接口

您可能已经注意到了 `snowman.Block` 接口中引用的 `snowman.VM` 类型。它描述了一个区块要成为线性\(Snowman\) 链中的一个区块而必须实现的方法。

让我们来看一下我们从 [`github.com/ava-labs/avalanchego/snow/consensus/snowman/block.go`.](https://github.com/ava-labs/avalanchego/blob/master/snow/consensus/snowman/block.go)复制的一个接口及其方法。

```cpp
// Block is a block in a blockchain.
//
// Blocks are guaranteed to be Verified, Accepted, and Rejected in topological
// order. Specifically, if Verify is called, then the parent has already been
// verified. If Accept is called, then the parent has already been accepted. If
// Reject is called, the parent has already been accepted or rejected.
//
// If the status of the block is Unknown, ID is assumed to be able to be called.
// If the status of the block is Accepted or Rejected; Parent, Verify, Accept,
// and Reject will never be called.
type Block interface {
    // ID returns this block's unique ID.
    //
    // Typically, a block's ID is a hash of its byte representation.
    // A block should return the same ID upon repeated calls.
    ID() ids.ID

    // Accept this block.
    //
    // This block will be accepted by every correct node in the network.
    Accept()

    // Reject this block.
    //
    // This block will not be accepted by any correct node in the network.
    Reject()

    // Status returns this block's current status.
    //
    // If Accept has been called on n block with this ID, Accepted should be
    // returned. Similarly, if Reject has been called on a block with this
    // ID, Rejected should be returned. If the contents of this block are
    // unknown, then Unknown should be returned. Otherwise, Processing should be
    // returned.
    Status() Status

    // Parent returns this block's parent.
    //
    // If the parent block is not known, a Block should be returned with the
    // status Unknown.
    Parent() Block

    // Verify that the state transition this block would make if accepted is
    // valid. If the state transition is invalid, a non-nil error should be
    // returned.
    //
    // It is guaranteed that the Parent has been successfully verified.
    Verify() error

    // Bytes returns the binary representation of this block.
    //
    // This is used for sending blocks to peers. The bytes should be able to be
    // parsed into the same block on another node.
    Bytes() []byte
}
```

## 库

我们创建了一些虚拟机实现可以嵌入 \(嵌入就像Go版本的继承\)的类型，以便处理样板代码。

在我们的实例中，我们使用了下面两种库型，并且我们建议您也使用它们。

### core.SnowmanVM

这个类型是一个结构体，包含所有`snowman.ChainVM` 接口实现所共有的方法和字段。

#### **Methods**

这个类型实现了以下方法，这些方法是`snowman.ChainVM` 接口的一部分：

* `SetPreference`
* `Shutdown`
* `LastAccepted`

如果您的虚拟机实现嵌入了一个`core.SnowmanVM`, ，那么您就不再需要实现这些方法，因为它们已经通过`core.SnowmanVM`实现了。如果愿意，您可以重写这些继承的方法。

#### **Fields**

这个类型包含您需要包含在虚拟机实现中的几个字段。其中：
* `DB`: 区块链的数据库
* `Ctx`: 区块链的运行时上下文
* `preferred`: 构建新区块的偏好区块的ID
* `lastAccepted`: 最近被接受的区块的ID
* `toEngine`: 向驱动区块链共识协议发送信息的通道
* `State`: 用于持久化数据，如区块 \(can be used to put/get any bytes\)

### core.Block

This type, a struct, contains methods and fields common to all implementations of the `snowman.Block` interface.

#### **Methods**

This type implements the following methods, which are part of the `snowman.Block` interface:

* `ID`
* `Parent`
* `Accept`
* `Reject`
* `Status`

Your VM implementation will probably override `Accept` and `Reject` so that these methods cause application-specific state changes.

#### **Fields**

`core.Block` has a field VM, which is a reference to a `core.SnowmanVM`. This means that a `core.Block` has access to all of the fields and methods of that type.

## Timestamp Server Implementation

Now, we know the interface our VM must implement and the libraries we can use to build a VM.

Let’s write our VM, which implements `snowman.VM` and whose blocks implement `snowman.Block`.

### Block

First, let’s look at our block implementation.

The type declaration is:

```cpp
// Block is a block on the chain.
// Each block contains:
// 1) A piece of data (the block's payload)
// 2) The (unix) timestamp when the block was created
type Block struct {
    *core.Block           `serialize:"true"`
    Data        [32]byte  `serialize:"true"`
    Timestamp   int64     `serialize:"true"`
}
```

The `serialize:"true"` tag indicates when a block is persisted in the database or sent to other nodes. The field with the tag is included in the serialized representation.

#### **Verify**

```cpp
// Verify returns nil iff this block is valid.
// To be valid, it must be that:
// b.parent.Timestamp < b.Timestamp <= [local time] + 1 hour
func (b *Block) Verify() error {
    // Check to see if this block has already been verified by calling Verify on the
    // embedded *core.Block.
    // If there is an error while checking, return an error.
    // If the core.Block says the block is accepted, return accepted.
    if accepted, err := b.Block.Verify(); err != nil || accepted {
        return err
    }

    // Get [b]'s parent
    parent, ok := b.Parent().(*Block)
    if !ok {
        return errors.New("error while retrieving block from database")
    }

    // Ensure [b]'s timestamp is after its parent's timestamp.
    if b.Timestamp < time.Unix(parent.Timestamp, 0).Unix() {
        return errors.New("block's timestamp is more than 1 hour ahead of local time")
    }

    // Ensure [b]'s timestamp is not more than an hour 
    // ahead of this node's time
    if b.Timestamp >= time.Now().Add(time.Hour).Unix() {
        return errors.New("block's timestamp is more than 1 hour ahead of local time")
    }

    // Our block inherits VM from *core.Block.
    // It holds the database we read/write, b.VM.DB
    // We persist this block to that database using VM's SaveBlock method.
    b.VM.SaveBlock(b.VM.DB, b)

    // Then we flush the database's contents
    return b.VM.DB.Commit()
}
```

That’s all the code for our block implementation! All of the other methods of `snowman.Block`, which our `Block` must implement, are inherited from `*core.Block`.

### Virtual Machine

Now, let’s look at the implementation of VM, which implements the `snowman.VM` interface.

The declaration is:

```cpp
// This Virtual Machine defines a blockchain that acts as a timestamp server
// Each block contains a piece of data (payload) and the timestamp when it was created
type VM struct {
    core.SnowmanVM

    // codec serializes and de-serializes structs to/from bytes
    codec codec.Codec

    // Proposed pieces of data that haven't been put into a block and proposed yet
    mempool [][32]byte
}
```

#### **Initialize**

```cpp
// Initialize this vm
// [ctx] is the execution context
// [db] is this database we read/write
// [toEngine] is used to notify the consensus engine that new blocks are
//   ready to be added to consensus
// The data in the genesis block is [genesisData]
func (vm *VM) Initialize(
    ctx *snow.Context,
    db database.Database,
    genesisData []byte,
    toEngine chan<- common.Message,
    _ []*common.Fx,
) error {
    // First, we initialize the core.SnowmanVM.
    // vm.ParseBlock, which we'll see further on, tells the core.SnowmanVM how to deserialize
    // a block from bytes
    if err := vm.SnowmanVM.Initialize(ctx, db, vm.ParseBlock, toEngine); err != nil {
        ctx.Log.Error("error initializing SnowmanVM: %v", err)
        return err
    }
    // Set vm's codec to a new codec, which we can use to 
    // serialize and deserialize blocks
    vm.codec = codec.NewDefault()

    // If the database is empty, initialize the state of this blockchain
    // using the genesis data
    if !vm.DBInitialized() {
        // Ensure that the genesis bytes are no longer than 32 bytes
        // (the genesis block, like all blocks, holds 32 bytes of data)
        if len(genesisData) > 32 {
            return errors.New("genesis data should be bytes (max length 32)")
        }

        // genesisData is a byte slice (because that's what the snowman.VM interface says)
        // but each block contains an byte array.
        // To make the types match, take the first [dataLen] bytes from genesisData
        // and put them in an array
        var genesisDataArr [dataLen]byte
        copy(genesisDataArr[:], genesisData)

        // Create the genesis block
        // Timestamp of genesis block is 0. It has no parent, so we say the parent's ID is empty.
        // We'll come to the definition of NewBlock later.
        genesisBlock, err := vm.NewBlock(ids.Empty, genesisDataArr, time.Unix(0, 0))
        if err != nil {
            vm.Ctx.Log.Error("error while creating genesis block: %v", err)
            return err
        }

        // Persist the genesis block to the database.
        // Normally, a block is saved to the database when Verify() is called on the block.
        // We don't call Verify on the genesis block, though. (It has no parent so
        // it wouldn't pass verification.)
        // vm.DB is the database, and was set when we initialized the embedded SnowmanVM.
        if err := vm.SaveBlock(vm.DB, genesisBlock); err != nil {
            vm.Ctx.Log.Error("error while saving genesis block: %v", err)
            return err
        }

        // Accept the genesis block.
        // Sets [vm.lastAccepted] and [vm.preferred] to the genesisBlock.
        genesisBlock.Accept()

        // Mark the database as initialized so that in the future when this chain starts
        // it pulls state from the database rather than starting over from genesis
        vm.SetDBInitialized()

        // Flush the database
        if err := vm.DB.Commit(); err != nil {
            vm.Ctx.Log.Error("error while commiting db: %v", err)
            return err
        }
    }
    return nil
}
```

#### **proposeBlock**

This method adds a piece of data to the mempool and notifies the consensus layer of the blockchain that a new block is ready to be built and voted on. We’ll see where this is called later.

```cpp
// proposeBlock appends [data] to [p.mempool].
// Then it notifies the consensus engine
// that a new block is ready to be added to consensus
// (namely, a block with data [data])
func (vm *VM) proposeBlock(data [dataLen]byte) {
    vm.mempool = append(vm.mempool, data)
    vm.NotifyBlockReady()
}
```

#### **ParseBlock**

```cpp
// ParseBlock parses [bytes] to a snowman.Block
// This function is used by the vm's state to unmarshal blocks saved in state
// and by the consensus layer when it receives the byte representation of a block
// from another node
func (vm *VM) ParseBlock(bytes []byte) (snowman.Block, error) {
    // A new empty block
    block := &Block{}

    // Unmarshal the byte repr. of the block into our empty block
    err := vm.codec.Unmarshal(bytes, block)

    // Initialize the block
    // (Block inherits Initialize from its embedded *core.Block)
    block.Initialize(bytes, &vm.SnowmanVM)
    return block, err
}
```

#### **NewBlock**

```cpp
// NewBlock returns a new Block where:
// - the block's parent has ID [parentID]
// - the block's data is [data]
// - the block's timestamp is [timestamp]
func (vm *VM) NewBlock(parentID ids.ID, data [dataLen]byte, timestamp time.Time) (*Block, error) {
    // Create our new block
    block := &Block{
        Block:     core.NewBlock(parentID),
        Data:      data,
        Timestamp: timestamp.Unix(),
    }

    // Get the byte representation of the block
    blockBytes, err := vm.codec.Marshal(block)
    if err != nil {
        return nil, err
    }

    // Initialize the block by providing it with its byte representation
    // and a reference to SnowmanVM
    block.Initialize(blockBytes, &vm.SnowmanVM)

    return block, nil
}
```

#### **BuildBlock**

This method is called by the consensus layer after the application layer tells it that a new block is ready to be built \(i.e., when `vm.NotifyConsensus()` is called\).

```cpp
// BuildBlock returns a block that this VM wants to add to consensus
func (vm *VM) BuildBlock() (snowman.Block, error) {
    // There is no data to put in a new block
    if len(vm.mempool) == 0 { 
        return nil, errors.New("there is no block to propose")
    }

    // Get the value to put in the new block
    value := vm.mempool[0]
    vm.mempool = vm.mempool[1:]

    // Notify consensus engine that there are more pending data for blocks
    // (if that is the case) when done building this block
    if len(vm.mempool) > 0 {
        defer vm.NotifyBlockReady()
    }

    // Build the block
    block, err := vm.NewBlock(vm.Preferred(), value, time.Now())
    if err != nil {
        return nil, err
    }
    return block, nil
}
```

#### **CreateHandlers**

```cpp
// CreateHandlers returns a map where:
// Keys: The path extension for this blockchain's API (empty in this case)
// Values: The handler for the API
// In this case, our blockchain has only one API, which we name timestamp,
// and it has no path extension, so the API endpoint:
// [Node IP]/ext/bc/[this blockchain's ID]
// See API section in documentation for more information
func (vm *VM) CreateHandlers() map[string]*common.HTTPHandler {
    // Create the API handler (we'll see the declaration of Service further on)
    handler := vm.NewHandler("timestamp", &Service{vm})
    return map[string]*common.HTTPHandler{
        "": handler,
    }
}
```

### Service

AvalancheGo uses [Gorilla’s RPC library](https://www.gorillatoolkit.org/pkg/rpc) to implement APIs.

Using Gorilla, there is a struct for each API service. In the case of this blockchain, there’s only one API service.

The service struct’s declaration is:

```cpp
// Service is the API service for this VM
type Service struct{ vm *VM }
```

For each API method, there is: \* A struct that defines the method’s arguments \* A struct that defines the method’s return values \* A method that implements the API method, and is parameterized on the above 2 structs

#### **ProposeBlock**

This API method allows clients to add a block to the blockchain.

```cpp
// ProposeBlockArgs are the arguments to ProposeValue
type ProposeBlockArgs struct {
    // Data for the new block. Must be base 58 encoding (with checksum) of 32 bytes.
    Data string
}

// ProposeBlockReply is the reply from function ProposeBlock
type ProposeBlockReply struct{ 
    // True if the operation was successful
    Success bool
}

// ProposeBlock is an API method to propose a new block whose data is [args].Data.
func (s *Service) ProposeBlock(_ *http.Request, args *ProposeBlockArgs, reply *ProposeBlockReply) error {
    // Parse the data given as argument to bytes
    byteFormatter := formatting.CB58{}
    if err := byteFormatter.FromString(args.Data); err != nil {
        return errBadData
    }
    // Ensure the data is 32 bytes
    dataSlice := byteFormatter.Bytes
    if len(dataSlice) != 32 {
        return errBadData
    }
    // Convert the data from a byte slice to byte array
    var data [dataLen]byte             
    copy(data[:], dataSlice[:dataLen])
    // Invoke proposeBlock to trigger creation of block with this data
    s.vm.proposeBlock(data)
    reply.Success = true
    return nil
}
```

#### **GetBlock**

This API method allows clients to get a block by its ID.

```cpp
// APIBlock is the API representation of a block
type APIBlock struct {
    Timestamp int64  `json:"timestamp"` // Timestamp of most recent block
    Data      string `json:"data"`      // Data in the most recent block. Base 58 repr. of 5 bytes.
    ID        string `json:"id"`        // String repr. of ID of the most recent block
    ParentID  string `json:"parentID"`  // String repr. of ID of the most recent block's parent
}

// GetBlockArgs are the arguments to GetBlock
type GetBlockArgs struct {
    // ID of the block we're getting.
    // If left blank, gets the latest block
    ID string
}

// GetBlockReply is the reply from GetBlock
type GetBlockReply struct {
    APIBlock
}

// GetBlock gets the block whose ID is [args.ID]
// If [args.ID] is empty, get the latest block
func (s *Service) GetBlock(_ *http.Request, args *GetBlockArgs, reply *GetBlockReply) error {
    // If an ID is given, parse its string representation to an ids.ID
    // If no ID is given, ID becomes the ID of last accepted block
    var ID ids.ID
    var err error
    if args.ID == "" {
        ID = s.vm.LastAccepted()
    } else {
        ID, err = ids.FromString(args.ID)
        if err != nil {
            return errors.New("problem parsing ID")
        }
    }

    // Get the block from the database
    blockInterface, err := s.vm.GetBlock(ID)
    if err != nil {
        return errors.New("error getting data from database")
    }

    block, ok := blockInterface.(*Block)
    if !ok { // Should never happen but better to check than to panic
        return errors.New("error getting data from database")
    }

    // Fill out the response with the block's data
    reply.APIBlock.ID = block.ID().String()
    reply.APIBlock.Timestamp = block.Timestamp
    reply.APIBlock.ParentID = block.ParentID().String()
    byteFormatter := formatting.CB58{Bytes: block.Data[:]}
    reply.Data = byteFormatter.String()

    return nil
}
```

#### **API**

The resulting API has the following methods:

**timestamp.getBlock**

Get a block by its ID. If no ID is provided, get the latest block.

**Signature**

```cpp
timestamp.getBlock({id: string}) ->
    {
        id: string,
        data: string,
        timestamp: int,
        parentID: string
    }
```

* `id` is the ID of the block being retrieved. If omitted from arguments, gets the latest block
* `data` is the base 58 \(with checksum\) representation of the block’s 32 byte payload
* `timestamp` is the Unix timestamp when this block was created
* `parentID` is the block’s parent

**Example Call**

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "timestamp.getBlock",
    "params":{
        "id":"xqQV1jDnCXDxhfnNT7tDBcXeoH2jC3Hh7Pyv4GXE1z1hfup5K"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/timestamp
```

**Example Response**

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "timestamp": "1581717416",
        "data": "11111111111111111111111111111111LpoYY",
        "id": "xqQV1jDnCXDxhfnNT7tDBcXeoH2jC3Hh7Pyv4GXE1z1hfup5K",
        "parentID": "22XLgiM5dfCwTY9iZnVk8ZPuPe3aSrdVr5Dfrbxd3ejpJd7oef"
    },
    "id": 1
}
```

**timestamp.proposeBlock**

Propose the creation of a new block.

**Signature**

```cpp
timestamp.proposeBlock({data: string}) -> {success: bool}
```

* `data` is the base 58 \(with checksum\) representation of the proposed block’s 32 byte payload.

**Example Call**

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "timestamp.proposeBlock",
    "params":{
        "data":"SkB92YpWm4Q2iPnLGCuDPZPgUQMxajqQQuz91oi3xD984f8r"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/timestamp
```

**Example Response**

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "Success": true
    },
    "id": 1
}
```

### Wrapping Up

That’s it! That’s the entire implementation of a VM which defines a blockchain-based timestamp server.

In this tutorial, we learned:

* The `snowman.ChainVM` interface, which all VMs that define a linear chain must implement
* The `snowman.Block` interface, which all blocks that are part of a linear chain must implement
* The `core.SnowmanVM` and `core.Block` library types, which make defining VMs faster

<!--stackedit_data:
eyJoaXN0b3J5IjpbODczMzg4MzQzXX0=
-->