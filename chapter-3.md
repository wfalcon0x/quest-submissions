## DAY 1

### 1. In words, list 3 reasons why structs are different from resources.
*Structs can be copied, but resources can only be moved
Structs can be created outside of contract too, but resources can be created only inside a contract with a special keyword: create
The value of struct can be overwritten easily, but resources cannot*

### 2. Describe a situation where a resource might be better to use than a struct.

*It is better to use resources for example for NFTs, the cadence language provides guarantees the resource will not get lost unless the developer want to explicitly destroy them.*

### 3. What is the keyword to make a new resource?
*create*

### 4. Can a resource be created in a script or transaction (assuming there isn't a public function to create one)?
*resource can be created only in the contract*

### 5. What is the type of the resource below?
```
pub resource Jacob {

}
```
*@Jacob*

### 6. Let's play the "I Spy" game from when we were kids. I Spy 4 things wrong with this code. Please fix them.
```
pub contract Test {

    // Hint: There's nothing wrong here ;)
    pub resource Jacob {
        pub let rocks: Bool
        init() {
            self.rocks = true
        }
    }

    pub fun createJacob(): Jacob { // there is 1 here
        let myJacob = Jacob() // there are 2 here
        return myJacob // there is 1 here
    }
}
```


*answer:*


```
pub contract Test {
    // Hint: There's nothing wrong here ;)
    pub resource Jacob {
        pub let rocks: Bool
        init() {
            self.rocks = true
        }
    }
    pub fun createJacob(): @Jacob { // there is 1 here
        let myJacob <- create Jacob() // there are 2 here
        return <- myJacob // there is 1 here
    }
}
```

---

## DAY 2



### 1. Write your own smart contract that contains two state variables: an array of resources, and a dictionary of resources. Add functions to remove and add to each of them. They must be different from the examples above.


```
pub contract Test {
    pub var arrayOfJutsus: @[Jutsu]
    pub var rankedJutsus: @{UInt64: Jutsu}
    pub resource Jutsu {
        pub let name: String
        pub let rank: UInt64
        init(name: String, rank: UInt64) {
            self.name = name
            self.rank = rank
        }
    }
    pub fun addJutsuToArray(jutsu: @Jutsu) {
        self.arrayOfJutsus.append(<- jutsu)
    }
    pub fun removeJutsuFromArray(index: Int): @Jutsu {
        return <- self.arrayOfJutsus.remove(at: index)
    }
    pub fun addRankedJutsu(jutsu: @Jutsu) {
        let key = jutsu.rank
        
        let oldJutsu<- self.rankedJutsus[key] <- jutsu
        destroy oldJutsu
    }
    pub fun removeJutsu(key: UInt64): @Jutsu {
        let jutsu <- self.rankedJutsus.remove(key: key) ?? panic("Could not find the jutsu!")
        return <- jutsu
    }
    init() {
        self.arrayOfJutsus <- []
        self.rankedJutsus <- {}
    }
}
```

---

### DAY 3


### 1. Define your own contract that stores a dictionary of resources. Add a function to get a reference to one of the resources in the dictionary.

```
pub contract Test {
    pub var rankedJutsus: @{UInt64: Jutsu}
    pub resource Jutsu {
        pub let name: String
        pub let rank: UInt64
        init(name: String, rank: UInt64) {
            self.name = name
            self.rank = rank
        }
    }
    
    pub fun getReference(key: UInt64): &Jutsu {
        return &self.rankedJutsus[key] as &Jutsu
    }

    init() {
        self.rankedJutsus <- {
            1: <- create Jutsu(name: "Infinite Tsukuyomi", rank: 1), 
            2: <- create Jutsu(name: "Kotoamatsukami", rank: 2),
            3: <- create Jutsu(name: "Chibaku Tensei", rank: 3),
            4: <- create Jutsu(name: "Indra’s Arrow", rank: 4)
        }
    }
}
```

### 2. Create a script that reads information from that resource using the reference from the function you defined in part 1.

```
import Test from 0x03
pub fun main(): String {
  let ref = Test.getReference(key: 2)
  return ref.name 
}
```

### 3. Explain, in your own words, why references can be useful in Cadence.


We get information from the resource without having to move it out from where it is atm

---

### DAY 4


### 1. Explain, in your own words, the 2 things resource interfaces can be used for (we went over both in today's content)

*Interface is useful for:*
- defining what the behaviour of the implementing resource, struct or contract should be by defining what functions and fields it must have
- limiting what member variable or method  is accessible by the caller

### 2. Define your own contract. Make your own resource interface and a resource that implements the interface. Create 2 functions. In the 1st function, show an example of not restricting the type of the resource and accessing its content. In the 2nd function, show an example of restricting the type of the resource and NOT being able to access its content.

