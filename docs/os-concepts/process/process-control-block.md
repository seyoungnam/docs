# Process Control Block

## Concurrency

A modern operating system uses concurrency, allowing multiple processes to execute by **alternating access** to the CPU. Internally, alternating access to the CPU is achieved by placing processes in a queue. At any given moment, only one process can use the CPU, while all others wait their turn.


## CPU Components

The CPU has the following internal components:
- **General purpose registers**
- **Instruction Register**
- **Address Register(Program Counter)**
- **Stack Pointer**
- **Flags**

## How CPU is used for a task

Imagine we want to run the following program:

``` rust
fn main(){
  let a = 25 + 100;
}
```

The above rust program is translated to the following assembly instructions:

``` assembly
LOAD R0 25
LOAD R1 100
ADD R0 R1
STORE R0 [a]
```




---

## Reference

<iframe width="560" height="315" src="https://www.youtube.com/embed/LDhoD4IVElk?si=KSLKqaAgDdPJrin6" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
