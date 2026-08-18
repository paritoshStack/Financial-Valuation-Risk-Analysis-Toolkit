---
name: Monte-Carlo-Simulation
description: Run a Monte Carlo simulation on a financial model — drawing random values from probability distributions for uncertain inputs, running the model many times, and summarizing the resulting distribution of outputs. Use whenever the user asks for a Monte Carlo simulation, a probability distribution of a valuation or forecast, a "what's the chance the value comes in above/below X" question, a confidence interval or percentile range around a financial estimate, or wants to combine several uncertain assumptions (growth rate, margin, discount rate, and so on) into one risk-weighted result. Pairs naturally with a DCF valuation model or any other model that produces a single output from a set of inputs.
---

# Monte Carlo simulation

The DCF valuation skill runs a model once, using one fixed value for every assumption. The sensitivity analysis skill runs it a handful of times, at specific chosen points — a range of test values, a low and a high, a few named scenarios. Monte Carlo simulation is a third, different approach: instead of fixed values or a small number of test points, every uncertain input gets a probability distribution describing the range of values it could plausibly take and how likely each part of that range is. The model is then run hundreds or thousands of times, drawing a fresh random value from each distribution on every run, and the resulting pile of outputs is summarized as a full distribution — a most likely value, a spread, and the odds of landing above or below any particular threshold.

This is the natural next step after building a DCF: instead of a single enterprise value, you get a range of plausible enterprise values and a sense of how confident to be in any one number within that range.

## When to use this

Trigger for requests like:
- "Run a Monte Carlo simulation on this valuation"
- "What's the probability the value comes in below $X?"
- "Give me a confidence interval around this forecast, not just a point estimate"
- "I'm unsure about three assumptions at once — combine the uncertainty into one result"
- "How risky is this projection, given how uncertain the growth rate and margin both are?"

## Step 1: decide which inputs are uncertain enough to randomize

Not every input needs a distribution. Pick the ones the user is genuinely unsure about — commonly revenue growth, EBITDA margin, the discount rate (WACC) or its components, and terminal growth rate, if you're feeding this into a DCF. Leave everything else fixed at its base value. Randomizing an input the user is actually confident about just adds noise without adding insight.

For every input you do randomize, you need three things: which type of distribution best describes its uncertainty, the parameters that distribution needs, and confirmation with the user that the range feels right before running anything.

## Step 2: choose a distribution type and its parameters, per input

Three distribution shapes cover almost every case in financial modeling.

**Uniform distribution.** Use this when any value between a minimum and a maximum is equally likely, with no reason to favor the middle or the edges. Needs two parameters: a minimum and a maximum.

**Triangular distribution.** Use this when there's a clear "most likely" value, but the input could still land anywhere between a low and a high, with values near the most-likely point more probable than values near the edges. This is usually the best default for financial assumptions like growth rates and margins, since people can usually state a low case, a base case, and a high case even if they can't state a precise statistical shape. Needs three parameters: a minimum, a most-likely value (called the mode), and a maximum.

**Normal (bell curve) distribution.** Use this when the input is expected to cluster symmetrically around an average, with values far from the average becoming steadily less likely in both directions equally. Needs two parameters: a mean (the average, and the center of the bell) and a standard deviation (how wide the spread is — roughly two-thirds of outcomes fall within one standard deviation of the mean, and about 95% fall within two).

**Lognormal distribution** (a less common fourth option, worth knowing). Use this for inputs that can never go negative and tend to have a long tail on the high side — asset prices and revenue multiples sometimes fit this better than a normal distribution. It's built from a normal distribution applied to the logarithm of the value rather than the value itself; the formula is given in step 3.

## Step 3: the formula for drawing one random sample from each distribution

Every method below starts from a single raw random number, call it U, drawn uniformly between 0 and 1 — meaning every value in that range is equally likely. Any standard random number generator produces this kind of number; it's the raw material every other distribution is built from.

**Uniform sample**, given a minimum and maximum:

Sample = Minimum + U × (Maximum − Minimum)

Worked example: Minimum 8, Maximum 12, U = 0.65. Sample = 8 + 0.65 × (12 − 8) = 8 + 2.6 = 10.6.

**Triangular sample**, given a Minimum, a Mode (most likely value), and a Maximum. This one has two cases depending on where the raw random number U falls relative to the mode's position within the range.

First, find the mode's position within the range:

Mode Position = (Mode − Minimum) ÷ (Maximum − Minimum)

If U is less than or equal to Mode Position, use:

Sample = Minimum + the square root of [U × (Maximum − Minimum) × (Mode − Minimum)]

If U is greater than Mode Position, use:

