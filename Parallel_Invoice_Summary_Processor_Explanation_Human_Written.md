# Parallel Invoice Summary Processor - Explanation and Solution

## Problem Explanation

The issue is caused by this dictionary:

```csharp
Dictionary<string, decimal> branchTotals = new Dictionary<string, decimal>();
```

This dictionary is updated inside:

```csharp
Parallel.ForEach(...)
```

A normal `Dictionary<TKey, TValue>` is not thread-safe. That means it should not be updated by multiple threads at the same time.

In this program, each invoice line is processed in parallel. So more than one thread may try to update `branchTotals` at the same time.

This part is unsafe:

```csharp
if (!branchTotals.ContainsKey(branch))
{
    branchTotals[branch] = 0;
}

branchTotals[branch] += amount;
```

The line below looks simple, but it is not one single operation:

```csharp
branchTotals[branch] += amount;
```

Internally, it performs three steps:

```text
1. Read the current branch total.
2. Add the current invoice amount.
3. Write the updated total back to the dictionary.
```

When two threads do these steps at the same time, one update may overwrite another update. In some cases, the dictionary itself can also become corrupted.

That is why the program may show wrong totals or fail at runtime.

---

# Solution 1: Using lock

One simple way to solve the issue is to protect the dictionary update using `lock`.

## Fixed Code

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace InvoiceSummaryBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var invoiceLines = new List<string>
            {
                "BranchA,1000",
                "BranchB,500",
                "BranchA,700",
                "BranchC,1200",
                "BranchB,300",
                "BranchA,200",
                "BranchC,800"
            };

            Dictionary<string, decimal> branchTotals = new Dictionary<string, decimal>();

            object lockObject = new object();

            Parallel.ForEach(invoiceLines, line =>
            {
                var parts = line.Split(',');
                string branch = parts[0];
                decimal amount = decimal.Parse(parts[1]);

                lock (lockObject)
                {
                    if (!branchTotals.ContainsKey(branch))
                    {
                        branchTotals[branch] = 0;
                    }

                    branchTotals[branch] += amount;
                }
            });

            foreach (var item in branchTotals)
            {
                Console.WriteLine($"{item.Key} = {item.Value}");
            }
        }
    }
}
```

## Output

```text
BranchA = 1900
BranchB = 800
BranchC = 2000
```

## Why this works

The `lock` statement allows only one thread at a time to enter the protected section.

This part is protected:

```csharp
lock (lockObject)
{
    if (!branchTotals.ContainsKey(branch))
    {
        branchTotals[branch] = 0;
    }

    branchTotals[branch] += amount;
}
```

So even though the loop runs in parallel, the dictionary update happens safely.

This removes the race condition and gives the correct total every time.

---

# Solution 2: Using ConcurrentDictionary

A better solution for this type of parallel update is to use `ConcurrentDictionary`.

`ConcurrentDictionary` is designed for multi-threaded read and write operations.

## Fixed Code

```csharp
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace InvoiceSummaryBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var invoiceLines = new List<string>
            {
                "BranchA,1000",
                "BranchB,500",
                "BranchA,700",
                "BranchC,1200",
                "BranchB,300",
                "BranchA,200",
                "BranchC,800"
            };

            ConcurrentDictionary<string, decimal> branchTotals =
                new ConcurrentDictionary<string, decimal>();

            Parallel.ForEach(invoiceLines, line =>
            {
                var parts = line.Split(',');
                string branch = parts[0];
                decimal amount = decimal.Parse(parts[1]);

                branchTotals.AddOrUpdate(
                    branch,
                    amount,
                    (key, existingAmount) => existingAmount + amount
                );
            });

            foreach (var item in branchTotals)
            {
                Console.WriteLine($"{item.Key} = {item.Value}");
            }
        }
    }
}
```

## Output

```text
BranchA = 1900
BranchB = 800
BranchC = 2000
```

## Explanation of AddOrUpdate

The `AddOrUpdate` method is useful when we need to add a new key or update an existing key safely.

```csharp
branchTotals.AddOrUpdate(
    branch,
    amount,
    (key, existingAmount) => existingAmount + amount
);
```

This means:

```text
If the branch is not already available:
    Add the branch with the current amount.

If the branch is already available:
    Add the current amount to the existing branch total.
```

Example:

```text
BranchA first record  = 1000
BranchA second record = 1000 + 700 = 1700
BranchA third record  = 1700 + 200 = 1900
```

---

# Recommended Solution

For this scenario, `ConcurrentDictionary` is the better choice.

The requirement is to calculate branch-wise totals using parallel processing. Since multiple threads are updating the same collection, a thread-safe collection is more suitable than a normal dictionary.

Recommended approach:

```text
Use ConcurrentDictionary with AddOrUpdate.
```

---

# Final Candidate Answer

The issue occurs because `Dictionary<string, decimal>` is not thread-safe. The program updates the same dictionary from multiple threads inside `Parallel.ForEach`. This causes a race condition, so the final branch totals may become incorrect.

The issue can be fixed either by using `lock` around the dictionary update or by replacing `Dictionary` with `ConcurrentDictionary`.

In this case, `ConcurrentDictionary` is the better solution because it is created for concurrent updates. Its `AddOrUpdate` method safely handles both adding a new branch and updating an existing branch total.

---

# Final Fixed Code

```csharp
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace InvoiceSummaryBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var invoiceLines = new List<string>
            {
                "BranchA,1000",
                "BranchB,500",
                "BranchA,700",
                "BranchC,1200",
                "BranchB,300",
                "BranchA,200",
                "BranchC,800"
            };

            ConcurrentDictionary<string, decimal> branchTotals =
                new ConcurrentDictionary<string, decimal>();

            Parallel.ForEach(invoiceLines, line =>
            {
                var parts = line.Split(',');
                string branch = parts[0];
                decimal amount = decimal.Parse(parts[1]);

                branchTotals.AddOrUpdate(
                    branch,
                    amount,
                    (key, existingAmount) => existingAmount + amount
                );
            });

            foreach (var item in branchTotals)
            {
                Console.WriteLine($"{item.Key} = {item.Value}");
            }
        }
    }
}
```

---

# Key Concepts Tested

```text
Parallel.ForEach
Race condition
Thread safety
Dictionary vs ConcurrentDictionary
lock keyword
AddOrUpdate method
Concurrent collection handling
```

---

# Interview Explanation

A normal dictionary should not be updated by multiple threads at the same time. In this program, every invoice line is processed in parallel, and multiple threads may try to update `branchTotals` together.

This can lead to incorrect totals or dictionary corruption.

To solve this, I used `ConcurrentDictionary`. It is thread-safe and supports safe add/update operations using `AddOrUpdate`.

This ensures that the branch-wise invoice totals are calculated correctly every time.
