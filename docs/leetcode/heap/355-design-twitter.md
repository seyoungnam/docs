# 355. Design Twitter

``` go
import (
    "container/heap"
)

type maxHeap [][4]int
func (h maxHeap) Len() int { return len(h) }
func (h maxHeap) Less(i, j int) bool { return h[i][0] > h[j][0] }
func (h maxHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *maxHeap) Push(x interface{}) { *h = append(*h, x.([4]int)) }
func (h *maxHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}
func (h maxHeap) Top() interface{} {
    return h[0]
}

type Twitter struct {
    count       int
    tweetMap    map[int][][2]int        // userId -> [count, tweetId]
    followMap   map[int]map[int]bool    // userId -> set of followeeIds
}


func Constructor() Twitter {
    return Twitter{
        count: 0,
        tweetMap: make(map[int][][2]int),
        followMap: make(map[int]map[int]bool),
    }
}


func (this *Twitter) PostTweet(userId int, tweetId int)  {
    this.tweetMap[userId] = append(this.tweetMap[userId], [2]int{this.count, tweetId})
    this.count++
}


func (this *Twitter) GetNewsFeed(userId int) []int {
    res := []int{}
    h := &maxHeap{}
    heap.Init(h)

    if this.followMap[userId] == nil {
        this.followMap[userId] = map[int]bool{}
    }
    this.followMap[userId][userId] = true

    followees := this.followMap[userId]
    for followeeId, _ := range followees {
        if listTweets, found := this.tweetMap[followeeId]; found {
            index := len(listTweets)-1
            count, tweetId := listTweets[index][0], listTweets[index][1]
            heap.Push(h, [4]int{count, tweetId, followeeId, index-1})
        }
    }

    for h.Len() > 0 && len(res) < 10 {
        curr := heap.Pop(h).([4]int)
        tweetId, followeeId, index := curr[1], curr[2], curr[3]
        res = append(res, tweetId)
        if index >= 0 {
            prevTweet := this.tweetMap[followeeId][index]
            prevCount, prevTweetId := prevTweet[0], prevTweet[1]
            heap.Push(h, [4]int{prevCount, prevTweetId, followeeId, index-1})
        }
    }
    return res
}
```