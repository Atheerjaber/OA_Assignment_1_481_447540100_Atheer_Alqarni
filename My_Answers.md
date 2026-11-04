# My Answers - Assignment 1 (CFLP: VNS vs Gurobi)

**Student Information:**
- **Student ID:** 447540100
- **Student Name:** Atheer Alqarni
- **Gurobi Version Used:** [x] Academic [ ] Free
- **Date Completed:** 2026-11-04

> Note about my data: all the numbers below come from my results file `results_student_447540100.csv` / `.xlsx` and the plots in `analysis_plots_student_447540100/`. The instances that were generated with my ID are smaller than the sizes in the guide, they have 15-27 facilities and 51-80 clients, so the labels small, medium and large overlap a lot and I mention this where it matters.

---

## Section A: Solution Quality Analysis (4 marks)

### A.1 (2 marks) - Gap Analysis by Categories

**Analysis of Plot 01:**

In Plot 01 the gap gets bigger when the problem gets bigger, the median gap is 11.27% for small, 12.73% for medium and 14.80% for large instances, and the large box is the widest (about 12.6% to 18.8%). For the difficulty it was different from what I expected, because the moderate instances are the worst (median 16.35%), then the expensive ones (12.91%), and the easy ones are the best (11.01%). The worst group is large/moderate with a median of 17.66%, and the best group is small/easy with 8.48%.

**Specific Observations:**
- Highest median gap size: Large
- Median gap value: 14.80% (small 11.27%, medium 12.73%)
- Effect of difficulty on gap: Easy 11.01% < Expensive 12.91% < Moderate 16.35% (medians), so it does not go up in a straight line with difficulty.

**Outliers identified:**

- Instance 038 (small / expensive) has the highest gap of all 144 runs, which is 26.65%, and the CSV shows that the VNS never improved anything after the start (0 improving iterations). I built this instance again with my ID and checked it, the greedy start is 38.3% above the optimum, the first local search brings it down to 26.7%, and then the VNS stays there.
- Instance 053 (medium / easy) is a low outlier with 3.68%, I think the starting solution was already close to the best set of open facilities, and Gurobi also solved it in only 0.29 s.
- There is also a high point at about 17% in the medium/easy box.

**Interpretation:**

Easy instances have cheap facilities so many of them are opened, and the result depends mostly on how the clients are assigned, so the VNS does better there. For the moderate instances the fixed cost and the service cost are balanced, so choosing the right facilities is the important part, and the VNS can only open or swap facilities starting from a weak solution, that is why it makes more mistakes. The gap grows with the size because there are more possible sets of facilities, and the VNS only looks at 15 random neighbours each time and stops after 20 iterations without improvement.

---

### A.2 (2 marks) - Overall Solution Quality Statistics

**From Excel File (Sheet "All Results"):**

- Total instances analyzed: 144
- Instances with gap < 5%: 2 ( 1.4% )
- Instances with gap < 10%: 31 ( 21.5% )
- Mean gap across all instances: 13.69%
- Standard deviation of gap: 4.55%

Extra numbers: the median is 13.20%, the minimum is 3.68%, the maximum is 26.65%, and 11 instances (7.6%) are above 20%.

**Interpretation:**

The standard deviation is small (4.55%) so the VNS behaves the same way on most instances, but the quality is not good, only 2 of 144 instances are within 5% of the optimum and about 78% are above 10%. An average of 13.7% above the optimum is not good for a problem that Gurobi solves to optimality in a few seconds. In 122 of the 144 runs the last improving iteration is 0, which means the VNS never improved its first local solution, and I think this is the real reason for the high gaps (I explain it in Section G).

---

## Section B: Computational Efficiency Analysis (4 marks)

### B.1 (2 marks) - Time Comparison

**Analysis of Plot 02:**

**Size at which VNS becomes significantly faster:**

Medium. The median times from my CSV are: small, Gurobi 1.49 s and VNS 0.85 s (1.8x), medium, 3.36 s and 1.49 s (2.3x), large, 3.06 s and 1.53 s (2.0x). In the means the VNS is already faster for small (8.29 s vs 1.01 s), but that mean is pushed up by a few very slow Gurobi runs, so the median is a fairer comparison. The difficulty matters more than the size, because for the easy instances Gurobi is actually faster (median 0.53 s vs 0.93 s for the VNS), for moderate it is 3.33 s vs 1.12 s, and for expensive 7.23 s vs 1.88 s.

