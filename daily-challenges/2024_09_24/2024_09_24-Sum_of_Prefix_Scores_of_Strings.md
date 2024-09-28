# [2416. Sum of Prefix Scores of Strings](https://leetcode.com/problems/sum-of-prefix-scores-of-strings/description/)
> Hard
> 
You are given an array `words` of size `n` consisting of non-empty strings.

We define the score of a string `word` as the number of strings `words[i]` such that `word` is a prefix of `words[i]`.

- For example, if `words = ["a", "ab", "abc", "cab"]`, then the score of `"ab"` is `2`, since `"ab"` is a prefix of both `"ab"` and `"abc"`.

Return an array `answer` of size `n` where `answer[i]` is the sum of scores of every non-empty prefix of `words[i]`.

__Note__ that a string is considered as a prefix of itself.

```
Example 1:

Input: words = ["abc","ab","bc","b"]
Output: [5,4,3,2]
Explanation: The answer for each string is the following:
- "abc" has 3 prefixes: "a", "ab", and "abc".
- There are 2 strings with the prefix "a", 2 strings with the prefix "ab", and 1 string with the prefix "abc".
The total is answer[0] = 2 + 2 + 1 = 5.
- "ab" has 2 prefixes: "a" and "ab".
- There are 2 strings with the prefix "a", and 2 strings with the prefix "ab".
The total is answer[1] = 2 + 2 = 4.
- "bc" has 2 prefixes: "b" and "bc".
- There are 2 strings with the prefix "b", and 1 string with the prefix "bc".
The total is answer[2] = 2 + 1 = 3.
- "b" has 1 prefix: "b".
- There are 2 strings with the prefix "b".
The total is answer[3] = 2.

Example 2:

Input: words = ["abcd"]
Output: [4]
Explanation:
"abcd" has 4 prefixes: "a", "ab", "abc", and "abcd".
Each prefix has a score of one, so the total is answer[0] = 1 + 1 + 1 + 1 = 4.
```

---

## Initial Thoughts
We are dealing with prefixes of a string. This is the third daily problem in a row in which we are dealing with prefixes. Nonetheless, recency bias shouldn't play a part in my intuition.

The problem states that we are looking for any prefix for any word in words such that the current word is a prefix of that word. 

The problem was intiially confusing since I thought we were looking for where the word is a prefix of other words, but we are actually looking for where every prefix of that given word is a prefx of another word.

Now the problem gets more interesting. We can clearly see that Trie traversal is favored here as we are dealing with multiple prefix traversals and comparisons.

We can also see that we don't care where a word ends in the Trie. We only care about the number of occurence that prefix as appeared. This solidified the use of Trie, but we need to modify it slighty.

```golang
type Trie struct {
    children []*Trie
    count int
}
```

This way we can count the number of occurence of each prefix.

Therefore after constructing the Trie. For each word in word, we will traverse the trie summing all counts we find. Each iteration is a prefix of the word word.

```
words = ["abc","ab","bc","b"]
Trie would look like this
    
root  -> (a,2) -> (b,2) -> (c,1)
      -> (b,2) -> (c,1)

Therefore, for "abc", we can calculate the score as follows.
"a"   -> (a,2)
"ab"  -> (b,2)
"abc" -> (c,1)

Summing the counts of each prefix, we get 2+2+1 = 5
```

---

## Trie
```golang
// Trie represents a node in a Trie structure.
type Trie struct {
    children []*Trie // List of child nodes (26 for each letter a-z)
    count    int     // Count of words passing through this node
}

// insert adds a word into the Trie.
func (t *Trie) insert(word string) {
    pointer := t
    for _, char := range word {
        index := int(char - 'a') // Convert character to index (0-25)
        if pointer.children[index] == nil {
            pointer.children[index] = &Trie{make([]*Trie, 26), 0}
        }
        pointer = pointer.children[index]
        pointer.count++ // Increment the count for each node in the path
    }
}

func (t *Trie) calculateScore(word string) int {
    pointer := t
    score := 0

    // Traverse the Trie for the current word's prefixes
    for _, char := range word {
        index := int(char - 'a')
        if pointer.children[index] == nil {
            break // Stop if the word path is no longer valid
        }
        pointer = pointer.children[index]
        score += pointer.count // Add count at each node to the score
    }

    return score
}

// sumPrefixScores calculates the prefix scores for each word in the input list.
func sumPrefixScores(words []string) []int {
    /*
        The score of a word is the number of words such that each prefix of the word
        is also a prefix of any word in the list.

        We use a Trie where each node holds a count of how many words passed through
        that node (representing prefixes). This allows us to calculate the score
        for each word by summing the counts of nodes corresponding to its prefixes.

        Example:
        words = ["abc", "ab", "bc", "b"]

        Trie representation:
        root -> (a,2) -> (b,2) -> (c,1)
             -> (b,2) -> (c,1)

        For "abc", the score is calculated by traversing the Trie:
        "a" -> count 2
        "ab" -> count 2
        "abc" -> count 1
        Total score for "abc" = 2 + 2 + 1 = 5
    */

    // Initialize the Trie with 26 children (for each letter a-z)
    trie := &Trie{make([]*Trie, 26), 0}

    // Insert each word into the Trie
    for _, word := range words {
        trie.insert(word)
    }

    // Result slice to store prefix scores for each word
    res := make([]int, len(words))

    // Calculate the score for each word
    for i, word := range words {
        res[i] = trie.calculateScore(word)
    }

    return res
}

```