# C# .NET 8 Console App Bug Fixing Assessment
## Topic: Task Parallel Library — Complex Real-World Scenarios

Prepared for: **Minimum 5 Years Experienced C# / .NET Candidates**  
Role Context: **Senior .NET SME Assessment**  
Application Type: **C# Console App using .NET 8**  
Assessment Type: **Bug Fixing / Code Analysis / TPL / Concurrency**

---

## Candidate Instructions

You are given a partially working C# console application.  
Each problem simulates a real production issue related to concurrency, parallel processing, task coordination, cancellation, exception handling, and shared state.

Your task is to:

1. Analyze the given code.
2. Identify why the program is producing incorrect, unstable, or unexpected output.
3. Fix the issue with minimal code changes.
4. Preserve the expected business behavior.
5. Do not remove parallelism completely unless it is logically required.
6. Use proper .NET 8 / C# practices.

> **Important:** The problem statement does not reveal the bug. The candidate must analyze the code and find the issue.

---

# Question 1: Parallel Invoice Summary Processor

## Business Scenario

A finance system receives invoice line items from multiple branches.  
The system should calculate the total invoice amount for each branch using parallel processing.

The current program sometimes gives inconsistent totals when executed multiple times.

## Input

```text
BranchA,1000
BranchB,500
BranchA,700
BranchC,1200
BranchB,300
BranchA,200
BranchC,800
```

## Expected Output

```text
BranchA = 1900
BranchB = 800
BranchC = 2000
```

## Given Code

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

            Parallel.ForEach(invoiceLines, line =>
            {
                var parts = line.Split(',');
                string branch = parts[0];
                decimal amount = decimal.Parse(parts[1]);

                if (!branchTotals.ContainsKey(branch))
                {
                    branchTotals[branch] = 0;
                }

                branchTotals[branch] += amount;
            });

            foreach (var item in branchTotals)
            {
                Console.WriteLine($"{item.Key} = {item.Value}");
            }
        }
    }
}
```

## Candidate Task

Fix the program so that it always calculates the correct branch-wise invoice total.

---

# Question 2: Parallel Notification Sender

## Business Scenario

A notification service sends messages to customers in parallel.

Each message send operation is asynchronous and takes some time.  
The program should wait until all notifications are sent before printing the final count.

Currently, the final count is sometimes incorrect.

## Input

```text
5 customers
```

## Expected Output

```text
Sent notification to Customer 1
Sent notification to Customer 2
Sent notification to Customer 3
Sent notification to Customer 4
Sent notification to Customer 5
Total Sent: 5
```

Order of customer messages does not matter.

## Given Code

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace NotificationSenderBug
{
    internal class Program
    {
        static int sentCount = 0;

        static void Main(string[] args)
        {
            var customers = new List<string>
            {
                "Customer 1",
                "Customer 2",
                "Customer 3",
                "Customer 4",
                "Customer 5"
            };

            Parallel.ForEach(customers, async customer =>
            {
                await SendNotificationAsync(customer);
                sentCount++;
            });

            Console.WriteLine($"Total Sent: {sentCount}");
        }

        static async Task SendNotificationAsync(string customer)
        {
            await Task.Delay(500);
            Console.WriteLine($"Sent notification to {customer}");
        }
    }
}
```

## Candidate Task

Fix the program so that all notifications are completed and the final count is always correct.

---

# Question 3: Inventory Reservation Engine

## Business Scenario

An e-commerce warehouse receives many reservation requests for the same product.

Each request reserves a quantity from available stock.  
The system must not allow stock to go below zero.

Currently, under load, the system sometimes allows more quantity to be reserved than available.

## Input

```text
Initial Stock: 10
Reservation Requests: 3, 4, 2, 5, 1
```

## Expected Behavior

Only valid reservations should succeed.  
Total reserved quantity should not exceed 10.

Example valid output:

```text
Reserved: 3
Reserved: 4
Reserved: 2
Rejected: 5
Reserved: 1
Final Stock: 0
```

Order may vary because processing is parallel.