**Most challenging configurations:**
- For VNS: medium / expensive with an average time of 2.91 seconds (the slowest run is instance 081 with 8.42 s)
- For Gurobi: small / expensive with an average time of 21.85 seconds (medium / expensive 20.88 s and large / expensive 17.61 s, the slowest run is instance 081 with 134.79 s)

**Detailed Comparison:**

The expensive instances are the hardest for both methods. For Gurobi the fixed costs are large so the LP relaxation is weak and the y variables stay fractional, which means branch-and-bound has to check many nodes. The VNS is also slowest there, I think because every neighbour needs all the clients to be assigned again, and the local search needs more rounds. The heat maps in Plot 02 show that the Gurobi time depends on the difficulty (easy 0.41-0.76 s, expensive 17.6-21.9 s) and almost not on the size.

---

### B.2 (2 marks) - Speedup Analysis

**Analysis of Plot 03:**

**Average speedup factors:**
- Small instances: 5.10x faster (median 2.33x)
- Medium instances: 4.82x faster (median 2.69x)
- Large instances: 5.72x faster (median 1.76x)

For all 144 instances the mean speedup is 5.21x and the median is 2.15x, and the VNS is faster in 102 of 144 cases (71%).

**Instances where Gurobi is faster (speedup < 1):**

Yes, there are 42 instances (11 small, 15 medium, 16 large), and 38 of them are easy, 3 are moderate and 1 is expensive. What they have in common is that Gurobi solves them very fast (0.79 s on average and never more than 4.14 s), while the VNS needs 1.33 s on average because it always runs at least 20 iterations.

**Practical Implications:**

The mean speedup of 5.2x comes mostly from a few instances where Gurobi is slow (up to 12.9x for large/expensive). In real time Gurobi needs 8.6 s on average and 135 s at most, so for this size it is fine to wait for the exact solution. The speedup is only useful if the model must be solved a very large number of times (like simulations or daily planning for many regions) or if the answer is needed immediately.

---

## Section C: Scalability Analysis (3 marks)

### C.1 (1.5 marks) - Scalability Behavior

**Analysis of Plot 04:**

(In Plot 04 the problem size is facilities x clients on a log axis, and in my data it only goes from about 765 to 1,836, so I can only comment on a small range.)

**VNS Scalability:**

The VNS time is close together, mostly between 0.2 s and 3 s with a few points up to 8 s. The average goes from 1.01 s (small) to 1.76 s (medium) and stays 1.76 s (large), so it is a small increase.

**Gurobi Scalability:**

The Gurobi points are spread out a lot, from 0.1 s to about 135 s for the same size. Its averages (8.29 s, 9.49 s, 7.95 s) do not go up with the size, because the time depends mainly on the difficulty (the fixed cost level) and not on the number of variables.

**Comparison:**

On my data the VNS is more predictable, its time changes by a factor of about 100 (0.08 s to 8.4 s) while the Gurobi time changes by more than 1,000 (0.12 s to 134.8 s). But I cannot say that the VNS scales better for large problems, because my large instances are only a little bigger than the small ones and Gurobi did not grow at all.

---

### C.2 (1.5 marks) - Growth Rate and Predictions

**Observed Growth of Average Runtime (from your results):**

| | Small | Medium | Large | Small -> Medium (x) | Medium -> Large (x) |
|---|---|---|---|---|---|
| VNS avg time (s) | 1.01 | 1.76 | 1.76 | 1.74 | 1.00 |
| Gurobi avg time (s) | 8.29 | 9.49 | 7.95 | 1.14 | 0.84 |

**Shape of growth in Plot 04:**
- VNS: [ ] Roughly steady [x] Steadily increasing [ ] Sharply increasing
- Gurobi: [x] Roughly steady [ ] Steadily increasing [ ] Sharply increasing

**Justification:**

The VNS average grows 1.74x from small to medium and then stays the same, so a small increasing trend is the closest answer. The Gurobi average changes by +14% and then by -16%, so it is steady, but with a very big scatter around it. Both ratios are close to 1 because the three size classes are close to each other in the number of variables.

