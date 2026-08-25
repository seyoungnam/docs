# 152. Maximum Product Subarray

``` go
func maxProduct(nums []int) int {
    res, minVal, maxVal := nums[0], nums[0], nums[0]
    for i := 1; i < len(nums); i++ {
        num := nums[i]
        maxCur := max(num, max(minVal*num, maxVal*num))
        minCur := min(num, min(minVal*num, maxVal*num))

        minVal, maxVal = minCur, maxCur
        res = max(res, maxVal)
    }
    return res
}
```