## Given Code

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace InventoryReservationBug
{
    internal class Program
    {
        static int availableStock = 10;

        static void Main(string[] args)
        {
            var reservationRequests = new List<int> { 3, 4, 2, 5, 1 };

            Parallel.ForEach(reservationRequests, quantity =>
            {
                if (availableStock >= quantity)
                {
                    Task.Delay(100).Wait();

                    availableStock -= quantity;
                    Console.WriteLine($"Reserved: {quantity}");
                }
                else
                {
                    Console.WriteLine($"Rejected: {quantity}");
                }
            });

            Console.WriteLine($"Final Stock: {availableStock}");
        }
    }
}
```

## Candidate Task

Fix the reservation logic so that stock consistency is guaranteed.

---

# Question 4: Customer Risk Score Aggregator

## Business Scenario

A banking application calculates risk scores for customers.

Each customer score is calculated in parallel.  
If one customer calculation fails, the application should still process the remaining customers and report failed customer IDs separately.

Currently, the application stops unexpectedly and does not give a proper summary.

## Input

```text
Customer IDs: 101, 102, 103, 104, 105
```

## Expected Output

```text
Successful Scores:
Customer 101 = 70
Customer 102 = 80
Customer 104 = 65
Customer 105 = 90

Failed Customers:
Customer 103 failed
```

## Given Code

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace RiskScoreAggregatorBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var customerIds = new List<int> { 101, 102, 103, 104, 105 };

            var scores = new Dictionary<int, int>();
            var failedCustomers = new List<int>();

            Parallel.ForEach(customerIds, customerId =>
            {
                int score = CalculateRiskScore(customerId);
                scores.Add(customerId, score);
            });

            Console.WriteLine("Successful Scores:");
            foreach (var score in scores)
            {
                Console.WriteLine($"Customer {score.Key} = {score.Value}");
            }

            Console.WriteLine();
            Console.WriteLine("Failed Customers:");
            foreach (var customerId in failedCustomers)
            {
                Console.WriteLine($"Customer {customerId} failed");
            }
        }

        static int CalculateRiskScore(int customerId)
        {
            if (customerId == 103)
            {
                throw new InvalidOperationException("Risk engine timeout");
            }

            return customerId switch
            {
                101 => 70,
                102 => 80,
                104 => 65,
                105 => 90,
                _ => 50
            };
        }
    }
}
```

## Candidate Task

Fix the program so that:

1. One failed customer does not stop all processing.
2. Successful results are stored safely.
3. Failed customer IDs are captured safely.

---

# Question 5: Parallel File Import Validator

## Business Scenario

A data import tool validates multiple files in parallel.

Each file has multiple rows.  
The application should stop processing quickly when cancellation is requested.

Currently, cancellation does not stop the process properly, and some unnecessary work continues.

## Input

```text
Files:
orders_1.csv
orders_2.csv
orders_3.csv
orders_4.csv
```

## Expected Behavior

When cancellation is requested:

```text
Processing stopped by cancellation.
```

The program should stop cleanly and should not continue validating all files.

## Given Code

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