**Rough Prediction for Extra Large Instance (n=50, m=20):**
- Predicted VNS time: about 1.8 - 3 seconds
- Predicted Gurobi time: about 7 - 9 seconds
- How I computed it: I used the medium to large ratio again (VNS 1.76 x 1.00 = 1.8 s, Gurobi 7.95 x 0.84 = 6.7 s), and as a higher estimate I used the small to medium ratio (VNS 1.76 x 1.74 = 3.1 s, Gurobi 7.95 x 1.14 = 9.1 s). The instance with 50 clients and 20 facilities is actually not bigger than my small ones, so this is only a rough guess.

**Suitability for Very Large Instances:**

The VNS time grows slowly, which is a good sign for very large problems where an exact solver may not be practical. But in my results the quality gets worse when the size grows (median gap 11.3% -> 12.7% -> 14.8%), so I would only recommend this VNS for very large instances after fixing the weak assignment step (Section G). With the fix the mean gap on my data goes down from 13.7% to 4.5%, but the VNS becomes about 11 times slower (1.5 s -> 16.7 s on average).

---

## Section D: Quality-Time Tradeoff Analysis (3 marks)

### D.1 (1.5 marks) - Sweet Spot Identification

**Analysis of Plot 05:**

**Sweet Spot:**

The best place in Plot 05 is the bottom left, where the gap and the time are both low. On my data the VNS does not reach that corner for the expensive and moderate instances (median gaps about 11-18%), but it gives a useful compromise, a gap of about 13% in about 2 s compared with 20 s for Gurobi. For the easy instances there is no sweet spot, because Gurobi is faster (0.57 s on average) and it is exact.

**Time Savings for High-Quality Solutions (gap < 5%):**

Only two instances have a gap below 5% (instance 014 with 4.50%, Gurobi 0.12 s and VNS 0.09 s, and instance 053 with 3.68%, Gurobi 0.29 s and VNS 0.70 s).
- Average VNS time: 0.40 seconds
- Average Gurobi time: 0.21 seconds
- Absolute savings: -0.19 seconds (the VNS is slower on average)
- Relative savings: -93% (negative)

For a wider comparison, the 31 instances with a gap below 10% have Gurobi 8.35 s vs VNS 1.43 s, which is a saving of 6.92 s (82.9%).

**Interpretation:**

The good VNS solutions are rare, and they happen on instances that Gurobi solves almost immediately, so there is no real saving. The big savings are on the expensive instances (Gurobi 20.11 s vs VNS 2.13 s on average, 89% less), but with an average gap of 13.6%. So someone in practice would use Gurobi for the easy cases and would only think about the heuristic for the slow ones.

---

### D.2 (1.5 marks) - Practical Decision Making

**Scenarios Justifying 10-15% Gap:**

1. **Early strategic screening:** when hundreds of scenarios (different demand or cost assumptions) must be compared fast, an error of 10-15% in the total cost matters less than being able to rank the scenarios.

2. **Very frequent re-planning:** when the decision must be updated every few seconds or minutes (for example after a disruption), a fast feasible solution is worth more than an optimal one that comes too late.

3. **Very large instances or no solver license:** when the instance is much bigger than what a commercial solver can handle in the time limit, a heuristic is the only option and 10-15% can be accepted as a start that is improved later.

**Recommendation for Real-World Logistics Company:**

For instances like the ones I tested I would recommend Gurobi, because it solved all 144 instances to proven optimality in 8.6 s on average and 135 s in the worst case, and a 13.7% extra cost on facilities and shipping is a lot of money for a company. The VNS I was given could only be an extra tool for very large or urgent runs, and only in the improved version (Section G).

---

## Section E: Algorithm Understanding (3 marks)

### E.1 (1.5 marks) - VNS Neighborhood Structures

**Role of Each Neighborhood:**

**N1 (Open Facility):**
- Purpose: open one closed facility and assign the clients again.
- Contribution: [x] Intensification [ ] Diversification
- Explanation: the local search accepts it only if the saved service cost is bigger than the new fixed cost, so it helps mostly on easy instances, and it is the only move that changes the number of open facilities (+1).

**N2 (Close Facility):**
- Purpose: in this code it closes one open facility and opens one of only 3 closed candidates (a limited swap).
- Contribution: [ ] Intensification [x] Diversification
- Explanation: even though the name says close, it never makes the number of open facilities smaller, it only moves the solution sideways in a limited way, so it is a small diversification move.

**N3 (Swap Facilities):**
- Purpose: close one open facility and open one closed facility, trying many pairs.
- Contribution: [ ] Intensification [x] Diversification
- Explanation: it makes the biggest change in the structure, it explores different sets of open facilities, and it is the main way to leave a local optimum.

