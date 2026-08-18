---
name: sensitivity-analysis
description: Test how sensitive a financial model's output is to changes in its inputs. Use whenever the user asks for a sensitivity analysis, a data table, a tornado chart or tornado diagram, a scenario analysis with probability-weighted outcomes, or a breakeven or "what value of X makes Y happen" calculation. Also covers building an Excel-style two-variable data table.
---

# Sensitivity analysis

This skill covers six related techniques for testing how a financial model's output responds to changes in its inputs: one-way sensitivity, two-way sensitivity, tornado analysis, scenario analysis, breakeven analysis, and Excel-style data tables. Each one is a different way of asking the same underlying question — "if this input were different, what would the output be?" — organized differently depending on what the user wants to see.

Before doing any of this, you need a working model that can be asked "what value of the variable am I testing right now?" and told "recalculate your output" after each change. Everything below assumes you can do those two things — set a variable to a new value, and read the resulting output — for whatever model you're analyzing.

## When to use this

Trigger for requests like:
- "How sensitive is this valuation to the growth rate?"
- "Build a data table showing value at different discount rates and margins"
- "What variables matter most here — make me a tornado chart"
- "Give me a best case, base case, worst case scenario analysis"
- "At what price does this deal break even?"

## Technique 1: one-way sensitivity — testing a single variable across a range

Use this when you want to see how the output changes as one single input moves up and down, holding everything else fixed.

**What you need:** the variable's name, its base (starting) value, how far above and below that base to test (as a percentage), how many test points to use, a way to recalculate the output, and a way to update the model with a new value for the variable.

**Step 1, work out the range to test.**

Minimum test value = base value × (1 − range percent)

Maximum test value = base value × (1 + range percent)

For example, with a base value of 1,000 and a range percent of 20% (0.20): minimum is 1,000 × 0.80 = 800, maximum is 1,000 × 1.20 = 1,200.

**Step 2, split that range into evenly spaced test points.**

If you're asked for a certain number of steps (say, 5), the points are spaced evenly from the minimum to the maximum, inclusive of both ends. The spacing between each point is:

Step size = (Maximum test value − Minimum test value) ÷ (Number of steps − 1)

With minimum 800, maximum 1,200, and 5 steps: step size = (1,200 − 800) ÷ (5 − 1) = 400 ÷ 4 = 100. The five test values are 800, 900, 1,000, 1,100, and 1,200.

**Step 3, for every one of those test values, do the following:**

1. Update the model so the variable equals this test value.
2. Recalculate the model's output.
3. Percent change from base = (This test value − Base value) ÷ Base value × 100. For 800 versus a base of 1,000: (800 − 1,000) ÷ 1,000 × 100 = −20%.
4. Output change from base = This output − Base output. If you haven't separately calculated a base-case output before starting, treat this as 0 rather than guessing.

Record, for each test value: the variable name, the test value itself, the percent change from base, the resulting output, and the output change from base.

**Step 4, reset.** After the last test value, set the model's variable back to its original base value so it isn't left on the final test point.

The end result is a small table, one row per test value, showing exactly how the output moved as that one variable moved.

## Technique 2: two-way sensitivity — testing two variables against each other in a grid

Use this when you want to see the output for every combination of two variables at once, arranged as a grid (rows for one variable, columns for the other).

**What you need:** both variables' names, base values, and a list of test values (a "range") for each one, plus a way to update the model with both variables at once and a way to recalculate the output.

**Step 1, build an empty grid** with one row for every value in the first variable's range, and one column for every value in the second variable's range.

**Step 2, fill in every cell.** For each row value (call it "value 1") and each column value (call it "value 2"), in turn:

1. Update the model so the first variable equals value 1 and the second variable equals value 2, at the same time.
2. Recalculate the output.
3. Write that output into the grid cell at this row and column.

Do this for every possible pairing of a row value with a column value — if there are 5 row values and 5 column values, that's 25 cells to fill.

**Step 3, reset.** Once every cell is filled, set both variables back to their original base values.

**Step 4, label the grid.** Each row label is the first variable's name plus its value for that row; each column label is the second variable's name plus its value for that column. If a value is less than 1, it's shown as a percentage with two decimal places (for example, "WACC=9.50%"); otherwise it's shown as a plain number with one decimal place (for example, "Multiple=10.0").

