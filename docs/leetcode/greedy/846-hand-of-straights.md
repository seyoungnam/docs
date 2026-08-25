# 846. Hand of Straights

``` go
func isNStraightHand(hand []int, groupSize int) bool {
    n := len(hand)
    if n % groupSize != 0 {
        return false
    }
    count := map[int]int{}
    for _, num := range hand {
        count[num]++
    }
    sort.Ints(hand)
    for _, num := range hand {
        if count[num] > 0 {
            for i := num; i < num+groupSize; i++ {
                if count[i] == 0 {
                    return false
                }
                count[i]--
            }
        }
    }
    return true
}
```