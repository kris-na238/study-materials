# **Top 10 Interview Patterns**

*Full revision guide — problem statement, optimal solution, complexity, and alternate approaches for 100 practice problems.*

## Overview

| **#** | **Pattern** | **Time Complexity Goal** | **Common Problems** |
| --- | --- | --- | --- |
| **1** | **Hash Map / Hash Set** | O(n²) → O(n) | Two Sum, Contains Duplicate, Valid Anagram, … |
| **2** | **Two Pointers** | O(n²) → O(n) | Pair Sum (Two Sum II - Sorted Array), Valid Palindrome, 3Sum, … |
| **3** | **Sliding Window** | O(n²) → O(n) | Longest Substring Without Repeating Characters, Maximum Sum Subarray of Size K, Longest Repeating Character Replacement, … |
| **4** | **Prefix Sum** | O(n²) → O(n) | Subarray Sum Equals K, Range Sum Query - Immutable, Product of Array Except Self, … |
| **5** | **Binary Search** | O(n) → O(log n) | Binary Search (classic), Search in Rotated Sorted Array, Find First and Last Position of Element in Sorted Array, … |
| **6** | **Sorting** | Simplifies logic | Merge Intervals, Meeting Rooms, Meeting Rooms II, … |
| **7** | **Stack** | O(n²) → O(n) | Valid Parentheses, Next Greater Element I & II, Daily Temperatures, … |
| **8** | **Queue / BFS** | Efficient traversal | Binary Tree Level Order Traversal, Shortest Path in Binary Matrix, Rotting Oranges, … |
| **9** | **Recursion + DFS** | Tree/Graph traversal | Maximum Depth of Binary Tree (Tree Height), Path Sum, Path Sum II, … |
| **10** | **Dynamic Programming** | Exponential → Polynomial | Fibonacci Number / Climbing Stairs, House Robber, House Robber II, … |

# **1. Hash Map / Hash Set**

**Goal:** O(n²) → O(n)

**Core idea:** *Trade space for time by storing seen values/indices/counts for O(1) lookup instead of nested loops.*

## **1.1  Two Sum**

**Problem:** Given an array of integers nums and a target, return the indices of the two numbers that add up to target. Assume exactly one solution exists.

**Approach:** Walk through the array once. For each number, check if (target - number) already exists in a hash map. If yes, return the stored index and current index. If no, store the current number → index in the map and continue.

**Optimal Solution (JavaScript):**

