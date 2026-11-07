# **DSA Complete Guide**

*Interview revision guide: 22 chapters in 5 parts and 165 problems, ordered so each topic builds on the one before. Every problem has a statement, approach, optimal JavaScript solution, complexity and alternate approaches. Number of Islands and Course Schedule appear twice on purpose, as a BFS and a DFS version. All solutions were run against test cases when this guide was compiled.*

## **Table of Contents**

- [Revision Sheet](#revision-sheet) — one-line key insight for every problem
- [Part 0 — Start Here](#part-0--start-here)
- [Part I — Arrays, Strings & Searching](#part-i--arrays-strings--searching)
  - [1. Arrays & Hashing](#1-arrays--hashing) — 10 problems
  - [2. Two Pointers](#2-two-pointers) — 10 problems
  - [3. Sliding Window](#3-sliding-window) — 12 problems
  - [4. Prefix Sum](#4-prefix-sum) — 8 problems
  - [5. Matrix](#5-matrix) — 3 problems
  - [6. Binary Search](#6-binary-search) — 10 problems
  - [7. Sorting & Intervals](#7-sorting--intervals) — 7 problems
- [Part II — Linear Data Structures](#part-ii--linear-data-structures)
  - [8. Stack & Monotonic Stack](#8-stack--monotonic-stack) — 11 problems
  - [9. Linked List](#9-linked-list) — 9 problems
- [Part III — Trees & Heaps](#part-iii--trees--heaps)
  - [10. Binary Trees](#10-binary-trees) — 14 problems
  - [11. Binary Search Trees](#11-binary-search-trees) — 3 problems
  - [12. Tries](#12-tries) — 3 problems
  - [13. Heaps / Priority Queue](#13-heaps--priority-queue) — 6 problems
- [Part IV — Graphs](#part-iv--graphs)
  - [14. Graph BFS](#14-graph-bfs) — 8 problems
  - [15. Graph DFS](#15-graph-dfs) — 4 problems
  - [16. Topological Sort](#16-topological-sort) — 4 problems
  - [17. Union-Find](#17-union-find) — 4 problems
  - [18. Weighted Graphs: Shortest Paths & MST](#18-weighted-graphs-shortest-paths--mst) — 3 problems
- [Part V — Search & Optimization](#part-v--search--optimization)
  - [19. Backtracking](#19-backtracking) — 8 problems
  - [20. Greedy](#20-greedy) — 5 problems
  - [21. Dynamic Programming](#21-dynamic-programming) — 17 problems
  - [22. Bit Manipulation](#22-bit-manipulation) — 6 problems
- [Appendix — Blind 75 Checklist](#appendix--blind-75-checklist)

# **Revision Sheet**

*Read this in under an hour before an interview. Each chapter gets one line on when to use it, one line on the core move, and a one-line key insight per problem. If an insight doesn't immediately remind you of the solution, open that section and re-solve it.*

**In the interview, every time:**
1. **Clarify:** input size, value ranges (negatives? duplicates? empty?), sorted or not, what to return.
2. **Examples:** walk through one normal case and one edge case by hand.
3. **Brute force first:** state it and its complexity in one sentence.
4. **Optimize:** use the input size to pick the target complexity (Part 0), then match the problem's wording to a pattern (cheat sheet in Part 0).
5. **Code:** talk through the template, and name your variables clearly.
6. **Test:** trace your example through the code, then check the edge cases below.
7. **Complexity:** state time and space, including recursion depth and output size.

**Edge cases to check:** empty input · one element · all elements equal · already sorted / reverse sorted · negatives and zero · duplicates · k = 0 or k ≥ n · no valid answer (-1 / [] / "") · disconnected graph · cycles · very deep recursion.

**§1 Arrays & Hashing** — *Use when:* pairs or complements, duplicates, frequencies, grouping by a key. *Core move:* one pass, check the map before inserting, `map.set(x, (map.get(x) || 0) + 1)`.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 1.1 | Two Sum | Store value → index; for each x, look up target − x before inserting x. | O(n) |
| 1.2 | Contains Duplicate | Set; return true the moment an element is already in it. | O(n) |
| 1.3 | Valid Anagram | Equal lengths, count letters of s, decrement for t; any count below 0 fails. | O(n) |
| 1.4 | Group Anagrams | Key = sorted letters (or a 26-count signature) → list of words. | O(n · k log k) |
| 1.5 | Top K Frequent Elements | Count, then bucket by frequency (index = count) and read buckets from the top. | O(n) |
| 1.6 | Longest Consecutive Sequence | Set; only start counting at x when x − 1 is missing. | O(n) |
| 1.7 | First Unique Character in a String | Count pass, then the first index with count 1. | O(n) |
| 1.8 | Isomorphic Strings | Two maps (s → t and t → s) so the mapping is one-to-one both ways. | O(n) |
| 1.9 | Ransom Note | Count the magazine letters and spend them on the note. | O(n + m) |
| 1.10 | Encode and Decode Strings | Length prefix `len#str`; decode reads the length, then exactly that many characters. | O(total chars) |

**§2 Two Pointers** — *Use when:* sorted input, pair or triplet sums, in-place compaction or partition, palindromes. *Core move:* `left`/`right` from both ends moving by comparison, or `slow` (write) / `fast` (read).

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 2.1 | Pair Sum (Two Sum II - Sorted Array) | Sum too small → left++, too big → right--. | O(n) |
| 2.2 | Valid Palindrome | Skip non-alphanumerics from both ends and compare lowercase. | O(n) |
| 2.3 | 3Sum | Sort; fix i, two-pointer the rest; skip duplicate anchors and duplicates after a match. | O(n²) |
| 2.4 | Container With Most Water | Area = min height × width; always move the shorter side inward. | O(n) |
| 2.5 | Remove Duplicates from Sorted Array | Write a value at slow only when fast sees a new one. | O(n) |
| 2.6 | Trapping Rain Water | Move the side with the lower wall; water = its running max − its height. | O(n) |
| 2.7 | Sort Colors (Dutch National Flag) | low/mid/high: 0 → swap to low (advance both), 2 → swap to high (don't advance mid). | O(n) |
| 2.8 | Move Zeroes | Swap each non-zero into the slow position. | O(n) |
| 2.9 | Merge Sorted Array | Fill nums1 from the back with the larger tail element. | O(m + n) |
| 2.10 | Boats to Save People | Sort; the heaviest always boards, taking the lightest along if they fit. | O(n log n) |

**§3 Sliding Window** — *Use when:* longest or shortest **contiguous** subarray or substring under a condition. *Core move:* grow right; `while (invalid)` shrink left; record longest after the loop and shortest inside it.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 3.1 | Maximum Sum Subarray of Size K | Fixed window: add the entering element, subtract the leaving one. | O(n) |
| 3.2 | Best Time to Buy and Sell Stock | Track the minimum price so far; best = max(price − min). | O(n) |
| 3.3 | Longest Substring Without Repeating Characters | Last-seen index map; jump left past the previous occurrence if it's inside the window. | O(n) |
| 3.4 | Fruit Into Baskets | Longest window with at most 2 distinct values (delete keys at count 0). | O(n) |
| 3.5 | Longest Substring with At Most K Distinct Characters | Shrink while map.size > k. | O(n) |
| 3.6 | Max Consecutive Ones III | Shrink while the zeros in the window exceed k. | O(n) |
| 3.7 | Longest Repeating Character Replacement | Valid while window length − max frequency ≤ k. | O(n) |
| 3.8 | Minimum Size Subarray Sum | Positive values: shrink while sum ≥ target, recording the length inside the loop. | O(n) |
| 3.9 | Permutation in String | Fixed window of length s1; compare 26-letter counts. | O(n) |
| 3.10 | Minimum Window Substring | Need-map plus a `formed` counter; shrink while all requirements are met. | O(s + t) |
| 3.11 | Subarrays with K Different Integers | exactly(k) = atMost(k) − atMost(k − 1); atMost adds the window length per step. | O(n) |
| 3.12 | Sliding Window Maximum | Deque of indices with decreasing values; the front is the max, dropped once it leaves the window. | O(n) |

**§4 Prefix Sum** — *Use when:* range sums; count or find subarrays with sum = k or divisible by k, **negatives allowed**. *Core move:* running sum + map of prefix → count (seed `0 → 1`) or prefix → first index (seed `0 → -1`).

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 4.1 | Subarray Sum Equals K | count += seen[sum − k], then record sum. | O(n) |
| 4.2 | Range Sum Query - Immutable | prefix[j + 1] − prefix[i]. | O(1) per query |
| 4.3 | Find Pivot Index | right = total − left − nums[i]. | O(n) |
| 4.4 | Product of Array Except Self | Left-product pass into the output, then multiply by a running right product. | O(n) |
| 4.5 | Contiguous Array (equal 0s and 1s) | Map 0 → −1; longest span between equal prefix sums (store first index). | O(n) |
| 4.6 | Maximum Size Subarray Sum Equals K | Store the first index of each prefix; length = i − first[sum − k]. | O(n) |
| 4.7 | Continuous Subarray Sum | Same remainder twice at least 2 indices apart; normalize negative %. | O(n) |
| 4.8 | Subarray Sums Divisible by K | count += seen[remainder]; normalize negative remainders. | O(n) |

**§5 Matrix** — *Use when:* grid traversal order, or an in-place grid transform. (Regions and paths are graph problems.) *Core move:* boundary pointers (top/bottom/left/right) or index math; reuse part of the grid as markers.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 5.1 | Set Matrix Zeroes | Use the first row and column as markers; remember their own zeros separately. | O(m · n) |
| 5.2 | Spiral Matrix | Walk 4 edges and shrink the bounds; re-check bounds before the bottom and left edges. | O(m · n) |
| 5.3 | Rotate Image | Transpose, then reverse each row. | O(n²) |

**§6 Binary Search** — *Use when:* sorted or rotated data, or a monotonic yes/no question ("binary search on the answer"). *Core move:* `while (lo < hi) { mid; if (feasible(mid)) hi = mid; else lo = mid + 1; }`.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 6.1 | Binary Search (classic) | lo ≤ hi, mid = lo + floor((hi − lo) / 2). | O(log n) |
| 6.2 | Search Insert Position | Classic search; when not found, lo is the insertion index. | O(log n) |
| 6.3 | Find First and Last Position of Element in Sorted Array | Two searches; on a match keep going left (first) or right (last). | O(log n) |
| 6.4 | Search a 2D Matrix | Treat it as one sorted array: row = floor(i / cols), col = i % cols. | O(log(m · n)) |
| 6.5 | Find Minimum in Rotated Sorted Array | nums[mid] > nums[hi] → min is right of mid; otherwise hi = mid. | O(log n) |
| 6.6 | Search in Rotated Sorted Array | One half is always sorted; check whether the target is inside it. | O(log n) |
| 6.7 | Find Peak Element | nums[mid] < nums[mid + 1] → a peak is on the right. | O(log n) |
| 6.8 | Koko Eating Bananas | Search speed in [1, max pile]; feasible if Σ ceil(pile / speed) ≤ h. | O(n log max) |
| 6.9 | Capacity To Ship Packages Within D Days | Search capacity in [max, sum]; greedily count the days needed. | O(n log sum) |
| 6.10 | Median of Two Sorted Arrays | Binary-search the cut in the smaller array so left halves ≤ right halves. | O(log min(m, n)) |

**§7 Sorting & Intervals** — *Use when:* intervals, meetings, events on a timeline, or a custom ordering. *Core move:* sort by start to merge; sort by end to keep the most non-overlapping.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 7.1 | Largest Number | Sort strings so a comes before b when a + b > b + a; an all-zero result becomes "0". | O(n log n) |
| 7.2 | Custom Sort String | Count s, emit letters in `order`, then the leftovers. | O(n) |
| 7.3 | Merge Intervals | Sort by start; extend the last merged end while start ≤ end. | O(n log n) |
| 7.4 | Insert Interval | Three phases: copy intervals before, merge overlapping ones, copy the rest. | O(n) |
| 7.5 | Non-overlapping Intervals | Sort by end; keep an interval if start ≥ previous end, otherwise remove it. | O(n log n) |
| 7.6 | Meeting Rooms | Sort by start; any start < the previous end means a conflict. | O(n log n) |
| 7.7 | Meeting Rooms II | Sorted starts and ends with two pointers (or a min-heap of end times); track the peak. | O(n log n) |

**§8 Stack & Monotonic Stack** — *Use when:* nesting and matching, expression evaluation, next greater/smaller element. *Core move:* monotonic stack of **indices**; pop while the current value beats the top, and resolve what you pop.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 8.1 | Valid Parentheses | Push openers; a closer must match the popped opener; the stack ends empty. | O(n) |
| 8.2 | Min Stack | A second stack holds the minimum so far at each depth. | O(1) per op |
| 8.3 | Evaluate Reverse Polish Notation | Push numbers; an operator pops b then a; use Math.trunc for division. | O(n) |
| 8.4 | Decode String | On '[' push [prevString, count]; on ']' pop and prev + current.repeat(count). | O(output) |
| 8.5 | Basic Calculator II | Apply the previous sign: push ±num, or apply * and / to the top immediately; sum at the end. | O(n) |
| 8.6 | Asteroid Collision | A new negative collides with positive tops: pop smaller ones, both explode if equal. | O(n) |
| 8.7 | Next Greater Element I | Monotonic stack over nums2 → map of value to next greater; answer nums1 from the map. | O(m + n) |
| 8.8 | Next Greater Element II (circular) | Loop i over 2n using i % n; only push during the first pass. | O(n) |
| 8.9 | Daily Temperatures | Pop colder days; answer = i − popped index. | O(n) |
| 8.10 | Largest Rectangle in Histogram | Increasing stack; on pop, width = i − newTop − 1; a sentinel 0 flushes the rest. | O(n) |
| 8.11 | Remove K Digits | Pop larger digits while k > 0; trim the end, then strip leading zeros. | O(n) |

**§9 Linked List** — *Use when:* list position (middle, cycle, n-th from end), reversal, merging, O(1) LRU. *Core move:* dummy head; save `next` before re-pointing; fast & slow pointers.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 9.1 | Reverse Linked List | prev/curr: save next, point curr back to prev, advance both. | O(n) |
| 9.2 | Merge Two Sorted Lists | Dummy plus tail; attach the smaller node, then append the remainder. | O(m + n) |
| 9.3 | Middle of the Linked List | Fast moves 2, slow moves 1; slow ends at the middle. | O(n) |
| 9.4 | Linked List Cycle I & II | Floyd: they meet inside the cycle; reset one to head, step both by 1 → entrance. | O(n) |
| 9.5 | Remove Nth Node From End of List | Move fast n + 1 ahead of slow (both start at the dummy), then walk together. | O(n) |
| 9.6 | Reorder List | Find the middle, reverse the second half, weave the halves. | O(n) |
| 9.7 | Add Two Numbers | Digit-by-digit sum with carry; loop while l1, l2 or carry remain. | O(max(m, n)) |
| 9.8 | Copy List with Random Pointer | Map original → copy (null → null), then wire next and random. | O(n) |
| 9.9 | LRU Cache | Map of key → node plus a doubly linked list with sentinels; move to front on access, evict tail.prev. | O(1) per op |

**§10 Binary Trees** — *Use when:* values built bottom-up (height, sums), constraints passed top-down, or anything per level. *Core move:* `dfs(node)` returns what the parent needs and updates a global best; BFS with one array per level.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 10.1 | Maximum Depth of Binary Tree (Tree Height) | 1 + max(depth(left), depth(right)). | O(n) |
| 10.2 | Same Tree | Both null → true; one null or different values → false; recurse on both sides. | O(n) |
| 10.3 | Invert Binary Tree | Swap the inverted right and left subtrees. | O(n) |
| 10.4 | Balanced Binary Tree | Height returns −1 once anything is unbalanced, so it is one pass. | O(n) |
| 10.5 | Diameter of Binary Tree | best = max(left height + right height); return 1 + max. | O(n) |
| 10.6 | Subtree of Another Tree | Run Same Tree at every node. | O(m · n) |
| 10.7 | Path Sum | Subtract as you go down; at a real leaf check remaining === val. | O(n) |
| 10.8 | Path Sum II | DFS with a path array; push, recurse, pop; copy the path at a matching leaf. | O(n²) |
| 10.9 | Binary Tree Maximum Path Sum | gain = val + max(0, left, right); best = val + both positive gains; start at −Infinity. | O(n) |
| 10.10 | Lowest Common Ancestor of a Binary Tree | Return the node if it is p or q; if both sides return non-null, this node is the LCA. | O(n) |
| 10.11 | Binary Tree Level Order Traversal | Level array → values, then build the next level from children. | O(n) |
| 10.12 | Binary Tree Right Side View | The last node of each BFS level. | O(n) |
| 10.13 | Construct Binary Tree from Preorder and Inorder Traversal | The next preorder value is the root; its inorder index splits left and right (map value → index). | O(n) |
| 10.14 | Serialize and Deserialize Binary Tree | Preorder with '#' for null; rebuild by reading tokens in the same order. | O(n) |

**§11 Binary Search Trees** — *Use when:* sorted order, k-th smallest, range, or a left/right decision by comparison. *Core move:* in-order traversal = sorted; pass (min, max) bounds down.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 11.1 | Validate Binary Search Tree | Pass (min, max) down; a node must sit strictly inside the bounds. | O(n) |
| 11.2 | Kth Smallest Element in a BST | Iterative in-order traversal; stop at the k-th popped node. | O(h + k) |
| 11.3 | Lowest Common Ancestor of a Binary Search Tree | Both smaller → go left, both larger → go right, otherwise this node. | O(h) |

**§12 Tries** — *Use when:* prefixes, autocomplete, wildcard dictionary, many words on one grid. *Core move:* node = { children: Map, isEnd }; walk one character at a time.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 12.1 | Implement Trie (Prefix Tree) | Shared walk(prefix); search also needs isEnd. | O(L) |
| 12.2 | Design Add and Search Words Data Structure | Trie plus DFS; '.' tries every child. | O(L) without dots |
| 12.3 | Word Search II | Build a trie of the words; DFS the grid while walking the trie; clear the word once found. | O(cells · 3^L) |

**§13 Heaps / Priority Queue** — *Use when:* k-th / top k / k closest, merge k sorted, running median, always take the cheapest next. *Core move:* k **largest** → **min**-heap of size k (evict the smallest); comparator `b − a` makes a max-heap.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 13.1 | Implement a Min-Heap (Priority Queue) | Array tree: push sifts up through parent (i − 1) >> 1; pop moves the last item to the root and sifts down. | O(log n) |
| 13.2 | Kth Largest Element in an Array | Quickselect with a three-way partition toward index n − k, or a min-heap of size k. | O(n) avg |
| 13.3 | K Closest Points to Origin | Max-heap of size k by x² + y²; pop the farthest. | O(n log k) |
| 13.4 | Merge K Sorted Lists | Min-heap of list heads; pop the smallest, push its next. | O(N log k) |
| 13.5 | Task Scheduler | Max-heap of counts plus a cooldown queue; or the formula max(tasks, (maxFreq − 1)(n + 1) + countMax). | O(T) |
| 13.6 | Find Median from Data Stream | Max-heap low and min-heap high; push to low, move its top to high, rebalance so low ≥ high. | O(log n) add |

**§14 Graph BFS** — *Use when:* fewest steps with equal-cost moves; spreading from many sources; implicit state graphs. *Core move:* queue + head index; mark visited when **enqueuing**; seed every source for multi-source BFS.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 14.1 | Number of Islands (BFS version) | For each unvisited '1', count it and flood-fill with BFS. | O(m · n) |
| 14.2 | Shortest Path in Binary Matrix | BFS in 8 directions; the distance at the first arrival is the answer. | O(n²) |
| 14.3 | Rotting Oranges | Multi-source BFS from all rotten oranges, one level per minute; −1 if fresh oranges remain. | O(m · n) |
| 14.4 | Walls and Gates | Multi-source BFS from all gates; fill rooms on first reach. | O(m · n) |
| 14.5 | 01 Matrix | Multi-source BFS from every 0. | O(m · n) |
| 14.6 | Open the Lock | BFS over 4-digit states (8 neighbours each); deadends act as walls. | O(10⁴) |
| 14.7 | Word Ladder | BFS over words; neighbours = change one letter and check the set; delete words once used. | O(n · L² · 26) |
| 14.8 | Snakes and Ladders | BFS over squares with dice 1–6; map square → (row, col) boustrophedon-style. | O(n²) |

**§15 Graph DFS** — *Use when:* count or measure regions, reachability, clone a structure that has cycles. *Core move:* mark visited before recursing; search backwards from targets when that's easier.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 15.1 | Number of Islands (DFS version) | Sink each island ('1' → '0') recursively, counting starts. | O(m · n) |
| 15.2 | Max Area of Island | DFS returns 1 + the areas of its 4 neighbours. | O(m · n) |
| 15.3 | Clone Graph | Map original → clone; create and store the clone before visiting neighbours. | O(V + E) |
| 15.4 | Pacific Atlantic Water Flow | DFS uphill from each ocean's border cells; answer = cells reached by both. | O(m · n) |

**§16 Topological Sort** — *Use when:* prerequisites, dependency or build order, "can all of them finish?". *Core move:* in-degrees; queue all zero in-degree nodes; output count < n means a cycle.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 16.1 | Course Schedule (BFS / Kahn's Algorithm) | Kahn's: processed === numCourses means no cycle. | O(V + E) |
| 16.2 | Course Schedule (DFS cycle detection) | Three states; reaching a "visiting" node means a cycle. | O(V + E) |
| 16.3 | Course Schedule II | Kahn's; the processing order is the answer ([] if there's a cycle). | O(V + E) |
| 16.4 | Alien Dictionary | The first differing letter of each adjacent word pair is an edge; topo sort; "abc" before "ab" is invalid. | O(total chars) |

**§17 Union-Find** — *Use when:* incremental connectivity, "does this edge make a cycle?", counting components. *Core move:* find with path compression, union by rank; `union` returns false when already connected.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 17.1 | Implement Union-Find (Disjoint Set Union) | parent[], rank[], count; path halving in find. | O(α(n)) |
| 17.2 | Number of Connected Components in an Undirected Graph | Union every edge; the answer is uf.count. | O(E · α(n)) |
| 17.3 | Graph Valid Tree | Exactly n − 1 edges and no failed union. | O(n · α(n)) |
| 17.4 | Redundant Connection | The first edge whose union fails. | O(n · α(n)) |

**§18 Weighted Graphs: Shortest Paths & MST** — *Use when:* weighted shortest path (Dijkstra), at-most-k edges (Bellman-Ford), connect everything cheaply (MST). *Core move:* Dijkstra = BFS + min-heap of [dist, node]; skip stale entries where d > dist[u].

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 18.1 | Network Delay Time | Dijkstra from k; the answer is the max distance, or −1 if any node is unreachable. | O((V + E) log V) |
| 18.2 | Cheapest Flights Within K Stops | Bellman-Ford for k + 1 rounds, relaxing from a copy of the previous round. | O(k · E) |
| 18.3 | Min Cost to Connect All Points | Prim with arrays: add the closest point outside the tree, update minDist. | O(n²) |

**§19 Backtracking** — *Use when:* "return all" subsets, permutations, combinations, partitions or placements; small n. *Core move:* choose → recurse → un-choose; push a copy; `start` index for combinations, `used[]` for permutations.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 19.1 | Subsets / Subsets II | Record at every node; sort and skip i > start duplicates. | O(n · 2ⁿ) |
| 19.2 | Permutations | used[] array; record when the path length is n. | O(n · n!) |
| 19.3 | Combination Sum | Recurse with i (not i + 1) so a number can be reused. | exponential |
| 19.4 | Letter Combinations of a Phone Number | One digit per level, one letter per branch. | O(n · 4ⁿ) |
| 19.5 | Generate Parentheses | Add '(' while open < n; add ')' while close < open. | O(Catalan(n)) |
| 19.6 | Palindrome Partitioning | Try every palindromic prefix from start, then recurse on the rest. | O(n · 2ⁿ) |
| 19.7 | Word Search | DFS from each cell; mark with '#', restore after. | O(m · n · 4^L) |
| 19.8 | N-Queens | One queen per row; sets for columns, r − c and r + c. | O(n!) |

**§20 Greedy** — *Use when:* a running best or farthest reach decides everything, or a failing prefix can be thrown away. *Core move:* one pass keeping a running value; be ready to say why the local choice is safe.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 20.1 | Maximum Subarray | Kadane: current = max(x, current + x). | O(n) |
| 20.2 | Jump Game | Track the farthest reach; fail if i > reach. | O(n) |
| 20.3 | Jump Game II | BFS levels over indices: jump when i reaches the current level's end. | O(n) |
| 20.4 | Gas Station | If total < 0, return −1; when the tank goes negative, restart at i + 1. | O(n) |
| 20.5 | Partition Labels | Last index of each letter; close the part when i reaches the farthest last index. | O(n) |

**§21 Dynamic Programming** — *Use when:* count the ways / min cost / max value / is it possible, with repeated subproblems. *Core move:* define the state in words → transition → base cases → fill order; then cut space to the last row or two values.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 21.1 | Fibonacci Number / Climbing Stairs | ways(n) = ways(n − 1) + ways(n − 2); two variables. | O(n) |
| 21.2 | House Robber | dp = max(skip: prev1, rob: prev2 + x). | O(n) |
| 21.3 | House Robber II | max(rob houses 0..n−2, rob houses 1..n−1). | O(n) |
| 21.4 | Decode Ways | Add prev1 if the last digit is 1–9; add prev2 if the last two digits are 10–26. | O(n) |
| 21.5 | Maximum Product Subarray | Track both max and min products ending here (a negative swaps them). | O(n) |
| 21.6 | Word Break | dp[i] = some j < i with dp[j] true and s[j..i) in the dictionary. | O(n³) |
| 21.7 | Coin Change | dp[a] = min(dp[a − coin] + 1); Infinity means impossible. | O(amount · coins) |
| 21.8 | Coin Change II | Coins in the outer loop → each combination counted once. | O(amount · coins) |
| 21.9 | Combination Sum IV | Amount in the outer loop → every order counted (permutations). | O(target · n) |
| 21.10 | 0/1 Knapsack | 1D dp, capacity looped **backwards** so each item is used once. | O(n · W) |
| 21.11 | Partition Equal Subset Sum | Odd total → false; otherwise 0/1 subset-sum to total / 2 with a boolean dp. | O(n · sum) |
| 21.12 | Longest Increasing Subsequence | tails[]: binary-search the first tail ≥ x and replace it, or append. | O(n log n) |
| 21.13 | Longest Common Subsequence | Characters match → diagonal + 1; otherwise max(up, left). | O(m · n) |
| 21.14 | Edit Distance | Characters match → diagonal; otherwise 1 + min(replace, delete, insert). | O(m · n) |
| 21.15 | Unique Paths | dp[c] += dp[c − 1], one row at a time. | O(m · n) |
| 21.16 | Longest Palindromic Substring | Expand around 2n − 1 centres (odd and even). | O(n²) |
| 21.17 | Palindromic Substrings | Center expansion; count every successful step. | O(n²) |

**§22 Bit Manipulation** — *Use when:* single number among pairs, missing number, counting or reversing bits, no `+` allowed. *Core move:* XOR cancels pairs; `n & (n − 1)` clears the lowest set bit; `>>> 0` reads a result as unsigned.

| **§** | **Problem** | **Key insight** | **Time** |
| --- | --- | --- | --- |
| 22.1 | Single Number | XOR everything; pairs cancel. | O(n) |
| 22.2 | Number of 1 Bits | Repeat n &= n − 1 and count the steps. | O(set bits) |
| 22.3 | Counting Bits | bits[i] = bits[i >> 1] + (i & 1). | O(n) |
| 22.4 | Reverse Bits | 32 times: result = (result << 1) \| (n & 1), then n >>>= 1; return result >>> 0. | O(1) |
| 22.5 | Missing Number | XOR all indices 0..n with all values. | O(n) |
| 22.6 | Sum of Two Integers | Repeat a = a ^ b, b = (a & b) << 1 until there is no carry. | O(1) |

# **Part 0 — Start Here**

## **How to Use This Guide**

1. **Read the chapters in order the first time.** Each chapter builds on earlier ones (see *How the Topics Connect* below), so later patterns reuse ideas you have already practiced.
2. **For each chapter, learn the intro before the problems.** The *Recognize it when* cues, the *Template* and the *Pitfalls* are what transfer to problems you have never seen.
3. **Solve each problem from a blank editor before reading the solution.** Give yourself 20–25 minutes, then compare. Tick ☐ Practiced when you have solved it unaided, and ☐ Revised when you have re-solved it a week later.
4. **Revise with the Revision Sheet.** Before an interview, read the *Revision Sheet* at the top of this guide (about an hour), then the *Pattern Recognition Cheat Sheet* below. Re-solve any problem whose one-line insight doesn't immediately bring back the solution.

## **How the Topics Connect**

The guide is ordered so that every new pattern is an extension of one you already know:

- **Linear scans:** Arrays & Hashing (§1) trades space for time. Two Pointers (§2) removes that space when the data is sorted. Sliding Window (§3) is two pointers moving in the same direction, usually with a hash map counting the window. Prefix Sum (§4) takes over when a window can't slide, for example with negative numbers, and pairs with a hash map again.
- **Halving the search space:** Binary Search (§6) starts on sorted arrays, then generalizes to *binary search on the answer* (any monotonic yes/no question). The same idea powers Binary Search Trees (§11) and the O(n log n) Longest Increasing Subsequence (§21).
- **Sort first:** Sorting (§7) unlocks Two Pointers (3Sum), interval merging, and most Greedy proofs (§20).
- **Stacks:** A stack (§8) handles nesting and "most recent first". A *monotonic* stack answers next-greater/smaller queries, and its cousin the monotonic deque solves Sliding Window Maximum. A stack is also what recursion uses under the hood, which leads to DFS.
- **Pointers in memory:** Linked lists (§9) reuse two pointers as *fast & slow* pointers. LRU Cache combines a hash map with a doubly linked list.
- **Recursion family:** Recursion on trees (§10) → DFS on graphs (§15) → Backtracking (§19), which is DFS that undoes its choice → Dynamic Programming (§21), which is DFS that remembers answers (memoization) or the same recurrence filled bottom-up.
- **BFS family:** Level-order traversal on trees (§10) → BFS on grids and implicit graphs (§14) for unweighted shortest paths → Kahn's topological sort (§16), which is BFS driven by in-degrees → Dijkstra (§18), which is BFS with a heap so it can handle edge weights.
- **Heaps:** A heap (§13) gives repeated access to the smallest/largest item: top-k, merge-k, and running median. It is also the engine inside Dijkstra and Prim (§18).
- **Connectivity:** Union-Find (§17) is an alternative to DFS for "are these connected?" questions, and it powers Kruskal's minimum spanning tree.
- **Greedy vs DP:** Greedy (§20) works when a locally best choice is provably safe; when it isn't, you need DP (§21). Kadane's algorithm sits on the boundary: it is DP with O(1) state.
- **Bits:** Bit Manipulation (§22) gives O(1)-space tricks (XOR cancels pairs) and represents subsets as bitmasks, an alternative to backtracking for small n.

## **Pattern Recognition Cheat Sheet**

| **If the problem says…** | **Try first** | **Chapter** |
| --- | --- | --- |
| Find a pair/complement, duplicates, frequencies, group by key | Hash map / set | §1 |
| Sorted array; pair or triplet with a target sum; in-place partition | Two pointers | §2 |
| Longest/shortest **contiguous** substring or subarray satisfying a condition | Sliding window | §3 |
| Count subarrays with sum = k / divisible by k (negatives allowed); many range-sum queries | Prefix sum + hash map | §4 |
| Traverse or transform a 2D grid in place (spiral, rotate) | Boundary pointers / index math | §5 |
| Sorted or rotated array search; "minimize the maximum" / "smallest capacity that works" | Binary search (on the answer) | §6 |
| Overlapping intervals, meetings, scheduling | Sort by start or end | §7 |
| Matching brackets, nested encodings, expression evaluation | Stack | §8 |
| Next greater/smaller element, "how many days until…", histogram areas | Monotonic stack | §8 |
| Linked list middle, cycle, n-th from end, reorder | Fast & slow pointers, dummy node | §9 |
| Tree height, path sums, balance, diameter | DFS returning a value to the parent | §10 |
| Sorted order / k-th smallest in a BST | In-order traversal | §11 |
| Prefix search, autocomplete, many words on one board | Trie | §12 |
| k-th largest, k closest, merge k sorted, running median | Heap | §13 |
| Fewest steps in an unweighted grid/graph; spreading from many sources | BFS (multi-source) | §14 |
| Count islands / regions, clone a graph, reachability | DFS (or BFS) | §15 |
| Prerequisites, dependencies, build order | Topological sort | §16 |
| Dynamic connectivity, "does this edge create a cycle?", components | Union-Find | §17 |
| Shortest path with weights; connect all points cheaply | Dijkstra / Bellman-Ford / MST | §18 |
| All subsets, permutations, combinations, placements (n ≤ ~20) | Backtracking | §19 |
| Reachability with jumps, local choice that never needs undoing | Greedy | §20 |
| Count the ways / min cost / max value, with overlapping sub-choices | Dynamic programming | §21 |
| Number appearing once among pairs, counting bits, no `+` allowed | Bit manipulation | §22 |

## **Complexity Target by Input Size**

Interview constraints hint at the expected complexity (roughly 10⁸ simple operations per second):

| **n up to** | **Target complexity** | **Typical technique** |
| --- | --- | --- |
| 10–12 | O(n!) | Permutations backtracking |
| 20–25 | O(2ⁿ) or O(n · 2ⁿ) | Subsets backtracking, bitmasks |
| 100–500 | O(n³) | Interval DP, 3 nested loops |
| 1,000–5,000 | O(n²) | 2D DP, all pairs |
| 10⁵–10⁶ | O(n log n) or O(n) | Sorting, heap, two pointers, sliding window, prefix sum |
| 10⁹ and above (value range) | O(log n) | Binary search on the answer, math |

## **JavaScript Toolkit**

JavaScript has a few traps that matter in interviews, and no built-in heap, queue or deque. The heap is implemented in *Implement a Min-Heap (Priority Queue)* (§13.1) and Union-Find in *Implement Union-Find (Disjoint Set Union)* (§17.1).

```js
// 1. Numeric sort: the default sort() compares values as strings
[10, 9, 1].sort();                 // [1, 10, 9]  wrong
[10, 9, 1].sort((a, b) => a - b);  // [1, 9, 10]  right

// 2. Integer division truncates toward zero in most languages; Math.floor does not
Math.trunc(-7 / 2); // -3   (Math.floor(-7 / 2) is -4)

// 3. % keeps the sign of the left operand, so normalize for a true modulo
const mod = (a, m) => ((a % m) + m) % m; // mod(-1, 5) === 4

// 4. 2D arrays: never fill() with an array, or every row is the same object
const grid = Array.from({ length: 3 }, () => new Array(4).fill(0));

// 5. Queue for BFS: array + head index. shift() is O(n) and gets slow for large queues
const queue = ['start'];
for (let head = 0; head < queue.length; head++) {
  const node = queue[head];
  // queue.push(neighbor) for each unvisited neighbor of node
}

// 6. Composite keys: arrays compare by reference in Map/Set, so build a string
const seen = new Set();
seen.add(`${2},${3}`);
seen.has('2,3'); // true

// 7. Letters to indices 0..25
const idx = (ch) => ch.charCodeAt(0) - 97; // 'a' -> 0

// 8. Math.max(...arr) passes every element as an argument and can throw a
//    RangeError for very large arrays; reduce is safe at any size
const largest = (arr) => arr.reduce((m, x) => Math.max(m, x), -Infinity);

// 9. Bitwise operators work on signed 32-bit integers; >>> 0 reads the result as unsigned
(-1 >>> 0); // 4294967295
```

# **Part I — Arrays, Strings & Searching**

## **1. Arrays & Hashing**

**Goal:** O(n²) → O(n)

**Core idea:** *Trade space for time by storing seen values/indices/counts for O(1) lookup instead of nested loops.*

**Recognize it when:**
- You would otherwise compare every pair ("does a complement exist?", "is there a duplicate?").
- You need frequencies (anagrams, top-k, first unique) or need to group items by a computed key.

**Connects to:** The building block for almost everything else. Window counts in Sliding Window (§3), prefix-sum lookups (§4), visited sets in graphs (§14) and memo tables in DP (§21) are all hash maps. *Subarray Sum Equals K* is a hash-map problem too; it lives in *Subarray Sum Equals K* (§4.1), where the prefix-sum idea is explained.

**Template:**

```js
// Frequency count
function frequencies(items) {
  const count = new Map();
  for (const x of items) count.set(x, (count.get(x) || 0) + 1);
  return count;
}

// "Have I already seen what I need?"
function findPair(nums, target) {
  const seen = new Map(); // value -> index
  for (let i = 0; i < nums.length; i++) {
    if (seen.has(target - nums[i])) return [seen.get(target - nums[i]), i];
    seen.set(nums[i], i);
  }
  return null;
}
```

**Pitfalls:**
- Plain objects turn keys into strings (`obj[1]` and `obj['1']` are the same key). Use `Map` for numeric keys.
- Arrays used as `Map` keys compare by reference. Use a string key such as `arr.join(',')`.
- Check the map *before* inserting the current element when an element must not pair with itself (Two Sum).

### **1.1  Two Sum**

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

### **1.2  Contains Duplicate**

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

### **1.3  Valid Anagram**

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

### **1.4  Group Anagrams**

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

### **1.5  Top K Frequent Elements**

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

**Complexity:** Time O(n) with bucket sort (O(n log k) with a size-k heap, O(n log n) with a full sort), Space O(n)

***Alternate approaches:*** *Use a min-heap of size k over frequency counts — push each unique number, pop when heap exceeds k. O(n log k) time, useful when k is small relative to n. The heap class is in *Implement a Min-Heap (Priority Queue)* (§13.1).*

☐ Practiced   ☐ Revised

### **1.6  Longest Consecutive Sequence**

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

### **1.7  First Unique Character in a String**

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

### **1.8  Isomorphic Strings**

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

### **1.9  Ransom Note**

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

### **1.10  Encode and Decode Strings**

**Problem:** Design `encode(strs)` that turns a list of strings into one string, and `decode(str)` that restores the original list. Strings may contain any character, including your delimiter.

**Approach:** Prefix each string with its length and a separator: `"4#leet5#co#de"`. To decode, read digits up to the next `#`, then take exactly that many characters. The content itself is never scanned for delimiters, so any character is safe.

**Optimal Solution (JavaScript):**

```js
function encode(strs) {
  return strs.map(s => `${s.length}#${s}`).join('');
}

function decode(str) {
  const result = [];
  let i = 0;
  while (i < str.length) {
    const hash = str.indexOf('#', i);
    const len = Number(str.slice(i, hash));
    result.push(str.slice(hash + 1, hash + 1 + len));
    i = hash + 1 + len;
  }
  return result;
}
```

**Complexity:** Time O(total characters), Space O(total characters) for the output

***Alternate approaches:*** *Join with a delimiter and escape every occurrence of it inside the strings (like CSV quoting). It works but is easy to get wrong. JSON.stringify solves it in one line, but interviewers usually disallow it because it skips the point of the question.*

☐ Practiced   ☐ Revised

## **2. Two Pointers**

**Goal:** O(n²) → O(n), O(1) extra space

**Core idea:** *Use two indices moving toward each other (or same direction) on sorted data / arrays to avoid nested loops.*

**Recognize it when:**
- The input is sorted (or can be sorted cheaply) and you're looking for a pair/triplet with a target property.
- You must modify an array in place: remove duplicates, move zeroes, partition into regions.
- You compare characters from both ends (palindromes).

**Connects to:** Builds on §1: it's the O(1)-space alternative to a hash map when data is sorted. Leads to Sliding Window (§3), which is two pointers moving in the same direction, fast & slow pointers on linked lists (§9), and the three-way partition reused by Quickselect (*Kth Largest Element in an Array* (§13.2)).

**Template:**

```js
// Opposite ends (sorted input)
function pairWithSum(nums, target) {
  let left = 0, right = nums.length - 1;
  while (left < right) {
    const sum = nums[left] + nums[right];
    if (sum === target) return [left, right];
    if (sum < target) left++; // need bigger
    else right--;             // need smaller
  }
  return null;
}

// Same direction: slow = write position, fast = read position
function compact(nums, keep) {
  let slow = 0;
  for (let fast = 0; fast < nums.length; fast++) {
    if (keep(nums[fast])) nums[slow++] = nums[fast];
  }
  return slow; // new length
}
```

**Pitfalls:**
- `while (left < right)` vs `left <= right`: use `<=` only when a single element is a valid answer (e.g., Boats to Save People).
- When results must be unique (3Sum), skip duplicates *after* recording a match, on both pointers.
- Sorting costs O(n log n) and loses original indices. Store `[value, index]` pairs first if indices are needed.

### **2.1  Pair Sum (Two Sum II - Sorted Array)**

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

### **2.2  Valid Palindrome**

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

### **2.3  3Sum**

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

### **2.4  Container With Most Water**

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

### **2.5  Remove Duplicates from Sorted Array**

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

### **2.6  Trapping Rain Water**

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

### **2.7  Sort Colors (Dutch National Flag)**

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

### **2.8  Move Zeroes**

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

### **2.9  Merge Sorted Array**

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

### **2.10  Boats to Save People**

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

## **3. Sliding Window**

**Goal:** O(n²) → O(n)

**Core idea:** *Maintain a shrinking/growing window over a contiguous range to track a running condition (sum, count, distinct chars).*

**Recognize it when:**
- The answer is a **contiguous** subarray or substring (longest, shortest, count of).
- The validity of a window changes monotonically: growing it can only make it "more invalid", shrinking it "more valid".
- Fixed size k → fixed window; "at most / at least" condition → variable window.

**Connects to:** Two Pointers (§2) moving in the same direction plus hash-map counts (§1). When elements can be negative, the monotonic property breaks, so switch to Prefix Sum (§4). The monotonic deque in *Sliding Window Maximum* (§3.12) is the queue version of the monotonic stack (§8).

**Template:**

```js
// Variable-size window: longest valid window
function longestValidWindow(arr, isInvalid, add, remove) {
  let left = 0, best = 0;
  for (let right = 0; right < arr.length; right++) {
    add(arr[right]);                 // grow
    while (isInvalid()) {            // shrink until valid again
      remove(arr[left]);
      left++;
    }
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

**Pitfalls:**
- For "shortest valid window", record the answer *inside* the shrink loop; for "longest", record it *after*.
- Counting "exactly k" directly is hard. Use `atMost(k) - atMost(k - 1)` (*Subarrays with K Different Integers* (§3.11)).
- Delete a key from the count map when its count hits 0, or `map.size` will be wrong.

### **3.1  Maximum Sum Subarray of Size K**

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

### **3.2  Best Time to Buy and Sell Stock**

**Problem:** Given daily stock prices, return the maximum profit from one buy followed by one later sell (0 if no profit is possible).

**Approach:** Scan once, keeping the lowest price seen so far. That is the best buy day for any sell day from here on. At each day, the best profit selling today is `price - minPrice`. The window runs from the cheapest day so far to today.

**Optimal Solution (JavaScript):**

```js
function maxProfit(prices) {
  let minPrice = Infinity, best = 0;
  for (const p of prices) {
    minPrice = Math.min(minPrice, p);
    best = Math.max(best, p - minPrice);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Brute force tries every buy/sell pair, which is O(n²). Equivalently, run Kadane's algorithm on the day-to-day price differences: the maximum subarray sum of the differences is the best profit.*

☐ Practiced   ☐ Revised

### **3.3  Longest Substring Without Repeating Characters**

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

### **3.4  Fruit Into Baskets**

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

### **3.5  Longest Substring with At Most K Distinct Characters**

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

### **3.6  Max Consecutive Ones III**

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

### **3.7  Longest Repeating Character Replacement**

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

### **3.8  Minimum Size Subarray Sum**

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

***Alternate approaches:*** *Binary search on prefix sums: because every element is positive, the prefix-sum array is strictly increasing, so for each start index you can binary-search the first end index whose prefix sum reaches the target — O(n log n) time, O(n) space. Both this and the sliding window depend on positive values; with negative numbers you need a monotonic deque over prefix sums (Shortest Subarray with Sum at Least K).*

☐ Practiced   ☐ Revised

### **3.9  Permutation in String**

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

### **3.10  Minimum Window Substring**

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

### **3.11  Subarrays with K Different Integers**

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

### **3.12  Sliding Window Maximum**

**Problem:** Given an array and window size k, return the maximum value in each sliding window as it moves from left to right.

**Approach:** Use a monotonic deque storing indices, keeping values in decreasing order. Pop smaller elements from the back before adding a new one (they can never be the max while the new element is in range). Pop from the front when the index falls out of the window. The front of the deque is always the current max.

**Optimal Solution (JavaScript):**

```js
function maxSlidingWindow(nums, k) {
  const deque = []; // indices; live part is deque[head..], values decreasing
  let head = 0;
  const result = [];

  for (let i = 0; i < nums.length; i++) {
    if (head < deque.length && deque[head] <= i - k) head++; // front left the window
    while (head < deque.length && nums[deque[deque.length - 1]] < nums[i]) deque.pop();
    deque.push(i);
    if (i >= k - 1) result.push(nums[deque[head]]);
  }
  return result;
}
```

**Complexity:** Time O(n) — each index pushed/popped once, Space O(n) for the backing array (the live part never exceeds k; a ring buffer would make it O(k)). A head index replaces `deque.shift()`, which is O(k) per call in JavaScript.

***Alternate approaches:*** *Brute force recomputes the max of each window — O(n·k) time. A max-heap with lazy deletion also works in O(n log n) time, simpler to write but slower asymptotically.*

☐ Practiced   ☐ Revised

## **4. Prefix Sum**

**Goal:** O(n²) → O(n)

**Core idea:** *Precompute cumulative sums so any range-sum query becomes O(1); pair with a hash map for subarray-sum problems.*

**Recognize it when:**
- Many range-sum queries on a fixed array.
- "Count/find subarrays whose sum equals k / is divisible by k", especially when **negative numbers** are allowed (sliding window fails there).
- "Equal number of X and Y": map one to +1 and the other to −1, then look for a zero-sum subarray.

**Connects to:** Hash maps (§1) store prefix sums seen so far. It's the fallback for Sliding Window (§3) when values can be negative. Prefix *products* (*Product of Array Except Self* (§4.4)) use the same idea. 2D prefix sums extend it to matrices.

**Template:**

```js
// sum(i..j) = prefix[j + 1] - prefix[i]
function buildPrefix(nums) {
  const prefix = [0];
  for (const n of nums) prefix.push(prefix[prefix.length - 1] + n);
  return prefix;
}

// Count subarrays with sum k: earlier prefix p with current - p === k
function countSubarrays(nums, k) {
  const seen = new Map([[0, 1]]); // the empty prefix
  let sum = 0, count = 0;
  for (const n of nums) {
    sum += n;
    count += seen.get(sum - k) || 0;
    seen.set(sum, (seen.get(sum) || 0) + 1);
  }
  return count;
}
```

**Pitfalls:**
- Seed the map with the empty prefix (`0 → 1` for counts, `0 → -1` for first index), or subarrays starting at index 0 are missed.
- For "longest", store only the *first* index of each prefix value; for "count", store how many times it occurred.
- `%` can be negative in JS. Normalize with `((sum % k) + k) % k`.

### **4.1  Subarray Sum Equals K**

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

### **4.2  Range Sum Query - Immutable**

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

### **4.3  Find Pivot Index**

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

### **4.4  Product of Array Except Self**

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

### **4.5  Contiguous Array (equal 0s and 1s)**

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

### **4.6  Maximum Size Subarray Sum Equals K**

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

### **4.7  Continuous Subarray Sum**

**Problem:** Given an integer array and an integer k ≥ 1, return true if the array has a contiguous subarray of size at least 2 whose sum is a multiple of k.

**Approach:** Track running sum modulo k. If the same remainder appears at two indices at least 2 apart, the subarray between them sums to a multiple of k. Store {remainder → first index seen} in a hash map.

**Optimal Solution (JavaScript):**

```js
function checkSubarraySum(nums, k) {
  const firstIndex = new Map([[0, -1]]);
  let sum = 0;
 
  for (let i = 0; i < nums.length; i++) {
    sum += nums[i];
    const rem = ((sum % k) + k) % k; // JS % can be negative
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

### **4.8  Subarray Sums Divisible by K**

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

## **5. Matrix**

**Goal:** In-place O(1)-space grid manipulation

**Core idea:** *Treat the grid as rows × cols index math: walk shrinking boundaries, or reuse part of the matrix as scratch space.*

**Recognize it when:**
- The output is a traversal order of the grid (spiral, diagonal).
- The grid must be transformed in place (rotate, mark rows/columns).
- If the question is about *regions or paths* through a grid, it's a graph problem instead: see BFS (§14) and DFS (§15).

**Connects to:** Index math also appears in *Search a 2D Matrix* (§6.4) (flatten r, c ↔ i). Grids become graphs in §14 and §15, and DP tables in §21 (*Unique Paths* (§21.15)).

**Template:**

```js
// 4-directional neighbors with bounds checking
function neighbors(grid, r, c) {
  const rows = grid.length, cols = grid[0].length;
  const result = [];
  for (const [dr, dc] of [[1, 0], [-1, 0], [0, 1], [0, -1]]) {
    const nr = r + dr, nc = c + dc;
    if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) result.push([nr, nc]);
  }
  return result;
}
```

**Pitfalls:**
- Square-only tricks (transpose) don't apply to m × n grids.
- When updating in place, don't let early writes corrupt values you still need to read (use markers or a second pass).
- In spiral traversal, re-check `top <= bottom` and `left <= right` before the 3rd and 4th edges.

### **5.1  Set Matrix Zeroes**

**Problem:** Given an m x n matrix, if an element is 0, set its entire row and column to 0, in place.

**Approach:** Use the first row and first column as marker storage: `matrix[r][0] = 0` means "zero row r" and `matrix[0][c] = 0` means "zero column c". Remember separately whether the first row/column themselves need zeroing, apply the markers to the inner cells, and finish with the first row/column.

**Optimal Solution (JavaScript):**

```js
function setZeroes(matrix) {
  const rows = matrix.length, cols = matrix[0].length;
  let firstRowZero = false, firstColZero = false;
  for (let c = 0; c < cols; c++) if (matrix[0][c] === 0) firstRowZero = true;
  for (let r = 0; r < rows; r++) if (matrix[r][0] === 0) firstColZero = true;

  // record zeros in the first row/column
  for (let r = 1; r < rows; r++)
    for (let c = 1; c < cols; c++)
      if (matrix[r][c] === 0) { matrix[r][0] = 0; matrix[0][c] = 0; }

  // zero inner cells using the markers
  for (let r = 1; r < rows; r++)
    for (let c = 1; c < cols; c++)
      if (matrix[r][0] === 0 || matrix[0][c] === 0) matrix[r][c] = 0;

  if (firstRowZero) for (let c = 0; c < cols; c++) matrix[0][c] = 0;
  if (firstColZero) for (let r = 0; r < rows; r++) matrix[r][0] = 0;
}
```

**Complexity:** Time O(m · n), Space O(1)

***Alternate approaches:*** *Record zero rows and columns in two sets (or boolean arrays), then do a second pass. That is O(m · n) time and O(m + n) space, and easier to get right; start with it and optimize if asked.*

☐ Practiced   ☐ Revised

### **5.2  Spiral Matrix**

**Problem:** Given an m x n matrix, return all elements in spiral order (right, down, left, up, repeating inward).

**Approach:** Keep four boundaries: top, bottom, left, right. Walk the top row, then the right column, then the bottom row backwards, then the left column upwards, shrinking the matching boundary after each edge. Re-check the boundaries before the bottom and left edges so a single remaining row or column isn't visited twice.

**Optimal Solution (JavaScript):**

```js
function spiralOrder(matrix) {
  const result = [];
  let top = 0, bottom = matrix.length - 1;
  let left = 0, right = matrix[0].length - 1;

  while (top <= bottom && left <= right) {
    for (let c = left; c <= right; c++) result.push(matrix[top][c]);
    top++;
    for (let r = top; r <= bottom; r++) result.push(matrix[r][right]);
    right--;
    if (top <= bottom) {
      for (let c = right; c >= left; c--) result.push(matrix[bottom][c]);
      bottom--;
    }
    if (left <= right) {
      for (let r = bottom; r >= top; r--) result.push(matrix[r][left]);
      left++;
    }
  }
  return result;
}
```

**Complexity:** Time O(m · n), Space O(1) extra (excluding output)

***Alternate approaches:*** *Simulate a walker with a direction vector that turns right whenever the next cell is out of bounds or already visited. Same O(m · n) time, but it needs an O(m · n) visited grid.*

☐ Practiced   ☐ Revised

### **5.3  Rotate Image**

**Problem:** Rotate an n x n matrix 90 degrees clockwise, in place.

**Approach:** A clockwise rotation equals a transpose (swap `matrix[r][c]` with `matrix[c][r]`) followed by reversing each row. Both steps are in place.

**Optimal Solution (JavaScript):**

```js
function rotate(matrix) {
  const n = matrix.length;
  for (let r = 0; r < n; r++)
    for (let c = r + 1; c < n; c++)
      [matrix[r][c], matrix[c][r]] = [matrix[c][r], matrix[r][c]];
  for (const row of matrix) row.reverse();
}
```

**Complexity:** Time O(n²), Space O(1)

***Alternate approaches:*** *Rotate layer by layer, moving four cells at a time in a cycle (top → right → bottom → left). Also O(n²) time and O(1) space with a single pass, but the index math is more error-prone. Counter-clockwise is transpose + reverse each column.*

☐ Practiced   ☐ Revised

## **6. Binary Search**

**Goal:** O(n) → O(log n)

**Core idea:** *Repeatedly halve the search space on sorted (or monotonic) data; also used for 'search on the answer' problems.*

**Recognize it when:**
- The input is sorted, or sorted then rotated.
- You can ask a yes/no question `feasible(x)` that flips exactly once as x grows ("can Koko finish at speed x?"). Then binary search on x itself ("binary search on the answer").
- The constraints are huge (values up to 10⁹), which rules out scanning every candidate.

**Connects to:** Needs sorted data from §7. BSTs (§11) are binary search stored as a tree. *Longest Increasing Subsequence* (§21.12) uses binary search inside a DP. *Minimum Size Subarray Sum* (§3.8) has a binary-search alternative.

**Template:**

```js
// Find the first x in [lo, hi] where feasible(x) is true
function firstTrue(lo, hi, feasible) {
  while (lo < hi) {
    const mid = lo + Math.floor((hi - lo) / 2);
    if (feasible(mid)) hi = mid; // mid might be the answer
    else lo = mid + 1;           // mid is definitely not
  }
  return lo;
}
```

**Pitfalls:**
- Pick one loop style and stick to it: `lo <= hi` with `hi = mid - 1` (exact match), or `lo < hi` with `hi = mid` (first true). Mixing them causes infinite loops.
- With `lo < hi`, always move `lo` to `mid + 1`; `lo = mid` loops forever when `hi = lo + 1`.
- Search-on-answer bounds must actually contain the answer (e.g., capacity from `max(weights)` to `sum(weights)`).

### **6.1  Binary Search (classic)**

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

### **6.2  Search Insert Position**

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

### **6.3  Find First and Last Position of Element in Sorted Array**

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

### **6.4  Search a 2D Matrix**

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

### **6.5  Find Minimum in Rotated Sorted Array**

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

### **6.6  Search in Rotated Sorted Array**

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

### **6.7  Find Peak Element**

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

### **6.8  Koko Eating Bananas**

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

### **6.9  Capacity To Ship Packages Within D Days**

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

### **6.10  Median of Two Sorted Arrays**

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

## **7. Sorting & Intervals**

**Goal:** Simplifies logic (O(n log n) sort, then an O(n) scan)

**Core idea:** *Sorting first often turns a hard problem into an easy scan/two-pointer problem; know the sort's complexity trade-offs.*

**Recognize it when:**
- Intervals, meetings, ranges, or events on a timeline.
- The order of the input doesn't matter for the answer, but a sorted order makes neighbours meaningful.
- A custom ordering defines the answer (Largest Number, Custom Sort String).

**Connects to:** Sorting is the setup step for Two Pointers (§2), Binary Search (§6) and most Greedy algorithms (§20). Interval-scheduling greedy (*Non-overlapping Intervals* (§7.5)) is the classic exchange-argument proof. Meeting Rooms II can also be solved with a heap (§13).

**Sorting facts to know:**
- JS `Array.prototype.sort` is stable (since ES2019) and O(n log n) (TimSort in V8), but compares as **strings** unless you pass a comparator.
- Merge sort: O(n log n) always, O(n) extra space, stable. Quick sort: O(n log n) average, O(n²) worst, in place. Counting/bucket sort: O(n + range) when values are small integers (*Top K Frequent Elements* (§1.5)).
- Quickselect finds the k-th element in O(n) average without fully sorting (*Kth Largest Element in an Array* (§13.2)).

**Template:**

```js
// Merge overlapping intervals
function mergeAll(intervals) {
  intervals.sort((a, b) => a[0] - b[0]);       // sort by start
  const merged = [];
  for (const [start, end] of intervals) {
    const last = merged[merged.length - 1];
    if (last && start <= last[1]) last[1] = Math.max(last[1], end);
    else merged.push([start, end]);
  }
  return merged;
}
```

**Pitfalls:**
- Sort by **start** to merge; sort by **end** to pick the maximum number of non-overlapping intervals.
- Decide whether touching intervals (`[1,2]` and `[2,3]`) overlap. Problems differ.
- Sorting mutates the input array. Copy it first if the caller needs the original.

### **7.1  Largest Number**

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

### **7.2  Custom Sort String**

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

### **7.3  Merge Intervals**

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

### **7.4  Insert Interval**

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

### **7.5  Non-overlapping Intervals**

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

### **7.6  Meeting Rooms**

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

### **7.7  Meeting Rooms II**

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

# **Part II — Linear Data Structures**

## **8. Stack & Monotonic Stack**

**Goal:** O(n²) → O(n)

**Core idea:** *LIFO structure ideal for matching pairs, nested structures, and 'next greater/smaller element' style scans (monotonic stack).*

**Recognize it when:**
- Brackets, nested encodings (`3[a2[c]]`), expression evaluation, undo/backspace.
- "Next/previous greater or smaller element", "how many days until warmer", largest rectangle.
- You need the most recent unmatched item.

**Connects to:** The monotonic deque in *Sliding Window Maximum* (§3.12) is the same idea with removal from the front. Recursion uses an implicit stack, so iterative DFS (§15) and iterative in-order traversal (*Kth Smallest Element in a BST* (§11.2)) use an explicit one.

**Template:**

```js
// Monotonic decreasing stack: next greater element for every index
function nextGreater(nums) {
  const result = new Array(nums.length).fill(-1);
  const stack = []; // indices whose answer is still unknown
  for (let i = 0; i < nums.length; i++) {
    while (stack.length && nums[stack[stack.length - 1]] < nums[i]) {
      result[stack.pop()] = nums[i]; // nums[i] is their next greater
    }
    stack.push(i);
  }
  return result;
}
```

**Pitfalls:**
- Store **indices** on the stack when you need distances or widths; store values only when that's enough.
- Decreasing stack → next *greater*; increasing stack → next *smaller*. Strict vs non-strict comparison decides how duplicates are handled.
- Process leftovers at the end: a sentinel (e.g., height 0 in Largest Rectangle) flushes the stack.

### **8.1  Valid Parentheses**

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

### **8.2  Min Stack**

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

### **8.3  Evaluate Reverse Polish Notation**

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

### **8.4  Decode String**

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

### **8.5  Basic Calculator II**

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

### **8.6  Asteroid Collision**

**Problem:** Given an array representing asteroids (sign = direction, magnitude = size), simulate collisions and return the final state.

**Approach:** Keep a stack of surviving asteroids. A collision happens only when a new **left-moving** (negative) asteroid meets **right-moving** (positive) asteroids on top of the stack. While that is the case, compare sizes: pop a smaller right-mover and keep going; if they're equal, both explode; if the right-mover is bigger, the new asteroid explodes. Push the new asteroid only if it survives.

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

### **8.7  Next Greater Element I**

**Problem:** Given `nums1`, a subset of the distinct array `nums2`, return for each value in nums1 the first greater value to its right in nums2 (or -1).

**Approach:** Run a monotonic decreasing stack over nums2 once, recording every value's next greater value in a hash map. Then answer each query from nums1 with a lookup.

**Optimal Solution (JavaScript):**

```js
function nextGreaterElement(nums1, nums2) {
  const nextGreater = new Map(); // value -> next greater value in nums2
  const stack = [];              // values, decreasing
  for (const n of nums2) {
    while (stack.length && stack[stack.length - 1] < n) {
      nextGreater.set(stack.pop(), n);
    }
    stack.push(n);
  }
  return nums1.map(n => nextGreater.get(n) ?? -1);
}
```

**Complexity:** Time O(m + n), Space O(n)

***Alternate approaches:*** *For each value in nums1, find it in nums2 and scan right for a bigger one. That is O(m · n) time, O(1) extra space, and fine for tiny inputs.*

☐ Practiced   ☐ Revised

### **8.8  Next Greater Element II (circular)**

**Problem:** Given a circular array (the element after the last one is the first), find for each element the next greater element, or -1 if none. The non-circular version is the previous problem, Next Greater Element I.

**Approach:** Use a monotonic decreasing stack of indices. Scan the array twice, using modulo, so elements near the end can see elements at the start; while the current value is greater than the value at the stack's top index, pop and record the current value as that index's answer.

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

### **8.9  Daily Temperatures**

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

### **8.10  Largest Rectangle in Histogram**

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

### **8.11  Remove K Digits**

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

## **9. Linked List**

**Goal:** O(n) time, O(1) extra space, by re-wiring pointers instead of copying

**Core idea:** *Manipulate `next` pointers directly; use a dummy head to avoid special cases and fast/slow pointers to find positions in one pass.*

**Recognize it when:**
- The input is a linked list (you can't index into it or know its length up front).
- You need the middle, a cycle, or the n-th node from the end in one pass → fast & slow pointers.
- You need to reverse or reorder all or part of the list.

**Connects to:** Fast & slow pointers are Two Pointers (§2) on a list. Merging sorted lists is the merge step of merge sort (§7), and merging k lists needs a heap (*Merge K Sorted Lists* (§13.4)). Copying a list with random pointers is the same idea as *Clone Graph* (§15.3). LRU Cache combines a hash map (§1) with a doubly linked list.

**Template:**

```js
class ListNode {
  constructor(val = 0, next = null) {
    this.val = val;
    this.next = next;
  }
}

// Dummy head: no special case for an empty result or removing the first node
function buildList(values) {
  const dummy = new ListNode(0);
  let tail = dummy;
  for (const v of values) {
    tail.next = new ListNode(v);
    tail = tail.next;
  }
  return dummy.next;
}

// Fast & slow pointers
function middle(head) {
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  return slow; // second middle for even lengths
}
```

**Pitfalls:**
- Save `curr.next` *before* re-pointing it, or you lose the rest of the list.
- Always check `fast && fast.next` before `fast.next.next`.
- Draw the pointers for a 2- or 3-node list before coding; most bugs are off-by-one at the ends.

### **9.1  Reverse Linked List**

**Problem:** Reverse a singly linked list and return the new head.

**Approach:** Walk the list with `prev` and `curr`. For each node, save `next`, point `curr.next` back to `prev`, then advance both pointers. When `curr` is null, `prev` is the new head.

**Optimal Solution (JavaScript):**

```js
function reverseList(head) {
  let prev = null, curr = head;
  while (curr) {
    const next = curr.next; // save before re-pointing
    curr.next = prev;
    prev = curr;
    curr = next;
  }
  return prev;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Recursive: reverse the rest of the list, then set `head.next.next = head` and `head.next = null`. Same O(n) time but O(n) call-stack space.*

☐ Practiced   ☐ Revised

### **9.2  Merge Two Sorted Lists**

**Problem:** Merge two sorted linked lists into one sorted list by splicing their nodes together.

**Approach:** Use a dummy head and a `tail` pointer. Repeatedly attach the smaller of the two current nodes and advance that list. When one list runs out, attach the remainder of the other.

**Optimal Solution (JavaScript):**

```js
function mergeTwoLists(l1, l2) {
  const dummy = new ListNode(0);
  let tail = dummy;
  while (l1 && l2) {
    if (l1.val <= l2.val) { tail.next = l1; l1 = l1.next; }
    else { tail.next = l2; l2 = l2.next; }
    tail = tail.next;
  }
  tail.next = l1 || l2; // attach whatever is left
  return dummy.next;
}
```

**Complexity:** Time O(m + n), Space O(1)

***Alternate approaches:*** *Recursive: the smaller head's `next` becomes the merge of the rest. Elegant, but O(m + n) call-stack space.*

☐ Practiced   ☐ Revised

### **9.3  Middle of the Linked List**

**Problem:** Return the middle node of a linked list (the second middle if there are two).

**Approach:** Fast & slow pointers: `fast` moves two steps for every one step of `slow`. When fast reaches the end, slow is at the middle.

**Optimal Solution (JavaScript):**

```js
function middleNode(head) {
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  return slow;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Count the nodes in one pass, then walk n/2 steps in a second pass. Same complexity, two passes instead of one.*

☐ Practiced   ☐ Revised

### **9.4  Linked List Cycle I & II**

**Problem:** (I) Return true if the linked list has a cycle. (II) Return the node where the cycle begins, or null.

**Approach:** Floyd's algorithm: move slow by 1 and fast by 2; if they meet, there's a cycle. For the cycle start, reset one pointer to the head and move both by 1. They meet at the cycle entrance, because the distance from the head to the entrance equals the distance from the meeting point to the entrance (mod the cycle length).

**Optimal Solution (JavaScript):**

```js
function hasCycle(head) {
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }
  return false;
}

function detectCycle(head) { // Linked List Cycle II
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) {
      let p = head;
      while (p !== slow) { p = p.next; slow = slow.next; }
      return p; // cycle entrance
    }
  }
  return null;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Store visited nodes in a Set; the first node seen twice is the cycle start. O(n) time but O(n) space.*

☐ Practiced   ☐ Revised

### **9.5  Remove Nth Node From End of List**

**Problem:** Remove the n-th node from the end of a linked list in one pass and return the head.

**Approach:** Start two pointers at a dummy node and move `fast` n + 1 steps ahead. Then move both until fast is null; `slow` now sits just before the node to remove. The dummy handles removing the head.

**Optimal Solution (JavaScript):**

```js
function removeNthFromEnd(head, n) {
  const dummy = new ListNode(0, head);
  let fast = dummy, slow = dummy;
  for (let i = 0; i <= n; i++) fast = fast.next; // gap of n + 1 nodes
  while (fast) {
    fast = fast.next;
    slow = slow.next;
  }
  slow.next = slow.next.next;
  return dummy.next;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Two passes: count the length L, then remove the (L − n + 1)-th node from the front. Same O(n) time, simpler but not one pass.*

☐ Practiced   ☐ Revised

### **9.6  Reorder List**

**Problem:** Reorder L0 → L1 → … → Ln into L0 → Ln → L1 → Ln−1 → L2 → …, in place.

**Approach:** Three linked-list basics combined: (1) find the middle with fast & slow pointers, (2) reverse the second half, (3) merge the two halves by alternating nodes.

**Optimal Solution (JavaScript):**

```js
function reorderList(head) {
  if (!head || !head.next) return;

  // 1. find the end of the first half
  let slow = head, fast = head;
  while (fast.next && fast.next.next) {
    slow = slow.next;
    fast = fast.next.next;
  }

  // 2. reverse the second half
  let prev = null, curr = slow.next;
  slow.next = null;
  while (curr) {
    const next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }

  // 3. weave the halves together
  let first = head, second = prev;
  while (second) {
    const n1 = first.next, n2 = second.next;
    first.next = second;
    second.next = n1;
    first = n1;
    second = n2;
  }
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Copy the nodes into an array and relink them with two pointers from both ends. O(n) time and O(n) space, simpler to write.*

☐ Practiced   ☐ Revised

### **9.7  Add Two Numbers**

**Problem:** Two non-negative integers are stored as linked lists with digits in reverse order. Return their sum as a linked list in the same format.

**Approach:** Walk both lists together like grade-school addition, adding digits plus the carry. Continue while either list has nodes **or** a carry remains, appending `sum % 10` and carrying `floor(sum / 10)`.

**Optimal Solution (JavaScript):**

```js
function addTwoNumbers(l1, l2) {
  const dummy = new ListNode(0);
  let tail = dummy, carry = 0;
  while (l1 || l2 || carry) {
    const sum = (l1 ? l1.val : 0) + (l2 ? l2.val : 0) + carry;
    carry = Math.floor(sum / 10);
    tail.next = new ListNode(sum % 10);
    tail = tail.next;
    if (l1) l1 = l1.next;
    if (l2) l2 = l2.next;
  }
  return dummy.next;
}
```

**Complexity:** Time O(max(m, n)), Space O(max(m, n)) for the result

***Alternate approaches:*** *Converting to numbers and back fails for long lists (beyond 2⁵³ in JS); BigInt works but misses the point. If digits are stored in forward order (Add Two Numbers II), push them onto two stacks first.*

☐ Practiced   ☐ Revised

### **9.8  Copy List with Random Pointer**

**Problem:** Each node has `next` and `random` pointers (random may point anywhere or be null). Return a deep copy of the list.

**Approach:** First pass: create a copy of every node and store original → copy in a hash map (null maps to null). Second pass: set each copy's `next` and `random` by looking up the originals' targets in the map.

**Optimal Solution (JavaScript):**

```js
function copyRandomList(head) {
  const copies = new Map([[null, null]]);
  for (let n = head; n; n = n.next) {
    copies.set(n, { val: n.val, next: null, random: null });
  }
  for (let n = head; n; n = n.next) {
    const copy = copies.get(n);
    copy.next = copies.get(n.next);
    copy.random = copies.get(n.random);
  }
  return copies.get(head);
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *O(1) extra space: interleave copies into the list (A → A' → B → B'), set `A'.random = A.random.next`, then split the two lists apart. It's the same original → copy idea as Clone Graph, with the map stored in the list itself.*

☐ Practiced   ☐ Revised

### **9.9  LRU Cache**

**Problem:** Design a cache with fixed capacity supporting `get(key)` and `put(key, value)` in O(1). When full, evict the least recently used key.

**Approach:** Combine a hash map (key → node) for O(1) lookup with a doubly linked list ordered by recency (most recent next to `head`, least recent next to `tail`). On every access, move the node to the front; on overflow, remove the node before `tail`. Sentinel head/tail nodes remove edge cases.

**Optimal Solution (JavaScript):**

```js
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.map = new Map(); // key -> node
    this.head = { prev: null, next: null }; // most recent side
    this.tail = { prev: null, next: null }; // least recent side
    this.head.next = this.tail;
    this.tail.prev = this.head;
  }
  _remove(node) {
    node.prev.next = node.next;
    node.next.prev = node.prev;
  }
  _addFront(node) {
    node.prev = this.head;
    node.next = this.head.next;
    this.head.next.prev = node;
    this.head.next = node;
  }
  get(key) {
    if (!this.map.has(key)) return -1;
    const node = this.map.get(key);
    this._remove(node);
    this._addFront(node);
    return node.val;
  }
  put(key, value) {
    if (this.map.has(key)) this._remove(this.map.get(key));
    const node = { key, val: value, prev: null, next: null };
    this.map.set(key, node);
    this._addFront(node);
    if (this.map.size > this.capacity) {
      const lru = this.tail.prev;
      this._remove(lru);
      this.map.delete(lru.key);
    }
  }
}
```

**Complexity:** Time O(1) per operation, Space O(capacity)

***Alternate approaches:*** *JS-specific shortcut: a `Map` iterates in insertion order, so on access `delete` then `set` the key to move it to the end, and evict `map.keys().next().value` (the oldest). It is also O(1) and very short, but interviewers usually want the hash map + doubly linked list design shown above.*

☐ Practiced   ☐ Revised

# **Part III — Trees & Heaps**

## **10. Binary Trees**

**Goal:** O(n): visit each node once

**Core idea:** *Solve the problem for the left and right subtrees recursively, then combine the results at the current node (DFS), or process level by level with a queue (BFS).*

**Recognize it when:**
- Height, depth, diameter, balance, path sums: a value that is built **bottom-up** from children → post-order DFS that returns something.
- A constraint passed **top-down** from parent to child (remaining sum, valid range) → pass it as a parameter.
- Anything "per level" (level order, right side view, zigzag) → BFS.

**Connects to:** Recursion here is the foundation for graph DFS (§15), backtracking (§19) and memoized DP (§21). Level-order BFS becomes grid/graph BFS (§14). BSTs (§11) add ordering; tries (§12) and heaps (§13) are specialized trees.

**Template:**

```js
class TreeNode {
  constructor(val = 0, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

// DFS: return information up; update a global answer at each node
function bottomUp(root) {
  let best = 0;
  const dfs = (node) => {
    if (!node) return 0;                    // base case
    const left = dfs(node.left);
    const right = dfs(node.right);
    best = Math.max(best, left + right);   // answer that passes through node
    return 1 + Math.max(left, right);       // what the parent needs
  };
  dfs(root);
  return best;
}

// BFS: one array per level (avoids queue.shift())
function levels(root) {
  const result = [];
  let level = root ? [root] : [];
  while (level.length) {
    result.push(level.map((n) => n.val));
    level = level.flatMap((n) => [n.left, n.right].filter(Boolean));
  }
  return result;
}
```

**Pitfalls:**
- Separate "what I return to my parent" from "the global answer" (Diameter, Max Path Sum).
- A leaf is `!node.left && !node.right`, not `!node`. Path problems usually need real leaves.
- Recursion depth equals tree height; a skewed tree of 10⁵ nodes can overflow the JS call stack. Mention the iterative version.

### **10.1  Maximum Depth of Binary Tree (Tree Height)**

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

### **10.2  Same Tree**

**Problem:** Given the roots of two binary trees, return true if they are structurally identical with equal values.

**Approach:** Recurse on both trees in lockstep. Two empty nodes match; one empty or different values don't; otherwise both subtrees must match.

**Optimal Solution (JavaScript):**

```js
function isSameTree(p, q) {
  if (!p && !q) return true;
  if (!p || !q || p.val !== q.val) return false;
  return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
}
```

**Complexity:** Time O(n), Space O(h)

***Alternate approaches:*** *Iterative BFS with a queue of node pairs, comparing each pair. Same complexity, no recursion.*

☐ Practiced   ☐ Revised

### **10.3  Invert Binary Tree**

**Problem:** Mirror a binary tree (swap every node's left and right children) and return its root.

**Approach:** Recursively invert both subtrees, then swap them at the current node.

**Optimal Solution (JavaScript):**

```js
function invertTree(root) {
  if (!root) return null;
  [root.left, root.right] = [invertTree(root.right), invertTree(root.left)];
  return root;
}
```

**Complexity:** Time O(n), Space O(h)

***Alternate approaches:*** *BFS level by level, swapping the children of each node as it's visited. Same O(n) time, O(width) space.*

☐ Practiced   ☐ Revised

### **10.4  Balanced Binary Tree**

**Problem:** Determine if a binary tree is height-balanced: at every node, the left and right subtree heights differ by at most 1.

**Approach:** Post-order DFS that returns the subtree height, or -1 as soon as any subtree is unbalanced. The -1 propagates up, so each node is visited once.

**Optimal Solution (JavaScript):**

```js
function isBalanced(root) {
  const height = (node) => { // -1 means "unbalanced"
    if (!node) return 0;
    const l = height(node.left);
    if (l === -1) return -1;
    const r = height(node.right);
    if (r === -1 || Math.abs(l - r) > 1) return -1;
    return 1 + Math.max(l, r);
  };
  return height(root) !== -1;
}
```

**Complexity:** Time O(n), Space O(h)

***Alternate approaches:*** *Top-down: at each node compute both heights with a separate function and recurse. Correct but O(n²) on skewed trees, because heights are recomputed. It's a good example of why returning information bottom-up matters.*

☐ Practiced   ☐ Revised

### **10.5  Diameter of Binary Tree**

**Problem:** Return the length (in edges) of the longest path between any two nodes in a binary tree.

**Approach:** The longest path through a node is `leftHeight + rightHeight`. DFS returns each subtree's height to its parent while updating a global best with the path that bends at the current node.

**Optimal Solution (JavaScript):**

```js
function diameterOfBinaryTree(root) {
  let best = 0;
  const height = (node) => {
    if (!node) return 0;
    const l = height(node.left), r = height(node.right);
    best = Math.max(best, l + r); // path bending at this node
    return 1 + Math.max(l, r);    // what the parent can extend
  };
  height(root);
  return best;
}
```

**Complexity:** Time O(n), Space O(h)

***Alternate approaches:*** *Compute heights separately for every node: O(n²). The same "return one thing, record another" pattern solves Binary Tree Maximum Path Sum.*

☐ Practiced   ☐ Revised

### **10.6  Subtree of Another Tree**

**Problem:** Return true if `subRoot` appears in `root` as a subtree (a node plus all its descendants).

**Approach:** For every node in root, check whether the tree starting there is the same tree as subRoot (reuse Same Tree).

**Optimal Solution (JavaScript):**

```js
function isSubtree(root, subRoot) {
  const same = (a, b) => {
    if (!a && !b) return true;
    if (!a || !b || a.val !== b.val) return false;
    return same(a.left, b.left) && same(a.right, b.right);
  };
  if (!root) return false;
  return same(root, subRoot) || isSubtree(root.left, subRoot) || isSubtree(root.right, subRoot);
}
```

**Complexity:** Time O(m · n), Space O(h)

***Alternate approaches:*** *Serialize both trees with null markers and delimiters (e.g., ",1,#,#"), then check whether one string contains the other using KMP. That is O(m + n) time. The markers are essential, or different shapes can produce the same string.*

☐ Practiced   ☐ Revised

### **10.7  Path Sum**

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

### **10.8  Path Sum II**

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

### **10.9  Binary Tree Maximum Path Sum**

**Problem:** A path is any sequence of connected nodes (each used once, not necessarily through the root). Return the maximum path sum. Values can be negative.

**Approach:** Like Diameter: DFS returns the best **downward** path starting at a node, which is the only shape a parent can extend. At each node, the best path bending there is `val + leftGain + rightGain`, where negative gains are dropped (replaced by 0).

**Optimal Solution (JavaScript):**

```js
function maxPathSum(root) {
  let best = -Infinity;
  const gain = (node) => { // best downward path starting at node
    if (!node) return 0;
    const l = Math.max(0, gain(node.left));  // drop negative branches
    const r = Math.max(0, gain(node.right));
    best = Math.max(best, node.val + l + r); // path bending at node
    return node.val + Math.max(l, r);
  };
  gain(root);
  return best;
}
```

**Complexity:** Time O(n), Space O(h)

***Alternate approaches:*** *Brute force: treat every pair of nodes as path endpoints and sum the path between them. O(n²) or worse, impractical. Start `best` at -Infinity, not 0, because all values may be negative.*

☐ Practiced   ☐ Revised

### **10.10  Lowest Common Ancestor of a Binary Tree**

**Problem:** Given a binary tree (not necessarily a BST) and two nodes p and q in it, return their lowest common ancestor.

**Approach:** DFS returns p or q if found in a subtree, otherwise null. If a node gets non-null results from **both** sides, p and q are split across it, so it's the LCA. If the node itself is p or q, return it, since it is the ancestor of the other or the other is elsewhere.

**Optimal Solution (JavaScript):**

```js
function lowestCommonAncestor(root, p, q) {
  if (!root || root === p || root === q) return root;
  const left = lowestCommonAncestor(root.left, p, q);
  const right = lowestCommonAncestor(root.right, p, q);
  if (left && right) return root; // p and q on different sides
  return left || right;
}
```

**Complexity:** Time O(n), Space O(h)

***Alternate approaches:*** *Record each node's parent with a traversal, collect p's ancestors in a Set, then walk up from q until hitting one. O(n) time and O(n) space. For a BST, use values to go left/right instead (next chapter).*

☐ Practiced   ☐ Revised

### **10.11  Binary Tree Level Order Traversal**

**Problem:** Given a binary tree, return the values of nodes level by level (left to right).

**Approach:** Process the tree one level at a time: keep the current level in an array, record its values, and build the next level from their children. A fresh array per level is the JavaScript-friendly BFS queue, since `queue.shift()` is O(n).

**Optimal Solution (JavaScript):**

```js
function levelOrder(root) {
  const result = [];
  let level = root ? [root] : [];
  while (level.length) {
    result.push(level.map(node => node.val));
    const next = [];
    for (const node of level) {
      if (node.left) next.push(node.left);
      if (node.right) next.push(node.right);
    }
    level = next;
  }
  return result;
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Recursive DFS that tracks depth and appends each value into result[depth] — same O(n) time, avoids an explicit queue but relies on the call stack instead.*

☐ Practiced   ☐ Revised

### **10.12  Binary Tree Right Side View**

**Problem:** Return the values of the nodes visible from the right side of a binary tree, top to bottom.

**Approach:** BFS level by level; the last node of each level is the visible one.

**Optimal Solution (JavaScript):**

```js
function rightSideView(root) {
  const result = [];
  let level = root ? [root] : [];
  while (level.length) {
    result.push(level[level.length - 1].val);
    const next = [];
    for (const node of level) {
      if (node.left) next.push(node.left);
      if (node.right) next.push(node.right);
    }
    level = next;
  }
  return result;
}
```

**Complexity:** Time O(n), Space O(width)

***Alternate approaches:*** *DFS visiting right before left, recording a node's value the first time a new depth is reached (`if (depth === result.length) result.push(val)`). Same O(n) time, O(h) space.*

☐ Practiced   ☐ Revised

### **10.13  Construct Binary Tree from Preorder and Inorder Traversal**

**Problem:** Given the preorder and inorder traversals of a binary tree with unique values, rebuild the tree.

**Approach:** The next preorder value is the root of the current subtree. Its position in inorder splits the remaining values into left and right subtrees. Map value → inorder index for O(1) lookups, and consume preorder with a running pointer, building the left subtree before the right.

**Optimal Solution (JavaScript):**

```js
function buildTree(preorder, inorder) {
  const inIndex = new Map(inorder.map((v, i) => [v, i]));
  let pre = 0;
  const build = (lo, hi) => { // subtree made of inorder[lo..hi]
    if (lo > hi) return null;
    const val = preorder[pre++];
    const node = new TreeNode(val);
    const mid = inIndex.get(val);
    node.left = build(lo, mid - 1);
    node.right = build(mid + 1, hi);
    return node;
  };
  return build(0, inorder.length - 1);
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Slice the arrays at each step and search inorder with indexOf. It's easier to read but O(n²) time and memory because of the copying and linear searches.*

☐ Practiced   ☐ Revised

### **10.14  Serialize and Deserialize Binary Tree**

**Problem:** Design functions that convert a binary tree to a string and back, so the same tree is reconstructed.

**Approach:** Preorder DFS that writes `#` for null children. Because the nulls are explicit, the preorder sequence alone fixes the shape. Deserialize by reading tokens in the same preorder: a value creates a node and recursively builds its left then right child; `#` returns null.

**Optimal Solution (JavaScript):**

```js
function serialize(root) {
  const out = [];
  const dfs = (node) => {
    if (!node) { out.push('#'); return; }
    out.push(node.val);
    dfs(node.left);
    dfs(node.right);
  };
  dfs(root);
  return out.join(',');
}

function deserialize(data) {
  const tokens = data.split(',');
  let i = 0;
  const build = () => {
    const t = tokens[i++];
    if (t === '#') return null;
    const node = new TreeNode(Number(t));
    node.left = build();
    node.right = build();
    return node;
  };
  return build();
}
```

**Complexity:** Time O(n), Space O(n)

***Alternate approaches:*** *Level-order (BFS) serialization with null markers, the format LeetCode uses for its examples. Same complexity; deserializing processes children pairwise with a queue.*

☐ Practiced   ☐ Revised

## **11. Binary Search Trees**

**Goal:** O(h) search/insert/delete, where h = log n when balanced

**Core idea:** *Every node is greater than everything in its left subtree and smaller than everything in its right subtree, so an in-order traversal yields sorted order and each comparison discards one side.*

**Recognize it when:**
- The tree is a BST and you need sorted order, the k-th smallest, a range, or a successor.
- You can decide to go left or right from a comparison (BST LCA, search, insert).

**Connects to:** Binary search (§6) stored as a tree. Validation passes a (min, max) range top-down, like Path Sum (§10). The iterative in-order traversal uses an explicit stack (§8).

**Template:**

```js
// Iterative in-order traversal: visits BST values in sorted order
function inorder(root, visit) {
  const stack = [];
  let node = root;
  while (node || stack.length) {
    while (node) { stack.push(node); node = node.left; }
    node = stack.pop();
    visit(node.val);
    node = node.right;
  }
}
```

**Pitfalls:**
- Checking only `left.val < node.val < right.val` is wrong; every node in a subtree must respect *all* ancestors. Pass bounds down.
- Clarify whether duplicates are allowed and on which side.
- O(log n) holds only for balanced trees; a sorted insertion order produces a linked list (O(n)).

### **11.1  Validate Binary Search Tree**

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

### **11.2  Kth Smallest Element in a BST**

**Problem:** Return the k-th smallest value (1-indexed) in a binary search tree.

**Approach:** An in-order traversal of a BST visits values in ascending order. Run it iteratively with an explicit stack and stop at the k-th visited node, without traversing the rest.

**Optimal Solution (JavaScript):**

```js
function kthSmallest(root, k) {
  const stack = [];
  let node = root;
  while (node || stack.length) {
    while (node) { stack.push(node); node = node.left; }
    node = stack.pop();
    if (--k === 0) return node.val;
    node = node.right;
  }
}
```

**Complexity:** Time O(h + k), Space O(h)

***Alternate approaches:*** *Recursive in-order collecting all values into an array, then return `arr[k - 1]`: O(n) time and space. If the tree changes often and kth queries are frequent, store subtree sizes in each node to answer in O(h).*

☐ Practiced   ☐ Revised

### **11.3  Lowest Common Ancestor of a Binary Search Tree**

**Problem:** Given a BST and two nodes p and q, return their lowest common ancestor.

**Approach:** Use BST ordering: if both values are smaller than the current node, the LCA is in the left subtree; if both are larger, it's in the right. Otherwise they split here (or one equals the current node), so this node is the LCA.

**Optimal Solution (JavaScript):**

```js
function lowestCommonAncestor(root, p, q) {
  let node = root;
  while (node) {
    if (p.val < node.val && q.val < node.val) node = node.left;
    else if (p.val > node.val && q.val > node.val) node = node.right;
    else return node;
  }
  return null;
}
```

**Complexity:** Time O(h), Space O(1)

***Alternate approaches:*** *The general binary-tree LCA (previous chapter) also works, but it is O(n) because it ignores the ordering.*

☐ Practiced   ☐ Revised

## **12. Tries**

**Goal:** O(L) insert/lookup per word of length L, independent of how many words are stored

**Core idea:** *A tree where each edge is a character and each path from the root spells a prefix; words sharing a prefix share nodes.*

**Recognize it when:**
- Prefix queries: autocomplete, "starts with", longest common prefix.
- Wildcard search over a dictionary.
- Searching for **many** words at once in a grid or text (one trie walk instead of one search per word).

**Connects to:** A specialized tree (§10) whose children are stored in a hash map (§1). Word Search II is DFS backtracking (§19) guided by a trie.

**Template:**

```js
function buildTrie(words) {
  const root = {};
  for (const word of words) {
    let node = root;
    for (const ch of word) node = node[ch] ??= {};
    node.isEnd = true;
  }
  return root;
}
```

**Pitfalls:**
- Mark the end of a word explicitly; a path existing doesn't mean the word was inserted ("app" vs "apple").
- With plain objects, guard against keys like `isEnd` colliding with characters (single characters are safe; multi-character keys are not).
- Memory is O(total characters). Mention it if the dictionary is huge.

### **12.1  Implement Trie (Prefix Tree)**

**Problem:** Implement a trie with `insert(word)`, `search(word)` (exact word present?) and `startsWith(prefix)` (any word with this prefix?).

**Approach:** Each node holds a map of child characters and an `isEnd` flag. Insert walks or creates one node per character and marks the last as an end. Search and startsWith share a walk that returns the node reached (or null); search additionally requires `isEnd`.

**Optimal Solution (JavaScript):**

```js
class Trie {
  constructor() {
    this.root = { children: new Map(), isEnd: false };
  }
  insert(word) {
    let node = this.root;
    for (const c of word) {
      if (!node.children.has(c)) node.children.set(c, { children: new Map(), isEnd: false });
      node = node.children.get(c);
    }
    node.isEnd = true;
  }
  _walk(prefix) {
    let node = this.root;
    for (const c of prefix) {
      node = node.children.get(c);
      if (!node) return null;
    }
    return node;
  }
  search(word) {
    const node = this._walk(word);
    return node !== null && node.isEnd;
  }
  startsWith(prefix) {
    return this._walk(prefix) !== null;
  }
}
```

**Complexity:** Time O(L) per operation for a word of length L, Space O(total characters inserted)

***Alternate approaches:*** *Store all words in a Set and check prefixes by scanning every word: search is O(L), but startsWith becomes O(n · L). For lowercase-only input, a fixed array of 26 children per node is slightly faster than a Map.*

☐ Practiced   ☐ Revised

### **12.2  Design Add and Search Words Data Structure**

**Problem:** Support `addWord(word)` and `search(word)`, where the search pattern may contain `.` matching any single letter.

**Approach:** Store words in a trie. Search with DFS: a normal letter follows one child; a `.` tries every child. A full match also requires the end-of-word flag.

**Optimal Solution (JavaScript):**

```js
class WordDictionary {
  constructor() {
    this.root = { children: new Map(), isEnd: false };
  }
  addWord(word) {
    let node = this.root;
    for (const c of word) {
      if (!node.children.has(c)) node.children.set(c, { children: new Map(), isEnd: false });
      node = node.children.get(c);
    }
    node.isEnd = true;
  }
  search(word) {
    const dfs = (node, i) => {
      if (i === word.length) return node.isEnd;
      if (word[i] === '.') {
        for (const child of node.children.values()) {
          if (dfs(child, i + 1)) return true;
        }
        return false;
      }
      const next = node.children.get(word[i]);
      return next ? dfs(next, i + 1) : false;
    };
    return dfs(this.root, 0);
  }
}
```

**Complexity:** Time O(L) for addWord; search is O(L) without dots and up to O(26^d · L) with d dots in the worst case. Space O(total characters)

***Alternate approaches:*** *Group words by length in a Map of arrays and compare the pattern character by character against each word of that length. Simple, but O(n · L) per search.*

☐ Practiced   ☐ Revised

### **12.3  Word Search II**

**Problem:** Given a letter grid and a list of words, return all words that can be formed by adjacent cells (no cell reused within a word).

**Approach:** Put all words in a trie, then DFS from every cell, walking the trie alongside the grid. A path that leaves the trie stops immediately, so all words are searched at once with shared prefixes. Store the full word at its end node and clear it once found, to avoid duplicates.

**Optimal Solution (JavaScript):**

```js
function findWords(board, words) {
  const root = new Map();
  for (const w of words) {
    let node = root;
    for (const c of w) {
      if (!node.has(c)) node.set(c, new Map());
      node = node.get(c);
    }
    node.word = w; // full word stored at its end node
  }

  const rows = board.length, cols = board[0].length;
  const result = [];
  const dfs = (r, c, parent) => {
    const ch = board[r][c];
    const node = parent.get(ch);
    if (!node) return;                       // no word continues this way
    if (node.word) { result.push(node.word); node.word = null; }
    board[r][c] = '#';                       // mark visited
    for (const [dr, dc] of [[1, 0], [-1, 0], [0, 1], [0, -1]]) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && board[nr][nc] !== '#') {
        dfs(nr, nc, node);
      }
    }
    board[r][c] = ch;                        // backtrack
  };

  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++) dfs(r, c, root);
  return result;
}
```

**Complexity:** Time O(rows · cols · 4 · 3^(L−1)) worst case for max word length L, Space O(total characters in words)

***Alternate approaches:*** *Run Word Search (single word) once per word: O(words · rows · cols · 3^L), far slower when there are many words. Extra optimization: remove trie leaves once their word is found so later searches skip dead branches.*

☐ Practiced   ☐ Revised

## **13. Heaps / Priority Queue**

**Goal:** O(log n) insert and remove-min/max, O(1) peek

**Core idea:** *A complete binary tree stored in an array where every parent is ≤ its children (min-heap), so the smallest item is always at index 0.*

**Recognize it when:**
- "k-th largest/smallest", "top k", "k closest": keep a heap of size k.
- Merge k sorted sources: the heap holds the current head of each.
- Running median: two heaps (max-heap for the lower half, min-heap for the upper half).
- Always process the cheapest/earliest next item (scheduling, Dijkstra).

**Connects to:** Faster than sorting (§7) when you only need the top k: O(n log k) vs O(n log n). Merge K Sorted Lists extends the linked-list merge (§9). The heap is what turns BFS into Dijkstra and powers Prim (§18).

**Index math:** parent of `i` is `(i - 1) >> 1`; children are `2i + 1` and `2i + 2`. JavaScript has no built-in heap, so be ready to write the class in *Implement a Min-Heap (Priority Queue)* (§13.1) in about 5 minutes. A max-heap is the same class with a reversed comparator.

**Template:**

```js
// Keep the k largest values seen so far: min-heap of size k
// (uses the MinHeap class from the first problem in this chapter)
function kLargest(nums, k, MinHeap) {
  const heap = new MinHeap();
  for (const n of nums) {
    heap.push(n);
    if (heap.size() > k) heap.pop(); // evict the smallest
  }
  return heap.data; // the k largest, in heap order
}
```

**Pitfalls:**
- For the k **largest**, use a **min**-heap of size k (evict the smallest), and vice versa.
- Comparator sign: `(a, b) => a - b` is a min-heap; `(a, b) => b - a` is a max-heap.
- Building a heap by pushing n items is O(n log n). Mention bottom-up heapify (O(n)) if asked.

### **13.1  Implement a Min-Heap (Priority Queue)**

**Problem:** JavaScript has no built-in priority queue. Implement a heap with `push`, `pop` (remove the top), `peek` and `size`, ordered by a comparator, so it works as a min-heap, a max-heap, or a heap of objects.

**Approach:** Store the complete binary tree in an array. `push` appends at the end and **sifts up** (swap with the parent while it has higher priority). `pop` takes the root, moves the last element to the root and **sifts down** (swap with the higher-priority child until both children are lower priority).

**Optimal Solution (JavaScript):**

```js
class MinHeap {
  constructor(compare = (a, b) => a - b) {
    this.data = [];
    this.compare = compare; // compare(a, b) < 0 means a comes out first
  }
  size() { return this.data.length; }
  peek() { return this.data[0]; }
  push(val) {
    const d = this.data;
    d.push(val);
    let i = d.length - 1;
    while (i > 0) {
      const parent = (i - 1) >> 1;
      if (this.compare(d[i], d[parent]) >= 0) break;
      [d[i], d[parent]] = [d[parent], d[i]];
      i = parent;
    }
  }
  pop() {
    const d = this.data;
    if (d.length === 0) return undefined;
    const top = d[0];
    const last = d.pop();
    if (d.length) {
      d[0] = last;
      let i = 0;
      while (true) {
        const l = 2 * i + 1, r = l + 1;
        let best = i;
        if (l < d.length && this.compare(d[l], d[best]) < 0) best = l;
        if (r < d.length && this.compare(d[r], d[best]) < 0) best = r;
        if (best === i) break;
        [d[i], d[best]] = [d[best], d[i]];
        i = best;
      }
    }
    return top;
  }
}
// Max-heap: new MinHeap((a, b) => b - a)
// By field:  new MinHeap((a, b) => a.dist - b.dist)
```

**Complexity:** Time O(log n) push/pop, O(1) peek and size, Space O(n)

***Alternate approaches:*** *A sorted array gives O(1) pop but O(n) insert. Re-sorting after every push is O(n log n) per operation. Neither is acceptable when the interviewer expects a heap. Building a heap from n known items bottom-up (sift down from the last parent to the root) is O(n) instead of O(n log n).*

☐ Practiced   ☐ Revised

### **13.2  Kth Largest Element in an Array**

**Problem:** Given an array and integer k, find the kth largest element (not necessarily distinct).

**Approach:** Quickselect: partition around a random pivot into `< pivot | = pivot | > pivot`, the same three-way partition as Sort Colors (Dutch National Flag). The kth largest sits at index n − k of the sorted order, so keep only the region containing that index and repeat. Unlike quicksort, only one side is processed, which gives O(n) on average, and the three-way split keeps arrays with many duplicates fast.

**Optimal Solution (JavaScript):**

```js
function findKthLargest(nums, k) {
  const target = nums.length - k; // index in ascending order
  let lo = 0, hi = nums.length - 1;
  while (true) {
    const pivot = nums[lo + Math.floor(Math.random() * (hi - lo + 1))];
    let lt = lo, i = lo, gt = hi; // [lo..lt) < pivot, [lt..i) = pivot, (gt..hi] > pivot
    while (i <= gt) {
      if (nums[i] < pivot) { [nums[lt], nums[i]] = [nums[i], nums[lt]]; lt++; i++; }
      else if (nums[i] > pivot) { [nums[i], nums[gt]] = [nums[gt], nums[i]]; gt--; }
      else i++;
    }
    if (target < lt) hi = lt - 1;
    else if (target > gt) lo = gt + 1;
    else return pivot;
  }
}
```

**Heap version (JavaScript):**

```js
function findKthLargestHeap(nums, k) {
  const heap = new MinHeap(); // holds the k largest values seen so far
  for (const n of nums) {
    heap.push(n);
    if (heap.size() > k) heap.pop(); // evict the smallest
  }
  return heap.peek();
}
```

**Complexity:** Quickselect: Time O(n) average, O(n²) worst case (very unlikely with a random pivot), Space O(1). Heap: Time O(n log k), Space O(k)

***Alternate approaches:*** *Sort and index: `nums.sort((a, b) => b - a)[k - 1]`, O(n log n). It's the simplest to write and a fine opening answer, but interviewers will ask for one of the faster versions above. Prefer the heap when data streams in or k is small; prefer quickselect when the whole array is in memory and may be reordered.*

☐ Practiced   ☐ Revised

### **13.3  K Closest Points to Origin**

**Problem:** Given points on a plane and k, return the k points closest to the origin (any order).

**Approach:** Keep a **max**-heap of size k keyed by squared distance. Push each point; if the heap grows beyond k, pop the farthest. What remains are the k closest. Squared distance avoids `Math.sqrt` and preserves the order.

**Optimal Solution (JavaScript):**

```js
function kClosest(points, k) {
  const dist = ([x, y]) => x * x + y * y;
  const heap = new MinHeap((a, b) => dist(b) - dist(a)); // max-heap by distance
  for (const p of points) {
    heap.push(p);
    if (heap.size() > k) heap.pop(); // drop the farthest
  }
  return heap.data;
}
```

**Complexity:** Time O(n log k), Space O(k)

***Alternate approaches:*** *Sort all points by distance and take the first k: O(n log n), simplest to write. Quickselect on distance gives O(n) average time.*

☐ Practiced   ☐ Revised

### **13.4  Merge K Sorted Lists**

**Problem:** Merge k sorted linked lists into one sorted list.

**Approach:** Put the head of each list into a min-heap keyed by value. Repeatedly pop the smallest node, append it to the result, and push that node's successor. The heap never holds more than k nodes.

**Optimal Solution (JavaScript):**

```js
function mergeKLists(lists) {
  const heap = new MinHeap((a, b) => a.val - b.val);
  for (const node of lists) if (node) heap.push(node);

  const dummy = new ListNode(0);
  let tail = dummy;
  while (heap.size()) {
    const node = heap.pop();
    tail.next = node;
    tail = node;
    if (node.next) heap.push(node.next);
  }
  return dummy.next;
}
```

**Complexity:** Time O(N log k) for N total nodes, Space O(k)

***Alternate approaches:*** *Divide and conquer: merge lists in pairs (1+2, 3+4, …) with Merge Two Sorted Lists, halving the count each round. Also O(N log k) time with O(1) extra space (iteratively). Merging one list at a time into a growing result is O(N · k).*

☐ Practiced   ☐ Revised

### **13.5  Task Scheduler**

**Problem:** Given CPU tasks (letters) and a cooldown n, identical tasks must be at least n time units apart. Each unit runs one task or idles. Return the minimum total time.

**Approach:** Greedy with a max-heap of remaining counts: at each time step, run the task with the most remaining copies, since it is the hardest to fit. A finished task waits in a FIFO cooldown queue with the time it becomes available again, then returns to the heap. When the heap is empty but tasks are cooling, jump time forward to the next release (idling).

**Optimal Solution (JavaScript):**

```js
function leastInterval(tasks, n) {
  const freq = new Map();
  for (const t of tasks) freq.set(t, (freq.get(t) || 0) + 1);
  const heap = new MinHeap((a, b) => b - a); // max-heap of remaining counts
  for (const f of freq.values()) heap.push(f);

  const cooling = []; // [remainingCount, readyAfterTime], in FIFO order
  let head = 0, time = 0;
  while (heap.size() || head < cooling.length) {
    time++;
    if (heap.size()) {
      const left = heap.pop() - 1;
      if (left > 0) cooling.push([left, time + n]);
    } else {
      time = cooling[head][1]; // idle until the next task is ready
    }
    if (head < cooling.length && cooling[head][1] === time) {
      heap.push(cooling[head++][0]);
    }
  }
  return time;
}
```

**Complexity:** Time O(T log 26) ≈ O(T) for T tasks, Space O(26)

***Alternate approaches:*** *Counting formula in O(T): with maxFreq the highest count and maxCount the number of tasks having it, the answer is `Math.max(tasks.length, (maxFreq - 1) * (n + 1) + maxCount)`. The most frequent tasks form maxFreq − 1 frames of length n + 1, plus a last partial frame. It's faster but harder to justify on the spot.*

☐ Practiced   ☐ Revised

### **13.6  Find Median from Data Stream**

**Problem:** Design a structure with `addNum(num)` and `findMedian()` for a stream of numbers.

**Approach:** Two heaps: a max-heap `low` for the smaller half and a min-heap `high` for the larger half. Keep every value in low ≤ every value in high, and low's size equal to high's or one more. The median is low's top, or the average of both tops.

**Optimal Solution (JavaScript):**

```js
class MedianFinder {
  constructor() {
    this.low = new MinHeap((a, b) => b - a); // max-heap: smaller half
    this.high = new MinHeap();               // min-heap: larger half
  }
  addNum(num) {
    this.low.push(num);
    this.high.push(this.low.pop());           // largest of the low half moves up
    if (this.high.size() > this.low.size()) { // rebalance: low keeps the extra one
      this.low.push(this.high.pop());
    }
  }
  findMedian() {
    if (this.low.size() > this.high.size()) return this.low.peek();
    return (this.low.peek() + this.high.peek()) / 2;
  }
}
```

**Complexity:** Time O(log n) addNum, O(1) findMedian, Space O(n)

***Alternate approaches:*** *Keep a sorted array with binary-search insertion: O(log n) to find the spot but O(n) to shift elements, with O(1) median. Fine for small streams. If values are bounded (e.g., 0–100), keep counts per value and walk them.*

☐ Practiced   ☐ Revised

# **Part IV — Graphs**

## **14. Graph BFS**

**Goal:** Efficient traversal: O(V + E), and shortest paths in unweighted graphs

**Core idea:** *FIFO structure for level-by-level exploration; guarantees shortest path in unweighted graphs.*

**Recognize it when:**
- "Minimum number of steps/moves/minutes" where every move costs the same.
- Something spreads from one or **many** sources at once (rotting oranges, distance to nearest gate): seed the queue with all sources (multi-source BFS).
- The graph is implicit: states (lock combinations, words) are nodes and legal moves are edges.

**Connects to:** Generalizes tree level-order traversal (§10). With in-degrees it becomes Kahn's topological sort (§16); with a heap instead of a queue it becomes Dijkstra (§18). DFS (§15) also explores everything but does **not** find shortest paths.

**Template:**

```js
// Grid BFS from one or many sources; returns the distance to every cell
function bfsGrid(grid, sources) {
  const rows = grid.length, cols = grid[0].length;
  const dist = Array.from({ length: rows }, () => new Array(cols).fill(-1));
  const queue = [];
  for (const [r, c] of sources) { dist[r][c] = 0; queue.push([r, c]); }

  for (let head = 0; head < queue.length; head++) {
    const [r, c] = queue[head];
    for (const [dr, dc] of [[1, 0], [-1, 0], [0, 1], [0, -1]]) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < rows && nc >= 0 && nc < cols &&
          grid[nr][nc] !== 1 && dist[nr][nc] === -1) { // 1 = wall
        dist[nr][nc] = dist[r][c] + 1; // mark when ENQUEUED, not when dequeued
        queue.push([nr, nc]);
      }
    }
  }
  return dist;
}
```

**Pitfalls:**
- Mark nodes visited when you **enqueue** them; marking on dequeue lets the same node be queued many times.
- `queue.shift()` is O(n) for large arrays. Use a head index (as above) or process one level array at a time.
- BFS gives shortest paths only when all edges have equal weight; otherwise use Dijkstra.

### **14.1  Number of Islands (BFS version)**

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
 
    let head = 0;
    while (head < queue.length) {
      const [r, c] = queue[head++];
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

### **14.2  Shortest Path in Binary Matrix**

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
 
  let head = 0;
  while (head < queue.length) {
    const [r, c, dist] = queue[head++];
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

***Alternate approaches:*** *A\* search with the Chebyshev distance max(|Δr|, |Δc|) as the heuristic can prune the search space faster in practice, though the worst case stays the same. Chebyshev is the right heuristic because it never overestimates with 8-directional moves; Manhattan distance does overestimate when diagonals are allowed, so A\* could return a path that isn't the shortest.*

☐ Practiced   ☐ Revised

### **14.3  Rotting Oranges**

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
 
  let head = 0;
  while (head < queue.length && fresh > 0) {
    const levelEnd = queue.length; // everything queued so far rots this minute
    while (head < levelEnd) {
      const [r, c] = queue[head++];
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

### **14.4  Walls and Gates**

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
 
  let head = 0;
  while (head < queue.length) {
    const [r, c] = queue[head++];
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

### **14.5  01 Matrix**

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
  let head = 0;
  while (head < queue.length) {
    const [r, c] = queue[head++];
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

### **14.6  Open the Lock**

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
 
  let head = 0;
  while (head < queue.length) {
    const [state, moves] = queue[head++];
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

### **14.7  Word Ladder**

**Problem:** Given a start word, end word, and a word list, find the length of the shortest transformation sequence changing one letter at a time, with each intermediate word in the list.

**Approach:** BFS over words, where an edge connects two words differing by exactly one letter. Generate neighbors by trying every letter substitution at every position and checking membership in a word set (removing found words to prevent revisiting).

**Optimal Solution (JavaScript):**

```js
function ladderLength(beginWord, endWord, wordList) {
  const wordSet = new Set(wordList);
  if (!wordSet.has(endWord)) return 0;
 
  const queue = [[beginWord, 1]];
  const alphabet = 'abcdefghijklmnopqrstuvwxyz';
 
  let head = 0;
  while (head < queue.length) {
    const [word, steps] = queue[head++];
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

### **14.8  Snakes and Ladders**

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
 
  let head = 0;
  while (head < queue.length) {
    const [square, moves] = queue[head++];
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

## **15. Graph DFS**

**Goal:** O(V + E) exhaustive exploration

**Core idea:** *Explore as deep as possible before backtracking; natural fit for trees, graphs, and combinatorial search.*

**Recognize it when:**
- Count or measure connected regions (islands, provinces, area).
- Reachability: "which cells can reach the ocean?". It's often easier to search backwards from the target.
- Copy/clone a structure with cycles (keep an original → copy map).

**Connects to:** The same recursion as trees (§10), plus a `visited` set because graphs have cycles. With an undo step it becomes backtracking (§19); with a memo it becomes DP (§21). Cycle detection with three colours is DFS-based topological sort (§16). Union-Find (§17) answers the same connectivity questions without traversal.

**Template:**

```js
// Adjacency list + visited set
function countComponents(n, edges) {
  const graph = Array.from({ length: n }, () => []);
  for (const [a, b] of edges) { graph[a].push(b); graph[b].push(a); }

  const visited = new Array(n).fill(false);
  const dfs = (u) => {
    visited[u] = true;
    for (const v of graph[u]) if (!visited[v]) dfs(v);
  };

  let components = 0;
  for (let u = 0; u < n; u++) {
    if (!visited[u]) { components++; dfs(u); }
  }
  return components;
}
```

**Pitfalls:**
- Mark visited *before* recursing into neighbours, or cycles recurse forever.
- Recursive DFS on a 1000 × 1000 grid can exceed the JS call stack. Use BFS or an explicit stack for very large inputs.
- Modifying the input grid to mark visited is fine in interviews, but say so (or copy it).

### **15.1  Number of Islands (DFS version)**

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

### **15.2  Max Area of Island**

**Problem:** Given a grid of 1s (land) and 0s (water), return the area of the largest island (4-directionally connected), or 0.

**Approach:** Same flood fill as Number of Islands, but each DFS returns the size of the region it sinks: 1 for the current cell plus the sizes from its four neighbours. Track the maximum.

**Optimal Solution (JavaScript):**

```js
function maxAreaOfIsland(grid) {
  const rows = grid.length, cols = grid[0].length;
  const area = (r, c) => {
    if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] !== 1) return 0;
    grid[r][c] = 0; // sink it so it isn't counted twice
    return 1 + area(r + 1, c) + area(r - 1, c) + area(r, c + 1) + area(r, c - 1);
  };
  let best = 0;
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (grid[r][c] === 1) best = Math.max(best, area(r, c));
  return best;
}
```

**Complexity:** Time O(rows · cols), Space O(rows · cols) worst case recursion

***Alternate approaches:*** *BFS flood fill counting dequeued cells, or Union-Find with component sizes. Both are the same complexity; Union-Find also handles land being added cell by cell (Number of Islands II).*

☐ Practiced   ☐ Revised

### **15.3  Clone Graph**

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

### **15.4  Pacific Atlantic Water Flow**

**Problem:** Water flows from a cell to a neighbour with equal or lower height. The Pacific touches the top and left edges, the Atlantic the bottom and right. Return all cells from which water can reach both oceans.

**Approach:** Searching forward from every cell is O((m·n)²). Instead reverse the flow: DFS **uphill** from every Pacific border cell to mark what can drain to the Pacific, then do the same for the Atlantic. The answer is the cells marked by both.

**Optimal Solution (JavaScript):**

```js
function pacificAtlantic(heights) {
  const rows = heights.length, cols = heights[0].length;
  const makeGrid = () => Array.from({ length: rows }, () => new Array(cols).fill(false));
  const pacific = makeGrid(), atlantic = makeGrid();

  const climb = (r, c, seen) => { // move to neighbours at least as high
    if (seen[r][c]) return;
    seen[r][c] = true;
    for (const [dr, dc] of [[1, 0], [-1, 0], [0, 1], [0, -1]]) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < rows && nc >= 0 && nc < cols &&
          heights[nr][nc] >= heights[r][c]) {
        climb(nr, nc, seen);
      }
    }
  };

  for (let r = 0; r < rows; r++) { climb(r, 0, pacific); climb(r, cols - 1, atlantic); }
  for (let c = 0; c < cols; c++) { climb(0, c, pacific); climb(rows - 1, c, atlantic); }

  const result = [];
  for (let r = 0; r < rows; r++)
    for (let c = 0; c < cols; c++)
      if (pacific[r][c] && atlantic[r][c]) result.push([r, c]);
  return result;
}
```

**Complexity:** Time O(rows · cols), Space O(rows · cols)

***Alternate approaches:*** *Multi-source BFS from each ocean's border cells instead of DFS. Same complexity, and no recursion-depth risk on large grids. "Search backwards from the targets" is the reusable idea.*

☐ Practiced   ☐ Revised

## **16. Topological Sort**

**Goal:** O(V + E) ordering of a directed acyclic graph

**Core idea:** *Order nodes so every edge points forward: repeatedly take nodes with no remaining prerequisites (Kahn's BFS), or output nodes in reverse post-order (DFS). If you can't order every node, there is a cycle.*

**Recognize it when:**
- Prerequisites, dependencies, build/compile order, task scheduling with "A before B".
- "Is it possible to finish?" is cycle detection in a directed graph.
- Ordering is implied by sorted data (alien dictionary: adjacent words reveal letter order).

**Connects to:** Kahn's algorithm is BFS (§14) driven by in-degrees; the DFS version is three-colour DFS (§15). DP on a DAG processes nodes in topological order.

**Template:**

```js
// Kahn's algorithm: returns an order, or [] if there is a cycle
function topoSort(n, edges) { // edge [a, b] means a must come before b
  const graph = Array.from({ length: n }, () => []);
  const inDegree = new Array(n).fill(0);
  for (const [a, b] of edges) { graph[a].push(b); inDegree[b]++; }

  const order = [];
  for (let u = 0; u < n; u++) if (inDegree[u] === 0) order.push(u);
  for (let head = 0; head < order.length; head++) {
    for (const v of graph[order[head]]) {
      if (--inDegree[v] === 0) order.push(v);
    }
  }
  return order.length === n ? order : [];
}
```

**Pitfalls:**
- Get the edge direction right: `[course, prereq]` means `prereq → course`.
- Undirected-graph cycle detection is different: use Union-Find (§17) or DFS that ignores the parent edge.
- In DFS, "visiting" (on the current path) and "visited" (finished) are different states; only "visiting" means a cycle.

### **16.1  Course Schedule (BFS / Kahn's Algorithm)**

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
  let head = 0;
  while (head < queue.length) {
    const course = queue[head++];
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

### **16.2  Course Schedule (DFS cycle detection)**

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

### **16.3  Course Schedule II**

**Problem:** Given numCourses and prerequisite pairs [course, prereq], return a valid order to take all courses, or [] if impossible.

**Approach:** Kahn's algorithm, as in Course Schedule, but keep the processing order. The `order` array doubles as the BFS queue: a head index walks forward through it while newly freed courses are appended.

**Optimal Solution (JavaScript):**

```js
function findOrder(numCourses, prerequisites) {
  const graph = Array.from({ length: numCourses }, () => []);
  const inDegree = new Array(numCourses).fill(0);
  for (const [course, pre] of prerequisites) {
    graph[pre].push(course);
    inDegree[course]++;
  }

  const order = [];
  for (let i = 0; i < numCourses; i++) if (inDegree[i] === 0) order.push(i);
  for (let head = 0; head < order.length; head++) {
    for (const next of graph[order[head]]) {
      if (--inDegree[next] === 0) order.push(next);
    }
  }
  return order.length === numCourses ? order : [];
}
```

**Complexity:** Time O(V + E), Space O(V + E)

***Alternate approaches:*** *DFS with three colours: append each course after all its prerequisites finish (post-order). With edges stored as course → prereq, the post-order is already a valid order. Same O(V + E).*

☐ Practiced   ☐ Revised

### **16.4  Alien Dictionary**

**Problem:** Given words sorted in an unknown alphabet's order, return a string of their letters in a valid alphabet order, or "" if the order is contradictory.

**Approach:** Adjacent words reveal one rule each: the first differing character of `a` comes before that of `b`. Build a graph of these rules over all letters present and topologically sort it with Kahn's algorithm. A cycle (not all letters output) or a word followed by its own prefix (`"abc"` before `"ab"`) means it's invalid.

**Optimal Solution (JavaScript):**

```js
function alienOrder(words) {
  const graph = new Map(); // char -> Set of chars that come after it
  const inDegree = new Map();
  for (const w of words) {
    for (const c of w) {
      if (!graph.has(c)) { graph.set(c, new Set()); inDegree.set(c, 0); }
    }
  }

  for (let i = 0; i + 1 < words.length; i++) {
    const a = words[i], b = words[i + 1];
    if (a.length > b.length && a.startsWith(b)) return ''; // prefix after longer word
    for (let j = 0; j < Math.min(a.length, b.length); j++) {
      if (a[j] !== b[j]) {
        if (!graph.get(a[j]).has(b[j])) {
          graph.get(a[j]).add(b[j]);
          inDegree.set(b[j], inDegree.get(b[j]) + 1);
        }
        break; // only the first difference carries information
      }
    }
  }

  const order = [...inDegree.keys()].filter(c => inDegree.get(c) === 0);
  for (let head = 0; head < order.length; head++) {
    for (const next of graph.get(order[head])) {
      inDegree.set(next, inDegree.get(next) - 1);
      if (inDegree.get(next) === 0) order.push(next);
    }
  }
  return order.length === graph.size ? order.join('') : '';
}
```

**Complexity:** Time O(C) where C = total characters in all words, Space O(U + rules) for U unique letters (at most 26)

***Alternate approaches:*** *DFS-based topological sort with cycle detection (three colours), reversing the post-order. Same complexity.*

☐ Practiced   ☐ Revised

## **17. Union-Find**

**Goal:** Near O(1) amortized per union/find (inverse Ackermann α(n))

**Core idea:** *Keep a forest where each set is a tree identified by its root; `find` walks to the root (compressing the path), `union` links two roots (by rank).*

**Recognize it when:**
- Edges arrive one at a time and you need to know whether two nodes are already connected.
- "Does adding this edge create a cycle?" in an **undirected** graph.
- Counting connected components, or grouping items by a transitive relation (accounts merge, similar strings).

**Connects to:** An alternative to DFS/BFS (§15, §14) for connectivity that works incrementally. It's the cycle check inside Kruskal's minimum spanning tree (*Min Cost to Connect All Points* (§18.3)).

**Template:** see *Implement Union-Find (Disjoint Set Union)* (§17.1). Every problem in this chapter reuses that class.

**Pitfalls:**
- Without path compression *and* union by rank/size, operations can degrade to O(n).
- Union-Find can't easily *remove* edges, and it doesn't give paths, only connectivity.
- Watch 0- vs 1-indexed node labels when sizing the parent array.

### **17.1  Implement Union-Find (Disjoint Set Union)**

**Problem:** Implement a structure over n elements supporting `find(x)` (which set is x in?) and `union(a, b)` (merge their sets), and track the number of sets.

**Approach:** Each set is a tree in a `parent` array whose root identifies the set. **Path compression** makes nodes on a find path point closer to the root, and **union by rank** attaches the shorter tree under the taller. Together they make both operations nearly O(1). `union` returns false when a and b were already connected, which is how cycles are detected.

**Optimal Solution (JavaScript):**

```js
class UnionFind {
  constructor(n) {
    this.parent = Array.from({ length: n }, (_, i) => i);
    this.rank = new Array(n).fill(0);
    this.count = n; // number of separate sets
  }
  find(x) {
    while (this.parent[x] !== x) {
      this.parent[x] = this.parent[this.parent[x]]; // path halving
      x = this.parent[x];
    }
    return x;
  }
  union(a, b) { // returns false if a and b were already connected
    let ra = this.find(a), rb = this.find(b);
    if (ra === rb) return false;
    if (this.rank[ra] < this.rank[rb]) [ra, rb] = [rb, ra];
    this.parent[rb] = ra;
    if (this.rank[ra] === this.rank[rb]) this.rank[ra]++;
    this.count--;
    return true;
  }
}
```

**Complexity:** Time O(α(n)) amortized per operation (effectively constant), Space O(n)

***Alternate approaches:*** *Without the optimizations, a chain of unions can make find O(n). Union by size (attach the smaller set) works as well as union by rank, and the size is often useful itself (largest component).*

☐ Practiced   ☐ Revised

### **17.2  Number of Connected Components in an Undirected Graph**

**Problem:** Given n nodes labelled 0..n−1 and a list of undirected edges, return the number of connected components.

**Approach:** Start with n separate sets and union the endpoints of every edge. Each successful union merges two components, so the remaining set count is the answer.

**Optimal Solution (JavaScript):**

```js
function countComponents(n, edges) {
  const uf = new UnionFind(n);
  for (const [a, b] of edges) uf.union(a, b);
  return uf.count;
}
```

**Complexity:** Time O(n + E · α(n)), Space O(n)

***Alternate approaches:*** *Build an adjacency list and count how many times a DFS/BFS starts from an unvisited node (the Graph DFS chapter template). O(n + E) time and space.*

☐ Practiced   ☐ Revised

### **17.3  Graph Valid Tree**

**Problem:** Given n nodes and a list of undirected edges, determine whether the edges form a valid tree (connected, no cycles).

**Approach:** A tree on n nodes has exactly n − 1 edges; check that first. Then union every edge: if any edge joins two nodes already in the same set, it closes a cycle. With n − 1 edges and no cycle, the graph is connected.

**Optimal Solution (JavaScript):**

```js
function validTree(n, edges) {
  if (edges.length !== n - 1) return false; // a tree has exactly n - 1 edges
  const uf = new UnionFind(n);
  for (const [a, b] of edges) {
    if (!uf.union(a, b)) return false; // already connected: cycle
  }
  return true;
}
```

**Complexity:** Time O(n · α(n)), Space O(n)

***Alternate approaches:*** *Check the edge count, then DFS from node 0 and verify that all n nodes were reached. O(n) time and space.*

☐ Practiced   ☐ Revised

### **17.4  Redundant Connection**

**Problem:** A tree with n nodes (labelled 1..n) had one extra edge added. Return the edge that can be removed so the result is a tree; if several work, return the last one in the input.

**Approach:** Union edges in input order. The first edge whose endpoints are already connected is the one that closes the cycle, and it is also the last cycle edge in the input order.

**Optimal Solution (JavaScript):**

```js
function findRedundantConnection(edges) {
  const uf = new UnionFind(edges.length + 1); // labels are 1..n
  for (const [a, b] of edges) {
    if (!uf.union(a, b)) return [a, b];
  }
  return [];
}
```

**Complexity:** Time O(n · α(n)), Space O(n)

***Alternate approaches:*** *For each edge, DFS to check whether its endpoints are already connected by earlier edges, then add it. O(n²) time.*

☐ Practiced   ☐ Revised

## **18. Weighted Graphs: Shortest Paths & MST**

**Goal:** Shortest paths with edge weights; cheapest way to connect everything

**Core idea:** *Dijkstra always expands the closest unfinished node (BFS with a min-heap). Bellman-Ford relaxes every edge repeatedly (handles negative weights and "at most k edges"). Prim/Kruskal grow a minimum spanning tree.*

**Recognize it when:**
- Weighted edges and "minimum total cost/time/distance" between nodes → Dijkstra (weights ≥ 0).
- A limit on the number of edges/stops, or negative weights → Bellman-Ford.
- "Connect all points/cities with minimum total cost" → Minimum Spanning Tree (Prim or Kruskal).

**Connects to:** Dijkstra = BFS (§14) + heap (§13). Kruskal = sort edges (§7) + Union-Find (§17). Bellman-Ford is DP over the number of edges used (§21).

| **Algorithm** | **Use for** | **Time** |
| --- | --- | --- |
| BFS | Unweighted shortest path | O(V + E) |
| Dijkstra | Non-negative weights | O((V + E) log V) |
| Bellman-Ford | Negative weights, at most k edges | O(V · E) |
| Prim (array) / Kruskal | Minimum spanning tree | O(V²) / O(E log E) |

**Template:**

```js
// Dijkstra with lazy deletion (uses the MinHeap class from the Heaps chapter)
function dijkstra(n, adj, src, MinHeap) { // adj[u] = [[v, w], ...]
  const dist = new Array(n).fill(Infinity);
  dist[src] = 0;
  const heap = new MinHeap((a, b) => a[0] - b[0]);
  heap.push([0, src]);
  while (heap.size()) {
    const [d, u] = heap.pop();
    if (d > dist[u]) continue; // stale entry
    for (const [v, w] of adj[u]) {
      if (d + w < dist[v]) { dist[v] = d + w; heap.push([dist[v], v]); }
    }
  }
  return dist;
}
```

**Pitfalls:**
- Dijkstra is wrong with negative edge weights.
- Skip stale heap entries (`d > dist[u]`) instead of trying to "decrease key".
- For "at most k stops" with Bellman-Ford, relax from a **copy** of the previous round's distances, or paths can use more edges than allowed.

### **18.1  Network Delay Time**

**Problem:** Given directed weighted edges `times[i] = [u, v, w]`, n nodes (1..n), and a source k, return the time for a signal from k to reach all nodes, or -1 if some node is unreachable.

**Approach:** Dijkstra: the answer is the largest shortest-path distance from k. A min-heap always expands the closest unsettled node. Stale heap entries (a shorter distance was found after they were pushed) are skipped.

**Optimal Solution (JavaScript):**

```js
function networkDelayTime(times, n, k) {
  const graph = Array.from({ length: n + 1 }, () => []);
  for (const [u, v, w] of times) graph[u].push([v, w]);

  const dist = new Array(n + 1).fill(Infinity);
  dist[k] = 0;
  const heap = new MinHeap((a, b) => a[0] - b[0]); // [distance, node]
  heap.push([0, k]);

  while (heap.size()) {
    const [d, u] = heap.pop();
    if (d > dist[u]) continue; // stale entry
    for (const [v, w] of graph[u]) {
      if (d + w < dist[v]) {
        dist[v] = d + w;
        heap.push([dist[v], v]);
      }
    }
  }
  let worst = 0;
  for (let i = 1; i <= n; i++) worst = Math.max(worst, dist[i]);
  return worst === Infinity ? -1 : worst;
}
```

**Complexity:** Time O((V + E) log V), Space O(V + E)

***Alternate approaches:*** *Bellman-Ford: relax every edge V − 1 times, O(V · E), and it handles negative weights. For a dense graph, Dijkstra with a plain array scan instead of a heap is O(V²).*

☐ Practiced   ☐ Revised

### **18.2  Cheapest Flights Within K Stops**

**Problem:** Given n cities, flights `[from, to, price]`, a source, a destination and k, return the cheapest price using at most k stops (k + 1 flights), or -1.

**Approach:** Bellman-Ford limited to k + 1 rounds: after round i, `cost[v]` is the cheapest price using at most i flights. Each round relaxes from a **copy** of the previous round's costs, so a single round can't chain several flights together.

**Optimal Solution (JavaScript):**

```js
function findCheapestPrice(n, flights, src, dst, k) {
  let cost = new Array(n).fill(Infinity);
  cost[src] = 0;
  for (let i = 0; i <= k; i++) { // k stops = k + 1 flights
    const next = [...cost];      // relax only from last round's costs
    for (const [u, v, price] of flights) {
      if (cost[u] !== Infinity && cost[u] + price < next[v]) {
        next[v] = cost[u] + price;
      }
    }
    cost = next;
  }
  return cost[dst] === Infinity ? -1 : cost[dst];
}
```

**Complexity:** Time O(k · E), Space O(n)

***Alternate approaches:*** *BFS by levels (one level per flight) with price pruning, or Dijkstra over (city, stops used) states. Plain Dijkstra on city alone is wrong here: a cheaper path with too many stops can block a slightly pricier valid one.*

☐ Practiced   ☐ Revised

### **18.3  Min Cost to Connect All Points**

**Problem:** Given points on a plane, the cost to connect two points is their Manhattan distance. Return the minimum cost to connect all points.

**Approach:** This is a minimum spanning tree on a complete graph. Prim's algorithm with arrays suits dense graphs: keep `minDist[v]`, the cheapest edge from the tree to each point. Repeatedly add the closest point not yet in the tree and update its neighbours' minDist. No heap is needed.

**Optimal Solution (JavaScript):**

```js
function minCostConnectPoints(points) {
  const n = points.length;
  const minDist = new Array(n).fill(Infinity); // cheapest edge into the tree
  const inTree = new Array(n).fill(false);
  minDist[0] = 0;
  let total = 0;

  for (let added = 0; added < n; added++) {
    let u = -1;
    for (let i = 0; i < n; i++) {
      if (!inTree[i] && (u === -1 || minDist[i] < minDist[u])) u = i;
    }
    inTree[u] = true;
    total += minDist[u];
    for (let v = 0; v < n; v++) {
      if (!inTree[v]) {
        const d = Math.abs(points[u][0] - points[v][0]) + Math.abs(points[u][1] - points[v][1]);
        if (d < minDist[v]) minDist[v] = d;
      }
    }
  }
  return total;
}
```

**Complexity:** Time O(n²), Space O(n)

***Alternate approaches:*** *Kruskal: generate all n² edges, sort them by cost, and add each edge whose endpoints Union-Find says are not yet connected. That is O(n² log n), slower here but the better choice for sparse graphs. Prim with a heap is O(E log V).*

☐ Practiced   ☐ Revised

# **Part V — Search & Optimization**

## **19. Backtracking**

**Goal:** Enumerate every valid configuration, pruning dead ends early (exponential, so n is small)

**Core idea:** *Build the answer one choice at a time: choose → recurse → un-choose. The recursion tree is the space of partial solutions; pruning cuts branches that can't succeed.*

**Recognize it when:**
- "Return **all** subsets / permutations / combinations / partitions / placements".
- Constraint satisfaction: N-Queens, Sudoku, word search on a grid.
- Small n (≤ ~20) and an answer count that is itself exponential.

**Connects to:** DFS (§15) with an undo step. If you only need a **count** or **best value** and subproblems repeat, memoize it and it becomes DP (§21). Subsets can also be generated with bitmasks (§22). Word Search II adds a trie (§12).

**Template:**

```js
function backtrackTemplate(candidates) {
  const result = [], path = [];
  const backtrack = (start) => {
    result.push([...path]);                 // record (subsets) or check a goal
    for (let i = start; i < candidates.length; i++) {
      if (i > start && candidates[i] === candidates[i - 1]) continue; // skip dup choices
      path.push(candidates[i]);             // choose
      backtrack(i + 1);                     // explore (i for unlimited reuse)
      path.pop();                           // un-choose
    }
  };
  candidates.sort((a, b) => a - b);         // needed for the duplicate skip
  backtrack(0);
  return result;
}
```

**Pitfalls:**
- Push a **copy** (`[...path]`) into the result; `path` keeps changing.
- `start` index → combinations (order doesn't matter); `used[]` array → permutations (order matters).
- Next call with `i` (reuse allowed) vs `i + 1` (each item once).
- Complexity counts the copying: subsets are O(n · 2ⁿ), permutations O(n · n!).

### **19.1  Subsets / Subsets II**

**Problem:** Given an array of (possibly duplicate, for II) integers, return all possible subsets (the power set), with II requiring no duplicate subsets.

**Approach:** Backtracking: at each index, decide to include or exclude the current element, recursing forward. Record the current subset at every recursive call (not just at the end). For duplicates (II), sort first and skip over repeated values at the same recursion depth to avoid duplicate subsets. With distinct input the skip never triggers, so the same code solves Subsets I.

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

**Complexity:** Time O(n · 2^n) — up to 2^n subsets, each copied in O(n), Space O(n) recursion depth (excluding output)

***Alternate approaches:*** *Iterative approach: start with [[]] and for each number, add it to every existing subset already built so far — O(2^n) time, avoids recursion but needs care to dedupe for the 'with duplicates' variant.*

☐ Practiced   ☐ Revised

### **19.2  Permutations**

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

### **19.3  Combination Sum**

**Problem:** Given distinct positive candidates and a target, return all unique combinations that sum to target; each candidate may be used unlimited times.

**Approach:** Backtracking with a start index so combinations come out in non-decreasing order (no permutation duplicates). Recurse with the **same** index `i` to allow reuse; skip candidates larger than the remaining sum.

**Optimal Solution (JavaScript):**

```js
function combinationSum(candidates, target) {
  const result = [], path = [];
  const backtrack = (start, remaining) => {
    if (remaining === 0) { result.push([...path]); return; }
    for (let i = start; i < candidates.length; i++) {
      if (candidates[i] > remaining) continue;
      path.push(candidates[i]);
      backtrack(i, remaining - candidates[i]); // i, not i + 1: reuse allowed
      path.pop();
    }
  };
  backtrack(0, target);
  return result;
}
```

**Complexity:** Time O(n^(T/m + 1)) worst case, where T = target and m = smallest candidate (depth ≤ T/m). Space O(T/m) recursion depth

***Alternate approaches:*** *Sort the candidates and `break` instead of `continue` once one is too large, which prunes more. If only the **number** of combinations is needed, it becomes DP: Coin Change II.*

☐ Practiced   ☐ Revised

### **19.4  Letter Combinations of a Phone Number**

**Problem:** Given a string of digits 2–9, return all letter combinations they could represent on a phone keypad.

**Approach:** Backtracking over positions: at digit i, try each of its letters and recurse to i + 1. When all digits are used, record the string. Strings are immutable, so passing `s + c` needs no explicit undo.

**Optimal Solution (JavaScript):**

```js
function letterCombinations(digits) {
  if (!digits) return [];
  const keys = { 2: 'abc', 3: 'def', 4: 'ghi', 5: 'jkl', 6: 'mno', 7: 'pqrs', 8: 'tuv', 9: 'wxyz' };
  const result = [];
  const backtrack = (i, s) => {
    if (i === digits.length) { result.push(s); return; }
    for (const c of keys[digits[i]]) backtrack(i + 1, s + c);
  };
  backtrack(0, '');
  return result;
}
```

**Complexity:** Time O(n · 4ⁿ), Space O(n) recursion depth (excluding output)

***Alternate approaches:*** *Iterative BFS: start with [''] and, for each digit, replace the list with every existing string extended by every letter. Same complexity.*

☐ Practiced   ☐ Revised

### **19.5  Generate Parentheses**

**Problem:** Given n, return all combinations of n pairs of well-formed parentheses.

**Approach:** Backtracking with two counters instead of generating everything and filtering. Add `(` while fewer than n are open; add `)` only while it closes an open one (`close < open`). Every finished string is then valid by construction.

**Optimal Solution (JavaScript):**

```js
function generateParenthesis(n) {
  const result = [];
  const backtrack = (s, open, close) => {
    if (s.length === 2 * n) { result.push(s); return; }
    if (open < n) backtrack(s + '(', open + 1, close);
    if (close < open) backtrack(s + ')', open, close + 1);
  };
  backtrack('', 0, 0);
  return result;
}
```

**Complexity:** Time O(4ⁿ / √n) (the n-th Catalan number of results, each of length 2n), Space O(n) recursion depth

***Alternate approaches:*** *Generate all 2^(2n) strings of brackets and keep the valid ones (using the Valid Parentheses check): O(n · 2^(2n)), far slower. A DP that builds `(A)B` from smaller results also works.*

☐ Practiced   ☐ Revised

### **19.6  Palindrome Partitioning**

**Problem:** Partition a string so that every part is a palindrome; return all possible partitions.

**Approach:** Backtracking over cut positions: from `start`, try every end index whose substring `s[start..end]` is a palindrome, add it to the path, and recurse from `end + 1`. Reaching the end of the string records a partition.

**Optimal Solution (JavaScript):**

```js
function partition(s) {
  const result = [], path = [];
  const isPalindrome = (l, r) => {
    while (l < r) if (s[l++] !== s[r--]) return false;
    return true;
  };
  const backtrack = (start) => {
    if (start === s.length) { result.push([...path]); return; }
    for (let end = start; end < s.length; end++) {
      if (!isPalindrome(start, end)) continue;
      path.push(s.slice(start, end + 1));
      backtrack(end + 1);
      path.pop();
    }
  };
  backtrack(0);
  return result;
}
```

**Complexity:** Time O(n · 2ⁿ), Space O(n) recursion depth (excluding output)

***Alternate approaches:*** *Precompute a table `isPal[i][j]` with DP (or center expansion) in O(n²), so each palindrome check is O(1). The worst case stays O(n · 2ⁿ) because of the output size, but it's faster in practice.*

☐ Practiced   ☐ Revised

### **19.7  Word Search**

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

### **19.8  N-Queens**

**Problem:** Place n queens on an n x n board so that no two attack each other; return all distinct boards.

**Approach:** Place one queen per row, trying each column. A placement is safe if its column, its diagonal (`r - c`) and its anti-diagonal (`r + c`) are all unused. Track these in three sets for O(1) checks, and undo after recursing.

**Optimal Solution (JavaScript):**

```js
function solveNQueens(n) {
  const result = [];
  const board = Array.from({ length: n }, () => new Array(n).fill('.'));
  const cols = new Set(), diag = new Set(), anti = new Set();

  const backtrack = (r) => {
    if (r === n) { result.push(board.map(row => row.join(''))); return; }
    for (let c = 0; c < n; c++) {
      if (cols.has(c) || diag.has(r - c) || anti.has(r + c)) continue;
      cols.add(c); diag.add(r - c); anti.add(r + c);
      board[r][c] = 'Q';
      backtrack(r + 1);
      board[r][c] = '.';
      cols.delete(c); diag.delete(r - c); anti.delete(r + c);
    }
  };
  backtrack(0);
  return result;
}
```

**Complexity:** Time O(n!), Space O(n²) for the board

***Alternate approaches:*** *Bitmasks for columns and diagonals instead of sets: the same search with much smaller constants, the standard trick for counting solutions (N-Queens II) quickly.*

☐ Practiced   ☐ Revised

## **20. Greedy**

**Goal:** Usually O(n) or O(n log n) with a single pass

**Core idea:** *Make the locally best choice at each step and never revisit it. It works only when you can argue that some optimal solution starts with that choice (exchange argument).*

**Recognize it when:**
- A running "best so far" or "farthest reachable" is enough to decide (Jump Game, Kadane).
- Intervals: earliest end time first maximizes how many fit (*Non-overlapping Intervals* (§7.5)).
- A failing prefix lets you discard everything before it (Gas Station).

**Connects to:** Often needs sorting first (§7). Many greedy problems also have a DP solution (§21); greedy is the faster one when it is provably safe. Jump Game II is BFS (§14) over index ranges. Boats to Save People (§2) and Task Scheduler (§13) are greedy too.

**Template:**

```js
// Kadane-style: extend the current run or restart, track the best
function bestRun(nums) {
  let current = nums[0], best = nums[0];
  for (let i = 1; i < nums.length; i++) {
    current = Math.max(nums[i], current + nums[i]);
    best = Math.max(best, current);
  }
  return best;
}
```

**Pitfalls:**
- Test the greedy rule against a small counter-example before coding (e.g., greedy coin change fails for coins [1, 3, 4], amount 6).
- Be ready to explain *why* the choice is safe. Interviewers ask.

### **20.1  Maximum Subarray**

**Problem:** Given an integer array, find the contiguous subarray with the largest sum and return the sum.

**Approach:** Kadane's algorithm: the best subarray ending at i either extends the best one ending at i − 1 or starts fresh at i, whichever is larger. A negative running sum can only hurt, so restart. Track the best seen.

**Optimal Solution (JavaScript):**

```js
function maxSubArray(nums) {
  let current = nums[0], best = nums[0];
  for (let i = 1; i < nums.length; i++) {
    current = Math.max(nums[i], current + nums[i]); // extend or restart
    best = Math.max(best, current);
  }
  return best;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Prefix sums: the answer is max over j of `prefix[j] - min(prefix[i])` for i < j, which is the same as Best Time to Buy and Sell Stock. Divide and conquer (best in left, in right, or crossing the middle) is O(n log n). Brute force is O(n²).*

☐ Practiced   ☐ Revised

### **20.2  Jump Game**

**Problem:** Each element is your maximum jump length from that position. Starting at index 0, return true if you can reach the last index.

**Approach:** Track the farthest index reachable so far. Walk forward; if you ever stand on an index beyond that reach, you're stuck. Otherwise extend the reach with `i + nums[i]`.

**Optimal Solution (JavaScript):**

```js
function canJump(nums) {
  let reach = 0;
  for (let i = 0; i < nums.length; i++) {
    if (i > reach) return false; // can't even get here
    reach = Math.max(reach, i + nums[i]);
  }
  return true;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *DP: `good[i]` is true if some jump from i lands on a good index, filled right to left. That is O(n²) time. Equivalent greedy: walk backwards moving a "goal" to any index that can reach it; answer goal === 0.*

☐ Practiced   ☐ Revised

### **20.3  Jump Game II**

**Problem:** Same setup as Jump Game (the end is always reachable). Return the minimum number of jumps to reach the last index.

**Approach:** Implicit BFS over index ranges: all indices reachable with j jumps form one contiguous "level". While scanning the current level, track the farthest index the next level can reach. When you hit the end of the current level, take a jump and the next level ends at that farthest point.

**Optimal Solution (JavaScript):**

```js
function jump(nums) {
  let jumps = 0, currentEnd = 0, farthest = 0;
  for (let i = 0; i < nums.length - 1; i++) {
    farthest = Math.max(farthest, i + nums[i]);
    if (i === currentEnd) { // finished this level
      jumps++;
      currentEnd = farthest;
    }
  }
  return jumps;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *DP: `dp[i]` = min jumps to reach i, updating every index each position can reach. That is O(n²) time. An explicit BFS over indices also works but needs a queue and a visited array.*

☐ Practiced   ☐ Revised

### **20.4  Gas Station**

**Problem:** Stations on a circle have `gas[i]` fuel; driving to the next costs `cost[i]`. Return the start index from which you can complete the circuit (unique if it exists), or -1.

**Approach:** If total gas < total cost, it's impossible. Otherwise run the tank from a candidate start. If the tank goes negative at station i, no start between the candidate and i can work either (each of them would arrive at i with even less fuel), so the next candidate is i + 1.

**Optimal Solution (JavaScript):**

```js
function canCompleteCircuit(gas, cost) {
  let total = 0, tank = 0, start = 0;
  for (let i = 0; i < gas.length; i++) {
    const diff = gas[i] - cost[i];
    total += diff;
    tank += diff;
    if (tank < 0) { // nobody from start..i can pass i
      start = i + 1;
      tank = 0;
    }
  }
  return total >= 0 ? start : -1;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Simulate the full circuit from every start: O(n²). The greedy skip is the key insight, and it's the same "discard a failing prefix" idea as Kadane.*

☐ Practiced   ☐ Revised

### **20.5  Partition Labels**

**Problem:** Split a string into as many parts as possible so that each letter appears in at most one part; return the part sizes.

**Approach:** Record the last index of every letter. Scan while extending the current part's end to the farthest last-occurrence of any letter seen in it. When the scan reaches that end, the part is closed and can't be extended.

**Optimal Solution (JavaScript):**

```js
function partitionLabels(s) {
  const last = new Map();
  for (let i = 0; i < s.length; i++) last.set(s[i], i);

  const result = [];
  let start = 0, end = 0;
  for (let i = 0; i < s.length; i++) {
    end = Math.max(end, last.get(s[i]));
    if (i === end) {
      result.push(end - start + 1);
      start = i + 1;
    }
  }
  return result;
}
```

**Complexity:** Time O(n), Space O(1) (at most 26 letters)

***Alternate approaches:*** *Convert each letter to an interval [first, last] and merge overlapping intervals (Merge Intervals). Same idea, more code.*

☐ Practiced   ☐ Revised

## **21. Dynamic Programming**

**Goal:** Exponential → Polynomial

**Core idea:** *Break a problem into overlapping subproblems, cache results (memoization / tabulation) to avoid recomputation.*

**Recognize it when:**
- "Number of ways", "minimum/maximum cost", "is it possible", "longest/shortest subsequence".
- A brute-force recursion makes choices (take/skip, which coin, which split) and the same arguments repeat.
- Greedy fails on a small example.

**The 4-step recipe:**
1. **State:** what does `dp[i]` (or `dp[i][j]`) mean, in words?
2. **Transition:** how is a state built from smaller states?
3. **Base cases:** the smallest states you know directly.
4. **Order & answer:** fill so dependencies come first; say which cell is the answer. Then check whether only the last row/two values are needed (space optimization).

**Problem families in this chapter:**
- *1D linear:* Climbing Stairs, House Robber I/II, Decode Ways, Maximum Product Subarray, Word Break.
- *Knapsack (unbounded vs 0/1):* Coin Change, Coin Change II, Combination Sum IV, 0/1 Knapsack, Partition Equal Subset Sum.
- *Subsequences & two strings:* Longest Increasing Subsequence, Longest Common Subsequence, Edit Distance.
- *Grid:* Unique Paths.
- *Palindromes (interval):* Longest Palindromic Substring, Palindromic Substrings.

**Connects to:** Top-down DP is DFS/backtracking (§19) plus a memo (§1). Kadane (*Maximum Subarray* (§20.1)) and Jump Game (§20) are DP problems with a greedy shortcut. LIS uses binary search (§6).

**Template:**

```js
// Top-down (memoized recursion) and bottom-up (tabulation) for the same recurrence
function countWaysTopDown(n) {
  const memo = new Map();
  const ways = (i) => {
    if (i <= 1) return 1;                     // base cases
    if (memo.has(i)) return memo.get(i);
    const result = ways(i - 1) + ways(i - 2); // transition
    memo.set(i, result);
    return result;
  };
  return ways(n);
}

function countWaysBottomUp(n) {
  const dp = new Array(n + 1).fill(0);
  dp[0] = 1; dp[1] = 1;
  for (let i = 2; i <= n; i++) dp[i] = dp[i - 1] + dp[i - 2];
  return dp[n];
}
```

**Pitfalls:**
- Knapsack loop order: iterate capacity **backwards** for 0/1 (each item once), **forwards** for unbounded (reuse allowed).
- Counting combinations vs permutations: put items in the outer loop for combinations (*Coin Change II* (§21.8)) and the amount in the outer loop for permutations (*Combination Sum IV* (§21.9)).
- Use `Infinity` for "impossible" in min-problems and convert to -1 at the end.

### **21.1  Fibonacci Number / Climbing Stairs**

**Problem:** Given n, find the number of ways to climb n stairs taking 1 or 2 steps at a time. This is the Fibonacci sequence shifted by one: ways(n) = Fib(n + 1), because the last step was either 1 (ways(n − 1)) or 2 (ways(n − 2)).

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

### **21.2  House Robber**

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

### **21.3  House Robber II**

**Problem:** Same as House Robber, but houses are arranged in a circle (first and last are adjacent).

**Approach:** Since the first and last house can't both be robbed, run the standard House Robber DP twice: once excluding the last house, once excluding the first house, and take the maximum of the two results.

**Optimal Solution (JavaScript):**

```js
function rob(nums) {
  if (nums.length === 1) return nums[0];

  const robRange = (start, end) => { // house robber over nums[start..end]
    let prev2 = 0, prev1 = 0;
    for (let i = start; i <= end; i++) {
      const curr = Math.max(prev1, prev2 + nums[i]);
      prev2 = prev1;
      prev1 = curr;
    }
    return prev1;
  };

  const n = nums.length;
  return Math.max(robRange(0, n - 2), robRange(1, n - 1));
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Slicing the array (`robLinear(nums.slice(0, -1))` and `robLinear(nums.slice(1))`) is shorter to write but costs O(n) extra space for the copies; passing an index range, as above, keeps it O(1).*

☐ Practiced   ☐ Revised

### **21.4  Decode Ways**

**Problem:** Letters map to numbers 'A' = 1 … 'Z' = 26. Given a digit string, return the number of ways to decode it.

**Approach:** Like Climbing Stairs, but steps are conditional. `ways(i)` (for the first i characters) adds `ways(i−1)` if the last digit is 1–9, and `ways(i−2)` if the last two digits form 10–26. Only the previous two values are needed.

**Optimal Solution (JavaScript):**

```js
function numDecodings(s) {
  let prev2 = 1;                       // ways to decode ""
  let prev1 = s[0] === '0' ? 0 : 1;    // ways to decode s[0]
  for (let i = 1; i < s.length; i++) {
    let curr = 0;
    if (s[i] !== '0') curr += prev1;   // last digit alone
    const two = Number(s.slice(i - 1, i + 1));
    if (two >= 10 && two <= 26) curr += prev2; // last two digits together
    prev2 = prev1;
    prev1 = curr;
  }
  return prev1;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Top-down recursion with memoization on the index. Same O(n) time, O(n) space, and often easier to derive first. Without memoization it's exponential.*

☐ Practiced   ☐ Revised

### **21.5  Maximum Product Subarray**

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

### **21.6  Word Break**

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

**Complexity:** Time O(n³) worst case — O(n²) (j, i) pairs, each slicing and hashing a substring of up to n characters; limiting j to the longest dictionary word length L gives O(n · L²). Space O(n)

***Alternate approaches:*** *BFS/DFS over 'reachable end positions' with memoization to avoid revisiting the same index — same complexity, framed as a graph search instead of tabulated DP.*

☐ Practiced   ☐ Revised

### **21.7  Coin Change**

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

### **21.8  Coin Change II**

**Problem:** Given coin denominations (unlimited supply) and an amount, return the number of **combinations** that make the amount (order doesn't matter).

**Approach:** Unbounded knapsack counting: `dp[a]` = ways to make a. Loop over **coins in the outer loop**, so each combination is built in one fixed coin order and counted once. Iterate amounts forwards, because a coin can be reused.

**Optimal Solution (JavaScript):**

```js
function change(amount, coins) {
  const dp = new Array(amount + 1).fill(0);
  dp[0] = 1;
  for (const coin of coins) {            // coins outer: combinations
    for (let a = coin; a <= amount; a++) {
      dp[a] += dp[a - coin];
    }
  }
  return dp[amount];
}
```

**Complexity:** Time O(amount · numCoins), Space O(amount)

***Alternate approaches:*** *2D table `dp[i][a]` using the first i coins: same time, O(amount · numCoins) space. Swapping the loops counts ordered sequences instead, which is Combination Sum IV.*

☐ Practiced   ☐ Revised

### **21.9  Combination Sum IV**

**Problem:** Given distinct positive integers and a target, return the number of **ordered** sequences that sum to target (different orders count separately).

**Approach:** `dp[t]` = number of sequences summing to t. The last number of a sequence can be any n ≤ t, so `dp[t] = Σ dp[t − n]`. Because the **amount is in the outer loop**, every order is counted. Compare with Coin Change II.

**Optimal Solution (JavaScript):**

```js
function combinationSum4(nums, target) {
  const dp = new Array(target + 1).fill(0);
  dp[0] = 1;
  for (let t = 1; t <= target; t++) {    // amount outer: permutations
    for (const n of nums) {
      if (n <= t) dp[t] += dp[t - n];
    }
  }
  return dp[target];
}
```

**Complexity:** Time O(target · n), Space O(target)

***Alternate approaches:*** *Top-down memoized recursion `count(remaining)`. Same complexity. Backtracking that lists every sequence is exponential.*

☐ Practiced   ☐ Revised

### **21.10  0/1 Knapsack**

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

### **21.11  Partition Equal Subset Sum**

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

### **21.12  Longest Increasing Subsequence**

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

### **21.13  Longest Common Subsequence**

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

### **21.14  Edit Distance**

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

### **21.15  Unique Paths**

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

### **21.16  Longest Palindromic Substring**

**Problem:** Return the longest palindromic substring of s.

**Approach:** Every palindrome mirrors around a center, which is a character (odd length) or a gap between two characters (even length). There are 2n − 1 centers. Expand outward from each while the ends match, and keep the longest.

**Optimal Solution (JavaScript):**

```js
function longestPalindrome(s) {
  let start = 0, maxLen = 0;
  const expand = (l, r) => {
    while (l >= 0 && r < s.length && s[l] === s[r]) { l--; r++; }
    if (r - l - 1 > maxLen) { maxLen = r - l - 1; start = l + 1; }
  };
  for (let i = 0; i < s.length; i++) {
    expand(i, i);     // odd length
    expand(i, i + 1); // even length
  }
  return s.slice(start, start + maxLen);
}
```

**Complexity:** Time O(n²), Space O(1)

***Alternate approaches:*** *Interval DP: `isPal[i][j] = s[i] === s[j] && isPal[i+1][j-1]`, filled by increasing length. O(n²) time and O(n²) space, which shows the DP structure explicitly. Manacher's algorithm is O(n) but rarely expected.*

☐ Practiced   ☐ Revised

### **21.17  Palindromic Substrings**

**Problem:** Return the number of palindromic substrings in s (substrings at different positions count separately).

**Approach:** Same center expansion as Longest Palindromic Substring, but count every successful expansion step: each one is a distinct palindrome.

**Optimal Solution (JavaScript):**

```js
function countSubstrings(s) {
  let count = 0;
  const expand = (l, r) => {
    while (l >= 0 && r < s.length && s[l] === s[r]) {
      count++;
      l--;
      r++;
    }
  };
  for (let i = 0; i < s.length; i++) {
    expand(i, i);     // odd length
    expand(i, i + 1); // even length
  }
  return count;
}
```

**Complexity:** Time O(n²), Space O(1)

***Alternate approaches:*** *The interval DP table `isPal[i][j]` counts true cells: O(n²) time and space.*

☐ Practiced   ☐ Revised

## **22. Bit Manipulation**

**Goal:** O(1) extra space tricks; O(1) per-number operations

**Core idea:** *Treat integers as arrays of bits: XOR cancels equal values, `n & (n - 1)` drops the lowest set bit, shifts multiply/divide by 2.*

**Recognize it when:**
- "Every element appears twice except one", "find the missing number" → XOR.
- Count set bits, check powers of two, reverse bits.
- "Without using + or −" → simulate addition with XOR and AND.
- Represent a subset of ≤ 20–30 items as an integer mask.

**Connects to:** Bitmask subsets are an alternative to backtracking (§19). *Counting Bits* (§22.3) is a tiny DP (§21).

| **Expression** | **Meaning** |
| --- | --- |
| `a ^ a = 0`, `a ^ 0 = a` | XOR cancels pairs |
| `n & 1` | Lowest bit (odd?) |
| `n & (n - 1)` | Clear the lowest set bit; `=== 0` means power of two |
| `n & -n` | Isolate the lowest set bit |
| `1 << k`, `n >> k` | Multiply / divide by 2ᵏ |
| `mask & (1 << i)` | Is item i in the subset? |

**Template:**

```js
// Enumerate all subsets of n items with bitmasks
function allSubsets(items) {
  const n = items.length, result = [];
  for (let mask = 0; mask < (1 << n); mask++) {
    result.push(items.filter((_, i) => mask & (1 << i)));
  }
  return result;
}
```

**Pitfalls:**
- JS bitwise operators convert to **signed 32-bit** integers: `1 << 31` is negative, and values ≥ 2³¹ change sign. Use `>>> 0` for unsigned results.
- Use `>>>` (logical shift) instead of `>>` when looping over the bits of a possibly negative number, or it never reaches 0.
- Operator precedence: `a & b === 0` means `a & (b === 0)`. Add parentheses.

### **22.1  Single Number**

**Problem:** Every element appears twice except one. Find it in O(n) time and O(1) space.

**Approach:** XOR all numbers. Pairs cancel (`a ^ a = 0`) and XOR is order-independent, so only the single number survives.

**Optimal Solution (JavaScript):**

```js
function singleNumber(nums) {
  let x = 0;
  for (const n of nums) x ^= n;
  return x;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *A hash set (add on first sight, delete on second) or counts map: O(n) time but O(n) space. Math: `2 * sum(unique) - sum(all)`, also O(n) space.*

☐ Practiced   ☐ Revised

### **22.2  Number of 1 Bits**

**Problem:** Return the number of set bits (Hamming weight) in an unsigned 32-bit integer.

**Approach:** `n & (n - 1)` clears the lowest set bit. Repeat until n is 0 and count the iterations, so the loop runs once per set bit rather than 32 times.

**Optimal Solution (JavaScript):**

```js
function hammingWeight(n) {
  let count = 0;
  while (n !== 0) {
    n &= n - 1; // clear the lowest set bit
    count++;
  }
  return count;
}
```

**Complexity:** Time O(number of set bits) ≤ O(32), Space O(1)

***Alternate approaches:*** *Check each of the 32 bits with `(n >>> i) & 1`. Use `>>>`, not `>>`, so values ≥ 2³¹ (negative in 32-bit) don't sign-extend. `n.toString(2)` and counting '1's works only for non-negative n.*

☐ Practiced   ☐ Revised

### **22.3  Counting Bits**

**Problem:** Given n, return an array where `ans[i]` is the number of set bits in i, for 0 ≤ i ≤ n, in O(n).

**Approach:** DP on bits: `i >> 1` is i with its last bit removed and has already been computed, so `bits[i] = bits[i >> 1] + (i & 1)`.

**Optimal Solution (JavaScript):**

```js
function countBits(n) {
  const bits = new Array(n + 1).fill(0);
  for (let i = 1; i <= n; i++) bits[i] = bits[i >> 1] + (i & 1);
  return bits;
}
```

**Complexity:** Time O(n), Space O(1) extra (excluding output)

***Alternate approaches:*** *`bits[i] = bits[i & (i - 1)] + 1` (drop the lowest set bit) also works. Calling Number of 1 Bits for every i is O(n log n).*

☐ Practiced   ☐ Revised

### **22.4  Reverse Bits**

**Problem:** Reverse the bits of a 32-bit unsigned integer.

**Approach:** 32 times: shift the result left, copy n's lowest bit into it, then shift n right logically. Finish with `>>> 0` so JS reads the result as unsigned.

**Optimal Solution (JavaScript):**

```js
function reverseBits(n) {
  let result = 0;
  for (let i = 0; i < 32; i++) {
    result = (result << 1) | (n & 1);
    n >>>= 1;
  }
  return result >>> 0; // read as unsigned 32-bit
}
```

**Complexity:** Time O(32) = O(1), Space O(1)

***Alternate approaches:*** *Divide and conquer with masks: swap the 16-bit halves, then 8-bit groups, 4, 2 and 1, giving 5 steps instead of 32. It's useful when called very often, and can be combined with a byte lookup table.*

☐ Practiced   ☐ Revised

### **22.5  Missing Number**

**Problem:** An array contains n distinct numbers from 0..n, so exactly one is missing. Find it in O(1) extra space.

**Approach:** XOR all indices 0..n with all values. Every present number cancels with its matching index, leaving only the missing one.

**Optimal Solution (JavaScript):**

```js
function missingNumber(nums) {
  let x = nums.length; // index n has no slot in the array
  for (let i = 0; i < nums.length; i++) x ^= i ^ nums[i];
  return x;
}
```

**Complexity:** Time O(n), Space O(1)

***Alternate approaches:*** *Gauss sum: `n * (n + 1) / 2 - sum(nums)`. It's simple, and safe in JS up to 2⁵³. Sorting and finding the gap is O(n log n). A hash set is O(n) space.*

☐ Practiced   ☐ Revised

### **22.6  Sum of Two Integers**

**Problem:** Return a + b without using `+` or `-`.

**Approach:** `a ^ b` adds without carries, and `(a & b) << 1` is the carry. Repeat with the partial sum and the carry until the carry is 0. JS bitwise operations are 32-bit two's complement, so negative numbers work and the carry disappears after at most 32 rounds.

**Optimal Solution (JavaScript):**

```js
function getSum(a, b) {
  while (b !== 0) {
    const carry = (a & b) << 1;
    a = a ^ b;  // sum without carry
    b = carry;
  }
  return a;
}
```

**Complexity:** Time O(32) = O(1), Space O(1)

***Alternate approaches:*** *Recursive form `getSum(a ^ b, (a & b) << 1)`. In Python, which has unbounded integers, you must mask to 32 bits manually; JS does this implicitly.*

☐ Practiced   ☐ Revised

# **Appendix — Blind 75 Checklist**

The Blind 75 is a widely used list of 75 core interview problems. All of them are in this guide:

| **Category** | **Problem** | **Section** |
| --- | --- | --- |
| Array | Two Sum | §1.1 |
| Array | Best Time to Buy and Sell Stock | §3.2 |
| Array | Contains Duplicate | §1.2 |
| Array | Product of Array Except Self | §4.4 |
| Array | Maximum Subarray | §20.1 |
| Array | Maximum Product Subarray | §21.5 |
| Array | Find Minimum in Rotated Sorted Array | §6.5 |
| Array | Search in Rotated Sorted Array | §6.6 |
| Array | 3Sum | §2.3 |
| Array | Container With Most Water | §2.4 |
| Binary | Sum of Two Integers | §22.6 |
| Binary | Number of 1 Bits | §22.2 |
| Binary | Counting Bits | §22.3 |
| Binary | Missing Number | §22.5 |
| Binary | Reverse Bits | §22.4 |
| Dynamic Programming | Fibonacci Number / Climbing Stairs | §21.1 |
| Dynamic Programming | Coin Change | §21.7 |
| Dynamic Programming | Longest Increasing Subsequence | §21.12 |
| Dynamic Programming | Longest Common Subsequence | §21.13 |
| Dynamic Programming | Word Break | §21.6 |
| Dynamic Programming | Combination Sum IV | §21.9 |
| Dynamic Programming | House Robber | §21.2 |
| Dynamic Programming | House Robber II | §21.3 |
| Dynamic Programming | Decode Ways | §21.4 |
| Dynamic Programming | Unique Paths | §21.15 |
| Dynamic Programming | Jump Game | §20.2 |
| Graph | Clone Graph | §15.3 |
| Graph | Course Schedule (BFS / Kahn's Algorithm) | §16.1 |
| Graph | Pacific Atlantic Water Flow | §15.4 |
| Graph | Number of Islands (DFS version) | §15.1 |
| Graph | Longest Consecutive Sequence | §1.6 |
| Graph | Alien Dictionary | §16.4 |
| Graph | Graph Valid Tree | §17.3 |
| Graph | Number of Connected Components in an Undirected Graph | §17.2 |
| Interval | Insert Interval | §7.4 |
| Interval | Merge Intervals | §7.3 |
| Interval | Non-overlapping Intervals | §7.5 |
| Interval | Meeting Rooms | §7.6 |
| Interval | Meeting Rooms II | §7.7 |
| Linked List | Reverse Linked List | §9.1 |
| Linked List | Linked List Cycle I & II | §9.4 |
| Linked List | Merge Two Sorted Lists | §9.2 |
| Linked List | Merge K Sorted Lists | §13.4 |
| Linked List | Remove Nth Node From End of List | §9.5 |
| Linked List | Reorder List | §9.6 |
| Matrix | Set Matrix Zeroes | §5.1 |
| Matrix | Spiral Matrix | §5.2 |
| Matrix | Rotate Image | §5.3 |
| Matrix | Word Search | §19.7 |
| String | Longest Substring Without Repeating Characters | §3.3 |
| String | Longest Repeating Character Replacement | §3.7 |
| String | Minimum Window Substring | §3.10 |
| String | Valid Anagram | §1.3 |
| String | Group Anagrams | §1.4 |
| String | Valid Parentheses | §8.1 |
| String | Valid Palindrome | §2.2 |
| String | Longest Palindromic Substring | §21.16 |
| String | Palindromic Substrings | §21.17 |
| String | Encode and Decode Strings | §1.10 |
| Tree | Maximum Depth of Binary Tree (Tree Height) | §10.1 |
| Tree | Same Tree | §10.2 |
| Tree | Invert Binary Tree | §10.3 |
| Tree | Binary Tree Maximum Path Sum | §10.9 |
| Tree | Binary Tree Level Order Traversal | §10.11 |
| Tree | Serialize and Deserialize Binary Tree | §10.14 |
| Tree | Subtree of Another Tree | §10.6 |
| Tree | Construct Binary Tree from Preorder and Inorder Traversal | §10.13 |
| Tree | Validate Binary Search Tree | §11.1 |
| Tree | Kth Smallest Element in a BST | §11.2 |
| Tree | Lowest Common Ancestor of a Binary Search Tree | §11.3 |
| Tree | Implement Trie (Prefix Tree) | §12.1 |
| Tree | Design Add and Search Words Data Structure | §12.2 |
| Tree | Word Search II | §12.3 |
| Heap | Top K Frequent Elements | §1.5 |
| Heap | Find Median from Data Stream | §13.6 |
