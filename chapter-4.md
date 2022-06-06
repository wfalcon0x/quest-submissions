## DAY 1


Quests

### 1. Explain what lives inside of an account.
*accounts store their own data, like NFTs*

### 2. What is the difference between the /storage/, /public/, and /private/ paths?
*/storage - only account owner can access, /private - account owner and those given access by the account owner, /public - everyone can access*

### 3. What does .save() do? What does .load() do? What does .borrow() do?
*save - save data to account's storage path, load - load data from the account's storage path*

### 4. Explain why we couldn't save something to our account storage inside of a script.
*storage can be accessed only by AuthAccount, which is available only in the transactions's prepare phase*

### 5. Explain why I couldn't save something to your account.
*in order the save to an account, the account owner has to sign it*

### 6. Define a contract that returns a resource that has at least 1 field in it. Then, write 2 transactions:
	1. A transaction that first saves the resource to account storage, then loads it out of account storage, logs a field inside the resource, and destroys it.
	2. A transaction that first saves the resource to account storage, then borrows a reference to it, and logs a field inside the resource.

*see below*


```
pub contract Stuff {
  pub resource Test {
    pub var name: String
    init() {
      self.name = "Jacob"
    }
  }
  pub fun createTest(): @Test {
    return <- create Test()
  }
}
```

```
import Stuff from 0x03
transaction() {
  prepare(signer: AuthAccount) {
    let testResource <- Stuff.createTest()
    signer.save(<- testResource, to: /storage/MyTestResource) 
    // saves `testResource` to my account storage at this path:
    // /storage/MyTestResource
    let testResourceBack <- signer.load<@Stuff.Test>(from: /storage/MyTestResource) ?? panic("A `@Stuff.Test` resource does not live here.")
    // takes `testResource` out of my account storage
    log(testResourceBack.name)
    destroy testResourceBack
  }
  execute {
  }
}
```

```
import Stuff from 0x03
transaction() {
  prepare(signer: AuthAccount) {
    let testResource <- Stuff.createTest()
    signer.save(<- testResource, to: /storage/MyTestResource) 
    // saves `testResource` to my account storage at this path:
    // /storage/MyTestResource
    let testResourceBack = signer.borrow<&Stuff.Test>(from: /storage/MyTestResource)
                          ?? panic("A `@Stuff.Test` resource does not live here.")
    log(testResourceBack.name)
  }
  execute {
  }
}
```

---

## DAY 2


### 1. What does .link() do?
*links /private or /public path to /storage, and by this the data in the storage can be exposed*

### 2. In your own words (no code), explain how we can use resource interfaces to only expose certain things to the /public/ path.
*we can define those fields and functions on the resource interface that we want to allow for the public to access and then we can define this interface when linking the storage path to the public path. If we did this, the code from a script will have access only to this interface when borrowing a public capability, in effect the public can access only those fields or functions that the interface has.*

### 3. Deploy a contract that contains a resource that implements a resource interface. Then, do the following:
	1. In a transaction, save the resource to storage and link it to the public with the restrictive interface.
	2. Run a script that tries to access a non-exposed field in the resource interface, and see the error pop up.
	3. Run the script and access something you CAN read from. Return it from the script.

*answer:*

```
pub contract DeathNoteAnime {
  pub resource interface IDeathNote {
    pub fun readNotes(): String
    pub fun writeName(newName: String)
  }
  pub resource DeathNote: IDeathNote {
    access(contract) var notes: String
    pub fun readNotes(): String {
      return self.notes
    }
    pub var ownerHasShinigamiEyes: Bool
    pub fun writeName(newName: String) {
      log(self.notes)
      self.notes = self.notes.concat(" ").concat(newName)
      log(self.notes)
    }
    pub fun tradeForShinigamiEyes() {
      self.ownerHasShinigamiEyes = true
    }
    init() {
      self.notes = ""
      self.ownerHasShinigamiEyes = false
    }
  }
  pub fun dropDeathNote(): @DeathNote {
    return <- create DeathNote()
  }
}
```

