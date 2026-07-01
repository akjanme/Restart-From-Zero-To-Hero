# DSA from Zero — Week 3
### Sliding Window (a moving frame over the data)

> Read slowly. This week builds directly on Week 2. Last week's lesson was "use two fingers at opposite ends, walking inward." This week's lesson is a close cousin: "use **two fingers walking the same direction**, keeping a **window** open between them, and slide that window across the data." Same family — one pass, little or no extra memory — but a different shape of problem: anything about a **contiguous chunk** (a run of neighbours) of an array or string.

---

## Part 1: A quick reminder of where we are

So far:
- **Week 1 — Hashing:** remember what you've seen with a `HashSet`/`Dictionary`, trading memory for instant lookups.
- **Week 2 — Two Pointers:** two fingers, usually at **opposite ends**, walking **toward** each other. Great for sorted pairs and checking from both ends.

This week — **Sliding Window:** two fingers that both start near the **left**, and instead of walking toward each other, they define a **window** (a contiguous slice) that **grows and shrinks as it slides rightward** across the data. It's built for questions about "the best contiguous chunk" — a subarray or a substring.

---

## Part 2: The big idea — a window sliding across the data

**Picture it:** you're looking through a physical picture frame that you slide along a long mural on a wall. The frame shows you a **chunk** of the mural at a time — some fixed or flexible width. As you slide it one step to the right, you **lose sight of the left edge** and **reveal a bit of new mural on the right edge.**

```
  window                      window (slid one step right)
 +-------+                    +-------+
 | 3 1 4 | 1 5 9 2 6          3 | 1 4 1 | 5 9 2 6
 +-------+                    +-------+
```

The key insight that makes this fast: when you slide the window one step, you don't need to re-look at everything inside it again. You just:
1. **Remove** the item that fell off the left edge, and
2. **Add** the item that just entered on the right edge.

That's it — O(1) work per slide, instead of re-scanning the whole window every time. Across the whole array, that's **O(n) total**, instead of the brute-force O(n²) or O(n·k) of re-checking every window from scratch.

> **The whole pattern in one line:** keep a left and right edge marking a contiguous chunk; grow the right edge to include more, shrink the left edge to remove what you don't need, and never restart from scratch.

---

## Part 3: The two shapes of Sliding Window

### Shape A — Fixed-size window
The window size **k** is given up front and never changes. You slide it across and check something (usually a sum or count) at each position.

**Signal:** the problem says "a window/subarray of size **k**."

### Shape B — Variable-size window (grow and shrink)
The window size is **not fixed**. You **grow** the right edge to try to satisfy a condition, and **shrink** the left edge whenever the window becomes invalid (or you want the smallest/best valid window).

**Signal:** the problem says "longest/shortest **substring/subarray** that satisfies some condition" — no fixed size mentioned.

We'll do Shape A first, then two Shape B problems (one for strings, one for arrays).

---

## Part 4: First problem — "Maximum Sum Subarray of Size K" (fixed window)

**The problem:** given an array and a fixed size `k`, find the maximum sum of any `k` consecutive numbers.
Example: `{ 2, 1, 5, 1, 3, 2 }`, `k = 3`. Answer: `9` (from `5 + 1 + 3`).

**Think like a human — the slow way first:** for every starting position, add up the next `k` numbers. That's a loop inside a loop: O(n·k).

**The sliding window way:** build the **first** window's sum normally. Then, to slide right by one step, just **subtract the number leaving on the left** and **add the number entering on the right**. Never re-add the whole window.

```csharp
int MaxSumSubarray(int[] nums, int k)
{
    int windowSum = 0;

    // build the FIRST window
    for (int i = 0; i < k; i++)
        windowSum += nums[i];

    int maxSum = windowSum;

    // slide the window one step at a time
    for (int end = k; end < nums.Length; end++)
    {
        windowSum += nums[end];          // add the new right edge
        windowSum -= nums[end - k];      // remove the old left edge
        maxSum = Math.Max(maxSum, windowSum);
    }

    return maxSum;
}
```

**Watch it run** on `{ 2, 1, 5, 1, 3, 2 }`, `k = 3`:

| Step | Window | Sum | Max so far |
|------|--------|-----|-----------|
| build first window | `[2,1,5]` | 8 | 8 |
| slide (add 1, remove 2) | `[1,5,1]` | 7 | 8 |
| slide (add 3, remove 1) | `[5,1,3]` | 9 | **9** |
| slide (add 2, remove 5) | `[1,3,2]` | 6 | 9 |

Answer: `9`. Each slide did just **one add and one subtract** — O(1) per step, O(n) total, instead of recomputing every window from scratch.

