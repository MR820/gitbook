[**LeetCode 热题 100**](https://leetcode.cn/studyplan/top-100-liked/)

# 哈希

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
func groupAnagrams(strs []string) [][]string {
	mp := make(map[string][]string)
	for _, str := range strs {
		s := []byte(str)
		sort.Slice(s, func(i, j int) bool { return s[i] < s[j] })
		sortedStr := string(s)
		mp[sortedStr] = append(mp[sortedStr], str)
	}
	ans := make([][]string, 0, len(mp))
	for _, v := range mp {
		ans = append(ans, v)
	}
	return ans
}
```

## 3. [最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

```go
func longestConsecutive(nums []int) int {
	if len(nums) <= 1 {
		return len(nums)
	}
	sort.Slice(nums, func(i, j int) bool {
		return nums[i] < nums[j]
	})
	maxNum := 1
	n := 1
	for i := 0; i < len(nums)-1; i++ {
		if nums[i]+1 == nums[i+1] {
			n++
			maxNum = max(maxNum, n)
		} else if nums[i]+1 < nums[i+1] {
			n = 1
		}
	}
	return maxNum
}

```

# 双指针

## 4. [移动零](https://leetcode.cn/problems/move-zeroes/)

```go
```

