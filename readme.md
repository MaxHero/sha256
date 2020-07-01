This is the copy of original [**crypto/sha256**](https://github.com/golang/go/tree/master/src/crypto/sha256) package. The only difference is that digest struct in this package is exported. It allows save/restore hash state without marshalling/unmarshalling
 
## Hash state copy example

    package main
    
    import (
        "encoding/hex"
        "fmt"
        "github.com/maxhero/sha256"
    )
    
    func main() {
        fixedBytes := make([]byte, 80)
    
        hashState1 := *sha256.New().(*sha256.Digest) //value of hash state, not a pointer
        hashState1.Write(fixedBytes) //write first 80 bytes as they are the same for both messages
    
        hashState2 := hashState1 //save hash state
    
        hashState1.Write([]byte{1, 2, 3, 4}) //write last 4 bytes of first message
        hashState2.Write([]byte{5, 6, 7, 8}) //write last 4 bytes of second message
    
        log.Println(hex.EncodeToString(hashState1.Sum(nil)))
        log.Println(hex.EncodeToString(hashState2.Sum(nil)))
    }
**Output is**

    429eb588aa975a12e4dee3db42bd26f406d5931e9f71886b875defe218a6d119
    019e19457ba8f51801274b7a487b74c2283e63d527b448ab1bd039beeb8f3f5e 

## We still can use this package as original one

    fixedBytes := make([]byte, 80)
    
    hashState1 := sha256.New() //pointer on hash state 
    hashState1.Write(fixedBytes)
    
    hashState2 := hashState1 //pointer copy
    
    hashState1.Write([]byte{1, 2, 3, 4})
    hashState2.Write([]byte{5, 6, 7, 8})
    
    fmt.Println(hex.EncodeToString(hashState1.Sum(nil)))
    fmt.Println(hex.EncodeToString(hashState2.Sum(nil)))    

**Output is**

    8a8143bb66f9e38fdecd45c6b8ad686845c0202e05d5e8bbb5daddcaafc73d3b
    8a8143bb66f9e38fdecd45c6b8ad686845c0202e05d5e8bbb5daddcaafc73d3b
    
As you can see, in this case output hashes are the same. This is because previous code equals to

    fixedBytes := make([]byte, 80)
        
    hashState := sha256.New() 
    hashState.Write(fixedBytes)        
    hashState.Write([]byte{1, 2, 3, 4, 5, 6, 7, 8})        	
        
    fmt.Println(hex.EncodeToString(hashState.Sum(nil)))    

**Output is**
    
    8a8143bb66f9e38fdecd45c6b8ad686845c0202e05d5e8bbb5daddcaafc73d3b