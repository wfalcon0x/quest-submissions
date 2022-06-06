## DAY 1

### 1. Describe what an event is, and why it might be useful to a client.
*When we emit an event, we send information to the outside world about something that happened as part of the smart contract code execution, and then we can  query them from the flow access api.*


### 2. Deploy a contract with an event in it, and emit the event somewhere else in the contract indicating that it happened.


```
pub contract TestEvent {
  // define an event here
  pub event NFTMinted(id: UInt64)
  pub resource NFT {
    pub let id: UInt64
    init() {
      self.id = self.uuid
      // broadcast the event to the outside world
      emit NFTMinted(id: self.id)
    }
  }
  pub fun createNFT(): @NFT {
    return <- create NFT()
  }
}
```

```
import TestEvent from 0x05
transaction() {
  prepare(signer: AuthAccount) {
    let testResource <- TestEvent.createNFT()
    destroy testResource
  }
  execute {
  }
}
```


### 3. Using the contract in step 2), add some pre conditions and post conditions to your contract to get used to writing them out.
```
pub contract TestEvent {
  // define an event here
  pub event NFTMinted(id: UInt64)
  pub resource NFT {
    pub let id: UInt64
    pub let url: String
    init(url: String) {
      self.id = self.uuid
      self.url = url
      // broadcast the event to the outside world
      emit NFTMinted(id: self.id)
    }
  }
  pub fun createNFT(url: String): @NFT {
      pre {
        url.length > 0: "Invalid url."
      }
    return <- create NFT(url: url)
  }
  pub fun makeNegativeResult(x: Int, y: Int): Int {
    post {
        result < 0: "The result is not negative."
    }
    return x + y
  }
}
```


### 4. For each of the functions below (numberOne, numberTwo, numberThree), follow the instructions.
```
pub contract Test {

  // TODO
  // Tell me whether or not this function will log the name.
  // name: 'Jacob'
  pub fun numberOne(name: String) {
    pre {
      // it will pass, as Jacob is 5 character long
      name.length == 5: "This name is not cool enough."
    }
    log(name)
  }

  // TODO
  // Tell me whether or not this function will return a value.
  // name: 'Jacob'
  pub fun numberTwo(name: String): String {
    pre {
      name.length >= 0: "You must input a valid name."
    }
    post {
      result == "Jacob Tucker"
    }
    // it will return Jacob Tucker
    return name.concat(" Tucker")
  }

  pub resource TestResource {
    pub var number: Int

    // TODO
    // Tell me whether or not this function will log the updated number.
    // Also, tell me the value of `self.number` after it's run.
    pub fun numberThree(): Int {
      post {
        before(self.number) == result + 1
      }
      self.number = self.number + 1
      return self.number
    }

    init() {
      self.number = 0
    }

  }

}
```

*answer:*

```
  pub fun createTestResource(): @TestResource {
     return <- create TestResource()
  }
```


```
pub contract Test {

  // TODO
  // Tell me whether or not this function will log the name.
  // name: 'Jacob'
  pub fun numberOne(name: String) {
    pre {
      // it will pass, as Jacob is 5 character long
      name.length == 5: "This name is not cool enough."
    }
    log(name)
  }

  // TODO
  // Tell me whether or not this function will return a value.
  // name: 'Jacob'
  pub fun numberTwo(name: String): String {
    pre {
      name.length >= 0: "You must input a valid name."
    }
    post {
      result == "Jacob Tucker"
    }
    // it will return Jacob Tucker
    return name.concat(" Tucker")
  }

  pub resource TestResource {
    pub var number: Int

    // TODO
    // Tell me whether or not this function will log the updated number.
    // Also, tell me the value of `self.number` after it's run.
    pub fun numberThree(): Int {
      post {
        // this fails as the result will be 1 after the first run, so the line should be here:
        // before(self.number) == result - 1
        before(self.number) == result + 1
      }
      self.number = self.number + 1
      log(self.number)
      return self.number
    }

    init() {
      self.number = 0
    }

  }


  pub fun createTestResource(): @TestResource {
     return <- create TestResource()
  }

}
```

```
import Test from 0x01
pub fun main(): Void {
  Test.numberOne(name: "Jacob")
  let name = Test.numberTwo(name: "Jacob")
  log(name)
}
```

```
import Test from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    let testResource <- Test.createTestResource()
    let number = testResource.numberThree()
    log(number)
    destroy testResource
  }
  execute {
  }
}
```

---

## DAY 2


### 1. Explain why standards can be beneficial to the Flow ecosystem.
*Dapps can interact with all the smart contracts implementing the same standard in the same way. So Dapps don't have to implement contract specific logic.*

### 2. What is YOUR favourite food?
*Sushi*

### 3. Please fix this code (Hint: There are two things wrong):
### The contract interface:

