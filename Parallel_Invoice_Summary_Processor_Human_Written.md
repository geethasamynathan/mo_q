# Question 1: Parallel Invoice Summary Processor

## Business Scenario

A finance application receives invoice line items from different branches.  
Each line contains the branch name and the invoice amount.

The requirement is to calculate the total invoice amount for each branch. Since the invoice lines may be high in number in a real system, the program uses parallel processing.

However, the current program sometimes gives incorrect or inconsistent totals when it is executed multiple times.

---

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

---

## Expected Output

```text
BranchA = 1900
BranchB = 800
BranchC = 2000
```

---

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

---

## Candidate Task

Fix the program so that it always calculates the correct branch-wise invoice total.

The candidate should identify why the result is inconsistent and update the program using a thread-safe approach.