namespace FileImportValidatorBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var files = new List<string>
            {
                "orders_1.csv",
                "orders_2.csv",
                "orders_3.csv",
                "orders_4.csv"
            };

            using CancellationTokenSource cts = new CancellationTokenSource();

            Task.Run(async () =>
            {
                await Task.Delay(700);
                cts.Cancel();
            });

            try
            {
                Parallel.ForEach(files, file =>
                {
                    ValidateFile(file, cts.Token);
                });

                Console.WriteLine("All files validated successfully.");
            }
            catch (OperationCanceledException)
            {
                Console.WriteLine("Processing stopped by cancellation.");
            }
        }

        static void ValidateFile(string fileName, CancellationToken token)
        {
            Console.WriteLine($"Started validating {fileName}");

            for (int row = 1; row <= 10; row++)
            {
                Thread.Sleep(200);

                Console.WriteLine($"{fileName} - Row {row} validated");
            }

            Console.WriteLine($"Completed validating {fileName}");
        }
    }
}
```

## Candidate Task

Fix the program so that cancellation is properly respected during parallel execution.

---

# Question 6: Parallel Payment Settlement Report

## Business Scenario

A payment settlement job processes transactions in parallel and creates a final report.

Each transaction has:

```text
TransactionId
Amount
Status
```

Only successful transactions should be included in the settlement total.

Currently, the report sometimes contains duplicate or missing transaction IDs.

## Input

```text
T001,1000,Success
T002,500,Failed
T003,700,Success
T004,300,Success
T005,200,Failed
T006,900,Success
```

## Expected Output

```text
Settlement Total: 2900
Settled Transactions:
T001
T003
T004
T006
```

## Given Code

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace PaymentSettlementBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var transactions = new List<string>
            {
                "T001,1000,Success",
                "T002,500,Failed",
                "T003,700,Success",
                "T004,300,Success",
                "T005,200,Failed",
                "T006,900,Success"
            };

            decimal settlementTotal = 0;
            List<string> settledTransactionIds = new List<string>();

            Parallel.For(0, transactions.Count, i =>
            {
                var parts = transactions[i].Split(',');

                string transactionId = parts[0];
                decimal amount = decimal.Parse(parts[1]);
                string status = parts[2];

                if (status == "Success")
                {
                    settlementTotal += amount;
                    settledTransactionIds.Add(transactionId);
                }
            });

            Console.WriteLine($"Settlement Total: {settlementTotal}");

            Console.WriteLine("Settled Transactions:");
            foreach (var transactionId in settledTransactionIds.OrderBy(x => x))
            {
                Console.WriteLine(transactionId);
            }
        }
    }
}
```

## Candidate Task

Fix the settlement report so that:

1. The settlement total is always correct.
2. Settled transaction IDs are not lost or duplicated.
3. Parallel processing is still used.

---

# Interviewer Answer Key

---

## Answer Key — Question 1

### Actual Issue

`Dictionary<TKey, TValue>` is not thread-safe for concurrent read/write operations.

This code is unsafe:

```csharp
if (!branchTotals.ContainsKey(branch))
{
    branchTotals[branch] = 0;
}

branchTotals[branch] += amount;
```

Multiple threads may check, add, and update the same branch at the same time.

### Recommended Fix

Use `ConcurrentDictionary`.

```csharp
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Linq;
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

            ConcurrentDictionary<string, decimal> branchTotals = new();

            Parallel.ForEach(invoiceLines, line =>
            {
                var parts = line.Split(',');
                string branch = parts[0];
                decimal amount = decimal.Parse(parts[1]);

                branchTotals.AddOrUpdate(
                    branch,
                    amount,
                    (_, existingAmount) => existingAmount + amount
                );
            });

            foreach (var item in branchTotals.OrderBy(x => x.Key))
            {
                Console.WriteLine($"{item.Key} = {item.Value}");
            }
        }
    }
}
```

### Senior-Level Evaluation Points

A strong candidate should mention:

- `Dictionary` is not safe for parallel writes.
- `ContainsKey` followed by update is not atomic.
- `ConcurrentDictionary.AddOrUpdate` is appropriate.
- Output order should not be assumed unless explicitly sorted.

---

## Answer Key — Question 2

### Actual Issue

`Parallel.ForEach` does not correctly await an `async` lambda in this form.

This code creates an `async void`-like behavior:

```csharp
Parallel.ForEach(customers, async customer =>
{
    await SendNotificationAsync(customer);
    sentCount++;
});
```

The loop completes before the asynchronous operations finish.

Also, `sentCount++` is not thread-safe.

### Recommended Fix for .NET 8

Use `Parallel.ForEachAsync`.

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

namespace NotificationSenderBug
{
    internal class Program
    {
        static int sentCount = 0;

        static async Task Main(string[] args)
        {
            var customers = new List<string>
            {
                "Customer 1",
                "Customer 2",
                "Customer 3",
                "Customer 4",
                "Customer 5"
            };

            await Parallel.ForEachAsync(customers, async (customer, cancellationToken) =>
            {
                await SendNotificationAsync(customer);
                Interlocked.Increment(ref sentCount);
            });

            Console.WriteLine($"Total Sent: {sentCount}");
        }