**N4 (Reassign Clients):**
- Purpose: move single clients to another open facility and keep the same open facilities.
- Contribution: [x] Intensification [ ] Diversification
- Explanation: it only fine-tunes the assignment (a local improvement), and it gives a feasible neighbour only rarely when the capacities are tight.

**How They Complement Each Other:**

N1, N2 and N3 change which facilities are open (the structure), and N4 improves who is served by which facility (the assignment). The shaking picks one of the four at random, and then the local search uses them one after the other with first improvement. The idea is good, but there are two gaps, N2 and N3 are both swaps so there is no pure close move, and the number of open facilities can never go down.

---

### E.2 (1.5 marks) - Convergence Behavior Analysis

**Selected Convergence Plots:** (grids from `convergence_plots_student_447540100/`, and copies of the samples are in `sample_convergence_plots/`)

**Plot 1:** [Instance name: convergence_grid_small_moderate.png, panel fac18cli68]
- Pattern: [ ] Smooth [x] Multiple jumps
- Description: it starts flat, then it drops at iteration 10 and again at iteration 15, and ends at a gap of 11.1%, while the other panels in this grid (fac20cli74, fac17cli70, fac20cli59) are completely flat.
- What it reveals: most of the time the start is already the best the search can reach, and when an improvement appears it comes as a single step after many failed tries, so the landscape has a few better regions that need the right shake.

**Plot 2:** [Instance name: convergence_grid_medium_easy.png, panel fac20cli68]
- Pattern: [x] Smooth [ ] Multiple jumps
- Description: the line is completely flat at the starting cost, the final gap is 14.4%, there is no improvement in 100 iterations, and only one panel in the grid (fac23cli68) shows one late drop.
- What it reveals: the search is stuck in a deep local optimum, because the shaken neighbours are rejected or they come back to the same solution after the local search.

**Plot 3:** [Instance name: convergence_grid_large_expensive.png, panel fac22cli66]
- Pattern: [ ] Smooth [x] Multiple jumps
- Description: a few small drops like steps, and then a flat line at a gap of 14.6%, and the panels fac24cli68 and fac26cli65 also show a few steps while fac20cli67 and fac27cli67 are flat.
- What it reveals: bigger instances have more room to improve (more neighbours), but the steps are small compared with the distance to the Gurobi line, so the search stops far from it.

**Role of Shaking Mechanism:**

The shaking jumps to a random feasible neighbour of the best solution from a random neighbourhood, and then the local search goes down from there. The few jumps in the plots (like fac18cli68 at iterations 10 and 15) are the cases where the shaking reached a different area with a lower cost. The many flat plots show the opposite, because with tight capacities most shaken solutions are infeasible or they fall back to the same local optimum, so the shaking cannot escape, and in 122 of 144 instances it never worked.

---

## Section F: Critical Reflection (3 marks)

### F.1 (1 mark) - Limitations and Improvements

**Main Limitations of Current VNS Implementation:**

1. The assignment function takes the clients in their file order and gives each one to the nearest open facility that still has space, and this is bad when the capacity is tight. For the optimal set of open facilities it gave an infeasible assignment or one that is 10-21% worse in 6 of the 6 instances I tested, so good neighbours get rejected.

2. No move can reduce the number of open facilities (N2 and N3 are both swaps), and the search only looks at up to 15 random neighbours and takes the first improvement.

3. The greedy start opens facilities by the average service cost and ignores the fixed costs, and the search stops after only 20 iterations without improvement.

**Suggested Improvements:**

1. Use a better assignment step that cares about capacity (regret order, repair, and a shift/swap polish), and this is the fix I made in Section G.

2. Add a real close-facility move, and use best improvement or more neighbours, so the search can reduce the cost on the expensive instances.

3. Start from a better solution that includes the fixed costs, and use a changing shaking strength and a longer stopping rule when no improvement is found.

---

### F.2 (1 mark) - Real-World Deployment Considerations

**Additional Constraints for Real Logistics Company:**

1. **Delivery time windows and service levels:** some clients must be served within a maximum distance or time, which limits which assignments are allowed.

2. **Vehicle routing:** the service cost is really a routing cost, so the facility decision and the routes should be optimized together (location-routing).

3. **Many periods and uncertain demand:** the capacities and the opening decisions last for years while the demand changes, so scenarios or robust models are needed.

