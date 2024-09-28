# [2707. Extra Characters in a String](https://leetcode.com/problems/extra-characters-in-a-string/description/)
>Medium

You are given a 0-indexed string `s` and a dictionary of words `dictionary`. You have to break `s` into one or more non-overlapping substrings such that each substring is present in `dictionary`. There may be some extra characters in `s` which are not present in any of the substrings.

Return the minimum number of extra characters left over if you break up `s` optimally.

---

## Initial Thoughts
The question is an optimization problem. Optimization problems tend to point towards Greedy or Dynamic Programming problems.

## Greedy?
Consider `s` = "foobarbaz", `dictionary` = ["foobarb", "foob", "arba"]
If we greedily select the largest string in `dictionary` such that we minimize our extra characters, we will not find the optimal solution as "foob" + "arba" produce a lower result.

## Dynamic Programming?
If we are to consider DP. We need to define 3 things.

### State Variables
`i` - we should consider a given index, such that we will consider all possible substrings starting at the index.

### Recurrence Relation
We need to define a way to transition between states.

`F(i) = max(F(j) + length of valid substring) for i <= j < n`

`where a substring is considered valid if it exists in dictionary`

### Base Case
We need a way to stop the recursion.

`F(n) = 0`

We reached the end of the string.

---

## Top Down Recursion (TLE)
![Recursion Tree](/IMG_6419.jpg)

```golang
func minExtraChar(s string, dictionary []string) int {
    n := len(s)

    // Initialize map for O(1) lookup
    dict := make(map[string]bool)
    for _, word := range dictionary {
        dict[word] = true
    }

    // Return total length - max valid chars
    return n - helper(s,0,dict)
}

func helper(s string, i int, dict map[string]bool) int {
    // Base case
    if i == len(s) {
        return 0
    }
    
    maxChars := 0

    // Consider all substrings starting at i
    for j := i; j < len(s); j++ {
        // Max valid chars starting at j+1
        totalChars := helper(s,j+1, dict)

        // Check if substring is valid
        if dict[string(s[i:j+1])] {
            totalChars += j - i + 1
        }

        // Update max valid chars
        maxChars = max(maxChars, totalChars)
    }

    return maxChars
}
```
### Time Complexity: $O(2^n)$
### Space Complexity: $O(n+m)$
Recursion stack can get up to `n` levels deep and the `dictionary` map is size `m`

---

## Add Memoization
We can see the repeated subproblems based off the image. Let's memoize the results so we don't recalculate solutions we already found.

```golang
func minExtraChar(s string, dictionary []string) int {
    n := len(s)

    // Initialize map for O(1) lookup
    dict := make(map[string]bool)
    for _, word := range dictionary {
        dict[word] = true
    }

    // Initialize memo to -1 for easy identification of when it is populated
    memo := make([]int, n)
    for i := range memo {
        memo[i] = -1
    }

    // Return total length - max valid chars
    return n - helper(s,0,dict,memo)
}

func helper(s string, i int, dict map[string]bool, memo []int) int {
    // Base case
    if i == len(s) {
        return 0
    }

    // Return if already computed
    if memo[i] != -1 {
        return memo[i]
    }
    
    maxChars := 0

    // Consider all substrings starting at i
    for j := i; j < len(s); j++ {
        // Max valid chars starting at j+1
        totalChars := helper(s,j+1, dict, memo)

        // Check if substring is valid
        if dict[string(s[i:j+1])] {
            totalChars += j - i + 1
        }

        // Update max valid chars
        maxChars = max(maxChars, totalChars)
    }

    // Store computed value for later use
    memo[i] = maxChars

    return memo[i]
}
```
### Time Complexity: $O(n^3)$
$O(n^2)$ Since we call our recursive function n times. In those n recursive calls, we iterate from i <= j < n which is upper bounded to n. In the n iterations, we construct a substring of upper bounded length n.
### Space Complexity: $O(n+m)$
Memo array is size `n`. Recursion stack can get up to `n` levels deep and the `dictionary` map is size `m`

---

## Bottom Up Tabulation

## Trie optimization

In our implementations, when we consider index `i` we will consider the following substrings, `s[i:j+1]`, `s[i:j+2]`, `s[i:j+k]` where j+k <= n. Each time we are reconstructing the substring from scratch. If we can reduce this redundant work, we can optimize our problem from $O(n^3)$ -> $O(n^2)$

This is where we utilize Trie data structure.

## Top Down Trie

```golang
// TrieNode structure
type TrieNode struct {
    children []*TrieNode
    isWord   bool
}

func (t *TrieNode) insert(word string) {
    pointer := t
    for _, char := range word {
        index := int(char - 'a')
        if pointer.children[index] == nil {
            pointer.children[index] = &TrieNode{make([]*TrieNode, 26), false}
        }
        pointer = pointer.children[index]
    }
    pointer.isWord = true
}

func minExtraChar(s string, dictionary []string) int {
    n := len(s)

    // Use Trie instead of Map for optimization
    trie := &TrieNode{make([]*TrieNode, 26), false}
    for _, word := range dictionary {
        trie.insert(word)
    }

    // Initialize memo to -1 for easy identification of when it is populated
    memo := make([]int, n)
    for i := range memo {
        memo[i] = -1
    }

    // Return total length - max valid chars
    return n - helper(s,0,trie,memo)
}

func helper(s string, i int, trie *TrieNode, memo []int) int {
    // Base case
    if i == len(s) {
        return 0
    }

    // Return if already computed
    if memo[i] != -1 {
        return memo[i]
    }
    
    include := 0

    pointer := trie
    // Consider all substrings starting at i
    for j := i; j < len(s); j++ {
        index := int(s[j] - 'a')
        // Traverse trie
        if pointer.children[index] == nil { // Not a valid substring
            break
        }

        pointer = pointer.children[index]

        if pointer.isWord {
            // Substring is valid, lets consider all substrings after this point
            include = max(include, helper(s,j+1, trie, memo) + (j - i + 1))
        }
    }

    // Consider all substrings starting after i
    notInclude := helper(s,i+1, trie, memo)

    // Store computed value for later use
    memo[i] = max(include, notInclude)

    return memo[i]
}
```
### Time Complexity: $O(n^2)$


---

## Bottom Up Trie