```
import DeathNoteAnime from 0x04
transaction() {
  prepare(signer: AuthAccount) {
    // Save the resource to account storage
    signer.save(<- DeathNoteAnime.dropDeathNote(), to: /storage/DeathNoteResource)
    signer.link<&DeathNoteAnime.DeathNote{DeathNoteAnime.IDeathNote}>(/public/DeathNoteResource, target: /storage/DeathNoteResource)
  }
  execute {
  }
}
```

```
import DeathNoteAnime from 0x04
transaction(address: Address) {
  prepare(signer: AuthAccount) {
  }
  execute {
    let publicCapability: Capability<&DeathNoteAnime.DeathNote> =
      getAccount(address).getCapability<&DeathNoteAnime.DeathNote>(/public/DeathNoteResource)
    // ERROR: "The capability doesn't exist or you did not 
    // specify the right type when you got the capability."
    let myDeathNote: &DeathNoteAnime.DeathNote = publicCapability.borrow() ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")
    myDeathNote.writeName(newName: "L")
  }
}
```

```
import DeathNoteAnime from 0x04
transaction(address: Address) {
  prepare(signer: AuthAccount) {
  }
  execute {
    let publicCapability: Capability<&DeathNoteAnime.DeathNote{DeathNoteAnime.IDeathNote}> =
      getAccount(address).getCapability<&DeathNoteAnime.DeathNote{DeathNoteAnime.IDeathNote}>(/public/DeathNoteResource)
    let myDeathNote: &DeathNoteAnime.DeathNote{DeathNoteAnime.IDeathNote} = publicCapability.borrow() ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")
    myDeathNote.writeName(newName: "L")
    log(myDeathNote.readNotes())
    // not accessible:
    //myDeathNote.tradeForShinigamiEyes()
  }
}
```

```
import DeathNoteAnime from 0x04
pub fun main(address: Address): String {
  let publicCapability: Capability<&DeathNoteAnime.DeathNote{DeathNoteAnime.IDeathNote}> =
      getAccount(address).getCapability<&DeathNoteAnime.DeathNote{DeathNoteAnime.IDeathNote}>(/public/DeathNoteResource)
  let myDeathNote: &DeathNoteAnime.DeathNote{DeathNoteAnime.IDeathNote} = publicCapability.borrow() ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")
  return myDeathNote.readNotes()
}
```

---


## DAY 3


### 1. Why did we add a Collection to this contract? List the two main reasons.
*we can store more than one NFT on the same storage path, and allows others to deposit NFTs*

### 2. What do you have to do if you have resources "nested" inside of another resource? ("Nested resources")
*we need to define and implement a destroy() function that destroys the nested resources*

### 3. Brainstorm some extra things we may want to add to this contract. Think about what might be problematic with this contract and how we could fix it.
	- Idea #1: Do we really want everyone to be able to mint an NFT? 🤔.
	- Idea #2: If we want to read information about our NFTs inside our Collection, right now we have to take it out of the Collection to do so. Is this good?

*admin account for NFT minting, store public data in structs so it can be read by anyone easily*



## DAY 4 


### Take our NFT contract so far and add comments to every single resource or function explaining what it's doing in your own words. Something like this:

```
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  // This is an NFT resource that contains a name,
  // favouriteFood, and luckyNumber
  pub resource NFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    // initialise NFT
    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  // This is a resource interface that allows us to... you get the point.
  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
  }

  // collection resource to store many NFTs on the same storage path
  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}

    // deposit an NFT in the collection
    pub fun deposit(token: @NFT) {
      self.ownedNFTs[token.id] <-! token
    }

    // withrow an NFT from the collection by it's ID
    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }

    // return IDs of all aowned NFTs
    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    // return a refernece to an NFT by ID
    pub fun borrowNFT(id: UInt64): &NFT {
      return &self.ownedNFTs[id] as &NFT
    }


    // initialise collection
    init() {
      self.ownedNFTs <- {}
    }

    // destroy all nested resources
    destroy() {
      destroy self.ownedNFTs
    }
  }

  // create a new emprty collection to be used to store the NFTs
  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }

  // the owner of the minter resource can mint the NFTs 
  pub resource Minter {

    // mint an NFTs
    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    // create another minter
    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  // initialise contract, set total supply to 0 and create a minter for the same account that created the contract
  init() {
    self.totalSupply = 0
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```