---

## Part 5: Second problem — "Longest Substring Without Repeating Characters" (variable window, strings)

**The problem:** given a string, find the length of the longest substring where **no character repeats**.
Example: `"abcabcbb"`. Answer: `3` (the substring `"abc"`).

**Think like a human.** Grow the window on the right by including new characters, and track which characters are currently inside the window with a `HashSet` (Week 1!). The moment you're about to add a character that's **already in the window**, that's a violation — so **shrink from the left**, removing characters, until the duplicate is gone. Then keep growing.

```csharp
int LongestUniqueSubstring(string s)
{
    HashSet<char> window = new HashSet<char>();
    int left = 0;
    int maxLen = 0;

    for (int right = 0; right < s.Length; right++)
    {
        // shrink from the left WHILE the new character would be a duplicate
        while (window.Contains(s[right]))
        {
            window.Remove(s[left]);
            left++;
        }

        window.Add(s[right]);                     // grow: include the new character
        maxLen = Math.Max(maxLen, right - left + 1);  // current window size
    }

    return maxLen;
}
```

**Watch it run** on `"abcabcbb"`:

| right → char | Window before | Duplicate? | Action | Window after | Length | Max so far |
|--------------|---------------|-----------|--------|---------------|--------|-----------|
| 0 → a | {} | no | add a | {a} | 1 | 1 |
| 1 → b | {a} | no | add b | {a,b} | 2 | 2 |
| 2 → c | {a,b} | no | add c | {a,b,c} | 3 | **3** |
| 3 → a | {a,b,c} | yes (a) | remove a, left→1 | {b,c} → add a → {b,c,a} | 3 | 3 |
| 4 → b | {b,c,a} | yes (b) | remove b, left→2 | {c,a} → add b → {c,a,b} | 3 | 3 |
| 5 → c | {c,a,b} | yes (c) | remove c, left→3 | {a,b} → add c → {a,b,c} | 3 | 3 |
| 6 → b | {a,b,c} | yes (b) | shrink until b gone, left→5 | add b → {c,b} | 2 | 3 |
| 7 → b | {c,b} | yes (b) | shrink until b gone, left→7 | add b → {b} | 1 | 3 |

Answer: `3`. Notice `left` only ever moves **forward**, never backward — that's why the whole scan is still O(n) even though it looks like a loop inside a loop (the inner `while` only ever does a total of `n` steps across the *entire* run, not per outer step).

---

## Part 6: Third problem — "Minimum Size Subarray Sum" (variable window, shrink to optimise)

**The problem:** given an array of **positive** numbers and a target sum, find the length of the **smallest** contiguous subarray whose sum is **at least** the target. Return `0` if no such subarray exists.
Example: `{ 2, 3, 1, 2, 4, 3 }`, target `7`. Answer: `2` (the subarray `{4, 3}`).

**Think like a human.** Grow the window on the right, adding to a running sum. The **moment** the running sum reaches the target, you have a **valid** window — now try to shrink it from the left as much as possible while it's still valid, recording the smallest length you find along the way. Then keep growing.

```csharp
int MinSubArrayLen(int target, int[] nums)
{
    int left = 0;
    int windowSum = 0;
    int minLen = int.MaxValue;

    for (int right = 0; right < nums.Length; right++)
    {
        windowSum += nums[right];                 // grow: include the new right edge

        // shrink from the left WHILE the window is still valid (>= target)
        while (windowSum >= target)
        {
            minLen = Math.Min(minLen, right - left + 1);
            windowSum -= nums[left];
            left++;
        }
    }

    return minLen == int.MaxValue ? 0 : minLen;
}
```

**Watch it run** on `{ 2, 3, 1, 2, 4, 3 }`, target `7`:

| right → value | windowSum | ≥ target? | Shrink action | left after | minLen so far |
|---------------|-----------|-----------|---------------|-----------|---------------|
| 0 → 2 | 2 | no | — | 0 | ∞ |
| 1 → 3 | 5 | no | — | 0 | ∞ |
| 2 → 1 | 6 | no | — | 0 | ∞ |
| 3 → 2 | 8 | yes | shrink: sum→6 (removed 2), left→1; stop (6<7) | 1 | 4 |
| 4 → 4 | 10 | yes | shrink: sum→7 (removed 3), len=3, left→2; shrink: sum→6 (removed 1), left→3; stop | 3 | **3** |
| 5 → 3 | 9 | yes | shrink: sum→7 (removed 2), len=2, left→4; shrink: sum→3 (removed 4), left→5; stop | 5 | **2** |