Sample = Maximum − the square root of [(1 − U) × (Maximum − Minimum) × (Maximum − Mode)]

Worked example: Minimum 800, Mode 1,000, Maximum 1,300, U = 0.65.

Mode Position = (1,000 − 800) ÷ (1,300 − 800) = 200 ÷ 500 = 0.40.

Since U (0.65) is greater than Mode Position (0.40), use the second formula:

Sample = 1,300 − square root of [(1 − 0.65) × (1,300 − 800) × (1,300 − 1,000)]
Sample = 1,300 − square root of [0.35 × 500 × 300]
Sample = 1,300 − square root of 52,500
Sample = 1,300 − 229.13
Sample ≈ 1,070.87

**Normal sample**, given a Mean and a Standard Deviation. A normal sample needs a "standard normal" building block first — call it Z, a random draw from a normal distribution with mean 0 and standard deviation 1 — and then it's rescaled:

Sample = Mean + Standard Deviation × Z

To get Z itself from two fresh raw random numbers, U1 and U2 (each drawn uniformly between 0 and 1, with U1 not equal to 0), use the Box-Muller transform:

Z = square root of (−2 × natural log of U1) × cosine of (2 × π × U2)

Worked example: U1 = 0.30, U2 = 0.70.

Natural log of 0.30 ≈ −1.20397

−2 × (−1.20397) = 2.40794

Square root of 2.40794 ≈ 1.5518

2 × π × 0.70 ≈ 4.3982 radians

Cosine of 4.3982 radians ≈ −0.3040

Z ≈ 1.5518 × (−0.3040) ≈ −0.4717

If Mean = 0.20 and Standard Deviation = 0.03: Sample = 0.20 + 0.03 × (−0.4717) ≈ 0.20 − 0.01415 ≈ 0.1859, or about 18.6%.