The result is a full table where you can look up the output for any combination of the two variables at a glance.

## Technique 3: tornado analysis — ranking variables by how much they move the output

Use this when the user wants to know which variables matter most, not just how one or two specific variables behave. The name comes from the resulting chart: when the variables are sorted from biggest impact to smallest and drawn as horizontal bars, the shape resembles a tornado, widest at the top.

**What you need:** a list of variables, where each one has a base value, a low value, a high value, and its own way to update the model; and a way to recalculate the output.

**Step 1, calculate the base output.** Before testing anything, recalculate the output with the model exactly as it currently stands. Keep this number — call it the base output. Every variable's impact gets measured against this same number.

**Step 2, for each variable in the list, do the following three things in order:**

1. Update the model so this variable equals its low value. Recalculate the output — call this the low output.
2. Update the model so this variable equals its high value. Recalculate the output — call this the high output.
3. Update the model so this variable equals its base value again, resetting it before moving to the next variable.

**Step 3, calculate four numbers for this variable:**

Impact = the absolute value of (High output − Low output). Absolute value means: if the result is negative, drop the minus sign — impact is always reported as a positive size, not a direction.

Low delta = Low output − Base output

High delta = High output − Base output

Impact percent = Impact ÷ Base output × 100

**Step 4, once every variable has been tested,** sort the whole list by Impact, from largest to smallest. The variable at the top of the sorted list moves the output the most; the one at the bottom moves it the least.

The finished table has one row per variable, showing its base/low/high input values, its low/high output values, both deltas, its impact, and its impact as a percentage of the base output — sorted so the biggest driver of the output is listed first.

## Technique 4: scenario analysis — probability-weighted outcomes across named scenarios

Use this when the user wants a small number of named, self-consistent scenarios (for example "Best Case," "Base Case," "Worst Case") rather than a mechanical range of one or two variables.

**What you need:** a set of named scenarios, where each scenario specifies particular values for one or more variables; a way to update each named variable; a way to recalculate the output; and, optionally, a probability for each scenario.

**Step 1, for each scenario, in turn:**

1. For every variable that this scenario specifies a value for, update the model with that value, using that variable's own update method.
2. Recalculate the output for the model in this state.
3. Determine this scenario's probability: use the probability the user gave for it, if any; otherwise, split probability equally across however many scenarios there are, so each one gets (1 ÷ number of scenarios).
4. Record the scenario's name, its probability, its output, and every variable value that was used to produce it.

Important caveat carried over from the original design: this process does not automatically restore the model to its base state between scenarios unless every variable used anywhere is explicitly reset as part of the next scenario's own variable list. If two scenarios don't specify the same set of variables, a value left over from an earlier scenario can leak into a later one. Reset every relevant variable explicitly at the start of each scenario to avoid this.

**Step 2, once every scenario has a recorded output and probability:**

Weighted output for each scenario = that scenario's output × that scenario's probability

Expected value = the sum of every scenario's weighted output, added together

**Step 3, add one more row to the results** labeled "Expected Value," with probability 1.0, and both the output and weighted output set equal to the expected value calculated above. This gives you a single probability-blended number alongside the individual named scenarios.

## Technique 5: breakeven analysis — finding the input value that hits a target output

Use this when the user asks a question shaped like "at what value of X does the output equal Y" — for example, "what growth rate gets us to a $10 valuation" or "what price do we need to break even."

**What you need:** the variable to adjust, a way to update it, a way to recalculate the output, the target output value you're trying to hit, a minimum and maximum bound to search between, and a tolerance (how close is "close enough" — defaults to 0.01 if not specified).

This uses a method called binary search (also called bisection): repeatedly cutting the search range in half, checking the midpoint, and narrowing the range based on whether the midpoint's output is too high or too low.

**Important precondition:** binary search only works correctly if the output moves in one consistent direction as the variable increases across the whole search range — always up, or always down, never both. If the relationship isn't consistently one-directional across your search bounds, this method can converge on the wrong answer or fail to converge. Check that assumption before trusting the result.

**The steps:**

