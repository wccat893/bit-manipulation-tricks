# bit manipulation interview questions: the 12 one-liners that solve most of them, plus the problem list actually worth practicing

Bit manipulation questions feel unfair because they aren't really problem-solving questions. They're recall questions wearing a costume.

There are roughly a dozen identities — `n & (n-1)`, `n & -n`, the XOR cancellation rule — and once you have them, most "hard" bit problems collapse into a two-line solution. Without them, no amount of staring at the whiteboard for 45 minutes is going to get you there. That's the whole difficulty curve. It's not like dynamic programming, where you can reason your way toward a recurrence from first principles. Bit tricks have to be in your hands before the interview starts.

So here's what's actually asked, how each one breaks down, and where the traps are that quietly cost people offers.

## The 12 tricks behind nearly every question

Memorize this table. Seriously — writing these from memory on a scratch pad before you start a bit problem is worth the 30 seconds.

| What you want | Expression | Note |
| --- | --- | --- |
| Test bit k | `n & (1 << k)` | Non-zero means set |
| Set bit k | `n \| (1 << k)` |  |
| Clear bit k | `n & ~(1 << k)` |  |
| Toggle bit k | `n ^ (1 << k)` |  |
| Multiply / divide by 2^k | `n << k` / `n >> k` | Right shift truncates toward negative infinity |
| Check even | `(n & 1) == 0` | Faster than `% 2` |
| Power of two | `n > 0 && (n & (n-1)) == 0` | The `n > 0` matters |
| Clear lowest set bit | `n & (n-1)` | Basis of Kernighan's popcount |
| Isolate lowest set bit | `n & -n` | Relies on two's complement |
| Lowest k bits as a number | `(1 << k) - 1` | Your subset/prefix mask |
| XOR cancellation | `x ^ x = 0`, `x ^ 0 = x` | Order doesn't matter |
| Swap two ints, no temp | `a ^= b; b ^= a; a ^= b;` | Interviewers ask this less than they used to |

Two things about the table that trip people up.

`n & (n-1)` works because subtracting 1 from a number flips the lowest set bit to 0 and everything below it to 1. AND those two together and the lowest set bit disappears. Run it in a loop and each iteration removes one more 1 bit, which is exactly Kernighan's algorithm for counting set bits — O(number of set bits) instead of O(32).

`n & -n` works because of two's complement. Negating a number inverts every bit and adds 1, which leaves the lowest set bit untouched and flips everything else. That's also why the mask trick is `n & ~(1 << k)` rather than some arithmetic subtraction.

## The questions asked most often, from warmup to genuinely hard

Roughly in order of how often they show up. I've grouped them by how much thinking they need beyond the table above.

**Warmups (should be reflex)**

