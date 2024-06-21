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
func moveZeroes(nums []int) {
	if len(nums) < 2 {
		return
	}
	j := 0
	for i := 0; i < len(nums); i++ {
		if nums[i] != 0 {
			nums[j] = nums[i]
			j++
		}
	}
	for i := j; i < len(nums); i++ {
		nums[i] = 0
	}
}

```

## 5. [盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

```go
func maxArea(height []int) int {
	l, r := 0, len(height)-1
	var maxA = 0
	for l < r {
		area := min(height[l], height[r]) * (r - l)
		maxA = max(maxA, area)
		if height[l] < height[r] {
			l++
		} else {
			r--
		}
	}
	return maxA
}

```

## 6. [三数之和](https://leetcode.cn/problems/3sum/)

```go
func threeSum(nums []int) [][]int {
	n := len(nums)
	sort.Ints(nums)
	ans := make([][]int, 0)

	for first := 0; first < n; first++ {
		if first > 0 && nums[first] == nums[first-1] {
			continue
		}
		third := n - 1
		target := -1 * nums[first]
		for second := first + 1; second < n; second++ {
			if second > first+1 && nums[second] == nums[second-1] {
				continue
			}
			for second < third && nums[second]+nums[third] > target {
				third--
			}
			if second == third {
				break
			}
			if nums[second]+nums[third] == target {
				ans = append(ans, []int{nums[first], nums[second], nums[third]})
			}
		}
	}
	return ans
}

```

## 7. [接雨水](https://leetcode.cn/problems/trapping-rain-water/)

```go
func trap(height []int) int {
	left, right := 0, len(height)-1
	leftMax, rightMax := 0, 0
	ans := 0
	for left < right {
		leftMax = max(leftMax, height[left])
		rightMax = max(rightMax, height[right])
		if height[left] < height[right] {
			ans += leftMax - height[left]
			left++
		} else {
			ans += rightMax - height[right]
			right--
		}
	}
	return ans
}

```

# 滑动窗口

## 8. [无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

```go
func lengthOfLongestSubstring(s string) int {
	if len(s) == 0 {
		return 0
	}
	start, end := 0, 0
	m := make(map[byte]int)
	m[s[end]] = end
	maxLength := 1
  l := len(s)-1
	for end < l {
		end++
		if i, exit := m[s[end]]; exit {
			if i >= start {
				start = i + 1
			}
		}
		m[s[end]] = end
		maxLength = max(maxLength, end-start+1)
	}
	return maxLength
}
```

## 9. [找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

```go
func findAnagrams(s string, p string) []int {
	l := len(s)
	n := len(p)
	if l < n {
		return nil
	}
	pList := make([]int, 26)
	sList := make([]int, 26)
	for i := 0; i < n; i++ {
		pList[p[i]-'a']++
		sList[s[i]-'a']++
	}

	res := make([]int, 0)
	if equal(pList, sList) {
		res = append(res, 0)
	}

	for start := 0; start < l-n; start++ {
		sList[s[start]-'a']--
		end := start + n
		sList[s[end]-'a']++
		if equal(sList, pList) {
			res = append(res, start+1)
		}
	}
	return res
}

func equal(a, b []int) bool {
	if len(a) != len(b) {
		return false
	}
	for i := uint8(0); i < 26; i++ {
		if a[i] != b[i] {
			return false
		}
	}
	return true
}

```

## 10. [和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)

```go
func subarraySum(nums []int, k int) int {
	count, pre := 0, 0
	m := map[int]int{}
	m[0] = 1
	for i := 0; i < len(nums); i++ {
		pre += nums[i]
		if _, ok := m[pre-k]; ok {
			count += m[pre-k]
		}
		m[pre]++
	}
	return count
}
```

## 11. [滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

```go
func maxSlidingWindow(nums []int, k int) []int {
	q := []int{}
	push := func(i int) {
		for len(q) > 0 && nums[i] >= nums[q[len(q)-1]] {
			q = q[:len(q)-1]
		}
		q = append(q, i)
	}
	for i := 0; i < k; i++ {
		push(i)
	}

	n := len(nums)
	ans := make([]int, 1, n-k+1)
	ans[0] = nums[q[0]]
	for i := k; i < n; i++ {
		push(i)
		for q[0] <= i-k {
			q = q[1:]
		}
		ans = append(ans, nums[q[0]])
	}
	return ans
}

```

## 12. [最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)