```
pub contract interface ITest {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    pre {
      newNumber >= 0: "We don't like negative numbers for some reason. We're mean."
    }
    post {
      self.number == newNumber: "Didn't update the number to be the new number."
    }
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff {
    pub var favouriteActivity: String
  }
}
```
The implementing contract:

```
pub contract Test {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    self.number = 5
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff: IStuff {
    pub var favouriteActivity: String

    init() {
      self.favouriteActivity = "Playing League of Legends."
    }
  }

  init() {
    self.number = 0
  }
}
```

*answer:*

```
import ITest from 0x04
// define the interface!
pub contract Test : ITest {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    self.number = 5
  }
  // implement the right interface!
  pub resource Stuff: ITest.IStuff {
    pub var favouriteActivity: String
    init() {
      self.favouriteActivity = "Playing League of Legends."
    }
  }
  init() {
    self.number = 0
  }
}
```

---


## DAY 3 



### 1. What does "force casting" with as! do? Why is it useful in our Collection?
*force casting means if the resource has the type, do the casting, otherwise panic. This way we can make sure we move resources to a collection with a specific type, in our specific case we can make sure NFTs with our specific @NFT type will be moved to our Collection by downcasting from @NonFungibleToken.NFT which is defined on the NFT standard*


### 2. What does auth do? When do we use it?
*when we want to downcast a reference, we need to have an "authorised" reference indicated with auth keyword to make it work*


### 3. This last quest will be your most difficult yet. Take this contract:
```
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return &self.ownedNFTs[id] as &NonFungibleToken.NFT
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    emit ContractInitialized()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```
### and add a function called borrowAuthNFTjust like we did in the section called "The Problem" above. Then, find a way to make it publically accessible to other people so they can read our NFT's metadata. Then, run a script to display the NFTs metadata for a certain id.

### You will have to write all the transactions to set up the accounts, mint the NFTs, and then the scripts to read the NFT's metadata. We have done most of this in the chapters up to this point, so you can look for help there🙂

*answer:*

```
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  pub resource interface CryptoPoopsCollectionPublic {
    pub fun deposit(token: @NonFungibleToken.NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT
    pub fun borrowAuthNFT(id: UInt64): &NFT? {
      post {
        (result == nil) || (result?.id == id): 
          "Cannot borrow CryptoPoops reference: The ID of the returned reference is incorrect"
      }
    }
  }

  pub resource Collection: CryptoPoopsCollectionPublic, NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return &self.ownedNFTs[id] as &NonFungibleToken.NFT
    }

    pub fun borrowAuthNFT(id: UInt64): &NFT {
      let ref = &self.ownedNFTs[id] as auth &NonFungibleToken.NFT
      return ref as! &NFT
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    emit ContractInitialized()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

```
import CryptoPoops from 0x01
transaction {
    prepare(acct: AuthAccount) {
        if acct.borrow<&CryptoPoops.Collection>(from: /storage/CryptoPoopsCollection) == nil {
            let collection <- CryptoPoops.createEmptyCollection() as! @CryptoPoops.Collection
            acct.save(<-collection, to: /storage/CryptoPoopsCollection)
            acct.link<&{CryptoPoops.CryptoPoopsCollectionPublic}>(/public/CryptoPoopsCollection, target: /storage/CryptoPoopsCollection)
        }
    }
}
```

```
import CryptoPoops from 0x01
transaction(
    recipientAddr: Address,
    name: String
    favouriteFood: String
    luckyNumber: Int
) {
    let minterRef: &CryptoPoops.Minter
    prepare(acct: AuthAccount) {
        self.minterRef = acct.borrow<&CryptoPoops.Minter>(from: /storage/Minter)!
    }
    execute {
        // Mint a new NFT
        let cryptoPoop <- self.minterRef.createNFT(
                                    name: name,
                                    favouriteFood: favouriteFood, 
                                    luckyNumber: luckyNumber
                                )
        let recipient = getAccount(recipientAddr)
        if recipient == nil { panic("Recipient doesn't exist") }
        let receiverRef = recipient.getCapability(/public/CryptoPoopsCollection).borrow<&{CryptoPoops.CryptoPoopsCollectionPublic}>()
            ?? panic("Cannot borrow a reference to the recipient's CryptoPoop collection")
        receiverRef.deposit(token: <-cryptoPoop)
    }
}
```


```
import CryptoPoops from 0x01

pub fun main(account: Address, id: UInt64): &CryptoPoops.NFT {

    let collectionRef = getAccount(account).getCapability(/public/CryptoPoopsCollection)
        .borrow<&{CryptoPoops.CryptoPoopsCollectionPublic}>()
        ?? panic("Could not get public CryptoPoop collection reference")

    let token = collectionRef.borrowAuthNFT(id: id)
        ?? panic("Could not borrow a reference to the specified CryptoPoop")

    log(token)

    return token
}
```