**Additional Objectives:**

Besides the total cost, a company also cares about the service level (maximum distance to the client), how strong the network is when something fails, carbon emissions, and the balance of work between facilities, so the problem becomes a multi-objective one.

---

### F.3 (1 mark) - Exact Method vs Heuristic in Practice

**Gurobi Optimality Status (from the `gurobi_status` column):**
- Instances solved to proven optimality: 144 out of 144
- Instances stopped at the time limit: 0 out of 144 (time limit 300 s, slowest run 134.79 s)
- Do the time-limit instances share a size/difficulty? Not applicable, but the slowest runs were all expensive instances (081, 046, 043).

**What VNS Gives Up and What It Gains:**

The VNS gives up the guarantee of optimality and, in my runs, 13.69% of cost on average (up to 26.65%). What it gains is time, in total Gurobi needed 1,235 s (about 20.6 min) and the VNS needed 217.7 s (3.6 min), which is 82% less, but this saving comes almost only from the moderate and expensive instances, and on the easy ones the VNS is slower.

**Practical Value of Heuristics:**

Heuristics are important when the exact solver is too slow, too expensive or not available, and also as a way to get quick feasible solutions. In my experiment the exact solver was never the problem, so the value of the heuristic is in what it could be, which only shows when its weak points are fixed (Section G).

---

## Section G: BONUS - VNS Improvement Challenge (+3 marks, optional)

### G.1 (1 mark) - Bug Identification

**Where is the bug?** (cell / function / line numbers in `Assignment_1.ipynb`):

First I checked the VNS loop that the guide talks about. In my copy of the notebook the acceptance criterion in `vns_solve_cflp` (Section 7, the `if new_cost < best_cost: ... else: no_improve_count += 1` block, around lines 98-111 of that cell) is **already correct**, a worse solution is never accepted and the shaking always starts from `best_solution`. So the bug described in the guide (accepting worse solutions in the `else` branch) is not in the version I received, and I did not change anything there.

The real reason for the high gaps is in the cell "4. Solution Encoding and Utility Functions", in the function `assign_clients_to_nearest_open_facility`:

```python
# Process clients in natural order (no sorting) for flexibility
for client_idx in range(num_clients):
    costs = costs_to_open[client_idx]
    sorted_fac_positions = np.argsort(costs)
    ...
        if remaining_capacity[fac_idx] >= client_demands[client_idx]:
            client_assignments[client_idx] = fac_idx
            remaining_capacity[fac_idx] -= client_demands[client_idx]
```

**What does the bug do?**

For a given set of open facilities it assigns the clients one by one in the order of the file, each one to the nearest open facility that still has room (first-fit). If a client does not fit anywhere, it is just assigned to its nearest facility, which makes the solution infeasible.

**Why does it degrade VNS performance?**

This function is used by the greedy start and by every neighbourhood (N1, N2, N3), and then every neighbour is checked with `is_feasible`. In my instances the capacity is very tight, in the optimal solutions the open facilities are 93-97% full. I built six of my instances again (002, 033, 038, 075, 123, 113) with my ID, solved them to optimality, and gave the optimal set of open facilities to this function, and in all 6 cases it returned an **infeasible** assignment, and its cost was 10-21% above the optimum (11.6, 9.7, 21.0, 10.9, 21.2, 18.9%). So the moves that lead to good sets of facilities are rejected or their cost is too high, and the search stays at the first local optimum. This matches my results, because in 122 of 144 instances the last improving iteration is 0, and the mean gap is 13.69% with 11 instances above 20%.

---

### G.2 (1 mark) - The Fix

**Corrected code:**

The whole new cell is in `447540100_Atheer_Alqarni_Assignment1_IMPROVED.ipynb` (the cell "BONUS FIX"). The switch itself is one line, and the rest is the new function:

```python
def assign_clients_improved(service_costs, facility_opened, facility_capacities, client_demands):
    """Drop-in replacement for assign_clients_to_nearest_open_facility.
    Builds the assignment with several client orderings and returns the cheapest feasible one."""
    best = None; best_key = None
    for key in _ORDERS:          # (1) regret order, (2) demand x regret, (3) demand decreasing
        a = _assign_core(service_costs, facility_opened, facility_capacities, client_demands, key)
        load = np.bincount(a, weights=client_demands, minlength=len(facility_capacities))
        over = float(np.maximum(load - facility_capacities, 0).sum())
        cost = float(service_costs[np.arange(len(a)), a].sum())
        k = (over > 1e-6, over, cost)          # feasible first, then least overload, then cheapest
        if best_key is None or k < best_key:
            best, best_key = a, k
    return best

# >>> the one-line switch: all neighbourhoods and the greedy start now use the improved assignment <<<
assign_clients_to_nearest_open_facility = assign_clients_improved
```

