# DSA from Zero — Week 2
### Two Pointers (two fingers walking through the data)

> Read slowly. This week builds directly on Week 1. Last week the big lesson was "use a HashSet/Dictionary to remember what you've seen, so you only need one pass." This week's lesson is different but just as powerful: "use **two markers** moving through the data so you still only need one pass — and often **no extra memory at all**." Go slow. By the end you'll solve three classic interview problems.

---

## Part 1: A quick reminder of where we are

Last week you learned:
- **Arrays** — a row of numbered boxes. Jump to a box = instant. Search by value = slow (O(n)).
- **HashSet / Dictionary** — instant "have I seen this?" lookups, at the cost of extra memory.
- The habit: when you feel a **loop inside a loop** (O(n²)) coming, stop and find a faster way.

This week is another tool for killing that loop-inside-a-loop — but a different shape of tool. Sometimes you don't need extra memory at all. You just need **two fingers.**

---

## Part 2: The big idea — two fingers instead of one

**Picture it:** you're reading a long shelf of books looking for two that fit together. The slow way is: pick book 1, compare it to every other book; then pick book 2, compare it to every other book; and so on. That's the handshake — O(n²).

The two-pointer way: put **one finger at the left end** of the shelf and **one finger at the right end.** Look at just those two. Based on what you see, move one finger inward. Keep going until the fingers meet in the middle.

```
   left finger →                        ← right finger
        ↓                                     ↓
      +-----+-----+-----+-----+-----+-----+-----+
      |  1  |  3  |  4  |  6  |  8  |  10 |  13 |
      +-----+-----+-----+-----+-----+-----+-----+
```

Because each finger only moves toward the other and they never go backwards, **together they take one pass** through the data. That's O(n) time. And because you're just holding two positions, you usually use **O(1) extra memory** — none of the extra storage a HashSet needs.

> **The whole pattern in one line:** keep two markers (pointers) into the array, look at what they point to, and move one or both based on a simple rule — until they meet.

---

## Part 3: The two common shapes of Two Pointers

Almost every two-pointer problem is one of these two shapes. Learn to recognise them.

### Shape A — "Opposite ends, walking inward"
Start one pointer at the **left (0)** and one at the **right (last index)**, and move them **toward each other.** Great for **sorted** arrays and for checking things from both ends (like palindromes).