**Lognormal sample**, given a mean and standard deviation stated in "log space" (these are not the same as the mean and standard deviation of the final values — they describe the underlying normal distribution before it's transformed):

Sample = e raised to the power of (Log-Space Mean + Log-Space Standard Deviation × Z), where Z is the same standard normal draw described above, and e is Euler's number, approximately 2.71828.

## Step 4: choose the number of trials

A "trial" is one full run of the model, using one freshly drawn random value for every randomized input. More trials produce a smoother, more reliable picture of the output distribution, at the cost of more calculation.

As a working guideline: a few hundred trials gives a rough picture; 1,000 to 10,000 trials is the normal range for a financial model and is enough for the summary statistics in step 6 to stabilize; going beyond that mostly sharpens the extreme percentiles (like the 1st or 99th) rather than the middle of the distribution. Step 7 gives a way to check whether you've run enough.

## Step 5: run the simulation

For each trial, from the first to the last:

1. For every input that was assigned a distribution in step 2, draw one fresh random sample from that distribution using the matching formula from step 3.
2. Leave every other input at its fixed base value — don't randomize anything that wasn't explicitly chosen for randomization in step 1.
3. Plug all of these values — the freshly drawn ones and the fixed ones — into the model (for example, all eight steps of the DCF valuation skill) and calculate the single output this combination produces (for example, enterprise value).
4. Record that output. Do not average or otherwise combine it with anything yet — just add it to a running list of one output per trial.

Once every trial has been run, you're left with one list of outputs, as long as the number of trials chosen in step 4. This list is the raw material for every summary statistic in step 6.

## Step 6: summarize the resulting distribution of outputs

Let N be the number of trials, and let the recorded outputs be the full list from step 5.

**Mean (average) output:**

Mean = (sum of every output in the list) ÷ N

**Standard deviation of the outputs** (how spread out the results are):

Standard Deviation = the square root of [ (sum, across every output, of (that output − Mean) squared) ÷ (N − 1) ]

**Minimum and maximum:** simply the smallest and largest values that appeared anywhere in the list of outputs.

**Percentiles.** To find the Pth percentile (for example, the 5th percentile, meaning "only 5% of trials came in below this"): sort every output in the list from smallest to largest, then take the value sitting at position P% of the way through that sorted list — for the 5th percentile of 1,000 sorted trials, that's roughly the 50th value from the bottom; for the 95th percentile, roughly the 950th value from the bottom.

**Probability of falling above or below a target value:** count how many of the outputs in the list are below (or above) the target, then divide that count by N and multiply by 100 to express it as a percentage. For example, if 68 out of 1,000 trials produced an enterprise value below $1,500, the probability of coming in below $1,500 is 68 ÷ 1,000 × 100 = 6.8%.

**Confidence interval.** A 90% confidence interval, for example, is simply the range from the 5th percentile to the 95th percentile of the sorted outputs — the middle 90% of everything the simulation produced, with the most extreme 5% trimmed off each end.

## Step 7 (optional): check whether you ran enough trials

The Standard Error of the Mean tells you how much the calculated mean itself might wobble if you reran the whole simulation with a fresh batch of random numbers:

Standard Error = Standard Deviation ÷ (square root of N)

As N grows, the standard error shrinks — quadrupling the number of trials only halves it, since it depends on the square root of N, not N directly. A simple practical check: run the simulation once, note the mean; double the number of trials and rerun; if the mean barely moves, you've run enough. If it's still moving noticeably, keep adding trials.

## Step 8 (optional, advanced): correlated inputs

The formulas above assume every randomized input is drawn independently — knowing one input's random draw tells you nothing about another's. In reality, some inputs move together: a company with unusually strong revenue growth in a given random draw might also tend to have a stronger EBITDA margin in that same draw, not by coincidence but because both are driven by the same underlying business conditions.

Ignoring a real correlation like this tends to understate how extreme the best and worst outcomes can be, because it lets favorable draws on one variable get randomly paired with unfavorable draws on the other, canceling out combinations that wouldn't actually happen together in practice.

Building correlation into the draws properly requires a technique called Cholesky decomposition: starting from a correlation matrix (a table of how strongly each pair of variables tends to move together, from −1 for perfectly opposite to +1 for perfectly in lockstep), that matrix is broken down into a form that lets you take a set of independent standard normal draws (the Z values from step 3) and blend them into a new set of draws that preserves the intended correlations, before converting each one into its target distribution. This is a meaningfully more advanced step than anything else in this skill — flag to the user that it requires either statistical software or a careful manual matrix calculation, and confirm they actually need it (most sensitivity questions don't) before taking it on.

## Worked example, start to finish, using a small number of trials for clarity

A simplified valuation model: Value = Revenue × Margin × Multiple, the same model used in the sensitivity analysis skill's worked example.

Assign distributions: Revenue is triangular, with Minimum 800, Mode 1,000, Maximum 1,300. Margin is normal, with Mean 0.20 and Standard Deviation 0.03. Multiple is uniform, with Minimum 8 and Maximum 12.

Suppose, for five trials, the random number generator produces the following already-converted samples (in practice you'd derive each one using the formulas in step 3; here they're given directly to keep the example readable):

Trial 1: Revenue 950, Margin 0.198, Multiple 9.5 → Value = 950 × 0.198 × 9.5 ≈ 1,787.6

Trial 2: Revenue 1,071, Margin 0.186, Multiple 10.6 → Value = 1,071 × 0.186 × 10.6 ≈ 2,111.4

Trial 3: Revenue 1,020, Margin 0.215, Multiple 8.8 → Value = 1,020 × 0.215 × 8.8 ≈ 1,929.8

Trial 4: Revenue 890, Margin 0.204, Multiple 11.2 → Value = 890 × 0.204 × 11.2 ≈ 2,033.9

Trial 5: Revenue 1,180, Margin 0.192, Multiple 9.9 → Value = 1,180 × 0.192 × 9.9 ≈ 2,242.9

The five recorded outputs, sorted smallest to largest: 1,787.6, 1,929.8, 2,033.9, 2,111.4, 2,242.9.

Mean = (1,787.6 + 1,929.8 + 2,033.9 + 2,111.4 + 2,242.9) ÷ 5 = 10,105.6 ÷ 5 = 2,021.1

For the standard deviation, first find each output's difference from the mean, squared:

(1,787.6 − 2,021.1)² ≈ (−233.5)² ≈ 54,522

(1,929.8 − 2,021.1)² ≈ (−91.3)² ≈ 8,336

(2,033.9 − 2,021.1)² ≈ (12.8)² ≈ 164

(2,111.4 − 2,021.1)² ≈ (90.3)² ≈ 8,154

(2,242.9 − 2,021.1)² ≈ (221.8)² ≈ 49,195

Sum of squared differences ≈ 54,522 + 8,336 + 164 + 8,154 + 49,195 = 120,371

Standard Deviation = square root of [120,371 ÷ (5 − 1)] = square root of 30,092.75 ≈ 173.5

With only 5 trials, this is far too few to draw real conclusions from — a genuine analysis would use at least several hundred — but the mechanics are identical at any scale: draw the samples, run the model, record the output, then apply the mean, standard deviation, and percentile formulas from step 6 to however many outputs you have.

If this were feeding into a DCF valuation, the "output" in every trial above would be the DCF's enterprise value from step 6 of that skill, recalculated in full for each trial's random combination of growth rate, margin, WACC, or whichever inputs were chosen for randomization here in step 1.