```
pub contract Bleach {

    pub resource interface HasZanpakuto {
      pub var power: Int
      pub fun swingZanpakuto(): Int
    }

    pub resource interface CapablaOfShikai {
      pub var shikaiPower: Int
      pub var shikaiReleased: Bool
      pub fun releaseShikai(): Void
    }

    pub resource interface CapablaOfBankai {
      pub var bankaiPower: Int
      pub var bankaiReleased: Bool
      pub fun releaseBankai(): Void
    }

    pub resource Shinigami: HasZanpakuto, CapablaOfShikai, CapablaOfBankai {
      pub var power: Int
      pub var shikaiPower: Int
      pub var shikaiReleased: Bool
      pub var bankaiPower: Int
      pub var bankaiReleased: Bool


      pub fun swingZanpakuto(): Int {
        var hitStrength = self.bankaiReleased ? self.bankaiPower : (self.shikaiReleased ? self.shikaiPower : self.power)
        return hitStrength
      }
      pub fun releaseShikai(): Void {
        self.shikaiReleased = true
      }
      pub fun releaseBankai(): Void {
        self.bankaiReleased = true
      }

      init() {
        self.power = 3
        self.shikaiPower = 15
        self.bankaiPower = 85
        self.shikaiReleased = false
        self.bankaiReleased = false
      }
    }

    pub fun noInterface() {
      let ichigo: @Shinigami <- create Shinigami()
      ichigo.releaseBankai()
      log(ichigo.swingZanpakuto())

      destroy ichigo
    }

    pub fun yesInterface() {
      let ichigo: @Shinigami{HasZanpakuto} <- create Shinigami()
      log(ichigo.swingZanpakuto())

      destroy ichigo
    }
}
```


### 3. How would we fix this code?
```
pub contract Stuff {

    pub struct interface ITest {
      pub var greeting: String
      pub var favouriteFruit: String
    }

    // ERROR:
    // `structure Stuff.Test does not conform 
    // to structure interface Stuff.ITest`
    pub struct Test: ITest {
      pub var greeting: String

      pub fun changeGreeting(newGreeting: String): String {
        self.greeting = newGreeting
        return self.greeting // returns the new greeting
      }

      init() {
        self.greeting = "Hello!"
      }
    }

    pub fun fixThis() {
      let test: Test{ITest} = Test()
      let newGreeting = test.changeGreeting(newGreeting: "Bonjour!") // ERROR HERE: `member of restricted type is not accessible: changeGreeting`
      log(newGreeting)
    }
}
```

*answer:*

```
pub contract Stuff {
    pub struct interface ITest {
      pub var greeting: String
      pub var favouriteFruit: String
      pub fun changeGreeting(newGreeting: String): String
    }

    pub struct Test: ITest {
      pub var greeting: String
      pub var favouriteFruit: String
      pub fun changeGreeting(newGreeting: String): String {
        self.greeting = newGreeting
        return self.greeting // returns the new greeting
      }
      init() {
        self.greeting = "Hello!"
        self.favouriteFruit = "durian"
      }
    }
    pub fun fixThis() {
      let test: Test{ITest} = Test()
      let newGreeting = test.changeGreeting(newGreeting: "Bonjour!") 
      log(newGreeting)
    }
}
```
---

## DAY 5


For today's quest, you will be looking at a contract and a script. You will be looking at 4 variables (a, b, c, d) and 3 functions (publicFunc, contractFunc, privateFunc) defined in SomeContract. In each AREA (1, 2, 3, and 4), I want you to do the following: for each variable (a, b, c, and d), tell me in which areas they can be read (read scope) and which areas they can be modified (write scope). For each function (publicFunc, contractFunc, and privateFunc), simply tell me where they can be called.

```
access(all) contract SomeContract {
    pub var testStruct: SomeStruct

    pub struct SomeStruct {

        //
        // 4 Variables
        //

        pub(set) var a: String

        pub var b: String

        access(contract) var c: String

        access(self) var d: String

        //
        // 3 Functions
        //

        pub fun publicFunc() {}

        access(contract) fun contractFunc() {}

        access(self) fun privateFunc() {}


        pub fun structFunc() {
            /**************/
            /*** AREA 1 ***/
            /**************/
        }

        init() {
            self.a = "a"
            self.b = "b"
            self.c = "c"
            self.d = "d"
        }
    }

    pub resource SomeResource {
        pub var e: Int

        pub fun resourceFunc() {
            /**************/
            /*** AREA 2 ***/
            /**************/
        }

        init() {
            self.e = 17
        }
    }

    pub fun createSomeResource(): @SomeResource {
        return <- create SomeResource()
    }

    pub fun questsAreFun() {
        /**************/
        /*** AREA 3 ****/
        /**************/
    }

    init() {
        self.testStruct = SomeStruct()
    }
}
```
This is a script that imports the contract above:

```
import SomeContract from 0x01

pub fun main() {
  /**************/
  /*** AREA 4 ***/
  /**************/
}
```

*answer:*

AREA 1

a, b, c, d, publicFunc, contractFunc, privateFunc

AREA 2

a, b, c, publicFunc, contractFunc

AREA 3

a, b, c, publicFunc, contractFunc

AREA 4 

a, b, publicFunc