### Shape B — "Both start at the left, one chases the other"
Both pointers start near the **beginning**; a **fast** one races ahead while a **slow** one follows. Great for filtering an array in place, removing duplicates, or "fast/slow" tricks. (You'll see the fast/slow idea again in Week 6 for linked lists.)

We'll do Shape A first because it's the most famous.

---

## Part 4: First problem — "Two Sum II" (sorted array)

This looks like last week's Two Sum, but with a twist: **the array is already sorted.** When data is sorted, two pointers usually beat a hash map — and use no extra memory.

**The problem:** given a **sorted** array and a target, find two numbers that add up to the target. Return their positions.
Example: numbers `{ 1, 3, 4, 6, 8, 10 }`, target `10`. Answer: `4 + 6 = 10`.

**Think like a human.** Put a finger on the smallest number (left) and the largest (right). Add them:
- If the sum is **too big**, the biggest number is too heavy — move the **right** finger left to a smaller number.
- If the sum is **too small**, you need more — move the **left** finger right to a bigger number.
- If it's **exactly right** — done!

Because the array is sorted, each move sends the sum in a predictable direction. That's the magic.

```csharp
int[] TwoSumSorted(int[] nums, int target)
{
    int left = 0;
    int right = nums.Length - 1;

    while (left < right)
    {
        int sum = nums[left] + nums[right];

        if (sum == target)
            return new[] { left, right };   // found the pair
        else if (sum < target)
            left++;                         // too small -> need a bigger number
        else
            right--;                        // too big -> need a smaller number
    }
    return Array.Empty<int>();              // no pair
}
```

**Watch it run** on `{ 1, 3, 4, 6, 8, 10 }`, target `10`:

| Step | left → value | right → value | Sum | Compare to 10 | Action |
|------|-------------|--------------|-----|---------------|--------|
| 1 | 0 → 1 | 5 → 10 | 11 | too big | move right left → 4 |
| 2 | 0 → 1 | 4 → 8 | 9 | too small | move left right → 3 |
| 3 | 1 → 3 | 4 → 8 | 11 | too big | move right left → 6 |
| 4 | 1 → 3 | 3 → 6 | 9 | too small | move left right → 4 |
| 5 | 2 → 4 | 3 → 6 | 10 | **exact!** | return [2, 3] |

One pass, no extra memory. The slow version would check every pair (O(n²)). Here the fingers walked toward each other just once — **O(n) time, O(1) space.**

---

## Part 5: Second problem — "Valid Palindrome" (read from both ends)

A **palindrome** is a word that reads the same forwards and backwards, like "racecar" or "level".

**The human way:** put one finger at the start, one at the end. Check the two letters match. If they do, step both fingers inward and check again. If you ever find a mismatch, it's not a palindrome. If the fingers cross without any mismatch, it is.

```csharp
bool IsPalindrome(string s)
{
    int left = 0;
    int right = s.Length - 1;

    while (left < right)
    {
        if (s[left] != s[right])
            return false;      // ends don't match -> not a palindrome
        left++;                // step both fingers inward
        right--;
    }
    return true;               // never mismatched -> it's a palindrome
}
```

**Watch it run** on `"racecar"`:

| Step | left → char | right → char | Match? | Action |
|------|------------|-------------|--------|--------|
| 1 | 0 → r | 6 → r | yes | move inward |
| 2 | 1 → a | 5 → a | yes | move inward |
| 3 | 2 → c | 4 → c | yes | move inward |
| 4 | 3 → e | 3 → e | left == right, loop ends | return **true** |

This is **Shape A** again: opposite ends walking inward. Same idea as Two Sum, different question. *(Real interview versions ask you to ignore spaces, punctuation, and capital letters — a nice follow-up once the basic version is solid.)*

---

## Part 6: Third problem — "Remove Duplicates from Sorted Array" (Shape B)

Now the other shape: **both pointers start at the left, one chases the other.**

**The problem:** you have a **sorted** array like `{ 1, 1, 2, 3, 3 }`. Remove the duplicates *in place* so the front of the array becomes `{ 1, 2, 3 }`. Return how many unique numbers there are (3).

**The human way:** keep a **slow** finger that marks "the last unique number I've locked in." Send a **fast** finger ahead to explore. Whenever the fast finger finds a number different from the slow finger's number, it's a new unique value — move slow forward one and copy it there.

```csharp
int RemoveDuplicates(int[] nums)
{
    if (nums.Length == 0) return 0;

    int slow = 0;   // last position holding a confirmed unique value

    for (int fast = 1; fast < nums.Length; fast++)   // fast explores ahead
    {
        if (nums[fast] != nums[slow])   // found a brand-new value
        {
            slow++;                     // make room
            nums[slow] = nums[fast];    // place the new unique value
        }
    }
    return slow + 1;                    // count of unique values
}
```

**Watch it run** on `{ 1, 1, 2, 3, 3 }`:

| fast → value | nums[slow] | New value? | Action | Array front so far |
|-------------|-----------|-----------|--------|--------------------|
| 1 → 1 | 1 | no (same) | skip | {1 \| ...} |
| 2 → 2 | 1 | yes | slow→1, nums[1]=2 | {1,2 \| ...} |
| 3 → 3 | 2 | yes | slow→2, nums[2]=3 | {1,2,3 \| ...} |
| 4 → 3 | 3 | no (same) | skip | {1,2,3 \| ...} |

Returns `slow + 1 = 3`. The first three slots are `{1, 2, 3}`. One pass, no extra array — **O(n) time, O(1) space.** A HashSet would also work but would cost extra memory; two pointers does it in place.

---

## Part 7: When do I reach for Two Pointers?

Train your eye to spot these signals:

- The array is **sorted** (or you can sort it first). Sorting unlocks the "move left/right based on the sum" trick.
- You're looking for a **pair** (or triple) that meets some condition — sum, difference, product.
- You're checking something **from both ends** — palindromes, reversing, "is it symmetric?"
- You need to **filter or rewrite an array in place** without extra memory — removing duplicates, moving zeros to the end, squashing values.
- The brute force is a **loop inside a loop**, and the data has order you can exploit.

**Two Pointers vs last week's HashSet — how to choose:**
- Array **already sorted**, or you must use **no extra memory** → **Two Pointers** usually wins.
- Array **unsorted** and you can spend memory for speed → a **HashSet/Dictionary** (Week 1) is often simpler.

Knowing *which* tool fits is exactly what interviewers are testing.

---

## Part 8: The big lesson of Week 2 (say this out loud)

> "When the data is **sorted**, or I'm comparing things **from both ends**, or I need to fix an array **in place** — I should reach for **two pointers** instead of a loop inside a loop. Two fingers walking through the data give me O(n) time and often O(1) memory."

Two patterns down (Hashing, Two Pointers). These two alone unlock a huge chunk of "easy" and "medium" interview questions.

---

## Part 9: Tiny practice (start very small)

Do these by hand on paper first, then in code.

**P1.** Run `TwoSumSorted` by hand on `{ 2, 3, 5, 8 }`, target `10`. Make the left/right table like in Part 4. (Answer: positions 0 and 3, because 2 + 8 = 10.)

**P2.** Run `IsPalindrome` by hand on `"level"` and on `"hello"`. One returns true, one returns false — show the step where "hello" fails.

**P3.** Run `RemoveDuplicates` by hand on `{ 5, 5, 5, 7 }`. What does the front of the array look like, and what number is returned? (Answer: `{5, 7}`, returns 2.)

**P4.** Write a method `ReverseInPlace(char[] s)` that reverses a character array using two pointers from both ends (swap left and right, then step inward). Test it on `['h','e','l','l','o']`.

**P5 (when ready).** On **neetcode.io**, in the "Two Pointers" section, try **"Valid Palindrome"** and then **"Two Sum II — Input Array Is Sorted."** You already wrote both above — now type them in and pass the tests. Two more solved interview problems. 🎉

---

## Part 10: Words to remember (your mini-dictionary)

- **Pointer** — here it just means a **position/index** into an array (a "finger"), not the C/C++ memory pointer.
- **Two Pointers** — a technique using two positions that move through the data, replacing a loop-inside-a-loop.
- **Opposite-ends (Shape A)** — left starts at 0, right starts at the end, they walk inward. Good for sorted pairs and palindromes.
- **Fast/slow (Shape B)** — both start at the front; fast explores ahead, slow marks confirmed results. Good for in-place filtering.
- **In place** — changing the array directly without making a new one (saves memory → O(1) space).
- **Sorted** — in order. Sorting is the key that makes the "move left/right based on the sum" logic work.
- **O(1) space** — uses a fixed, tiny amount of extra memory no matter how big the input is.

---

## Self-check before moving on
You are ready when you can, in your own simple words:
1. Explain the two-fingers picture and why it's one pass (O(n)) instead of O(n²).
2. Describe the two shapes (opposite-ends, and fast/slow) and when each fits.
3. Solve Two Sum on a **sorted** array with two pointers — and say why you move left vs right.
4. Check a palindrome from both ends.
5. Say when you'd pick Two Pointers over a HashSet (sorted data / no extra memory).

When all five feel easy, you've finished your second DSA pattern. Two down — real momentum now. Next week's pattern: **Sliding Window** (a cousin of Two Pointers).