        static async Task SendNotificationAsync(string customer)
        {
            await Task.Delay(500);
            Console.WriteLine($"Sent notification to {customer}");
        }
    }
}
```

### Senior-Level Evaluation Points

A strong candidate should mention:

- `Parallel.ForEach` is designed for synchronous delegates.
- `Parallel.ForEachAsync` should be used for async workloads.
- `sentCount++` is not atomic.
- `Interlocked.Increment` is safer for shared counters.
- `async Task Main` is valid in modern C#.

---

## Answer Key — Question 3

### Actual Issue

The check and update of `availableStock` is not atomic.

This code is unsafe:

```csharp
if (availableStock >= quantity)
{
    availableStock -= quantity;
}
```

Two threads may read the same stock value before either thread updates it.

### Recommended Fix

Use locking around the complete check-and-update section.

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace InventoryReservationBug
{
    internal class Program
    {
        static int availableStock = 10;
        static readonly object stockLock = new object();

        static void Main(string[] args)
        {
            var reservationRequests = new List<int> { 3, 4, 2, 5, 1 };

            Parallel.ForEach(reservationRequests, quantity =>
            {
                bool reserved = false;

                lock (stockLock)
                {
                    if (availableStock >= quantity)
                    {
                        availableStock -= quantity;
                        reserved = true;
                    }
                }

                if (reserved)
                {
                    Console.WriteLine($"Reserved: {quantity}");
                }
                else
                {
                    Console.WriteLine($"Rejected: {quantity}");
                }
            });

            Console.WriteLine($"Final Stock: {availableStock}");
        }
    }
}
```

### Senior-Level Evaluation Points

A strong candidate should mention:

- This is a classic race condition.
- The check and update must be atomic.
- `volatile` does not solve compound operations.
- `Interlocked` is useful for simple increments/decrements, but conditional decrement is better handled with lock or CAS loop.
- The lock should protect only the critical section, not unnecessary work.

---

## Answer Key — Question 4

### Actual Issue

There are two problems:

1. Exception inside `Parallel.ForEach` stops the loop and is wrapped in `AggregateException`.
2. `Dictionary` and `List` are not safe for concurrent writes.

### Recommended Fix

Use thread-safe collections and handle exceptions inside each iteration.

```csharp
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace RiskScoreAggregatorBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var customerIds = new List<int> { 101, 102, 103, 104, 105 };

            var scores = new ConcurrentDictionary<int, int>();
            var failedCustomers = new ConcurrentBag<int>();

            Parallel.ForEach(customerIds, customerId =>
            {
                try
                {
                    int score = CalculateRiskScore(customerId);
                    scores.TryAdd(customerId, score);
                }
                catch
                {
                    failedCustomers.Add(customerId);
                }
            });

            Console.WriteLine("Successful Scores:");
            foreach (var score in scores.OrderBy(x => x.Key))
            {
                Console.WriteLine($"Customer {score.Key} = {score.Value}");
            }

            Console.WriteLine();
            Console.WriteLine("Failed Customers:");
            foreach (var customerId in failedCustomers.OrderBy(x => x))
            {
                Console.WriteLine($"Customer {customerId} failed");
            }
        }

        static int CalculateRiskScore(int customerId)
        {
            if (customerId == 103)
            {
                throw new InvalidOperationException("Risk engine timeout");
            }

            return customerId switch
            {
                101 => 70,
                102 => 80,
                104 => 65,
                105 => 90,
                _ => 50
            };
        }
    }
}
```

### Senior-Level Evaluation Points

A strong candidate should mention:

- Exceptions inside parallel loops need careful handling.
- `AggregateException` may occur if exceptions are not handled inside iterations.
- `Dictionary` and `List` are not safe for concurrent writes.
- Use `ConcurrentDictionary` and `ConcurrentBag`.
- Do not swallow exceptions blindly in production; log the exception details.