1. **Number of 1 Bits / Hamming weight.** Loop with `n &= n - 1` and count iterations. Mention that most languages have a builtin (`Integer.bitCount`, `__builtin_popcount`, Python's `int.bit_count()`), then note the interviewer probably wants the manual version. That's usually the real question.
2. **Power of Two.** `n > 0 && (n & (n-1)) == 0`. The edge case they're testing is `n = 0` and negative inputs.
3. **Single Number.** Every element appears twice except one. XOR the whole array; pairs cancel; you're left with the answer. O(n) time, O(1) space.
4. **Missing Number.** Array holds 0 to n with one missing. XOR all indices and all values — everything except the missing one cancels. The sum formula `n(n+1)/2` also works but can overflow in fixed-width languages; XOR can't.
5. **Counting Bits.** For every i from 0 to num, return its popcount. The trick is a DP: `dp[i] = dp[i >> 1] + (i & 1)`, or `dp[i] = dp[i & (i-1)] + 1`. Either one turns an O(n log n) loop into O(n).

**Middle tier (needs the table plus a step)**

6. **Reverse Bits.** Pull the lowest bit off the input, push it onto the result, repeat 32 times. O(1) since the loop is fixed-length.
7. **Two numbers appear once, everything else twice.** XOR everything to get `a ^ b`. Find the lowest bit where they differ with `xor & -xor`. That bit splits the array into two groups, and XOR-ing each group separately gives you `a` and `b`. The partition step is the actual question.
8. **Every element appears three times except one.** XOR is useless here. Count how many times each of the 32 bit positions is set across the array, take each count mod 3, and reassemble the number. Slower and less elegant, and that's fine — interviewers asking this want to see whether you abandon the XOR reflex when it doesn't fit.
9. **Bitwise AND of a range [m, n].** Shift both numbers right until they're equal, remember how many shifts, shift back. What you're really doing is keeping the common binary prefix and zeroing everything below it.
10. **Hamming distance / bits to flip to convert A into B.** `popcount(a ^ b)`. That's a one-liner and a good warmup question.
11. **Sum of two integers without `+`.** Loop: `sum = a ^ b`, `carry = (a & b) << 1`, then repeat with `a = sum, b = carry` until the carry is zero. In Python you must mask with `0xFFFFFFFF` and handle the sign at the end, because Python integers are arbitrary precision and never wrap. If you get this in a Python interview, say that out loud before you code — it shows you know the runtime's semantics, not just the algorithm.
12. **Divide two integers without `*`, `/`, or `%`.** Shift-and-subtract: find the largest shift of the divisor that still fits into the dividend, subtract, accumulate the quotient bit. Getting the sign handling and the overflow case (`INT_MIN / -1`) right is most of the work.
13. **Swap odd and even bits.** Mask with `0xAAAAAAAA` and `0x55555555`, shift each half by one, OR them back. This is the "do you know hex masks" question.

**The bitmask family (different skill, higher ceiling)**

These aren't about bit tricks so much as using an integer as a set. If you're targeting senior loops or competitive-programming-adjacent teams, this is where the real difficulty lives.

14. **Subsets.** Represent each subset as an integer from 0 to `(1 << n) - 1` and include element `i` when bit `i` is set. O(n · 2ⁿ), and much shorter than the recursive version.
15. **Maximum XOR of two numbers in an array.** Build a binary trie of the 32-bit representations, then for each number walk the opposite branch wherever possible. Greedy prefix sets also work. This one explicitly combines a data structure with bit reasoning.
16. **Bitmask DP.** `dp[mask][i]` for traveling salesman-style problems, or DP over subsets for partition-into-k-equal-subsets. The state is a bitmask because the set of visited nodes is the state.
17. **Total Hamming distance across all pairs.** Count set bits per bit position, and each position contributes `ones × zeros`.

If a problem says "only one element appears once," "no extra space," "O(1) space," or "without using the `+` operator," the intended answer is almost certainly a bit trick.

## The four mistakes that actually cost people the offer

Most candidates who know the tricks still fumble something adjacent. These come up constantly.

**Sign extension on right shift.** In Java and most C-family languages, `>>` on a negative number fills with the sign bit, so `-1 >> 1` is still `-1`, never `0`. If your loop is `while (c != 0) c >>= 1`, a negative input loops forever. Java has `>>>` for a logical shift; Python doesn't have this problem because its shift is arithmetic on arbitrary-precision ints, but that introduces the mirror-image problem of never wrapping at 32 bits. Say which language you're assuming and why it matters.

**Python's unbounded integers.** `~5` is `-6` in Python, not `0b...1010`. Any solution that relies on a fixed 32-bit width needs an explicit mask. This is the single most common source of "the code is right but the test fails" moments in Python bit problems.

**Operator precedence.** The shift operators bind tighter than `&`, `^`, and `|`, but *looser* than `+`. So `1 << k - 1` parses as `1 << (k - 1)`, not `(1 << k) - 1`. And `n & 1 << k` is `n & (1 << k)`. Just write the parentheses. Every time. Nobody has ever been marked down for clear parentheses in an interview.

**Treating bit problems as arithmetic.** The moment you catch yourself writing `n / 2` or a modulo, stop and ask whether a shift or mask does it more directly. That reflex — noticing that a problem is secretly about binary representation — is the thing being evaluated, more than the final code.

## How to narrate a bit solution without sounding like you memorized it

The gap between "I think XOR works" and an answer that lands is mostly explanation. A version that works:

> "Since XOR cancels identical values and preserves unique ones, I can XOR the entire array in one pass. Duplicates vanish, and what's left is the single number. That's O(n) time and O(1) space — no hash set needed."

That tells the interviewer the *property* you're relying on, not just the operator you typed. For the two-singletons variant, add the reason for the partition: "`a ^ b` is non-zero, so there's at least one bit where they differ; I can use the lowest such bit as a dividing line, because every pair still lands on the same side of it."

When you get a problem where bit tricks don't apply — say, the "appears three times" variant — saying so is a plus. "XOR cancellation doesn't help here because there's no pair to cancel. I'll fall back to counting bits per position and taking mod 3." Candidates who switch tools visibly read as stronger than candidates who force the trick they practiced.

## A five-day plan that's realistic about the size of this topic

Bit manipulation is a small topic. It is not worth two weeks of grinding, and it is definitely not worth ignoring either, because it shows up as the "one weird question" in otherwise standard loops.

- **Day 1:** The table above, from memory, until you can reproduce it without thinking. Then the five warmups.
- **Day 2:** Middle tier — reverse bits, Hamming distance, AND of a range, swap odd/even bits. Write each in your interview language *and* note where that language's integer semantics differ.
- **Day 3:** The two XOR partition problems and the "appears three times" variant. These are the ones most likely to be asked at a level above new-grad.
- **Day 4:** Bitmasks — subsets, then one bitmask DP. If TSP-style DP feels out of reach, at least be able to explain the state definition.
- **Day 5:** Re-solve everything cold, timed, no notes. Whatever you can't reproduce is what you actually need to review.

That's maybe ten hours total, which is the right amount.

## Where pattern-based prep fits, if you want the tricks organized for you

The reason bit questions feel like a separate universe is that they *are* a separate pattern. They don't share much machinery with sliding window or BFS. Which is also why they're easy to skip and easy to get blindsided by.

[AlgoMonster](https://algo.monster/subscribe?fpr=li69) is built around exactly that idea — a fixed library of interview patterns, with bit manipulation as one of the categories, plus decision flowcharts for figuring out which pattern a new problem belongs to. The relevant features for this topic specifically:

- **A short conceptual lesson plus a code template per pattern**, then three to five problems that use it. For bit manipulation that's the right shape, because the lesson *is* the trick.
- **Speedrun mode**, which serves multiple-choice questions about technique and template selection rather than full code. For bit manipulation, where recognition of the trick is 80% of the battle, that's a better use of time than writing `n & (n-1)` out for the ninth time.
- **An inline IDE with an AI assistant** that can explain selected text or debug a failing case, which helps when the failure is a Python masking issue rather than a logic error.
- **A free practice list** including Blind 75, and free-tier access to the question sets, so you can evaluate the format before paying.

The platform was built by ex-Googlers and competitive programmers, and the site advertises 3,000+ editorials alongside its lesson library. It's entirely text and diagrams — no video walkthroughs — which is a real limitation if you learn better from watching someone solve a problem on screen.

### Current AlgoMonster plans and pricing

Pricing moves with their promotions, so treat these as the current promotional figures rather than fixed numbers. The list prices were the higher figures shown as struck through at the time of writing.

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| **Free** | Access to practice questions and sample content; enough to test the teaching format | $0 | — | [Start free and try the pattern lessons](https://algo.monster?fpr=li69) |
| **Pro — Yearly** | All patterns, lessons and problems unlocked, unlimited AI assistant and Speedrun, company-specific question sets, 12 months of access | $99/year promo (list ~$198), about $8.25/month | Billed annually | [Check the current yearly price](https://algo.monster/subscribe?fpr=li69) |
| **Pro — Lifetime** | Everything in the yearly plan plus permanent access and all future updates, and an internal referral to a top tech company for qualified candidates | $189 one-time (list ~$378) | One-time payment | [See the lifetime plan](https://algo.monster/subscribe?fpr=li69) |

A note on those numbers: the yearly price has been seen as low as $69.99 and the lifetime as low as $89 during deeper sales, so the discount depth genuinely fluctuates. If you're deciding between the two, the math is straightforward — if you expect to interview more than twice over the next several years, the lifetime plan pays for itself; if you have one interview loop in the next two months, the yearly plan is the sensible buy and there's no reason to overpay for access you won't use.

One thing the platform doesn't cover well at any price: system design for senior roles, and live mock interviews with a real person. Bit manipulation prep won't be your bottleneck there, but budget for a separate resource.

## Quick answers to the questions people search for

**How many bit manipulation questions will I actually get?** Usually zero to one per loop, and it's typically a warmup or a "one weird problem" late in the round. It's a small slice, but the formatting matters — it's an easy place to look unprepared because the failure mode is a blank stare rather than a slow solution.

**Do I need to memorize the tricks?** For the twelve in the first table, yes. There's no reliable way to re-derive `n & -n` under time pressure if you've never seen it. For everything else — trie-based maximum XOR, bitmask DP — understanding the idea is enough; those are hard for everyone.

**Is the `n & (n-1)` trick worth learning separately from popcount?** They're the same trick. Understanding why it clears the lowest set bit is what lets you use it in power-of-two checks, missing-number problems, and the DP for counting bits. Learn it once, properly.

**What language should I practice in?** The one you'll interview in, and specifically for its integer semantics. Python's arbitrary-precision integers and Java's `>>>` versus `>>` create different bug profiles, and both show up in real interviews. If you're doing bit questions in Python, write the 32-bit mask into your template before you start, not after your test fails.