1. Set Low = the minimum bound, High = the maximum bound.
2. While (High − Low) is still greater than the tolerance, repeat the following:
   a. Midpoint = (Low + High) ÷ 2
   b. Update the model so the variable equals Midpoint.
   c. Recalculate the output at Midpoint.
   d. If the absolute difference between this output and the target value is less than the tolerance, you've found the breakeven point — it's Midpoint, and you can stop here.
   e. Otherwise, if the output at Midpoint is still less than the target, the true answer must be higher than Midpoint, so set Low = Midpoint and repeat.
   f. Otherwise (the output at Midpoint is greater than the target), the true answer must be lower than Midpoint, so set High = Midpoint and repeat.
3. If the loop ends because (High − Low) shrank down to the tolerance without ever satisfying step 2d exactly, the answer is (Low + High) ÷ 2 at that point.

Each pass through the loop cuts the remaining search range in half, so it converges quickly even if the starting range is wide.

## Technique 6: Excel-style data table — same idea as technique 2, framed as separate row/column inputs

This produces the same kind of grid as the two-way sensitivity in technique 2, but is set up slightly differently: instead of one function that updates both variables together, you have two completely separate variables, each with its own name, its own list of test values, and its own independent update method.

**Step 1,** for every value in the row variable's list, and for every value in the column variable's list:

1. Update the row variable to this row value, using its own update method.
2. Update the column variable to this column value, using its own update method.
3. Recalculate the output.
4. Store that output in the grid at this row and column position.

**Step 2,** label the finished grid with the row variable's name and its list of values down the side, and the column variable's name and its list of values across the top.

This is functionally identical to technique 2 — the difference is purely in how the two variables are updated (one combined update versus two separate ones), not in the resulting table.

## Worked example, start to finish

A simple valuation model: Revenue of 1,000, a Margin of 20% (0.20), and a Multiple of 10.

Its output formula is: EBITDA = Revenue × Margin, then Value = EBITDA × Multiple. With the numbers above: EBITDA = 1,000 × 0.20 = 200, and Value = 200 × 10 = 2,000.

**One-way sensitivity on Revenue,** base 1,000, range 20% (0.20), 5 steps. From technique 1: test values are 800, 900, 1,000, 1,100, 1,200 (worked out above). Holding Margin at 0.20 and Multiple at 10, the outputs are:

- Revenue 800: Value = 800 × 0.20 × 10 = 1,600, percent change −20%
- Revenue 900: Value = 900 × 0.20 × 10 = 1,800, percent change −10%
- Revenue 1,000: Value = 1,000 × 0.20 × 10 = 2,000, percent change 0%
- Revenue 1,100: Value = 1,100 × 0.20 × 10 = 2,200, percent change +10%
- Revenue 1,200: Value = 1,200 × 0.20 × 10 = 2,400, percent change +20%

**Tornado analysis on three variables:** Revenue (base 1,000, low 800, high 1,200), Margin (base 0.20, low 0.15, high 0.25), and Multiple (base 10, low 8, high 12). Base output, with all three at their base values, is 2,000 as calculated above.

Revenue: low output = 800 × 0.20 × 10 = 1,600; high output = 1,200 × 0.20 × 10 = 2,400; impact = |2,400 − 1,600| = 800; low delta = 1,600 − 2,000 = −400; high delta = 2,400 − 2,000 = +400; impact percent = 800 ÷ 2,000 × 100 = 40%.

Margin: low output = 1,000 × 0.15 × 10 = 1,500; high output = 1,000 × 0.25 × 10 = 2,500; impact = |2,500 − 1,500| = 1,000; low delta = 1,500 − 2,000 = −500; high delta = 2,500 − 2,000 = +500; impact percent = 1,000 ÷ 2,000 × 100 = 50%.

Multiple: low output = 1,000 × 0.20 × 8 = 1,600; high output = 1,000 × 0.20 × 12 = 2,400; impact = |2,400 − 1,600| = 800; low delta = −400; high delta = +400; impact percent = 40%.

Sorted by impact, largest first: Margin (impact 1,000, 50%) comes first, then Revenue and Multiple tied at an impact of 800 (40% each). In the finished tornado chart, Margin's bar would be drawn widest, since it moves the output the most for the size of range tested.

Use this worked example to check your own arithmetic when running any of these six techniques against a real model.