---

## Answer Key — Question 5

### Actual Issue

The cancellation token is passed to `ValidateFile`, but it is never checked inside the method.

Also, `Parallel.ForEach` itself is not configured with the cancellation token.

### Recommended Fix

Use `ParallelOptions` and check the token inside the row-processing loop.

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

namespace FileImportValidatorBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var files = new List<string>
            {
                "orders_1.csv",
                "orders_2.csv",
                "orders_3.csv",
                "orders_4.csv"
            };

            using CancellationTokenSource cts = new CancellationTokenSource();

            Task.Run(async () =>
            {
                await Task.Delay(700);
                cts.Cancel();
            });

            try
            {
                var options = new ParallelOptions
                {
                    CancellationToken = cts.Token,
                    MaxDegreeOfParallelism = Environment.ProcessorCount
                };

                Parallel.ForEach(files, options, file =>
                {
                    ValidateFile(file, options.CancellationToken);
                });

                Console.WriteLine("All files validated successfully.");
            }
            catch (OperationCanceledException)
            {
                Console.WriteLine("Processing stopped by cancellation.");
            }
        }

        static void ValidateFile(string fileName, CancellationToken token)
        {
            Console.WriteLine($"Started validating {fileName}");

            for (int row = 1; row <= 10; row++)
            {
                token.ThrowIfCancellationRequested();

                Thread.Sleep(200);

                Console.WriteLine($"{fileName} - Row {row} validated");
            }

            Console.WriteLine($"Completed validating {fileName}");
        }
    }
}
```

### Senior-Level Evaluation Points

A strong candidate should mention:

- Passing a token is not enough.
- The method must actively observe the token.
- `ParallelOptions.CancellationToken` should be used.
- Cancellation is cooperative, not automatic.
- Long-running loops should check cancellation regularly.

---

## Answer Key — Question 6

### Actual Issue

Both of these are unsafe under parallel execution:

```csharp
settlementTotal += amount;
settledTransactionIds.Add(transactionId);
```

`decimal` updates are not atomic, and `List<T>` is not safe for concurrent writes.

### Recommended Fix Option 1: Lock-Based Fix

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace PaymentSettlementBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var transactions = new List<string>
            {
                "T001,1000,Success",
                "T002,500,Failed",
                "T003,700,Success",
                "T004,300,Success",
                "T005,200,Failed",
                "T006,900,Success"
            };

            decimal settlementTotal = 0;
            List<string> settledTransactionIds = new List<string>();
            object settlementLock = new object();

            Parallel.For(0, transactions.Count, i =>
            {
                var parts = transactions[i].Split(',');

                string transactionId = parts[0];
                decimal amount = decimal.Parse(parts[1]);
                string status = parts[2];

                if (status == "Success")
                {
                    lock (settlementLock)
                    {
                        settlementTotal += amount;
                        settledTransactionIds.Add(transactionId);
                    }
                }
            });

            Console.WriteLine($"Settlement Total: {settlementTotal}");

            Console.WriteLine("Settled Transactions:");
            foreach (var transactionId in settledTransactionIds.OrderBy(x => x))
            {
                Console.WriteLine(transactionId);
            }
        }
    }
}
```

