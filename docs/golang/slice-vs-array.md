# Slice vs Array

Arrays are regid and fixed-size, while {++ slices ++} are {++ dynamic, flexible windows ++} into underlying arrays.

## Arrays: Fixed and Rigid

Once you create an array, its size can never change. Because the size is literally part of the array's type, an array of `[3]int` and an array of `[4]int` are considered two entirely different types in Go.

Arrays in Go are **value types**. If you assign an array to a new variable or pass it to a function, Go creates a complete, independent copy of all the data.

``` go
package main

import "fmt"

func main() {
    // Declaring an array of size 3
    arr1 := [3]int{1, 2, 3} 
    
    // Assigning it to a new variable makes a FULL copy
    arr2 := arr1 
    arr2[0] = 99 

    fmt.Println("arr1:", arr1) // Output: [1 2 3] (Original is untouched)
    fmt.Println("arr2:", arr2) // Output: [99 2 3]
}
```

---

## Slices: Dynamic and Reference-Based

A slice does not actually store any data itself. Instead, it acts as a "view" or a "window" into an underlying, hidden array.

Under the hood, a slice is a tiny data structure containing just three pieces of information:

1. **Pointer**: Points to the first element of the underlying array it has access to.
1. **Length(`len`)**: The number of elements currently in the slice.
1. **Capacity(`cap`)**: The total amount of space in the underlying array starting from the pointer.

Because slices are essentially pointers to an underlying array, **assigning a slice to a new variable or passing it to a function does not copy the data**. They will both point to the same underlying array.

``` go
package main

import "fmt"

func main() {
    // Declaring a slice (Notice the empty brackets)
    slice1 := []int{1, 2, 3} 
    
    // Assigning it to a new variable points to the SAME backing array
    slice2 := slice1 
    slice2[0] = 99 

    fmt.Println("slice1:", slice1) // Output: [99 2 3] (Original is modified!)
    fmt.Println("slice2:", slice2) // Output: [99 2 3]
}
```

