# 1117. Building H2O

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/building-h2o/description/)

## Solution: Channel-Based Barrier Synchronization

To coordinate the execution of hydrogen and oxygen threads so they always form H2O molecules (2 Hydrogen and 1 Oxygen), we can construct a synchronization barrier. Using Go channels, we can cleanly regulate thread progression. A background manager goroutine dispenses tokens for exactly two Hydrogen threads and one Oxygen thread at a time, then waits for all three threads to finish execution before issuing tokens for the next molecule.

### Thought Process

1.  **Token Dispensation**:
    - `hCh`: A buffered channel of capacity 2, representing hydrogen permits.
    - `oCh`: A buffered channel of capacity 1, representing oxygen permits.
2.  **Barrier Release**:
    - `releaseCh`: An unbuffered channel used by the active threads to signal completion back to the manager.
3.  **Manager Goroutine**:
    - Runs a loop that performs the following steps in sequence:
        1. Fill `hCh` with 2 tokens (blocks if previous tokens haven't been consumed).
        2. Fill `oCh` with 1 token (blocks if previous token hasn't been consumed).
        3. Block until 3 release signals are received from `releaseCh` (indicating that 2 hydrogen threads and 1 oxygen thread have successfully completed).
4.  **Thread Execution (`Hydrogen` / `Oxygen`)**:
    - Wait to acquire a token from the corresponding permit channel (`hCh` or `oCh`).
    - Execute the callback (`releaseHydrogen` or `releaseOxygen`).
    - Send a signal to `releaseCh` to let the manager know the thread has completed its execution.

### Go Code

``` go
type H2O struct {
    hCh         chan struct{}
    oCh         chan struct{}
    releaseCh   chan struct{}
}

func NewH2O() *H2O {
	h2o := &H2O{
        hCh:        make(chan struct{}, 2),
        oCh:        make(chan struct{}, 1),
        releaseCh:  make(chan struct{}),
    }

    go func() {
        for {
            h2o.hCh <-struct{}{}
            h2o.hCh <-struct{}{}
            h2o.oCh <-struct{}{}

            <-h2o.releaseCh
            <-h2o.releaseCh
            <-h2o.releaseCh
        }
    }()

    return h2o
}

func (h *H2O) Hydrogen(releaseHydrogen func()) {
	<-h.hCh
	releaseHydrogen()
    h.releaseCh <-struct{}{}
}

func (h *H2O) Oxygen(releaseOxygen func()) {
	<-h.oCh
	releaseOxygen()
    h.releaseCh <-struct{}{}
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Where $n$ is the total number of hydrogen and oxygen threads. Each thread executes a constant number of channel sends and receives.
- **Space Complexity**: $O(1)$
    - The channels are initialized with fixed sizes (capacities of 2, 1, and 0) and do not scale with the input size.