### Recommended Fix Option 2: Better Senior-Level Fix Using Local Aggregation

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace PaymentSettlementBug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var transactions = new List<string>
            {
                "T001,1000,Success",
                "T002,500,Failed",
                "T003,700,Success",
                "T004,300,Success",
                "T005,200,Failed",
                "T006,900,Success"
            };

            object finalLock = new object();
            decimal settlementTotal = 0;
            List<string> settledTransactionIds = new List<string>();

            Parallel.ForEach(
                transactions,
                () => new SettlementPartialResult(),
                (transaction, state, localResult) =>
                {
                    var parts = transaction.Split(',');

                    string transactionId = parts[0];
                    decimal amount = decimal.Parse(parts[1]);
                    string status = parts[2];

                    if (status == "Success")
                    {
                        localResult.Total += amount;
                        localResult.TransactionIds.Add(transactionId);
                    }

                    return localResult;
                },
                localResult =>
                {
                    lock (finalLock)
                    {
                        settlementTotal += localResult.Total;
                        settledTransactionIds.AddRange(localResult.TransactionIds);
                    }
                });

            Console.WriteLine($"Settlement Total: {settlementTotal}");

            Console.WriteLine("Settled Transactions:");
            foreach (var transactionId in settledTransactionIds.OrderBy(x => x))
            {
                Console.WriteLine(transactionId);
            }
        }
    }

    internal class SettlementPartialResult
    {
        public decimal Total { get; set; }
        public List<string> TransactionIds { get; } = new();
    }
}
```

### Senior-Level Evaluation Points

A strong candidate should mention:

- Shared mutable state causes data corruption in parallel loops.
- `decimal` addition is not atomic.
- `List<T>.Add` is not thread-safe.
- Locking works but may reduce parallel performance.
- Local aggregation is usually better for parallel reduction scenarios.

---

# Scoring Rubric

| Skill Area | Expected From 5+ Years Candidate |
|---|---|
| TPL Knowledge | Understands `Parallel.For`, `Parallel.ForEach`, `Parallel.ForEachAsync`, `Task`, `CancellationToken` |
| Race Condition Analysis | Can identify shared mutable state issues |
| Thread-Safe Collections | Knows when to use `ConcurrentDictionary`, `ConcurrentBag`, local aggregation |
| Async Awareness | Understands why `async` lambda inside `Parallel.ForEach` is dangerous |
| Cancellation Handling | Knows cancellation is cooperative |
| Exception Handling | Can isolate per-item failure in parallel processing |
| Production Thinking | Avoids unnecessary locks, avoids swallowing exceptions silently, keeps parallelism meaningful |

---

# Suggested Interview Follow-Up Questions

1. Why is `Dictionary<TKey, TValue>` unsafe in parallel writes?
2. What is the difference between `Task.WhenAll` and `Parallel.ForEachAsync`?
3. Why is `sentCount++` not thread-safe?
4. When would you use `lock` instead of `ConcurrentDictionary`?
5. What is cooperative cancellation?
6. How does `Parallel.ForEach` handle exceptions internally?
7. What is local aggregation in TPL?
8. Why should we avoid blocking calls like `.Wait()` or `.Result()` in async code?
9. What is the difference between CPU-bound and I/O-bound parallelism?
10. How would you control maximum parallelism in .NET 8?

---

# Optional Practical Evaluation Format

## Round 1: Code Analysis

Ask the candidate to explain what the program is trying to do.

Expected from candidate:

- Understand the business requirement.
- Identify input and expected output.
- Identify shared variables.
- Identify parallel execution points.

## Round 2: Bug Identification

Ask the candidate:

- What can go wrong in this program?
- Will this issue happen every time?
- Why is the output non-deterministic?

Expected from candidate:

- Should explain race condition or async/TPL issue clearly.
- Should not only say “use lock” without explaining why.

## Round 3: Code Fix

Ask the candidate to fix the program.

Expected from candidate:

- Uses correct thread-safe approach.
- Keeps code readable.
- Does not remove parallelism unnecessarily.
- Handles exceptions/cancellation where required.

## Round 4: Production Discussion

Ask the candidate:

- How would you log this issue?
- How would you test this issue?
- How would you reproduce this issue under load?
- What are the performance implications of your fix?

Expected from candidate:

- Should discuss load testing, repeated execution, concurrent test scenarios, and observability.

---

# Additional Senior SME Notes

These questions are intentionally different from common coding platform questions because they focus on real-world production bugs instead of algorithm-only problems.

The goal is to evaluate whether the candidate understands:

- Real concurrency behavior.
- Non-deterministic output.
- Race conditions.
- Thread-safe collections.
- Async programming mistakes.
- Cancellation behavior.
- Exception handling in parallel workloads.
- Performance versus correctness trade-offs.

A good 5+ years candidate should not only provide a working fix but also explain the reason behind the failure.
