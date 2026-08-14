# Program vs Process

## Program (Passive Entity)

- An executable file that the CPU needs to accomplish a specific task 
- Usually refers to the executable file
- To run a program, it first needs to be loaded into memory as that's where the CPU can start fetching instructions and data

## Process (Active Entity)

- A program in execution
- Each process has its own memory space

## Memory Layout of the Process

- **TEXT**: contains the executable code. Never change in size and content.
- **DATA**: contains global variables and constant values. Doesn't change in size but its content varies.
- **HEAP**: contains all the data/files
- **STACK**: store all the data generated at runtime like user input and temporary results 

* **Process**: a program in execu

![Memory Layout of the Process](../assets/img/os-concepts/process-memory.svg)

---

## Reference

<iframe width="560" height="315" src="https://www.youtube.com/embed/7ge7u5VUSbE?si=1B-RenJ72OfaoKqH" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
