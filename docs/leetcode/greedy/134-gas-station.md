# 134. Gas Station

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/gas-station/description/)

There are `n` gas stations along a circular route, where the amount of gas at the $i$-th station is `gas[i]`.

You have a car with an unlimited gas tank and it costs `cost[i]` of gas to travel from the $i$-th station to its next $(i + 1)$-th station. You begin the journey with an empty tank at one of the gas stations.

Given two integer arrays `gas` and `cost`, return *the starting gas station's index if you can travel around the circuit once in the clockwise direction, otherwise return* `-1`. If there exists a solution, it is **guaranteed** to be **unique**.

---

## Solution: Greedy (Single Pass)

We can solve this problem in a single linear pass by checking two conditions:
1.  **Global Feasibility**: If the total gas is less than the total cost ($\sum \text{gas} < \sum \text{cost}$), completing the circuit is mathematically impossible; return `-1`.
2.  **Local Start Determination**: If $\sum \text{gas} \ge \sum \text{cost}$, a unique starting station is guaranteed to exist. We can greedily identify it by checking where our fuel tank runs dry.

### Thought Process

1.  **Variables**:
    *   `totalGas`, `totalCost`: Accumulate the total gas and cost respectively.
    *   `start`: The candidate starting station index (initially `0`).
    *   `tank`: The running gas level in our car's tank.
2.  **Linear Scan**:
    *   Loop through each gas station `i`:
        *   Accumulate `totalGas += gas[i]` and `totalCost += cost[i]`.
        *   Update the running tank: `tank += gas[i] - cost[i]`.
        *   **Check Tank**: If `tank < 0`, it means we cannot reach station `i + 1` from our current `start` candidate.
        *   **Reset Candidate**: Since we started with a positive (or zero) tank and failed to reach `i + 1`, any intermediate starting station between `start` and `i` would start with less (or zero) gas and also fail. Thus, we greedily reset our starting candidate to the next station: `start = i + 1`, and reset `tank = 0`.
3.  **Validation**:
    *   If `totalGas < totalCost`, return `-1`.
    *   Otherwise, return the final `start` candidate.

### Go Code

``` go
func canCompleteCircuit(gas []int, cost []int) int {
    totalGas, totalCost := 0, 0
    start, tank := 0, 0
    
    for i := range gas {
        totalGas += gas[i]
        totalCost += cost[i]
        tank += gas[i] - cost[i]
        
        // If the tank goes negative, we cannot start from any index <= i
        if tank < 0 {
            start = i + 1
            tank = 0
        }
    }
    
    // If total gas is less than total cost, completing the loop is impossible
    if totalGas < totalCost {
        return -1
    }
    
    return start
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of gas stations. We iterate through the array exactly once.
- **Space Complexity**: $O(1)$
    - We only use constant auxiliary variables, achieving optimal space complexity.