```js
function twoSum(nums, target) {
  const seen = new Map(); // value -> index
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (seen.has(complement)) {
      return [seen.get(complement), i];
    }
    seen.set(nums[i], i);
  }
  return [];
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Brute force: check every pair with nested loops — O(n²) time, O(1) space. Sort + two pointers works too, but you lose original indices unless you store them alongside values first — O(n log n) time.*

☐ Practiced   ☐ Revised

## **1.2  Contains Duplicate**

**Problem:** Given an array of integers, return true if any value appears at least twice.

**Approach:** Add each element to a hash set as you iterate. If an element is already in the set, a duplicate exists — return true immediately.

**Optimal Solution (JavaScript):**

```js
function containsDuplicate(nums) {
  const seen = new Set();
  for (const n of nums) {
    if (seen.has(n)) return true;
    seen.add(n);
  }
  return false;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Sort the array first, then check adjacent elements for equality — O(n log n) time, O(1) extra space (if in-place sort allowed).*

☐ Practiced   ☐ Revised

## **1.3  Valid Anagram**

**Problem:** Given two strings s and t, return true if t is an anagram of s (same letters, same frequency).

**Approach:** Count character frequencies of s in a hash map, then decrement counts while scanning t. If any count goes negative or a character in t isn't in the map, return false.

**Optimal Solution (JavaScript):**

```js
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  const count = new Map();
  for (const c of s) count.set(c, (count.get(c) || 0) + 1);
  for (const c of t) {
    if (!count.get(c)) return false;
    count.set(c, count.get(c) - 1);
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(1) — at most 26 lowercase letters

***Alternate approaches:*** *Sort both strings and compare for equality — O(n log n) time, O(1) extra space beyond sort overhead. Simple but slower than counting.*

☐ Practiced   ☐ Revised

## **1.4  Group Anagrams**

**Problem:** Given an array of strings, group the anagrams together and return the groups (in any order).

**Approach:** For each string, compute a canonical key — either the sorted characters or a 26-length count signature. Use that key in a hash map pointing to a list of original strings that share it.

**Optimal Solution (JavaScript):**

```js
function groupAnagrams(strs) {
  const groups = new Map();
  for (const s of strs) {
    const key = s.split('').sort().join('');
    if (!groups.has(key)) groups.set(key, []);
    groups.get(key).push(s);
  }
  return [...groups.values()];
}
```

**Complexity:** Time O(n · k log k) where k = max string length, Space O(n · k)

***Alternate approaches:*** *Use a frequency-count tuple (26 numbers) as the key instead of sorting — avoids the log k factor, giving O(n · k) time at the cost of slightly more complex key construction.*

☐ Practiced   ☐ Revised

## **1.5  Top K Frequent Elements**

**Problem:** Given an integer array and k, return the k most frequent elements.

**Approach:** Count frequencies with a hash map. Then either sort by frequency, or use bucket sort where bucket index = frequency (frequency can't exceed array length), and read off the top k from the highest buckets.

**Optimal Solution (JavaScript):**

```js
function topKFrequent(nums, k) {
  const freq = new Map();
  for (const n of nums) freq.set(n, (freq.get(n) || 0) + 1);
 
  const buckets = Array.from({ length: nums.length + 1 }, () => []);
  for (const [num, f] of freq) buckets[f].push(num);
 
  const result = [];
  for (let f = buckets.length - 1; f >= 0 && result.length < k; f--) {
    for (const num of buckets[f]) {
      result.push(num);
      if (result.length === k) break;
    }
  }
  return result;
}
```

**Complexity:** Time O(n) with bucket sort (O(n log n) with a heap/sort), Space O(n)

***Alternate approaches:*** *Use a min-heap of size k over frequency counts — push each unique number, pop when heap exceeds k. O(n log k) time, useful when k is small relative to n.*

☐ Practiced   ☐ Revised

## **1.6  Longest Consecutive Sequence**

**Problem:** Given an unsorted array of integers, find the length of the longest run of consecutive integers (order doesn't matter, must run in O(n)).

**Approach:** Put all numbers in a hash set. For each number that is a 'sequence start' (i.e., number - 1 is NOT in the set), count upward (num+1, num+2, ...) while they exist in the set, tracking the longest streak.

**Optimal Solution (JavaScript):**

```js
function longestConsecutive(nums) {
  const set = new Set(nums);
  let longest = 0;
 
  for (const n of set) {
    if (!set.has(n - 1)) { // only start counting from sequence starts
      let length = 1;
      while (set.has(n + length)) length++;
      longest = Math.max(longest, length);
    }
  }
  return longest;
}
```

**Complexity:** Time O(n) — each number is visited at most twice, Space O(n)

***Alternate approaches:*** *Sort the array first, then scan once counting consecutive runs (skip duplicates) — O(n log n) time, O(1) extra space if sorting in place. Simpler to reason about but slower asymptotically.*

☐ Practiced   ☐ Revised

## **1.7  Subarray Sum Equals K**

**Problem:** Given an array of integers and an integer k, return the number of contiguous subarrays whose sum equals k.

**Approach:** Track running prefix sum while scanning. Keep a hash map of {prefix sum → count of times seen}. At each index, if (currentSum - k) exists in the map, add its count to the answer — those are subarrays ending here that sum to k.

**Optimal Solution (JavaScript):**

```js
function subarraySum(nums, k) {
  const prefixCount = new Map([[0, 1]]);
  let sum = 0, count = 0;
 
  for (const n of nums) {
    sum += n;
    count += prefixCount.get(sum - k) || 0;
    prefixCount.set(sum, (prefixCount.get(sum) || 0) + 1);
  }
  return count;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Brute force: compute the sum of every subarray with two nested loops — O(n²) time, O(1) space. Fine only for small inputs.*

☐ Practiced   ☐ Revised

## **1.8  First Unique Character in a String**

**Problem:** Given a string, find the index of its first non-repeating character. Return -1 if none exists.

**Approach:** First pass: count frequency of every character in a hash map. Second pass: scan the string in order and return the index of the first character whose count is exactly 1.

**Optimal Solution (JavaScript):**

```js
function firstUniqChar(s) {
  const count = new Map();
  for (const c of s) count.set(c, (count.get(c) || 0) + 1);
  for (let i = 0; i < s.length; i++) {
    if (count.get(s[i]) === 1) return i;
  }
  return -1;
}
```

**Complexity:** Time O(n), Space O(1) — bounded alphabet

***Alternate approaches:*** *Use an array of size 26 (or 256 for extended ASCII) instead of a Map for slightly faster constant factors when the character set is known and small.*

☐ Practiced   ☐ Revised

## **1.9  Isomorphic Strings**

**Problem:** Given two strings s and t, return true if the characters in s can be replaced to get t, with a consistent one-to-one mapping both ways.

**Approach:** Maintain two hash maps: one mapping s-chars → t-chars, one mapping t-chars → s-chars. Walk both strings together; if a mapping already exists and conflicts with the current pair, return false.

**Optimal Solution (JavaScript):**

```js
function isIsomorphic(s, t) {
  if (s.length !== t.length) return false;
  const mapST = new Map(), mapTS = new Map();
 
  for (let i = 0; i < s.length; i++) {
    const a = s[i], b = t[i];
    if (mapST.has(a) && mapST.get(a) !== b) return false;
    if (mapTS.has(b) && mapTS.get(b) !== a) return false;
    mapST.set(a, b);
    mapTS.set(b, a);
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(1) — bounded alphabet

***Alternate approaches:*** *Encode both strings by 'first occurrence index' pattern (e.g. 'egg' → [0,1,1]) and compare the encoded arrays for equality — same complexity, different implementation style.*

☐ Practiced   ☐ Revised

## **1.10  Ransom Note**

**Problem:** Given strings ransomNote and magazine, return true if ransomNote can be constructed using letters from magazine (each letter used at most as many times as it appears in magazine).

**Approach:** Count character frequencies of magazine in a hash map. Then scan ransomNote, decrementing counts; if any character is missing or its count hits zero, return false.

**Optimal Solution (JavaScript):**

```js
function canConstruct(ransomNote, magazine) {
  const count = new Map();
  for (const c of magazine) count.set(c, (count.get(c) || 0) + 1);
 
  for (const c of ransomNote) {
    if (!count.get(c)) return false;
    count.set(c, count.get(c) - 1);
  }
  return true;
}
```

**Complexity:** Time O(n + m), Space O(1) — bounded alphabet

***Alternate approaches:*** *Use a fixed-size array of 26 counters instead of a Map for a small constant-factor speedup when only lowercase letters are involved.*

☐ Practiced   ☐ Revised

# **2. Two Pointers**

**Goal:** O(n²) → O(n)

**Core idea:** *Use two indices moving toward each other (or same direction) on sorted data / arrays to avoid nested loops.*

## **2.1  Pair Sum (Two Sum II - Sorted Array)**

**Problem:** Given a sorted array and a target, return the 1-indexed indices of two numbers that add up to target.

**Approach:** Place a left pointer at the start and right pointer at the end. If the sum is too small, move left forward; if too large, move right backward; if equal, return the indices.

**Optimal Solution (JavaScript):**

```js
function twoSumSorted(nums, target) {
  let left = 0, right = nums.length - 1;
  while (left < right) {
    const sum = nums[left] + nums[right];
    if (sum === target) return [left + 1, right + 1];
    if (sum < target) left++;
    else right--;
  }
  return [];
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Hash map of value → index works in O(n) time too, but uses O(n) space — worse than two pointers here since the array is already sorted.*

☐ Practiced   ☐ Revised

## **2.2  Valid Palindrome**

**Problem:** Given a string, determine if it's a palindrome considering only alphanumeric characters and ignoring case.

**Approach:** Use left and right pointers starting at both ends. Skip non-alphanumeric characters on either side, then compare characters (case-insensitive); mismatch means not a palindrome.

**Optimal Solution (JavaScript):**

```js
function isPalindrome(s) {
  let left = 0, right = s.length - 1;
  const isAlnum = c => /[a-z0-9]/i.test(c);
 
  while (left < right) {
    while (left < right && !isAlnum(s[left])) left++;
    while (left < right && !isAlnum(s[right])) right--;
    if (s[left].toLowerCase() !== s[right].toLowerCase()) return false;
    left++; right--;
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Filter and lowercase the string into a clean array first, then compare it to its reverse — O(n) time but O(n) extra space; simpler to write, less efficient.*

☐ Practiced   ☐ Revised

## **2.3  3Sum**

**Problem:** Given an array of integers, find all unique triplets that sum to zero.

**Approach:** Sort the array. Fix one number, then use two pointers (left, right) on the remaining sorted range to find pairs summing to its negation. Skip duplicate values at every level to avoid repeat triplets.

**Optimal Solution (JavaScript):**

```js
function threeSum(nums) {
  nums.sort((a, b) => a - b);
  const result = [];
 
  for (let i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue; // skip dup anchors
    let left = i + 1, right = nums.length - 1;
 
    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];
      if (sum === 0) {
        result.push([nums[i], nums[left], nums[right]]);
        while (left < right && nums[left] === nums[left + 1]) left++;
        while (left < right && nums[right] === nums[right - 1]) right--;
        left++; right--;
      } else if (sum < 0) left++;
      else right--;
    }
  }
  return result;
}
```

**Complexity:** Time O(n²), Space O(1) extra (excluding output / sort space)

***Alternate approaches:*** *Hash-set approach: fix one number, then for the rest use a set to check if the needed complement has been seen — also O(n²) time but O(n) space and trickier duplicate handling.*

☐ Practiced   ☐ Revised

## **2.4  Container With Most Water**

**Problem:** Given heights of vertical lines, find two lines that together with the x-axis form a container holding the most water.

**Approach:** Start with left and right pointers at the two ends (widest container). Compute area, then move the pointer at the shorter line inward — moving the taller one can never improve the area since width only shrinks.

**Optimal Solution (JavaScript):**

```js
function maxArea(height) {
  let left = 0, right = height.length - 1, best = 0;
 
  while (left < right) {
    const h = Math.min(height[left], height[right]);
    best = Math.max(best, h * (right - left));
    if (height[left] < height[right]) left++;
    else right--;
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Brute force checks every pair of lines — O(n²) time, O(1) space. Correct but far slower for large inputs.*

☐ Practiced   ☐ Revised

## **2.5  Remove Duplicates from Sorted Array**

**Problem:** Given a sorted array, remove duplicates in place so each unique element appears once, and return the new length.

**Approach:** Use a slow pointer marking the position of the last unique element written, and a fast pointer scanning ahead. Whenever the fast pointer finds a new value, advance slow and write it there.

**Optimal Solution (JavaScript):**

```js
function removeDuplicates(nums) {
  if (nums.length === 0) return 0;
  let slow = 0;
  for (let fast = 1; fast < nums.length; fast++) {
    if (nums[fast] !== nums[slow]) {
      slow++;
      nums[slow] = nums[fast];
    }
  }
  return slow + 1;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Copy unique elements into a new array/Set — O(n) time, O(n) space; not truly 'in place', which most interview variants require.*

☐ Practiced   ☐ Revised

## **2.6  Trapping Rain Water**

**Problem:** Given an elevation map (array of heights), compute how much water it can trap after raining.

**Approach:** Two pointers from both ends, tracking the max height seen so far from each side (leftMax, rightMax). At each step, move the pointer on the side with the smaller max — water trapped at that position is leftMax/rightMax minus the current bar height.

**Optimal Solution (JavaScript):**

```js
function trap(height) {
  let left = 0, right = height.length - 1;
  let leftMax = 0, rightMax = 0, water = 0;
 
  while (left < right) {
    if (height[left] < height[right]) {
      leftMax = Math.max(leftMax, height[left]);
      water += leftMax - height[left];
      left++;
    } else {
      rightMax = Math.max(rightMax, height[right]);
      water += rightMax - height[right];
      right--;
    }
  }
  return water;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Precompute leftMax[] and rightMax[] arrays in two passes, then a third pass sums min(leftMax[i], rightMax[i]) - height[i] — O(n) time but O(n) space. A monotonic stack approach also works in O(n) time/O(n) space, useful for visualizing trapped 'basins'.*

☐ Practiced   ☐ Revised

## **2.7  Sort Colors (Dutch National Flag)**

**Problem:** Given an array with values 0, 1, 2 representing colors, sort it in place in one pass without using a library sort.

**Approach:** Use three pointers: low (boundary for 0s), mid (current element), high (boundary for 2s). Swap 0s to the front, 2s to the back, and advance mid only when the value is confirmed placed (1, or after a 0-swap).

**Optimal Solution (JavaScript):**

```js
function sortColors(nums) {
  let low = 0, mid = 0, high = nums.length - 1;
  while (mid <= high) {
    if (nums[mid] === 0) {
      [nums[low], nums[mid]] = [nums[mid], nums[low]];
      low++; mid++;
    } else if (nums[mid] === 1) {
      mid++;
    } else {
      [nums[mid], nums[high]] = [nums[high], nums[mid]];
      high--;
    }
  }
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Counting sort: count occurrences of 0, 1, 2 in one pass, then overwrite the array in a second pass — O(n) time, O(1) space, simpler but two passes instead of one.*

☐ Practiced   ☐ Revised

## **2.8  Move Zeroes**

**Problem:** Given an array, move all zeroes to the end while maintaining the relative order of non-zero elements, in place.

**Approach:** Use a slow pointer marking where the next non-zero element should go. Fast pointer scans the array; whenever it finds a non-zero, swap it into the slow position and advance slow.

**Optimal Solution (JavaScript):**

```js
function moveZeroes(nums) {
  let slow = 0;
  for (let fast = 0; fast < nums.length; fast++) {
    if (nums[fast] !== 0) {
      [nums[slow], nums[fast]] = [nums[fast], nums[slow]];
      slow++;
    }
  }
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Two-pass approach: first pass copies non-zero elements to the front in order, second pass fills the remaining positions with zero — same O(n) time, slightly easier to reason about.*

☐ Practiced   ☐ Revised

## **2.9  Merge Sorted Array**

**Problem:** Given two sorted arrays nums1 (with extra trailing space) and nums2, merge nums2 into nums1 in place as one sorted array.

**Approach:** Fill from the back. Use pointers at the last real elements of nums1 and nums2, and a pointer at the very end of nums1's buffer. Repeatedly place the larger of the two candidates at the end, moving inward — avoids overwriting unprocessed nums1 elements.

**Optimal Solution (JavaScript):**

```js
function merge(nums1, m, nums2, n) {
  let i = m - 1, j = n - 1, k = m + n - 1;
  while (j >= 0) {
    if (i >= 0 && nums1[i] > nums2[j]) {
      nums1[k--] = nums1[i--];
    } else {
      nums1[k--] = nums2[j--];
    }
  }
}
```

**Complexity:** Time O(m + n), Space O(1)

***Alternate approaches:*** *Merge into a new array front-to-back (classic merge-sort merge step), then copy back into nums1 — O(m + n) time but O(m + n) extra space.*

☐ Practiced   ☐ Revised

## **2.10  Boats to Save People**

**Problem:** Given people's weights and a boat weight limit (each boat carries at most 2 people), find the minimum number of boats needed to carry everyone.

**Approach:** Sort weights. Use two pointers: lightest and heaviest person. If they fit together under the limit, pair them and move both pointers; otherwise the heaviest goes alone. Count boats either way.

**Optimal Solution (JavaScript):**

```js
function numRescueBoats(people, limit) {
  people.sort((a, b) => a - b);
  let left = 0, right = people.length - 1, boats = 0;
 
  while (left <= right) {
    if (people[left] + people[right] <= limit) left++;
    right--;
    boats++;
  }
  return boats;
}
```

**Complexity:** Time O(n log n) for sorting, Space O(1) extra

***Alternate approaches:*** *Greedy without explicit pointers using a deque (pop lightest and heaviest each round) — same complexity, different data structure framing.*

☐ Practiced   ☐ Revised

# **3. Sliding Window**

**Goal:** O(n²) → O(n)

**Core idea:** *Maintain a shrinking/growing window over a contiguous range to track a running condition (sum, count, distinct chars).*

## **3.1  Longest Substring Without Repeating Characters**

**Problem:** Given a string, find the length of the longest substring without repeating characters.

**Approach:** Expand the window with a right pointer, tracking characters seen in a hash set/map. When a repeat is found, shrink from the left until the repeat is removed. Track the max window size throughout.

**Optimal Solution (JavaScript):**

```js
function lengthOfLongestSubstring(s) {
  const lastSeen = new Map();
  let left = 0, best = 0;
 
  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    if (lastSeen.has(c) && lastSeen.get(c) >= left) {
      left = lastSeen.get(c) + 1;
    }
    lastSeen.set(c, right);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(min(n, alphabet size))

***Alternate approaches:*** *Brute force checks every substring for uniqueness — O(n³) time (or O(n²) with a running set). Sliding window with a plain set (shrink one at a time instead of jumping) also works in O(n) amortized time.*

☐ Practiced   ☐ Revised

## **3.2  Maximum Sum Subarray of Size K**

**Problem:** Given an array and integer k, find the maximum sum of any contiguous subarray of size exactly k.

**Approach:** Compute the sum of the first window of size k. Then slide the window one step at a time: subtract the element leaving, add the element entering, and track the max sum seen.

**Optimal Solution (JavaScript):**

```js
function maxSumSubarray(nums, k) {
  let windowSum = 0;
  for (let i = 0; i < k; i++) windowSum += nums[i];
  let best = windowSum;
 
  for (let right = k; right < nums.length; right++) {
    windowSum += nums[right] - nums[right - k];
    best = Math.max(best, windowSum);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Brute force recomputes the sum of every window from scratch — O(n·k) time. Prefix sums also give O(n) time with O(n) space, useful if you need many different range queries.*

☐ Practiced   ☐ Revised

## **3.3  Longest Repeating Character Replacement**

**Problem:** Given a string and integer k, find the length of the longest substring you can get after replacing at most k characters, such that all characters in it are the same.

**Approach:** Expand the window while tracking counts of each character and the count of the most frequent character in the window. If (window size - max frequency) exceeds k, too many characters need replacing — shrink from the left.

**Optimal Solution (JavaScript):**

```js
function characterReplacement(s, k) {
  const count = new Map();
  let left = 0, maxFreq = 0, best = 0;
 
  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    count.set(c, (count.get(c) || 0) + 1);
    maxFreq = Math.max(maxFreq, count.get(c));
 
    while (right - left + 1 - maxFreq > k) {
      count.set(s[left], count.get(s[left]) - 1);
      left++;
    }
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1) — bounded alphabet

***Alternate approaches:*** *Instead of shrinking the window, keep it the same size once maxFreq is achieved and just slide it forward (never shrinking) — a common optimization that still yields the correct max length in O(n) time.*

☐ Practiced   ☐ Revised

## **3.4  Minimum Window Substring**

**Problem:** Given strings s and t, find the smallest substring of s that contains all characters of t (including duplicates).

**Approach:** Track required character counts from t in a map. Expand the window in s, decrementing counts and tracking how many required characters are fully satisfied. Once all are satisfied, shrink from the left to minimize the window, recording the best result along the way.

**Optimal Solution (JavaScript):**

```js
function minWindow(s, t) {
  if (t.length > s.length) return "";
  const need = new Map();
  for (const c of t) need.set(c, (need.get(c) || 0) + 1);
 
  let required = need.size, formed = 0;
  const windowCount = new Map();
  let left = 0, bestLen = Infinity, bestStart = 0;
 
  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    windowCount.set(c, (windowCount.get(c) || 0) + 1);
    if (need.has(c) && windowCount.get(c) === need.get(c)) formed++;
 
    while (formed === required) {
      if (right - left + 1 < bestLen) {
        bestLen = right - left + 1;
        bestStart = left;
      }
      const lc = s[left];
      windowCount.set(lc, windowCount.get(lc) - 1);
      if (need.has(lc) && windowCount.get(lc) < need.get(lc)) formed--;
      left++;
    }
  }
  return bestLen === Infinity ? "" : s.slice(bestStart, bestStart + bestLen);
}
```

**Complexity:** Time O(|s| + |t|), Space O(|s| + |t|)

***Alternate approaches:*** *Brute force checks every substring of s against t's requirement — O(n² · k) time. A fixed-size 128/256-array instead of a Map can slightly speed up constant factors for ASCII input.*

☐ Practiced   ☐ Revised

## **3.5  Permutation in String**

**Problem:** Given strings s1 and s2, return true if s2 contains a permutation (anagram) of s1 as a contiguous substring.

**Approach:** Build a frequency count of s1. Slide a fixed-size window of length s1.length over s2, maintaining a running frequency count of the window and comparing it to s1's count (or tracking a 'matches' counter) for equality.

**Optimal Solution (JavaScript):**

```js
function checkInclusion(s1, s2) {
  if (s1.length > s2.length) return false;
  const need = new Array(26).fill(0);
  const window = new Array(26).fill(0);
  const idx = c => c.charCodeAt(0) - 97;
 
  for (const c of s1) need[idx(c)]++;
 
  for (let i = 0; i < s2.length; i++) {
    window[idx(s2[i])]++;
    if (i >= s1.length) window[idx(s2[i - s1.length])]--;
    if (i >= s1.length - 1 && need.every((v, k) => v === window[k])) return true;
  }
  return false;
}
```

**Complexity:** Time O(n · 26) ≈ O(n), Space O(1)

***Alternate approaches:*** *Track a single 'matches' counter that increments/decrements as counts align/misalign, avoiding the O(26) array comparison each step — brings it to strict O(n) time with a tighter constant.*

☐ Practiced   ☐ Revised

## **3.6  Sliding Window Maximum**

**Problem:** Given an array and window size k, return the maximum value in each sliding window as it moves from left to right.

**Approach:** Use a monotonic deque storing indices, keeping values in decreasing order. Pop smaller elements from the back before adding a new one (they can never be the max while the new element is in range). Pop from the front when the index falls out of the window. The front of the deque is always the current max.

**Optimal Solution (JavaScript):**

```js
function maxSlidingWindow(nums, k) {
  const deque = []; // stores indices, values decreasing
  const result = [];
 
  for (let i = 0; i < nums.length; i++) {
    while (deque.length && deque[0] <= i - k) deque.shift();
    while (deque.length && nums[deque[deque.length - 1]] < nums[i]) deque.pop();
    deque.push(i);
    if (i >= k - 1) result.push(nums[deque[0]]);
  }
  return result;
}
```

**Complexity:** Time O(n) — each index pushed/popped once, Space O(k)

***Alternate approaches:*** *Brute force recomputes the max of each window — O(n·k) time. A max-heap with lazy deletion also works in O(n log n) time, simpler to write but slower asymptotically.*

☐ Practiced   ☐ Revised

## **3.7  Fruit Into Baskets**

**Problem:** Given an array of fruit types, find the length of the longest subarray containing at most 2 distinct types.

**Approach:** This is 'longest substring with at most 2 distinct characters' in disguise. Expand the window with a right pointer, tracking counts of each fruit type in a map. When distinct types exceed 2, shrink from the left until it's back to 2.

**Optimal Solution (JavaScript):**

```js
function totalFruit(fruits) {
  const count = new Map();
  let left = 0, best = 0;
 
  for (let right = 0; right < fruits.length; right++) {
    count.set(fruits[right], (count.get(fruits[right]) || 0) + 1);
 
    while (count.size > 2) {
      const lf = fruits[left];
      count.set(lf, count.get(lf) - 1);
      if (count.get(lf) === 0) count.delete(lf);
      left++;
    }
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1) — at most 3 distinct types in the map at once

***Alternate approaches:*** *Generalize to 'at most k distinct types' by replacing the constant 2 with k — same technique, same complexity, useful as a template for a family of problems.*

☐ Practiced   ☐ Revised

## **3.8  Longest Substring with At Most K Distinct Characters**

**Problem:** Given a string and integer k, find the length of the longest substring containing at most k distinct characters.

**Approach:** Expand the window with a right pointer, tracking character counts in a map. Whenever the number of distinct characters (map size) exceeds k, shrink from the left, removing characters from the map once their count hits zero.

**Optimal Solution (JavaScript):**

```js
function lengthOfLongestSubstringKDistinct(s, k) {
  const count = new Map();
  let left = 0, best = 0;
 
  for (let right = 0; right < s.length; right++) {
    count.set(s[right], (count.get(s[right]) || 0) + 1);
 
    while (count.size > k) {
      const lc = s[left];
      count.set(lc, count.get(lc) - 1);
      if (count.get(lc) === 0) count.delete(lc);
      left++;
    }
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(k)

***Alternate approaches:*** *If k is very large relative to the alphabet, an array-based counter can replace the map for a small constant-factor speedup.*

☐ Practiced   ☐ Revised

## **3.9  Max Consecutive Ones III**

**Problem:** Given a binary array and integer k, find the maximum number of consecutive 1s you can get by flipping at most k zeroes.

**Approach:** Expand the window, counting zeroes inside it. If the zero count exceeds k, shrink from the left until it's back within budget. Track the max window size throughout — this is the flip-budget variant of longest-substring sliding window.

**Optimal Solution (JavaScript):**

```js
function longestOnes(nums, k) {
  let left = 0, zeroCount = 0, best = 0;
 
  for (let right = 0; right < nums.length; right++) {
    if (nums[right] === 0) zeroCount++;
 
    while (zeroCount > k) {
      if (nums[left] === 0) zeroCount--;
      left++;
    }
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Same 'never shrink, only slide' optimization as Longest Repeating Character Replacement applies here too, since we only care about the max window length, not every valid window.*

☐ Practiced   ☐ Revised

## **3.10  Subarrays with K Different Integers**

**Problem:** Given an array and integer k, count the number of contiguous subarrays with exactly k distinct integers.

**Approach:** Exactly k is hard to slide directly, but 'at most k' minus 'at most (k-1)' gives 'exactly k'. Write one helper for 'count subarrays with at most k distinct' using standard sliding window, and call it twice.

**Optimal Solution (JavaScript):**

```js
function subarraysWithKDistinct(nums, k) {
  const atMost = (limit) => {
    const count = new Map();
    let left = 0, total = 0;
    for (let right = 0; right < nums.length; right++) {
      count.set(nums[right], (count.get(nums[right]) || 0) + 1);
      while (count.size > limit) {
        const lv = nums[left];
        count.set(lv, count.get(lv) - 1);
        if (count.get(lv) === 0) count.delete(lv);
        left++;
      }
      total += right - left + 1; // all subarrays ending at 'right' within window
    }
    return total;
  };
  return atMost(k) - atMost(k - 1);
}
```

**Complexity:** Time O(n), Space O(k)

***Alternate approaches:*** *Direct 'exactly k' tracking with two windows in parallel (upper bound k, upper bound k-1) simultaneously — same complexity, avoids calling the helper twice but is harder to read.*

☐ Practiced   ☐ Revised

# **4. Prefix Sum**

**Goal:** O(n²) → O(n)

**Core idea:** *Precompute cumulative sums so any range-sum query becomes O(1); pair with a hash map for subarray-sum problems.*

## **4.1  Subarray Sum Equals K**

**Problem:** Given an array of integers and an integer k, return the number of contiguous subarrays whose sum equals k.

**Approach:** Track a running prefix sum. Keep a hash map of {prefix sum → number of times seen}. At each step, if (currentSum - k) was seen before, every one of those occurrences marks a valid subarray ending here.

**Optimal Solution (JavaScript):**

```js
function subarraySum(nums, k) {
  const prefixCount = new Map([[0, 1]]);
  let sum = 0, count = 0;
 
  for (const n of nums) {
    sum += n;
    count += prefixCount.get(sum - k) || 0;
    prefixCount.set(sum, (prefixCount.get(sum) || 0) + 1);
  }
  return count;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Brute force sums every subarray directly with nested loops — O(n²) time, O(1) space.*

☐ Practiced   ☐ Revised

## **4.2  Range Sum Query - Immutable**

**Problem:** Design a class that answers multiple sum(i, j) queries efficiently over a fixed, unchanging array.

**Approach:** Precompute a prefix sum array where prefix[i] = sum of nums[0..i-1]. Then sum(i, j) = prefix[j+1] - prefix[i], answered in O(1) after O(n) preprocessing.

**Optimal Solution (JavaScript):**

```js
class NumArray {
  constructor(nums) {
    this.prefix = [0];
    for (const n of nums) this.prefix.push(this.prefix[this.prefix.length - 1] + n);
  }
  sumRange(i, j) {
    return this.prefix[j + 1] - this.prefix[i];
  }
}
```

**Complexity:** Time O(n) build, O(1) per query, Space O(n)

***Alternate approaches:*** *Without precomputing, sum a subrange directly on every query — O(1) build but O(n) per query, only worth it if queries are rare.*

☐ Practiced   ☐ Revised

## **4.3  Product of Array Except Self**

**Problem:** Given an array, return an array where each element is the product of all other elements, without using division.

**Approach:** Compute a 'prefix product' pass left-to-right (product of everything before i) and a 'suffix product' pass right-to-left (product of everything after i). Multiply the two for each index. Can be done in O(1) extra space by reusing the output array for prefixes and folding in suffixes on a second pass.

**Optimal Solution (JavaScript):**

```js
function productExceptSelf(nums) {
  const n = nums.length;
  const result = new Array(n).fill(1);
 
  let prefix = 1;
  for (let i = 0; i < n; i++) {
    result[i] = prefix;
    prefix *= nums[i];
  }
 
  let suffix = 1;
  for (let i = n - 1; i >= 0; i--) {
    result[i] *= suffix;
    suffix *= nums[i];
  }
  return result;
}
```

**Complexity:** Time O(n), Space O(1) extra (excluding output array)

***Alternate approaches:*** *If division were allowed: compute the total product, then divide by nums[i] for each index — O(n) time but breaks on zeroes and is often disallowed in interviews for that reason.*

☐ Practiced   ☐ Revised

## **4.4  Contiguous Array (equal 0s and 1s)**

**Problem:** Given a binary array, find the length of the longest contiguous subarray with an equal number of 0s and 1s.

**Approach:** Convert 0s to -1s conceptually. Track a running sum; a subarray has equal 0s and 1s exactly when the running sum returns to a previously seen value. Store {sum → first index it was seen} in a hash map to find the longest such span.

**Optimal Solution (JavaScript):**

```js
function findMaxLength(nums) {
  const firstIndex = new Map([[0, -1]]);
  let sum = 0, best = 0;
 
  for (let i = 0; i < nums.length; i++) {
    sum += nums[i] === 1 ? 1 : -1;
    if (firstIndex.has(sum)) {
      best = Math.max(best, i - firstIndex.get(sum));
    } else {
      firstIndex.set(sum, i);
    }
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Brute force checks every subarray's 0/1 balance directly — O(n²) time, O(1) space.*

☐ Practiced   ☐ Revised

## **4.5  Continuous Subarray Sum**

**Problem:** Given an array and integer k, return true if the array has a contiguous subarray of size at least 2 whose sum is a multiple of k.

**Approach:** Track running sum modulo k. If the same remainder appears at two indices at least 2 apart, the subarray between them sums to a multiple of k. Store {remainder → first index seen} in a hash map.

**Optimal Solution (JavaScript):**

```js
function checkSubarraySum(nums, k) {
  const firstIndex = new Map([[0, -1]]);
  let sum = 0;
 
  for (let i = 0; i < nums.length; i++) {
    sum += nums[i];
    const rem = k === 0 ? sum : sum % k;
    if (firstIndex.has(rem)) {
      if (i - firstIndex.get(rem) > 1) return true;
    } else {
      firstIndex.set(rem, i);
    }
  }
  return false;
}
```

**Complexity:** Time O(n), Space O(min(n, k))

***Alternate approaches:*** *Brute force checks every subarray sum directly and tests divisibility — O(n²) time, O(1) space.*

☐ Practiced   ☐ Revised

## **4.6  Find Pivot Index**

**Problem:** Given an array, find the leftmost index where the sum of elements to the left equals the sum of elements to the right.

**Approach:** Compute the total sum first. Then scan left to right maintaining a running left-sum; at each index, the right-sum is total - leftSum - current. Compare left-sum to right-sum directly without a separate array.

**Optimal Solution (JavaScript):**

```js
function pivotIndex(nums) {
  const total = nums.reduce((a, b) => a + b, 0);
  let leftSum = 0;
 
  for (let i = 0; i < nums.length; i++) {
    const rightSum = total - leftSum - nums[i];
    if (leftSum === rightSum) return i;
    leftSum += nums[i];
  }
  return -1;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Precompute explicit prefix and suffix sum arrays, then compare index by index — O(n) time, O(n) space; functionally identical, more memory used.*

☐ Practiced   ☐ Revised

## **4.7  Minimum Size Subarray Sum**

**Problem:** Given an array of positive integers and a target sum, find the minimal length of a contiguous subarray whose sum is at least target.

**Approach:** This is really a sliding window over a running (prefix-like) sum: expand the window's right edge accumulating sum; once sum ≥ target, shrink from the left while still valid, tracking the minimum window length found.

**Optimal Solution (JavaScript):**

```js
function minSubArrayLen(target, nums) {
  let left = 0, sum = 0, best = Infinity;
 
  for (let right = 0; right < nums.length; right++) {
    sum += nums[right];
    while (sum >= target) {
      best = Math.min(best, right - left + 1);
      sum -= nums[left];
      left++;
    }
  }
  return best === Infinity ? 0 : best;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Since all elements are positive, binary search on window length combined with prefix sums also works in O(n log n) time — useful if elements could be non-positive, forcing away from the sliding window.*

☐ Practiced   ☐ Revised

## **4.8  Subarray Sums Divisible by K**

**Problem:** Given an array and integer k, return the number of contiguous subarrays whose sum is divisible by k.

**Approach:** Track running sum modulo k (normalized to be non-negative). Keep a hash map of {remainder → count of times seen}. At each step, add the count of the current remainder seen before — those mark subarrays divisible by k ending here.

**Optimal Solution (JavaScript):**

```js
function subarraysDivByK(nums, k) {
  const remCount = new Map([[0, 1]]);
  let sum = 0, count = 0;
 
  for (const n of nums) {
    sum += n;
    let rem = sum % k;
    if (rem < 0) rem += k; // normalize for negative sums
    count += remCount.get(rem) || 0;
    remCount.set(rem, (remCount.get(rem) || 0) + 1);
  }
  return count;
}
```

**Complexity:** Time O(n), Space O(k)

***Alternate approaches:*** *Brute force computes every subarray sum directly and checks divisibility — O(n²) time, O(1) space.*

☐ Practiced   ☐ Revised

## **4.9  Maximum Size Subarray Sum Equals K**

**Problem:** Given an array and integer k, find the length of the longest contiguous subarray that sums to exactly k.

**Approach:** Track running prefix sum. Keep a hash map of {prefix sum → earliest index it occurred}. At each step, if (currentSum - k) was seen before, the subarray from just after that index to here sums to k — track the longest such span.

**Optimal Solution (JavaScript):**

```js
function maxSubArrayLen(nums, k) {
  const firstIndex = new Map([[0, -1]]);
  let sum = 0, best = 0;
 
  for (let i = 0; i < nums.length; i++) {
    sum += nums[i];
    if (firstIndex.has(sum - k)) {
      best = Math.max(best, i - firstIndex.get(sum - k));
    }
    if (!firstIndex.has(sum)) firstIndex.set(sum, i); // keep earliest only
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Brute force computes every subarray sum with nested loops, tracking the longest match — O(n²) time, O(1) space.*

☐ Practiced   ☐ Revised

# **5. Binary Search**

**Goal:** O(n) → O(log n)

**Core idea:** *Repeatedly halve the search space on sorted (or monotonic) data; also used for 'search on the answer' problems.*

## **5.1  Binary Search (classic)**

**Problem:** Given a sorted array and a target, return its index, or -1 if not found.

**Approach:** Maintain low and high bounds. Compute mid; if nums[mid] equals target, return it. If nums[mid] is smaller, search the right half; otherwise search the left half.

**Optimal Solution (JavaScript):**

```js
function search(nums, target) {
  let low = 0, high = nums.length - 1;
  while (low <= high) {
    const mid = low + Math.floor((high - low) / 2);
    if (nums[mid] === target) return mid;
    if (nums[mid] < target) low = mid + 1;
    else high = mid - 1;
  }
  return -1;
}
```

**Complexity:** Time O(log n), Space O(1)

***Alternate approaches:*** *Linear scan works in O(n) time, O(1) space — only reasonable if the array isn't sorted or n is tiny.*

☐ Practiced   ☐ Revised

## **5.2  Search in Rotated Sorted Array**

**Problem:** Given a rotated sorted array (no duplicates) and a target, return its index or -1.

**Approach:** Standard binary search, but at each step determine which half (left of mid or right of mid) is properly sorted, then check if the target lies within that half's range to decide which side to continue on.

**Optimal Solution (JavaScript):**

```js
function search(nums, target) {
  let low = 0, high = nums.length - 1;
  while (low <= high) {
    const mid = low + Math.floor((high - low) / 2);
    if (nums[mid] === target) return mid;
 
    if (nums[low] <= nums[mid]) { // left half is sorted
      if (nums[low] <= target && target < nums[mid]) high = mid - 1;
      else low = mid + 1;
    } else { // right half is sorted
      if (nums[mid] < target && target <= nums[high]) low = mid + 1;
      else high = mid - 1;
    }
  }
  return -1;
}
```

**Complexity:** Time O(log n), Space O(1)

***Alternate approaches:*** *Find the rotation pivot first with a separate binary search, then do a normal binary search on the correct half — same complexity, two-step and arguably easier to reason about.*

☐ Practiced   ☐ Revised

## **5.3  Find First and Last Position of Element in Sorted Array**

**Problem:** Given a sorted array and a target, find the starting and ending index of target's occurrences (or [-1, -1] if absent).

**Approach:** Run two binary searches: one biased to find the leftmost occurrence (keep searching left even after finding a match), one biased to find the rightmost occurrence (keep searching right after a match).

**Optimal Solution (JavaScript):**

```js
function searchRange(nums, target) {
  const findBound = (isFirst) => {
    let low = 0, high = nums.length - 1, result = -1;
    while (low <= high) {
      const mid = low + Math.floor((high - low) / 2);
      if (nums[mid] === target) {
        result = mid;
        if (isFirst) high = mid - 1; else low = mid + 1;
      } else if (nums[mid] < target) low = mid + 1;
      else high = mid - 1;
    }
    return result;
  };
  return [findBound(true), findBound(false)];
}
```

**Complexity:** Time O(log n), Space O(1)

***Alternate approaches:*** *Linear scan from both ends inward, or a single binary search to find any occurrence followed by linear expansion outward — O(n) worst case if the target is very frequent.*

☐ Practiced   ☐ Revised

## **5.4  Search a 2D Matrix**

**Problem:** Given an m x n matrix where each row is sorted and the first element of each row is greater than the last element of the previous row, determine if target exists.

**Approach:** Treat the matrix as one flattened sorted array of size m*n. Binary search over indices 0..m*n-1, converting each mid index back to (row, col) via division/modulo.

**Optimal Solution (JavaScript):**

```js
function searchMatrix(matrix, target) {
  const rows = matrix.length, cols = matrix[0].length;
  let low = 0, high = rows * cols - 1;
 
  while (low <= high) {
    const mid = low + Math.floor((high - low) / 2);
    const val = matrix[Math.floor(mid / cols)][mid % cols];
    if (val === target) return true;
    if (val < target) low = mid + 1;
    else high = mid - 1;
  }
  return false;
}
```

**Complexity:** Time O(log(m·n)), Space O(1)

***Alternate approaches:*** *Start from the top-right corner and move left or down based on comparisons — O(m + n) time, O(1) space; works even without the stricter 'first element > previous row's last' guarantee.*

☐ Practiced   ☐ Revised

## **5.5  Find Minimum in Rotated Sorted Array**

**Problem:** Given a rotated sorted array with no duplicates, find the minimum element.

**Approach:** Binary search comparing nums[mid] to nums[high]. If nums[mid] > nums[high], the minimum is in the right half (search low = mid + 1); otherwise it's in the left half including mid (search high = mid).

**Optimal Solution (JavaScript):**

```js
function findMin(nums) {
  let low = 0, high = nums.length - 1;
  while (low < high) {
    const mid = low + Math.floor((high - low) / 2);
    if (nums[mid] > nums[high]) low = mid + 1;
    else high = mid;
  }
  return nums[low];
}
```

**Complexity:** Time O(log n), Space O(1)

***Alternate approaches:*** *Linear scan for the point where nums[i] > nums[i+1] — O(n) time, O(1) space; correct but ignores the sorted structure.*

☐ Practiced   ☐ Revised

## **5.6  Koko Eating Bananas**

**Problem:** Given piles of bananas and h hours, find the minimum eating speed k such that Koko can eat all bananas within h hours.

**Approach:** Binary search on the answer (the speed k), ranging from 1 to the max pile size. For each candidate speed, compute hours needed (sum of ceil(pile/k) for all piles); if it fits within h, try a smaller speed, otherwise try larger.

**Optimal Solution (JavaScript):**

```js
function minEatingSpeed(piles, h) {
  let low = 1, high = Math.max(...piles);
 
  const hoursNeeded = (k) =>
    piles.reduce((total, p) => total + Math.ceil(p / k), 0);
 
  while (low < high) {
    const mid = low + Math.floor((high - low) / 2);
    if (hoursNeeded(mid) <= h) high = mid;
    else low = mid + 1;
  }
  return low;
}
```

**Complexity:** Time O(n log(max pile)), Space O(1)

***Alternate approaches:*** *Linear scan over every possible speed from 1 upward, checking feasibility — O(max pile · n) time, far slower for large pile sizes.*

☐ Practiced   ☐ Revised

## **5.7  Capacity To Ship Packages Within D Days**

**Problem:** Given package weights and D days, find the minimum ship capacity so all packages can be shipped (in given order) within D days.

**Approach:** Binary search on the answer (capacity), ranging from max(weights) to sum(weights). For each candidate capacity, greedily count how many days it takes to ship everything; adjust the search bound based on whether that's within D.

**Optimal Solution (JavaScript):**

```js
function shipWithinDays(weights, days) {
  let low = Math.max(...weights);
  let high = weights.reduce((a, b) => a + b, 0);
 
  const daysNeeded = (capacity) => {
    let days = 1, load = 0;
    for (const w of weights) {
      if (load + w > capacity) { days++; load = 0; }
      load += w;
    }
    return days;
  };
 
  while (low < high) {
    const mid = low + Math.floor((high - low) / 2);
    if (daysNeeded(mid) <= days) high = mid;
    else low = mid + 1;
  }
  return low;
}
```

**Complexity:** Time O(n log(sum of weights)), Space O(1)

***Alternate approaches:*** *Dynamic programming over (package index, days used) to find the exact minimum max-load partition — more general but higher complexity; binary search on the answer is the standard efficient approach here.*

☐ Practiced   ☐ Revised

## **5.8  Median of Two Sorted Arrays**

**Problem:** Given two sorted arrays, find the median of the combined array in better than O(m+n) time.

**Approach:** Binary search on the smaller array to find a partition point such that all elements on the left side of both partitions are ≤ all elements on the right side. Adjust the partition based on comparing boundary elements until the partition is balanced.

**Optimal Solution (JavaScript):**

```js
function findMedianSortedArrays(nums1, nums2) {
  if (nums1.length > nums2.length) [nums1, nums2] = [nums2, nums1];
  const m = nums1.length, n = nums2.length;
  let low = 0, high = m;
 
  while (low <= high) {
    const cut1 = low + Math.floor((high - low) / 2);
    const cut2 = Math.floor((m + n + 1) / 2) - cut1;
 
    const left1 = cut1 === 0 ? -Infinity : nums1[cut1 - 1];
    const left2 = cut2 === 0 ? -Infinity : nums2[cut2 - 1];
    const right1 = cut1 === m ? Infinity : nums1[cut1];
    const right2 = cut2 === n ? Infinity : nums2[cut2];
 
    if (left1 <= right2 && left2 <= right1) {
      if ((m + n) % 2 === 0) {
        return (Math.max(left1, left2) + Math.min(right1, right2)) / 2;
      }
      return Math.max(left1, left2);
    } else if (left1 > right2) {
      high = cut1 - 1;
    } else {
      low = cut1 + 1;
    }
  }
}
```

**Complexity:** Time O(log(min(m, n))), Space O(1)

***Alternate approaches:*** *Merge both arrays (or simulate the merge without full materialization) and pick the middle element(s) — O(m + n) time, simpler but doesn't meet the classic log-time constraint.*

☐ Practiced   ☐ Revised

## **5.9  Find Peak Element**

**Problem:** Given an array where adjacent elements differ, find the index of any peak element (greater than both neighbors; edges compare to a virtual -infinity).

**Approach:** Binary search comparing nums[mid] to nums[mid+1]. If nums[mid] < nums[mid+1], a peak must exist to the right, so move low = mid + 1; otherwise a peak exists at mid or to the left, so move high = mid.

**Optimal Solution (JavaScript):**

```js
function findPeakElement(nums) {
  let low = 0, high = nums.length - 1;
  while (low < high) {
    const mid = low + Math.floor((high - low) / 2);
    if (nums[mid] < nums[mid + 1]) low = mid + 1;
    else high = mid;
  }
  return low;
}
```

**Complexity:** Time O(log n), Space O(1)

***Alternate approaches:*** *Linear scan for any index where the sequence stops increasing — O(n) time, O(1) space; simple but doesn't exploit the structure.*

☐ Practiced   ☐ Revised

## **5.10  Search Insert Position**

**Problem:** Given a sorted array and a target, return the index if found; if not, return the index where it would be inserted to keep the array sorted.

**Approach:** Standard binary search; when the loop ends without finding the target, the low pointer naturally lands on the correct insertion index.

**Optimal Solution (JavaScript):**

```js
function searchInsert(nums, target) {
  let low = 0, high = nums.length - 1;
  while (low <= high) {
    const mid = low + Math.floor((high - low) / 2);
    if (nums[mid] === target) return mid;
    if (nums[mid] < target) low = mid + 1;
    else high = mid - 1;
  }
  return low;
}
```

**Complexity:** Time O(log n), Space O(1)

***Alternate approaches:*** *Linear scan for the first element ≥ target — O(n) time, O(1) space; correct but unnecessary given sorted input.*

☐ Practiced   ☐ Revised

# **6. Sorting**

**Goal:** Simplifies logic

**Core idea:** *Sorting first often turns a hard problem into an easy scan/two-pointer problem; know the sort's complexity trade-offs.*

## **6.1  Merge Intervals**

**Problem:** Given a collection of intervals, merge all overlapping intervals and return the result.

**Approach:** Sort intervals by start time. Walk through them, keeping a 'current merged interval'; if the next interval overlaps (its start ≤ current end), extend the current interval's end, otherwise push current and start a new one.

**Optimal Solution (JavaScript):**

```js
function merge(intervals) {
  intervals.sort((a, b) => a[0] - b[0]);
  const result = [intervals[0]];
 
  for (let i = 1; i < intervals.length; i++) {
    const last = result[result.length - 1];
    const [start, end] = intervals[i];
    if (start <= last[1]) {
      last[1] = Math.max(last[1], end);
    } else {
      result.push([start, end]);
    }
  }
  return result;
}
```

**Complexity:** Time O(n log n), Space O(n)

***Alternate approaches:*** *Without sorting, you'd need to compare every interval to every other — O(n²) time. A sweep-line over start/end events also works in O(n log n) time and generalizes well to counting overlaps.*

☐ Practiced   ☐ Revised

## **6.2  Meeting Rooms**

**Problem:** Given meeting time intervals, determine if a person could attend all meetings (i.e., no two meetings overlap).

**Approach:** Sort intervals by start time. Scan through consecutive pairs; if any meeting starts before the previous one ends, overlap exists — return false.

**Optimal Solution (JavaScript):**

```js
function canAttendMeetings(intervals) {
  intervals.sort((a, b) => a[0] - b[0]);
  for (let i = 1; i < intervals.length; i++) {
    if (intervals[i][0] < intervals[i - 1][1]) return false;
  }
  return true;
}
```

**Complexity:** Time O(n log n), Space O(1) extra

***Alternate approaches:*** *Brute force compares every pair of intervals for overlap — O(n²) time, O(1) space.*

☐ Practiced   ☐ Revised

## **6.3  Meeting Rooms II**

**Problem:** Given meeting time intervals, find the minimum number of conference rooms required to hold all meetings.

**Approach:** Separate start times and end times into two sorted arrays. Use two pointers: advance the start pointer, and whenever a meeting starts before the earliest ongoing meeting ends, a new room is needed; otherwise a room frees up. Track the max concurrent rooms.

**Optimal Solution (JavaScript):**

```js
function minMeetingRooms(intervals) {
  const starts = intervals.map(i => i[0]).sort((a, b) => a - b);
  const ends = intervals.map(i => i[1]).sort((a, b) => a - b);
 
  let rooms = 0, maxRooms = 0, s = 0, e = 0;
  while (s < starts.length) {
    if (starts[s] < ends[e]) {
      rooms++;
      s++;
    } else {
      rooms--;
      e++;
    }
    maxRooms = Math.max(maxRooms, rooms);
  }
  return maxRooms;
}
```

**Complexity:** Time O(n log n), Space O(n)

***Alternate approaches:*** *Min-heap of end times: for each meeting (sorted by start), pop the heap if its earliest end time ≤ current start, then push the current end time. Heap size at any point is the rooms in use — O(n log n) time, conceptually maps directly to 'rooms in use'.*

☐ Practiced   ☐ Revised

## **6.4  Valid Anagram (via sorting)**

**Problem:** Given two strings s and t, return true if t is an anagram of s.

**Approach:** Sort both strings' characters and compare the results directly — anagrams produce identical sorted strings.

**Optimal Solution (JavaScript):**

```js
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  return s.split('').sort().join('') === t.split('').sort().join('');
}
```

**Complexity:** Time O(n log n), Space O(n)

***Alternate approaches:*** *Character frequency counting in a hash map or 26-length array gives O(n) time, O(1) space — faster, and the more common interview-preferred solution.*

☐ Practiced   ☐ Revised

## **6.5  Sort Colors**

**Problem:** Given an array with values 0, 1, 2, sort it in place without using a library sort.

**Approach:** Either run counting sort (count each value, then overwrite the array), or the one-pass Dutch National Flag three-pointer partition.

**Optimal Solution (JavaScript):**

```js
function sortColors(nums) {
  const counts = [0, 0, 0];
  for (const n of nums) counts[n]++;
 
  let idx = 0;
  for (let color = 0; color < 3; color++) {
    for (let i = 0; i < counts[color]; i++) nums[idx++] = color;
  }
}
```

**Complexity:** Time O(n), Space O(1) — fixed-size count array

***Alternate approaches:*** *The Dutch National Flag three-pointer swap achieves the same O(n) time in a single pass without a separate counting pass — see the Two Pointers section for that version.*

☐ Practiced   ☐ Revised

## **6.6  Largest Number**

**Problem:** Given a list of non-negative integers, arrange them to form the largest possible number (as a string).

**Approach:** Convert numbers to strings and sort with a custom comparator: for two strings a and b, put a before b if (a+b) as a concatenation is numerically greater than (b+a). Join the sorted strings.

**Optimal Solution (JavaScript):**

```js
function largestNumber(nums) {
  const strs = nums.map(String);
  strs.sort((a, b) => (b + a).localeCompare(a + b));
  const result = strs.join('');
  return result[0] === '0' ? '0' : result; // handle all-zero case
}
```

**Complexity:** Time O(n log n · k) where k = avg digit length, Space O(n)

***Alternate approaches:*** *Brute force tries every permutation and picks the numerically largest — O(n!) time, completely impractical beyond tiny inputs.*

☐ Practiced   ☐ Revised

## **6.7  Non-overlapping Intervals**

**Problem:** Given a set of intervals, find the minimum number of intervals to remove to make the rest non-overlapping.

**Approach:** Sort intervals by end time. Greedily keep the interval with the earliest end time whenever there's overlap, since it leaves the most room for future intervals; count how many must be removed.

**Optimal Solution (JavaScript):**

```js
function eraseOverlapIntervals(intervals) {
  intervals.sort((a, b) => a[1] - b[1]);
  let removed = 0, prevEnd = -Infinity;
 
  for (const [start, end] of intervals) {
    if (start >= prevEnd) {
      prevEnd = end;
    } else {
      removed++;
    }
  }
  return removed;
}
```

**Complexity:** Time O(n log n), Space O(1) extra

***Alternate approaches:*** *Dynamic programming (longest chain of non-overlapping intervals, then subtract from total) also solves it in O(n log n) or O(n²) depending on formulation — the greedy approach is simpler and equally optimal here.*

☐ Practiced   ☐ Revised

## **6.8  Insert Interval**

**Problem:** Given a sorted list of non-overlapping intervals and a new interval, insert it and merge as necessary.

**Approach:** Walk through intervals in three phases: add all intervals ending before the new one starts unchanged, merge all intervals that overlap the new one (expanding its bounds), then add the merged interval and all remaining intervals unchanged.

**Optimal Solution (JavaScript):**

```js
function insert(intervals, newInterval) {
  const result = [];
  let i = 0, n = intervals.length;
  let [start, end] = newInterval;
 
  while (i < n && intervals[i][1] < start) result.push(intervals[i++]);
 
  while (i < n && intervals[i][0] <= end) {
    start = Math.min(start, intervals[i][0]);
    end = Math.max(end, intervals[i][1]);
    i++;
  }
  result.push([start, end]);
 
  while (i < n) result.push(intervals[i++]);
  return result;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Append the new interval to the list and run the general Merge Intervals routine (sort + merge) — O(n log n) time; simpler to write, but ignores that the input was already sorted.*

☐ Practiced   ☐ Revised

## **6.9  Kth Largest Element in an Array**

**Problem:** Given an array and integer k, find the kth largest element (not necessarily distinct).

**Approach:** Sort the array descending (or ascending and index from the end) and return the element at position k-1. A more efficient alternative uses Quickselect for average O(n).

**Optimal Solution (JavaScript):**

```js
function findKthLargest(nums, k) {
  nums.sort((a, b) => b - a);
  return nums[k - 1];
}
```

**Complexity:** Time O(n log n), Space O(1) extra (excluding sort internals)

***Alternate approaches:*** *Quickselect (partition-based, like quicksort but recursing into only one side) gives average O(n) time, worst case O(n²). A min-heap of size k gives O(n log k) time and is the standard approach when data streams in.*

☐ Practiced   ☐ Revised

## **6.10  Custom Sort String**

**Problem:** Given a string 'order' representing a custom character priority, and a string s, sort the characters of s according to that order.

**Approach:** Count the frequency of each character in s. Build the result by appending each character in 'order' as many times as it appears, then append any remaining characters (not in order) at the end.

**Optimal Solution (JavaScript):**

```js
function customSortString(order, s) {
  const count = new Map();
  for (const c of s) count.set(c, (count.get(c) || 0) + 1);
 
  let result = '';
  for (const c of order) {
    if (count.has(c)) {
      result += c.repeat(count.get(c));
      count.delete(c);
    }
  }
  for (const [c, freq] of count) result += c.repeat(freq);
  return result;
}
```

**Complexity:** Time O(|s| + |order|), Space O(|s|)

***Alternate approaches:*** *Build a priority map from 'order' (character → rank) and sort s's characters using that rank as a custom comparator — O(|s| log |s|) time, more general if 'order' isn't guaranteed to cover all characters cleanly.*

☐ Practiced   ☐ Revised

# **7. Stack**

**Goal:** O(n²) → O(n)

**Core idea:** *LIFO structure ideal for matching pairs, nested structures, and 'next greater/smaller element' style scans (monotonic stack).*

## **7.1  Valid Parentheses**

**Problem:** Given a string containing '()[]{}', determine if the brackets are balanced and correctly nested.

**Approach:** Push opening brackets onto a stack. On a closing bracket, check the stack's top matches the corresponding opener; if so pop it, otherwise it's invalid. At the end, the stack must be empty.

**Optimal Solution (JavaScript):**

```js
function isValid(s) {
  const pairs = { ')': '(', ']': '[', '}': '{' };
  const stack = [];
 
  for (const c of s) {
    if (c === '(' || c === '[' || c === '{') {
      stack.push(c);
    } else {
      if (stack.pop() !== pairs[c]) return false;
    }
  }
  return stack.length === 0;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Repeatedly replace matched pairs '()', '[]', '{}' with empty string until no more replacements occur, then check if the string is empty — O(n²) time in the worst case due to repeated scanning, much less efficient.*

☐ Practiced   ☐ Revised

## **7.2  Next Greater Element I & II**

**Problem:** Given an array, find for each element the next element to its right that is greater (or -1 if none). Variant II treats the array as circular.

**Approach:** Use a monotonic decreasing stack of indices. Scan the array (twice for the circular variant, using modulo); while the current value is greater than the value at the stack's top index, pop and record the current value as that index's answer.

**Optimal Solution (JavaScript):**

```js
function nextGreaterElements(nums) {
  const n = nums.length;
  const result = new Array(n).fill(-1);
  const stack = []; // indices, values decreasing
 
  for (let i = 0; i < 2 * n; i++) {
    const val = nums[i % n];
    while (stack.length && nums[stack[stack.length - 1]] < val) {
      result[stack.pop()] = val;
    }
    if (i < n) stack.push(i);
  }
  return result;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Brute force checks every element against all elements to its right (wrapping for the circular case) — O(n²) time, O(1) extra space.*

☐ Practiced   ☐ Revised

## **7.3  Daily Temperatures**

**Problem:** Given daily temperatures, return for each day how many days you'd have to wait for a warmer temperature (0 if none).

**Approach:** Monotonic decreasing stack of indices. While the current temperature is higher than the temperature at the stack's top index, pop it and set its answer as the distance between indices.

**Optimal Solution (JavaScript):**

```js
function dailyTemperatures(temperatures) {
  const result = new Array(temperatures.length).fill(0);
  const stack = []; // indices, temps decreasing
 
  for (let i = 0; i < temperatures.length; i++) {
    while (stack.length && temperatures[stack[stack.length - 1]] < temperatures[i]) {
      const idx = stack.pop();
      result[idx] = i - idx;
    }
    stack.push(i);
  }
  return result;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Brute force scans forward from each day until a warmer one is found — O(n²) worst case (e.g., strictly decreasing temperatures).*

☐ Practiced   ☐ Revised

## **7.4  Min Stack**

**Problem:** Design a stack that supports push, pop, top, and retrieving the minimum element, all in O(1).

**Approach:** Maintain a second stack that tracks the minimum at each point. Whenever you push a value, also push the current minimum (min of new value and previous min) onto the min-stack; pop both together.

**Optimal Solution (JavaScript):**

```js
class MinStack {
  constructor() {
    this.stack = [];
    this.minStack = [];
  }
  push(val) {
    this.stack.push(val);
    const currMin = this.minStack.length ? Math.min(val, this.minStack[this.minStack.length - 1]) : val;
    this.minStack.push(currMin);
  }
  pop() {
    this.stack.pop();
    this.minStack.pop();
  }
  top() {
    return this.stack[this.stack.length - 1];
  }
  getMin() {
    return this.minStack[this.minStack.length - 1];
  }
}
```

**Complexity:** Time O(1) for all operations, Space O(n)

***Alternate approaches:*** *Store (value, minSoFar) pairs in a single stack instead of two parallel stacks — same complexity, marginally simpler bookkeeping. Alternatively, recompute the min by scanning the whole stack on demand — O(n) per getMin call, defeats the purpose.*

☐ Practiced   ☐ Revised

## **7.5  Evaluate Reverse Polish Notation**

**Problem:** Given tokens representing an arithmetic expression in Reverse Polish (postfix) notation, evaluate it.

**Approach:** Scan tokens left to right. Push numbers onto a stack. On an operator, pop the top two numbers, apply the operator, and push the result back.

**Optimal Solution (JavaScript):**

```js
function evalRPN(tokens) {
  const stack = [];
  const ops = {
    '+': (a, b) => a + b,
    '-': (a, b) => a - b,
    '*': (a, b) => a * b,
    '/': (a, b) => Math.trunc(a / b),
  };
 
  for (const token of tokens) {
    if (ops[token]) {
      const b = stack.pop(), a = stack.pop();
      stack.push(ops[token](a, b));
    } else {
      stack.push(Number(token));
    }
  }
  return stack[0];
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Convert postfix to an expression tree recursively and evaluate the tree — same O(n) complexity but more overhead building the tree structure; only useful if you need the tree for other purposes.*

☐ Practiced   ☐ Revised

## **7.6  Largest Rectangle in Histogram**

**Problem:** Given heights of bars in a histogram, find the area of the largest rectangle that can be formed.

**Approach:** Monotonic increasing stack of indices. When a bar shorter than the stack's top is found, pop and compute the area using the popped bar's height, with width determined by the current index and the new stack top (the previous shorter bar).

**Optimal Solution (JavaScript):**

```js
function largestRectangleArea(heights) {
  const stack = []; // indices, heights increasing
  let maxArea = 0;
 
  for (let i = 0; i <= heights.length; i++) {
    const h = i === heights.length ? 0 : heights[i];
    while (stack.length && heights[stack[stack.length - 1]] >= h) {
      const height = heights[stack.pop()];
      const width = stack.length ? i - stack[stack.length - 1] - 1 : i;
      maxArea = Math.max(maxArea, height * width);
    }
    stack.push(i);
  }
  return maxArea;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Brute force checks every pair (or every bar as a potential min-height anchor, expanding outward) — O(n²) time, O(1) space.*

☐ Practiced   ☐ Revised

## **7.7  Asteroid Collision**

**Problem:** Given an array representing asteroids (sign = direction, magnitude = size), simulate collisions and return the final state.

**Approach:** Use a stack to represent surviving asteroids moving left (negative) at the top. For each new right-moving asteroid encountering left-moving ones on the stack, resolve collisions by comparing sizes, popping smaller ones, until either it's destroyed, survives, or the stack top no longer conflicts.

**Optimal Solution (JavaScript):**

```js
function asteroidCollision(asteroids) {
  const stack = [];
 
  for (const a of asteroids) {
    let alive = true;
    while (alive && a < 0 && stack.length && stack[stack.length - 1] > 0) {
      const top = stack[stack.length - 1];
      if (top < -a) {
        stack.pop(); // top asteroid destroyed, keep checking
      } else if (top === -a) {
        stack.pop(); // both destroyed
        alive = false;
      } else {
        alive = false; // current asteroid destroyed
      }
    }
    if (alive) stack.push(a);
  }
  return stack;
}
```

**Complexity:** Time O(n) — each asteroid pushed/popped at most once, Space O(n)

***Alternate approaches:*** *Simulate with a plain array and repeated linear scans for collisions until stable — O(n²) worst case, much slower.*

☐ Practiced   ☐ Revised

## **7.8  Decode String**

**Problem:** Given an encoded string like '3[a2[c]]', decode it to 'accaccacc'.

**Approach:** Use a stack to hold (previous string, repeat count) pairs. On a digit, build the number. On '[', push the current string and count, reset both. On ']', pop and repeat the current segment, prepending the previous string. Append plain characters directly.

**Optimal Solution (JavaScript):**

```js
function decodeString(s) {
  const stack = [];
  let currentStr = '', currentNum = 0;
 
  for (const c of s) {
    if (!isNaN(c)) {
      currentNum = currentNum * 10 + Number(c);
    } else if (c === '[') {
      stack.push([currentStr, currentNum]);
      currentStr = ''; currentNum = 0;
    } else if (c === ']') {
      const [prevStr, num] = stack.pop();
      currentStr = prevStr + currentStr.repeat(num);
    } else {
      currentStr += c;
    }
  }
  return currentStr;
}
```

**Complexity:** Time O(n · maxRepeat), Space O(n)

***Alternate approaches:*** *Recursive descent parser that processes the string with an index pointer passed by reference, recursing on '[' and returning on ']' — same complexity, arguably more readable for deeply nested cases.*

☐ Practiced   ☐ Revised

## **7.9  Remove K Digits**

**Problem:** Given a non-negative integer as a string and an integer k, remove k digits to make the smallest possible number.

**Approach:** Use a monotonic increasing stack of digits. For each new digit, while the stack's top digit is larger and removals remain (k > 0), pop it. Push the current digit. After processing, trim any remaining removals from the end and strip leading zeroes.

**Optimal Solution (JavaScript):**

```js
function removeKdigits(num, k) {
  const stack = [];
  for (const digit of num) {
    while (k > 0 && stack.length && stack[stack.length - 1] > digit) {
      stack.pop();
      k--;
    }
    stack.push(digit);
  }
  while (k-- > 0) stack.pop();
 
  const result = stack.join('').replace(/^0+/, '');
  return result === '' ? '0' : result;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Brute force tries every combination of k digits to remove and picks the smallest result — exponential time, only viable for tiny inputs.*

☐ Practiced   ☐ Revised

## **7.10  Basic Calculator II**

**Problem:** Given a string expression with +, -, *, / and non-negative integers (no parentheses), evaluate it respecting operator precedence.

**Approach:** Scan digit by digit, tracking the current number and the last seen operator. For + or -, push the number (with sign) onto a stack. For * or /, pop the last stack value, apply the operation with the current number, and push the result back. Sum the stack at the end.

**Optimal Solution (JavaScript):**

```js
function calculate(s) {
  const stack = [];
  let num = 0, sign = '+';
 
  for (let i = 0; i < s.length; i++) {
    const c = s[i];
    if (/\d/.test(c)) num = num * 10 + Number(c);
 
    if ((!/\s/.test(c) && !/\d/.test(c)) || i === s.length - 1) {
      if (sign === '+') stack.push(num);
      else if (sign === '-') stack.push(-num);
      else if (sign === '*') stack.push(stack.pop() * num);
      else if (sign === '/') stack.push(Math.trunc(stack.pop() / num));
      sign = c;
      num = 0;
    }
  }
  return stack.reduce((a, b) => a + b, 0);
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Two-pass approach: first resolve all * and / left to right into a reduced list, then sum the remaining + and - terms — same complexity, splits logic into two clearer passes.*

☐ Practiced   ☐ Revised

# **8. Queue / BFS**

**Goal:** Efficient traversal

**Core idea:** *FIFO structure for level-by-level exploration; guarantees shortest path in unweighted graphs.*

## **8.1  Binary Tree Level Order Traversal**

**Problem:** Given a binary tree, return the values of nodes level by level (left to right).

**Approach:** Use a queue starting with the root. At each iteration, process exactly the nodes currently in the queue (the current level), collecting their values and enqueuing their children for the next level.

**Optimal Solution (JavaScript):**

```js
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];
 
  while (queue.length) {
    const levelSize = queue.length;
    const level = [];
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(level);
  }
  return result;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Recursive DFS that tracks depth and appends each value into result[depth] — same O(n) time, avoids an explicit queue but relies on the call stack instead.*

☐ Practiced   ☐ Revised

## **8.2  Shortest Path in Binary Matrix**

**Problem:** Given an n x n binary grid, find the length of the shortest clear path (0s only, 8-directional movement) from top-left to bottom-right.

**Approach:** BFS from the starting cell, exploring all 8 neighbors at each step, tracking the path length as the BFS depth. The first time the destination is reached, that depth is the shortest path.

**Optimal Solution (JavaScript):**

```js
function shortestPathBinaryMatrix(grid) {
  const n = grid.length;
  if (grid[0][0] !== 0 || grid[n - 1][n - 1] !== 0) return -1;
 
  const dirs = [[-1,-1],[-1,0],[-1,1],[0,-1],[0,1],[1,-1],[1,0],[1,1]];
  const queue = [[0, 0, 1]]; // row, col, path length
  grid[0][0] = 1; // mark visited
 
  while (queue.length) {
    const [r, c, dist] = queue.shift();
    if (r === n - 1 && c === n - 1) return dist;
 
    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] === 0) {
        grid[nr][nc] = 1;
        queue.push([nr, nc, dist + 1]);
      }
    }
  }
  return -1;
}
```

**Complexity:** Time O(n²), Space O(n²)

***Alternate approaches:*** *A* search with a Manhattan/Chebyshev distance heuristic can prune the search space faster in practice, though worst-case complexity remains similar for dense grids.*

☐ Practiced   ☐ Revised

## **8.3  Rotting Oranges**

**Problem:** Given a grid of fresh (1), rotten (2), and empty (0) oranges, find the minimum minutes until no fresh orange remains (or -1 if impossible).

**Approach:** Multi-source BFS: start with all rotten oranges in the queue simultaneously. Each BFS layer represents one minute; rot adjacent fresh oranges and enqueue them. Track elapsed minutes and remaining fresh count.

**Optimal Solution (JavaScript):**

```js
function orangesRotting(grid) {
  const rows = grid.length, cols = grid[0].length;
  const queue = [];
  let fresh = 0;
 
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === 2) queue.push([r, c]);
      if (grid[r][c] === 1) fresh++;
    }
 
  const dirs = [[1,0],[-1,0],[0,1],[0,-1]];
  let minutes = 0;
 
  while (queue.length && fresh > 0) {
    const size = queue.length;
    for (let i = 0; i < size; i++) {
      const [r, c] = queue.shift();
      for (const [dr, dc] of dirs) {
        const nr = r + dr, nc = c + dc;
        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] === 1) {
          grid[nr][nc] = 2;
          fresh--;
          queue.push([nr, nc]);
        }
      }
    }
    minutes++;
  }
  return fresh === 0 ? minutes : -1;
}
```

**Complexity:** Time O(rows · cols), Space O(rows · cols)

***Alternate approaches:*** *Repeatedly scan the whole grid, rotting adjacent oranges each pass, until no changes occur — O((rows·cols)²) worst case, far slower than multi-source BFS.*

☐ Practiced   ☐ Revised

## **8.4  Word Ladder**

**Problem:** Given a start word, end word, and a word list, find the length of the shortest transformation sequence changing one letter at a time, with each intermediate word in the list.

**Approach:** BFS over words, where an edge connects two words differing by exactly one letter. Generate neighbors by trying every letter substitution at every position and checking membership in a word set (removing found words to prevent revisiting).

**Optimal Solution (JavaScript):**

```js
function ladderLength(beginWord, endWord, wordList) {
  const wordSet = new Set(wordList);
  if (!wordSet.has(endWord)) return 0;
 
  const queue = [[beginWord, 1]];
  const alphabet = 'abcdefghijklmnopqrstuvwxyz';
 
  while (queue.length) {
    const [word, steps] = queue.shift();
    if (word === endWord) return steps;
 
    for (let i = 0; i < word.length; i++) {
      for (const letter of alphabet) {
        const next = word.slice(0, i) + letter + word.slice(i + 1);
        if (wordSet.has(next)) {
          wordSet.delete(next);
          queue.push([next, steps + 1]);
        }
      }
    }
  }
  return 0;
}
```

**Complexity:** Time O(n · L² · 26) where n = word count, L = word length, Space O(n · L)

***Alternate approaches:*** *Bidirectional BFS (searching from both begin and end words simultaneously) can dramatically reduce the search space in practice, especially for large word lists.*

☐ Practiced   ☐ Revised

## **8.5  Number of Islands (BFS version)**

**Problem:** Given a grid of '1' (land) and '0' (water), count the number of islands (connected groups of land, 4-directionally).

**Approach:** Scan every cell; whenever an unvisited land cell is found, start a BFS to mark the entire connected island as visited, and increment the island count.

**Optimal Solution (JavaScript):**

```js
function numIslands(grid) {
  const rows = grid.length, cols = grid[0].length;
  let islands = 0;
 
  const bfs = (sr, sc) => {
    const queue = [[sr, sc]];
    grid[sr][sc] = '0';
    const dirs = [[1,0],[-1,0],[0,1],[0,-1]];
 
    while (queue.length) {
      const [r, c] = queue.shift();
      for (const [dr, dc] of dirs) {
        const nr = r + dr, nc = c + dc;
        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] === '1') {
          grid[nr][nc] = '0';
          queue.push([nr, nc]);
        }
      }
    }
  };
 
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (grid[r][c] === '1') { islands++; bfs(r, c); }
 
  return islands;
}
```

**Complexity:** Time O(rows · cols), Space O(rows · cols)

***Alternate approaches:*** *DFS (recursive or with an explicit stack) does the same flood-fill in the same complexity — a matter of traversal style, not efficiency. Union-Find also solves it in near O(rows·cols) with path compression.*

☐ Practiced   ☐ Revised

## **8.6  Course Schedule (BFS / Kahn's Algorithm)**

**Problem:** Given numCourses and prerequisite pairs, determine if it's possible to finish all courses (i.e., the prerequisite graph has no cycle).

**Approach:** Build an adjacency list and in-degree count for each course. Start BFS from all courses with in-degree 0. As each course is 'completed', decrement its neighbors' in-degrees; if a neighbor reaches 0, enqueue it. If all courses get processed, there's no cycle.

**Optimal Solution (JavaScript):**

```js
function canFinish(numCourses, prerequisites) {
  const graph = Array.from({ length: numCourses }, () => []);
  const inDegree = new Array(numCourses).fill(0);
 
  for (const [course, pre] of prerequisites) {
    graph[pre].push(course);
    inDegree[course]++;
  }
 
  const queue = [];
  for (let i = 0; i < numCourses; i++) if (inDegree[i] === 0) queue.push(i);
 
  let processed = 0;
  while (queue.length) {
    const course = queue.shift();
    processed++;
    for (const next of graph[course]) {
      inDegree[next]--;
      if (inDegree[next] === 0) queue.push(next);
    }
  }
  return processed === numCourses;
}
```

**Complexity:** Time O(V + E), Space O(V + E)

***Alternate approaches:*** *DFS-based cycle detection with a 'visiting/visited' color-marking scheme achieves the same O(V + E) complexity and additionally can reconstruct the exact cycle if needed.*

☐ Practiced   ☐ Revised

## **8.7  Snakes and Ladders**

**Problem:** Given a boustrophedon-numbered board with snakes and ladders, find the minimum number of dice rolls to reach the last square.

**Approach:** BFS over board squares 1..n², treating each square as a graph node with edges to squares reachable by rolling 1-6 (redirected by any snake/ladder at the destination). Track visited squares and BFS depth as the roll count.

**Optimal Solution (JavaScript):**

```js
function snakesAndLadders(board) {
  const n = board.length;
  const getValue = (square) => {
    const row = Math.floor((square - 1) / n);
    let col = (square - 1) % n;
    if (row % 2 === 1) col = n - 1 - col; // boustrophedon
    return board[n - 1 - row][col];
  };
 
  const visited = new Set([1]);
  const queue = [[1, 0]];
 
  while (queue.length) {
    const [square, moves] = queue.shift();
    if (square === n * n) return moves;
 
    for (let dice = 1; dice <= 6; dice++) {
      let next = square + dice;
      if (next > n * n) continue;
      const val = getValue(next);
      if (val !== -1) next = val;
      if (!visited.has(next)) {
        visited.add(next);
        queue.push([next, moves + 1]);
      }
    }
  }
  return -1;
}
```

**Complexity:** Time O(n²), Space O(n²)

***Alternate approaches:*** *Dijkstra's algorithm would also work (treating every move as equal weight 1) but is unnecessary overhead — BFS is optimal since all edges have the same weight.*

☐ Practiced   ☐ Revised

## **8.8  Walls and Gates**

**Problem:** Given a grid of rooms (INF), walls (-1), and gates (0), fill each empty room with the distance to its nearest gate.

**Approach:** Multi-source BFS starting from all gates simultaneously. Each BFS layer increases the distance by 1; update rooms with the first (shortest) distance they're reached at, since BFS guarantees shortest path first.

**Optimal Solution (JavaScript):**

```js
function wallsAndGates(rooms) {
  const rows = rooms.length, cols = rooms[0].length;
  const queue = [];
 
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (rooms[r][c] === 0) queue.push([r, c]);
 
  const dirs = [[1,0],[-1,0],[0,1],[0,-1]];
 
  while (queue.length) {
    const [r, c] = queue.shift();
    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && rooms[nr][nc] === 2147483647) {
        rooms[nr][nc] = rooms[r][c] + 1;
        queue.push([nr, nc]);
      }
    }
  }
}
```

**Complexity:** Time O(rows · cols), Space O(rows · cols)

***Alternate approaches:*** *Run BFS/DFS independently from each empty room to find its nearest gate — O((rows·cols)²) worst case, much slower than multi-source BFS from the gates.*

☐ Practiced   ☐ Revised

## **8.9  01 Matrix**

**Problem:** Given a binary matrix, find the distance of the nearest 0 for each cell.

**Approach:** Multi-source BFS starting from all 0-cells at once. Propagate outward layer by layer, marking 1-cells with the current distance as they're first reached.

**Optimal Solution (JavaScript):**

```js
function updateMatrix(mat) {
  const rows = mat.length, cols = mat[0].length;
  const dist = Array.from({ length: rows }, () => new Array(cols).fill(-1));
  const queue = [];
 
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (mat[r][c] === 0) { dist[r][c] = 0; queue.push([r, c]); }
 
  const dirs = [[1,0],[-1,0],[0,1],[0,-1]];
  while (queue.length) {
    const [r, c] = queue.shift();
    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && dist[nr][nc] === -1) {
        dist[nr][nc] = dist[r][c] + 1;
        queue.push([nr, nc]);
      }
    }
  }
  return dist;
}
```

**Complexity:** Time O(rows · cols), Space O(rows · cols)

***Alternate approaches:*** *Two-pass dynamic programming (top-left to bottom-right, then bottom-right to top-left, taking min distances from processed neighbors) also solves it in O(rows · cols) time without a queue.*

☐ Practiced   ☐ Revised

## **8.10  Open the Lock**

**Problem:** Given a 4-digit combination lock (each wheel 0-9, wrapping), a list of deadend combinations, and a target, find the minimum number of moves to reach the target (or -1 if impossible).

**Approach:** BFS over all reachable lock states, where each move changes one wheel by ±1. Skip deadend states entirely and track visited states to avoid revisiting; BFS depth gives the minimum move count.

**Optimal Solution (JavaScript):**

```js
function openLock(deadends, target) {
  const dead = new Set(deadends);
  if (dead.has('0000')) return -1;
 
  const queue = [['0000', 0]];
  const visited = new Set(['0000']);
 
  const neighbors = (state) => {
    const results = [];
    for (let i = 0; i < 4; i++) {
      const digit = Number(state[i]);
      for (const delta of [-1, 1]) {
        const newDigit = (digit + delta + 10) % 10;
        results.push(state.slice(0, i) + newDigit + state.slice(i + 1));
      }
    }
    return results;
  };
 
  while (queue.length) {
    const [state, moves] = queue.shift();
    if (state === target) return moves;
 
    for (const next of neighbors(state)) {
      if (!dead.has(next) && !visited.has(next)) {
        visited.add(next);
        queue.push([next, moves + 1]);
      }
    }
  }
  return -1;
}
```

**Complexity:** Time O(10^4) states explored worst case, Space O(10^4)

***Alternate approaches:*** *Bidirectional BFS from both '0000' and the target simultaneously can significantly cut down the explored state space in practice.*

☐ Practiced   ☐ Revised

# **9. Recursion + DFS**

**Goal:** Tree/Graph traversal

**Core idea:** *Explore as deep as possible before backtracking; natural fit for trees, graphs, and combinatorial search.*

## **9.1  Maximum Depth of Binary Tree (Tree Height)**

**Problem:** Given a binary tree, find its maximum depth (number of nodes along the longest path from root to a leaf).

**Approach:** Recursively compute the depth of the left and right subtrees, then return 1 plus the larger of the two. Base case: an empty tree has depth 0.

**Optimal Solution (JavaScript):**

```js
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

**Complexity:** Time O(n), Space O(h) — h is tree height, for the recursion stack

***Alternate approaches:*** *Iterative BFS level-by-level, counting levels until the queue empties — O(n) time, O(n) space for the widest level's queue contents.*

☐ Practiced   ☐ Revised

## **9.2  Path Sum**

**Problem:** Given a binary tree and a target sum, determine if the tree has a root-to-leaf path whose values add up to the target.

**Approach:** Recurse down the tree, subtracting each node's value from the remaining target. At a leaf, check if the remaining target equals that leaf's value. Return true if either subtree finds a valid path.

**Optimal Solution (JavaScript):**

```js
function hasPathSum(root, targetSum) {
  if (!root) return false;
  if (!root.left && !root.right) return targetSum === root.val;
 
  const remaining = targetSum - root.val;
  return hasPathSum(root.left, remaining) || hasPathSum(root.right, remaining);
}
```

**Complexity:** Time O(n), Space O(h)

***Alternate approaches:*** *Iterative DFS with an explicit stack storing (node, remainingSum) pairs — same complexity, avoids recursion depth limits for very unbalanced trees.*

☐ Practiced   ☐ Revised

## **9.3  Path Sum II**

**Problem:** Given a binary tree and a target sum, return all root-to-leaf paths where the values sum to the target.

**Approach:** DFS while maintaining the current path in an array. At each node, add it to the path and subtract its value from the remaining target. At a leaf with remaining target 0, record a copy of the path. Backtrack by removing the node before returning.

**Optimal Solution (JavaScript):**

```js
function pathSum(root, targetSum) {
  const result = [];
 
  const dfs = (node, remaining, path) => {
    if (!node) return;
    path.push(node.val);
    remaining -= node.val;
 
    if (!node.left && !node.right && remaining === 0) {
      result.push([...path]);
    } else {
      dfs(node.left, remaining, path);
      dfs(node.right, remaining, path);
    }
    path.pop(); // backtrack
  };
 
  dfs(root, targetSum, []);
  return result;
}
```

**Complexity:** Time O(n²) worst case (copying paths), Space O(n)

***Alternate approaches:*** *Pass path arrays by value (creating a new array at each recursive call instead of mutating and backtracking) — functionally identical result but higher constant-factor memory overhead.*

☐ Practiced   ☐ Revised

## **9.4  Validate Binary Search Tree**

**Problem:** Given a binary tree, determine if it is a valid binary search tree (left subtree values < node < right subtree values, recursively).

**Approach:** Recurse while carrying down a valid (min, max) range for each node. A node must fall strictly within its allowed range; recurse into children with updated bounds (left child's max becomes the node's value, right child's min becomes the node's value).

**Optimal Solution (JavaScript):**

```js
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
 
  return isValidBST(root.left, min, root.val) && isValidBST(root.right, root.val, max);
}
```

**Complexity:** Time O(n), Space O(h)

***Alternate approaches:*** *In-order traversal should produce strictly increasing values for a valid BST — traverse in-order and compare each value to the previous one; O(n) time, O(h) space, sometimes considered more intuitive.*

☐ Practiced   ☐ Revised

## **9.5  Number of Islands (DFS version)**

**Problem:** Given a grid of '1' (land) and '0' (water), count the number of islands (connected land groups, 4-directionally).

**Approach:** Scan every cell; when an unvisited land cell is found, recursively DFS into all connected land cells, marking them visited (e.g., flipping to '0'), and increment the island count once per new island found.

**Optimal Solution (JavaScript):**

```js
function numIslands(grid) {
  const rows = grid.length, cols = grid[0].length;
 
  const dfs = (r, c) => {
    if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] !== '1') return;
    grid[r][c] = '0';
    dfs(r + 1, c); dfs(r - 1, c); dfs(r, c + 1); dfs(r, c - 1);
  };
 
  let islands = 0;
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (grid[r][c] === '1') { islands++; dfs(r, c); }
 
  return islands;
}
```

**Complexity:** Time O(rows · cols), Space O(rows · cols) worst case for recursion stack

***Alternate approaches:*** *BFS with an explicit queue achieves the same complexity while avoiding recursion depth issues on very large, densely connected grids.*

☐ Practiced   ☐ Revised

## **9.6  Clone Graph**

**Problem:** Given a reference to a node in a connected undirected graph, return a deep copy (clone) of the graph.

**Approach:** DFS from the given node, using a hash map from original node → cloned node to avoid infinite loops and duplicate clones. For each unvisited node, create its clone first, store it in the map, then recursively clone its neighbors.

**Optimal Solution (JavaScript):**

```js
function cloneGraph(node) {
  if (!node) return null;
  const cloned = new Map();
 
  const dfs = (n) => {
    if (cloned.has(n)) return cloned.get(n);
    const copy = { val: n.val, neighbors: [] };
    cloned.set(n, copy);
    for (const neighbor of n.neighbors) {
      copy.neighbors.push(dfs(neighbor));
    }
    return copy;
  };
 
  return dfs(node);
}
```

**Complexity:** Time O(V + E), Space O(V)

***Alternate approaches:*** *BFS with the same visited-map trick, using a queue instead of recursion — same complexity, avoids recursion depth issues on very large graphs.*

☐ Practiced   ☐ Revised

## **9.7  Course Schedule (DFS cycle detection)**

**Problem:** Given numCourses and prerequisite pairs, determine if it's possible to finish all courses (no cycle in the prerequisite graph).

**Approach:** DFS with three states per node: unvisited, visiting (currently in the recursion stack), and visited (fully processed). If DFS reaches a node marked 'visiting', a cycle exists. Mark nodes 'visited' once all their dependencies are cleared.

**Optimal Solution (JavaScript):**

```js
function canFinish(numCourses, prerequisites) {
  const graph = Array.from({ length: numCourses }, () => []);
  for (const [course, pre] of prerequisites) graph[course].push(pre);
 
  const state = new Array(numCourses).fill(0); // 0=unvisited, 1=visiting, 2=visited
 
  const dfs = (course) => {
    if (state[course] === 1) return false; // cycle detected
    if (state[course] === 2) return true;  // already confirmed safe
 
    state[course] = 1;
    for (const pre of graph[course]) {
      if (!dfs(pre)) return false;
    }
    state[course] = 2;
    return true;
  };
 
  for (let i = 0; i < numCourses; i++) {
    if (!dfs(i)) return false;
  }
  return true;
}
```

**Complexity:** Time O(V + E), Space O(V + E)

***Alternate approaches:*** *Kahn's algorithm (BFS with in-degree tracking) detects cycles by checking if all nodes get processed — same O(V + E) complexity, often preferred when you also want a valid topological order.*

☐ Practiced   ☐ Revised

## **9.8  Subsets / Subsets II**

**Problem:** Given an array of (possibly duplicate, for II) integers, return all possible subsets (the power set), with II requiring no duplicate subsets.

**Approach:** Backtracking: at each index, decide to include or exclude the current element, recursing forward. Record the current subset at every recursive call (not just at the end). For duplicates (II), sort first and skip over repeated values at the same recursion depth to avoid duplicate subsets.

**Optimal Solution (JavaScript):**

```js
function subsetsWithDup(nums) {
  nums.sort((a, b) => a - b);
  const result = [];
  const path = [];
 
  const backtrack = (start) => {
    result.push([...path]);
    for (let i = start; i < nums.length; i++) {
      if (i > start && nums[i] === nums[i - 1]) continue; // skip duplicates at this level
      path.push(nums[i]);
      backtrack(i + 1);
      path.pop();
    }
  };
 
  backtrack(0);
  return result;
}
```

**Complexity:** Time O(2^n), Space O(n) recursion depth (excluding output)

***Alternate approaches:*** *Iterative approach: start with [[]] and for each number, add it to every existing subset already built so far — O(2^n) time, avoids recursion but needs care to dedupe for the 'with duplicates' variant.*

☐ Practiced   ☐ Revised

## **9.9  Permutations**

**Problem:** Given an array of distinct integers, return all possible permutations.

**Approach:** Backtracking: maintain a 'used' marker per index (or swap-based in-place approach). At each recursive step, try placing every unused number next, recurse, then undo the choice (backtrack) before trying the next option.

**Optimal Solution (JavaScript):**

```js
function permute(nums) {
  const result = [];
  const path = [];
  const used = new Array(nums.length).fill(false);
 
  const backtrack = () => {
    if (path.length === nums.length) {
      result.push([...path]);
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (used[i]) continue;
      used[i] = true;
      path.push(nums[i]);
      backtrack();
      path.pop();
      used[i] = false;
    }
  };
 
  backtrack();
  return result;
}
```

**Complexity:** Time O(n · n!), Space O(n) recursion depth (excluding output)

***Alternate approaches:*** *In-place swap-based backtracking (swap current index with each candidate, recurse, swap back) avoids the 'used' array — same complexity, slightly less memory overhead.*

☐ Practiced   ☐ Revised

## **9.10  Word Search**

**Problem:** Given a grid of letters and a word, determine if the word can be constructed from adjacent cells (up/down/left/right, no cell reused).

**Approach:** DFS/backtracking from every cell matching the word's first letter. At each step, mark the current cell visited, recurse into neighbors looking for the next letter, and unmark (backtrack) before trying other paths.

**Optimal Solution (JavaScript):**

```js
function exist(board, word) {
  const rows = board.length, cols = board[0].length;
 
  const dfs = (r, c, idx) => {
    if (idx === word.length) return true;
    if (r < 0 || r >= rows || c < 0 || c >= cols || board[r][c] !== word[idx]) return false;
 
    const temp = board[r][c];
    board[r][c] = '#'; // mark visited
 
    const found = dfs(r + 1, c, idx + 1) || dfs(r - 1, c, idx + 1) ||
                  dfs(r, c + 1, idx + 1) || dfs(r, c - 1, idx + 1);
 
    board[r][c] = temp; // backtrack
    return found;
  };
 
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (dfs(r, c, 0)) return true;
 
  return false;
}
```

**Complexity:** Time O(rows · cols · 4^L) where L = word length, Space O(L) recursion depth

***Alternate approaches:*** *Trie-based search when checking multiple words against the same board (Word Search II) — builds a trie of all target words and DFS explores all of them simultaneously, sharing prefixes for efficiency.*

☐ Practiced   ☐ Revised

# **10. Dynamic Programming**

**Goal:** Exponential → Polynomial

**Core idea:** *Break a problem into overlapping subproblems, cache results (memoization / tabulation) to avoid recomputation.*

## **10.1  Fibonacci Number / Climbing Stairs**

**Problem:** Given n, find the nth Fibonacci number (or, equivalently, the number of ways to climb n stairs taking 1 or 2 steps at a time).

**Approach:** Bottom-up DP: since each state only depends on the previous two, keep two rolling variables instead of a full array, updating them as you count up to n.

**Optimal Solution (JavaScript):**

```js
function climbStairs(n) {
  if (n <= 2) return n;
  let prev2 = 1, prev1 = 2;
  for (let i = 3; i <= n; i++) {
    [prev2, prev1] = [prev1, prev2 + prev1];
  }
  return prev1;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Naive recursion without memoization is O(2^n) time — exponential blowup from recomputing the same subproblems. Top-down memoized recursion fixes that to O(n) time, O(n) space for the cache + call stack.*

☐ Practiced   ☐ Revised

## **10.2  House Robber**

**Problem:** Given amounts of money in houses arranged in a line, find the max sum you can rob without robbing two adjacent houses.

**Approach:** DP where dp[i] = max loot considering houses 0..i, either skipping house i (dp[i-1]) or robbing it (dp[i-2] + nums[i]). Reduce to two rolling variables since only the last two states matter.

**Optimal Solution (JavaScript):**

```js
function rob(nums) {
  let prev2 = 0, prev1 = 0;
  for (const n of nums) {
    const curr = Math.max(prev1, prev2 + n);
    prev2 = prev1;
    prev1 = curr;
  }
  return prev1;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Full DP array dp[i] storing every state explicitly — same O(n) time but O(n) space; easier to trace by hand while learning the pattern.*

☐ Practiced   ☐ Revised

## **10.3  House Robber II**

**Problem:** Same as House Robber, but houses are arranged in a circle (first and last are adjacent).

**Approach:** Since the first and last house can't both be robbed, run the standard House Robber DP twice: once excluding the last house, once excluding the first house, and take the maximum of the two results.

**Optimal Solution (JavaScript):**

```js
function rob(nums) {
  if (nums.length === 1) return nums[0];
 
  const robLinear = (houses) => {
    let prev2 = 0, prev1 = 0;
    for (const n of houses) {
      const curr = Math.max(prev1, prev2 + n);
      prev2 = prev1;
      prev1 = curr;
    }
    return prev1;
  };
 
  return Math.max(robLinear(nums.slice(0, -1)), robLinear(nums.slice(1)));
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *A single pass tracking four rolling states (with/without first house, at each position) can avoid slicing the array, saving the O(n) copy — same asymptotic complexity, slightly better constants.*

☐ Practiced   ☐ Revised

## **10.4  0/1 Knapsack**

**Problem:** Given item weights, values, and a capacity, find the max value achievable without exceeding capacity, each item usable at most once.

**Approach:** DP table dp[i][w] = max value using the first i items with capacity w. For each item, either skip it (dp[i-1][w]) or take it if it fits (value[i] + dp[i-1][w-weight[i]]). Can be compressed to a 1D array by iterating capacity in reverse.

**Optimal Solution (JavaScript):**

```js
function knapsack(weights, values, capacity) {
  const dp = new Array(capacity + 1).fill(0);
 
  for (let i = 0; i < weights.length; i++) {
    for (let w = capacity; w >= weights[i]; w--) {
      dp[w] = Math.max(dp[w], values[i] + dp[w - weights[i]]);
    }
  }
  return dp[capacity];
}
```

**Complexity:** Time O(n · capacity), Space O(capacity)

***Alternate approaches:*** *Full 2D dp[i][w] table — same O(n · capacity) time, O(n · capacity) space; easier to trace but uses more memory. Brute force tries every subset — O(2^n) time.*

☐ Practiced   ☐ Revised

## **10.5  Coin Change**

**Problem:** Given coin denominations and a target amount, find the fewest coins needed to make that amount (or -1 if impossible).

**Approach:** Bottom-up DP where dp[a] = minimum coins to make amount a. Initialize dp[0] = 0 and all others to infinity; for each amount, try every coin and take the best (1 + dp[amount - coin]).

**Optimal Solution (JavaScript):**

```js
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;
 
  for (let a = 1; a <= amount; a++) {
    for (const coin of coins) {
      if (coin <= a) dp[a] = Math.min(dp[a], dp[a - coin] + 1);
    }
  }
  return dp[amount] === Infinity ? -1 : dp[amount];
}
```

**Complexity:** Time O(amount · numCoins), Space O(amount)

***Alternate approaches:*** *BFS over amounts (each coin as an edge) also finds the minimum count since all edges have equal weight — same complexity, different framing. Greedy (always take the largest coin) fails for arbitrary denominations.*

☐ Practiced   ☐ Revised

## **10.6  Longest Common Subsequence**

**Problem:** Given two strings, find the length of their longest common subsequence (not necessarily contiguous).

**Approach:** 2D DP where dp[i][j] = LCS length of the first i characters of s1 and first j characters of s2. If characters match, dp[i][j] = dp[i-1][j-1] + 1; otherwise take the max of dropping a character from either string.

**Optimal Solution (JavaScript):**

```js
function longestCommonSubsequence(text1, text2) {
  const m = text1.length, n = text2.length;
  const dp = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));
 
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (text1[i - 1] === text2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }
  return dp[m][n];
}
```

**Complexity:** Time O(m · n), Space O(m · n) (reducible to O(min(m, n)) with row compression)

***Alternate approaches:*** *Recursive brute force trying every subsequence combination is O(2^(m+n)) time — memoizing the recursive calls brings it back down to O(m · n), equivalent to the DP table.*

☐ Practiced   ☐ Revised

## **10.7  Longest Increasing Subsequence**

**Problem:** Given an array, find the length of the longest strictly increasing subsequence.

**Approach:** Maintain a 'tails' array where tails[k] = smallest possible tail value of an increasing subsequence of length k+1. For each number, binary search for its position in tails and replace or extend accordingly. The final length of tails is the answer.

**Optimal Solution (JavaScript):**

```js
function lengthOfLIS(nums) {
  const tails = [];
 
  for (const n of nums) {
    let low = 0, high = tails.length;
    while (low < high) {
      const mid = low + Math.floor((high - low) / 2);
      if (tails[mid] < n) low = mid + 1;
      else high = mid;
    }
    tails[low] = n; // replace or extend
  }
  return tails.length;
}
```

**Complexity:** Time O(n log n), Space O(n)

***Alternate approaches:*** *Classic O(n²) DP: dp[i] = length of LIS ending at i, computed by checking all previous smaller elements — simpler to derive, but much slower for large inputs.*

☐ Practiced   ☐ Revised

## **10.8  Edit Distance**

**Problem:** Given two strings, find the minimum number of insert/delete/replace operations to convert one into the other.

**Approach:** 2D DP where dp[i][j] = edit distance between the first i characters of word1 and first j of word2. If characters match, dp[i][j] = dp[i-1][j-1]; otherwise take 1 + min of insert, delete, replace options.

**Optimal Solution (JavaScript):**

```js
function minDistance(word1, word2) {
  const m = word1.length, n = word2.length;
  const dp = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));
 
  for (let i = 0; i <= m; i++) dp[i][0] = i;
  for (let j = 0; j <= n; j++) dp[0][j] = j;
 
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        dp[i][j] = 1 + Math.min(dp[i - 1][j - 1], dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }
  return dp[m][n];
}
```

**Complexity:** Time O(m · n), Space O(m · n) (reducible to O(min(m, n)) with row compression)

***Alternate approaches:*** *Recursive brute force exploring every combination of operations is exponential time — memoization reduces it to the same O(m · n) as the tabulated DP.*

☐ Practiced   ☐ Revised

## **10.9  Unique Paths**

**Problem:** Given an m x n grid, find how many unique paths exist from top-left to bottom-right, moving only right or down.

**Approach:** DP where dp[r][c] = number of ways to reach cell (r, c), equal to dp[r-1][c] + dp[r][c-1] (paths from above plus paths from the left). Base case: first row and first column all have exactly 1 path.

**Optimal Solution (JavaScript):**

```js
function uniquePaths(m, n) {
  const dp = new Array(n).fill(1);
 
  for (let r = 1; r < m; r++) {
    for (let c = 1; c < n; c++) {
      dp[c] += dp[c - 1];
    }
  }
  return dp[n - 1];
}
```

**Complexity:** Time O(m · n), Space O(n)

***Alternate approaches:*** *Direct combinatorics: the answer is C(m+n-2, m-1) (choosing which steps are 'down' among all moves) — O(m + n) time to compute the binomial coefficient, faster than DP but relies on a combinatorial insight.*

☐ Practiced   ☐ Revised

## **10.10  Partition Equal Subset Sum**

**Problem:** Given an array of positive integers, determine if it can be partitioned into two subsets with equal sum.

**Approach:** If total sum is odd, it's impossible. Otherwise this reduces to a 0/1 knapsack subset-sum problem: can we pick a subset summing to exactly half the total? Use a boolean DP array over achievable sums, updating in reverse to respect the 'each item once' constraint.

**Optimal Solution (JavaScript):**

```js
function canPartition(nums) {
  const total = nums.reduce((a, b) => a + b, 0);
  if (total % 2 !== 0) return false;
  const target = total / 2;
 
  const dp = new Array(target + 1).fill(false);
  dp[0] = true;
 
  for (const n of nums) {
    for (let s = target; s >= n; s--) {
      dp[s] = dp[s] || dp[s - n];
    }
  }
  return dp[target];
}
```

**Complexity:** Time O(n · target), Space O(target)

***Alternate approaches:*** *Brute force tries every subset to check if any sums to target — O(2^n) time, impractical beyond small arrays.*

☐ Practiced   ☐ Revised

## **10.11  Word Break**

**Problem:** Given a string and a dictionary of words, determine if the string can be segmented into a sequence of one or more dictionary words.

**Approach:** DP where dp[i] = true if s[0..i) can be segmented using dictionary words. For each end position i, check all start positions j < i where dp[j] is true and s[j..i) is a valid word.

**Optimal Solution (JavaScript):**

```js
function wordBreak(s, wordDict) {
  const wordSet = new Set(wordDict);
  const dp = new Array(s.length + 1).fill(false);
  dp[0] = true;
 
  for (let i = 1; i <= s.length; i++) {
    for (let j = 0; j < i; j++) {
      if (dp[j] && wordSet.has(s.slice(j, i))) {
        dp[i] = true;
        break;
      }
    }
  }
  return dp[s.length];
}
```

**Complexity:** Time O(n² ) (plus substring hashing cost), Space O(n)

***Alternate approaches:*** *BFS/DFS over 'reachable end positions' with memoization to avoid revisiting the same index — same complexity, framed as a graph search instead of tabulated DP.*

☐ Practiced   ☐ Revised

## **10.12  Maximum Product Subarray**

**Problem:** Given an integer array, find the contiguous subarray with the largest product.

**Approach:** Track both the max and min product ending at the current index (since a negative number can flip a very small min into the new max). At each step, compute candidates using the current number times the previous max, previous min, or alone, and update both running max and min.

**Optimal Solution (JavaScript):**

```js
function maxProduct(nums) {
  let maxProd = nums[0], minProd = nums[0], result = nums[0];
 
  for (let i = 1; i < nums.length; i++) {
    const n = nums[i];
    const candidates = [n, maxProd * n, minProd * n];
    maxProd = Math.max(...candidates);
    minProd = Math.min(...candidates);
    result = Math.max(result, maxProd);
  }
  return result;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Brute force checks the product of every subarray — O(n²) time, O(1) space. Splitting the array at zeroes and handling each segment's sign-flips manually also works but is more error-prone than tracking running min/max.*

☐ Practiced   ☐ Revised