Answer: `2`. Notice how the window **grows and shrinks like breathing** — expanding until valid, then contracting as far as possible while staying valid — always chasing the smallest valid size.

---

## Part 7: When do I reach for Sliding Window?

Train your eye to spot these signals:

- The question is about a **contiguous** chunk — a **subarray** or **substring** (neighbours only, not "any pair" like Two Pointers/Two Sum).
- You see the words **"longest," "shortest," "maximum," "minimum,"** or **"at most/at least K"** applied to a run of elements.
- A **fixed window size `k`** is mentioned directly → Shape A (fixed window).
- No fixed size is mentioned, but there's a **condition to satisfy** (sum, character uniqueness, count of something) → Shape B (variable window, grow/shrink).
- The brute force is "check every possible contiguous chunk" — O(n²) or worse — and you notice that sliding one step only changes the window by one element at each edge.

**Sliding Window vs Two Pointers — how to tell them apart:**
- Two Pointers (Week 2): fingers usually move **toward each other** from opposite ends; about **pairs** or checking from both ends.
- Sliding Window (this week): fingers move in the **same direction**, left chasing right; about a **contiguous run** that grows and shrinks.

They're cousins from the same "avoid the loop inside a loop" family — the difference is the *shape* of what you're looking for: a pair vs. a stretch of neighbours.

---

## Part 8: The big lesson of Week 3 (say this out loud)

> "When I need the best (longest, shortest, max-sum) **contiguous** chunk of an array or string, I should reach for a **sliding window**: grow the right edge to include more, shrink the left edge when the window becomes invalid or I want it smaller, and never recompute the whole window from scratch. That's O(n) time instead of O(n²)."

Three patterns down now — Hashing, Two Pointers, Sliding Window. Together these three cover a huge share of "easy" and "medium" array/string interview questions.

---

## Part 9: Tiny practice (start very small)

Do these by hand on paper first, then in code.

**P1.** Run `MaxSumSubarray` by hand on `{ 1, 4, 2, 10, 2, 3, 1, 0, 20 }`, `k = 4`. Build the add/remove table like in Part 4. (Answer: `24`, from the last window `{3,1,0,20}`.)

**P2.** Run `LongestUniqueSubstring` by hand on `"pwwkew"`. Show the step where the window shrinks because of the second `w`. (Answer: `3`, from `"wke"`.)

**P3.** Run `MinSubArrayLen` by hand on `{ 1, 4, 4 }`, target `4`. (Answer: `1`, because a single `4` already meets the target.)

**P4.** Write `MaxVowelsInWindow(string s, int k)` — the maximum number of vowels in any window of size `k` (fixed window, Shape A, similar structure to P1 but counting vowels instead of summing numbers).

**P5 (when ready).** On **neetcode.io**, in the "Sliding Window" section, try **"Best Time to Buy and Sell Stock"** and **"Longest Substring Without Repeating Characters."** You already wrote the second one above — now type it in and pass the tests.

---

## Part 10: Words to remember (your mini-dictionary)

- **Window** — a contiguous chunk of the array/string, marked by a `left` and `right` edge.
- **Fixed window (Shape A)** — the window size `k` never changes; you slide it and update the running value by removing the old left edge and adding the new right edge.
- **Variable window (Shape B)** — the window size changes; you grow the right edge to try to satisfy a condition, and shrink the left edge when the window becomes invalid (or to find the smallest valid one).
- **Contiguous** — neighbours, in a row, with no gaps — as opposed to "any pair anywhere" like Week 2's Two Sum.
- **Running sum/count** — a value you update incrementally (add/remove one item at a time) instead of recalculating from scratch each time — this is *why* the pattern is O(n) instead of O(n²).

---

## Self-check before moving on
You are ready when you can, in your own simple words:
1. Explain the sliding-window picture and why sliding costs O(1) per step instead of re-scanning the whole window.
2. Describe the two shapes (fixed size, and grow/shrink) and how to recognise which one a problem wants.
3. Solve "max sum subarray of size k" using the add-new/remove-old trick.
4. Solve "longest substring without repeating characters" using a `HashSet` plus shrink-on-duplicate.
5. Explain why the inner `while` loop in the variable-window problems doesn't break the O(n) time — because `left` only ever moves forward, across the whole run.
6. Say when you'd pick Sliding Window over Two Pointers (contiguous run vs. a pair/both-ends check).

When all six feel easy, you've finished your third DSA pattern. Three down — Hashing, Two Pointers, Sliding Window. Next week's pattern: **Stack** (paired with Async/Await & Concurrency in your C# track).