The function `_assign_core` does three things: (1) it places the clients in the chosen order (the client with the biggest regret goes first, regret is the difference between the second best and the best open facility), (2) it repairs the clients that do not fit using ejection chains, and (3) it improves the result with best-improvement shift and swap moves.

**Why does the fix work and why does it not break the rest of the algorithm?**

The regret order gives the clients that would lose the most from a bad place the first choice, the ejection chains find a feasible packing when the simple greedy way fails, and the swap moves bring the cost down again. On the six instances above, with the optimal set of open facilities, the new function returns feasible assignments that are only 1-9% above the optimum (same test) and it takes about 2-8 ms per call. It has the same inputs and the same output type as the old function (an array with one facility index for each client), `is_feasible` still checks every neighbour, and the acceptance criterion, the shaking, the local search and the neighbourhoods are not changed. All 144 final solutions of the improved run are feasible (checked with `is_feasible`).

---

### G.3 (1 mark) - Demonstrated Improvement

**Before vs After (same instances):**

| Metric | Original VNS | Improved VNS |
|---|---|---|
| Mean gap (%) | 13.69 | 4.53 |
| Std of gap (%) | 4.55 | 2.32 |
| Median gap (%) | 13.20 | 4.10 |
| Max gap (%) | 26.65 | 13.18 |
| Instances with gap < 5% | 2 | 92 |
| Instances with gap < 10% | 31 | 141 |
| Instances with gap > 20% | 11 | 0 |
| Instances where VNS improved on its start | 22 | 46 |
| Feasible final solutions | - | 144 / 144 |
| Mean VNS time (s) | 1.51 | 16.68 |

Mean gap by size / difficulty (original -> improved, %): small easy 9.36 -> 3.20, small moderate 14.39 -> 4.77, small expensive 12.78 -> 4.14, medium easy 11.40 -> 3.33, medium moderate 16.73 -> 4.78, medium expensive 12.66 -> 4.75, large easy 13.46 -> 4.55, large moderate 17.07 -> 4.64, large expensive 15.35 -> 6.60.

**Analysis:**

The mean gap went down by 9.2 percentage points (a reduction of 66%), the gap is below 5% in 92 of the 144 instances (before it was 2), and no instance is above 20% anymore. It became worse in only 1 of the 144 instances, and the weakest group is still large / expensive (6.6%). The price is the time, the mean VNS time grew from 1.51 s to 16.68 s (median 15.0 s, slowest run 59 s), because every neighbour now builds three assignments and improves them. These times were measured on my laptop while other work was also running, so the real times may be a bit lower, but the ratio of about 11x is what the code change gives. Because of this the improved VNS is faster than Gurobi (8.58 s on average) in only 22 of 144 instances (mean speedup 0.66x), so for instances of this size the exact solver is still the better tool. The improved VNS shows that the heuristic can reach good quality, and its time could be reduced (fewer orderings, caching, evaluating the neighbours step by step), but I did not do that.

**Improved results files (names in my repository):** `447540100_Atheer_Alqarni_Assignment1_IMPROVED.ipynb`, `results_improved_student_447540100.csv`, `results_improved_student_447540100.xlsx`, `improved_vs_original_student_447540100.png`


---

## Additional Comments (Optional)

The bug described in Section 8 of the guide (the acceptance criterion in the `else` branch) was already corrected in the notebook I received. So in Section G I wrote what I measured myself, which is the client assignment function that ignores the capacity. I checked my claims on six of my instances by building them again with my ID and solving them exactly, and my solver gave the same optimal costs as the Gurobi values in my CSV for the three instances I compared.

---

## Declaration

I hereby declare that:
- I have completed this assignment independently
- All numerical results are from MY experiments using MY student ID
- All analysis and interpretations are MY own work
- I have not copied results or answers from other students
- I understand the consequences of academic dishonesty

**Signature:** Atheer Alqarni

**Date:** 4/11/2026

---

**END OF MY ANSWERS**
