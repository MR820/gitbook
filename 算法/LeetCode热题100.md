[**LeetCode 热题 100**](https://leetcode.cn/studyplan/top-100-liked/)

## 1. [两数之和](https://leetcode.cn/problems/two-sum/)

```go
func twoSum(nums []int, target int) []int {
	m := make(map[int]int)
	for i, num := range nums {
		if j, exit := m[target-num]; exit {
			return []int{j, i}
		}
		m[num] = i
	}
	return []int{-1, -1}
}

```

## 2. [字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

```go
```

