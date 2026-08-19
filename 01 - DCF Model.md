---
name: dcf-valuation

description: Build a discounted cash flow (DCF) enterprise valuation from historical financials and projection assumptions. Use whenever the user asks to value a company, build a DCF model, calculate WACC, project free cash flow, estimate terminal value, run an EV/EBITDA or equity value analysis, or run sensitivity analysis on a valuation. Also covers helper calculations: beta from stock/market returns, and FCF CAGR.

---

# DCF valuation

This skill builds a full company valuation from historical financials through to a per-share equity value. Every formula used is written out below in full, with every variable named. Follow the steps in order — each one feeds the next.

## When to use this

Trigger for requests like:
- "Value this company using a DCF"
- "Build a 5-year cash flow projection and discount it"
- "Calculate WACC for this company"
- "What's the enterprise value, equity value, or per-share value here?"
- "Run a sensitivity table on WACC vs terminal growth"

## Step 1: historical financials and two derived ratios

Collect, for each historical year: Revenue, EBITDA, Capex, NWC (net working capital), and the year label.

From these, calculate two ratios for every historical year:

EBITDA Margin for year = EBITDA for that year, divided by Revenue for that year.

Capex Percent for year = Capex for that year, divided by Revenue for that year.

Example, using year 2024 with Revenue 1,000 and EBITDA 220:
EBITDA Margin = 220 divided by 1,000 = 0.22, or 22%.

Keep every historical year's EBITDA Margin and Capex Percent. You'll use their average in step 2 if the user doesn't give you projection assumptions directly.

## Step 2: projection assumptions, with exact fallback values

Decide the number of projection years. Fallback: 5.

For every projection year, you need six things. Here is each one and its exact fallback if the user doesn't supply it:

1. Revenue Growth Rate — fallback: 10% (0.10) for every year.
2. EBITDA Margin — fallback: the average of all historical EBITDA Margins from step 1, calculated as (sum of every historical year's EBITDA Margin) divided by (number of historical years). If there is no historical data at all, fallback: 20% (0.20).
3. Capex Percent (capex as a percent of revenue) — fallback: 5% (0.05).
4. NWC Percent (net working capital as a percent of revenue) — fallback: 10% (0.10).
5. Tax Rate — fallback: 25% (0.25).
6. Terminal Growth Rate (the rate the business grows forever after the projection window ends) — fallback: 3% (0.03).

Any fallback used should be flagged to the user, since these are placeholder numbers, not researched ones.

## Step 3: WACC, the discount rate — full formula

WACC (weighted average cost of capital) is the rate used to bring every future dollar of cash flow back to a present-day value. You need five inputs to compute it:

- Risk-Free Rate (commonly the 10-year U.S. Treasury yield)
- Beta (the company's equity beta, a measure of volatility relative to the market)
- Market Premium (the equity market risk premium)
- Cost of Debt (the company's pre-tax cost of borrowing)
- Debt-to-Equity (the company's debt divided by its equity, as a ratio)
- Tax Rate — if not given separately for this step, use the Tax Rate from step 2, and if that's also missing, use 25% (0.25)

Formula 1, Cost of Equity, using the CAPM (Capital Asset Pricing Model):

Cost of Equity = Risk-Free Rate + (Beta × Market Premium)

Formula 2, Equity Weight:

Equity Weight = 1 ÷ (1 + Debt-to-Equity)

Formula 3, Debt Weight:

Debt Weight = Debt-to-Equity ÷ (1 + Debt-to-Equity)

Formula 4, WACC itself:

WACC = (Equity Weight × Cost of Equity) + (Debt Weight × Cost of Debt × (1 − Tax Rate))

The (1 − Tax Rate) term is there because interest on debt is tax-deductible, so the true after-tax cost of debt is lower than its stated rate.

Worked example, using Risk-Free Rate 4% (0.04), Beta 1.2, Market Premium 7% (0.07), Cost of Debt 5% (0.05), Debt-to-Equity 0.5, Tax Rate 25% (0.25):

Cost of Equity = 0.04 + (1.2 × 0.07) = 0.04 + 0.084 = 0.124, or 12.4%

Equity Weight = 1 ÷ (1 + 0.5) = 1 ÷ 1.5 = 0.6667

Debt Weight = 0.5 ÷ 1.5 = 0.3333

WACC = (0.6667 × 0.124) + (0.3333 × 0.05 × (1 − 0.25))
WACC = 0.08267 + (0.3333 × 0.05 × 0.75)
WACC = 0.08267 + 0.0125
WACC = 0.09517, or about 9.52%

Keep every one of these intermediate numbers (Cost of Equity, Equity Weight, Debt Weight, WACC) — the summary in step 8 reports them individually.

## Step 4: projecting cash flow, year by year — full formula for every line item

Starting point before year 1:

Base Revenue = the most recent historical year's revenue, or 1,000 if no historical data exists.

Starting NWC = Base Revenue × 0.10

Then, for each projection year, in order from year 1 to the final year, calculate the following ten lines, using that year's assumptions from step 2 and the previous year's Revenue and NWC (starting from Base Revenue and Starting NWC for year 1):

1. Revenue = Previous Year's Revenue × (1 + This Year's Revenue Growth Rate)

2. EBITDA = This Year's Revenue × This Year's EBITDA Margin

3. Depreciation = This Year's Revenue × This Year's Capex Percent
   (This model assumes depreciation equals capex. That's a simplification made for the sake of a single formula, not a real accounting rule — flag it if capex is running well above what real depreciation would be.)

4. EBIT = EBITDA − Depreciation

5. Tax = EBIT × Tax Rate

6. NOPAT (Net Operating Profit After Tax) = EBIT − Tax

7. Capex = This Year's Revenue × This Year's Capex Percent
   (Note this uses the same formula as Depreciation in line 3 — under this model's assumption, they are always equal.)

8. NWC (this year) = This Year's Revenue × This Year's NWC Percent

9. Change in NWC = This Year's NWC − Previous Year's NWC

10. Free Cash Flow (FCF) = NOPAT + Depreciation − Capex − Change in NWC

After finishing a year, that year's Revenue becomes "Previous Year's Revenue" and that year's NWC becomes "Previous Year's NWC" for the next year's calculations. Repeat for every projection year.

Worked example, year 1 only, using Base Revenue 1,000, Starting NWC 100, Revenue Growth Rate 15% (0.15), EBITDA Margin 23% (0.23), Capex Percent 5% (0.05), NWC Percent 10% (0.10), Tax Rate 25% (0.25):

Revenue = 1,000 × (1 + 0.15) = 1,150

EBITDA = 1,150 × 0.23 = 264.5

Depreciation = 1,150 × 0.05 = 57.5

EBIT = 264.5 − 57.5 = 207

Tax = 207 × 0.25 = 51.75

NOPAT = 207 − 51.75 = 155.25

Capex = 1,150 × 0.05 = 57.5

NWC (this year) = 1,150 × 0.10 = 115

Change in NWC = 115 − 100 = 15

FCF = 155.25 + 57.5 − 57.5 − 15 = 140.25

Year 1 FCF is 140.25. Year 2 would repeat this whole process, starting from Previous Year's Revenue = 1,150 and Previous Year's NWC = 115.

## Step 5: terminal value — full formula for both methods

The terminal value stands in for every year of cash flow beyond the projection window. Use one of these two methods.

**Method A — perpetuity growth (the default).**

Terminal FCF = Final Projected Year's FCF × (1 + Terminal Growth Rate)

Terminal Value = Terminal FCF ÷ (WACC − Terminal Growth Rate)

Check before using this result: if Terminal Growth Rate is close to WACC or higher than WACC, this division produces either an unreasonably huge number or a negative one. Don't report a terminal value from this formula in that case — flag the problem instead.

**Method B — exit multiple.**

Terminal Value = Final Projected Year's EBITDA × Exit Multiple

Exit Multiple fallback if not given: 10 (meaning 10 times EBITDA).

## Step 6: discounting to present value and enterprise value — full formula

You need every year's FCF from step 4 and the WACC from step 3.

For each projection year (call its position in the sequence t, so t = 1 for the first projection year, t = 2 for the second, and so on up to the final year):

Discount Factor for year t = (1 + WACC) raised to the power of t

Present Value of FCF for year t = FCF for year t ÷ Discount Factor for year t

Present Value of FCF (total) = the sum of Present Value of FCF for every projection year, added together.

For the terminal value from step 5, using n = the total number of projection years:

Discount Factor for terminal value = (1 + WACC) raised to the power of n

Present Value of Terminal Value = Terminal Value ÷ Discount Factor for terminal value

Enterprise Value = Present Value of FCF (total) + Present Value of Terminal Value

Also calculate:

Terminal Value Percent of Enterprise Value = (Present Value of Terminal Value ÷ Enterprise Value) × 100

If this comes out above roughly 75%, flag it to the user — it means the valuation depends almost entirely on the terminal value assumptions, which are the least certain part of the model.

Worked example, using a 5-year projection with a WACC of 9.52% (0.0952) and a Terminal Value of, say, 3,000, discounted:

Discount Factor for terminal value = (1 + 0.0952) raised to the power of 5 = 1.0952^5 ≈ 1.5751

Present Value of Terminal Value = 3,000 ÷ 1.5751 ≈ 1,904.7

(Each individual year's FCF gets the same treatment, using t = 1, 2, 3, 4, 5 in place of 5, then all of those present values plus this terminal present value are added together for Enterprise Value.)

## Step 7: equity value and value per share — full formula (optional)

Only do this step if the user wants equity value or a per-share price, not just enterprise value.

You need: Net Debt (total debt minus cash), Cash (only if it wasn't already subtracted into Net Debt), and Shares Outstanding.

Equity Value = Enterprise Value − Net Debt + Cash

Value Per Share = Equity Value ÷ Shares Outstanding

If Shares Outstanding is zero, Value Per Share is undefined — don't calculate it.

Worked example, using Enterprise Value 5,000, Net Debt 200, Cash 0, Shares Outstanding 50:

Equity Value = 5,000 − 200 + 0 = 4,800

Value Per Share = 4,800 ÷ 50 = 96

## Step 8: the summary report

Write a plain report with these fields, all of which were produced in the steps above:

- Company name
- Number of projection years
- Average Revenue Growth Rate across the projection = (sum of every projection year's Revenue Growth Rate) ÷ (number of projection years)
- Average EBITDA Margin across the projection = (sum of every projection year's EBITDA Margin) ÷ (number of projection years)
- Terminal Growth Rate
- WACC
- Enterprise Value
- Present Value of FCF (total)
- Present Value of Terminal Value
- Terminal Value Percent of Enterprise Value

If step 7 was done, also include:

- Equity Value
- Shares Outstanding
- Value Per Share

## Optional: sensitivity analysis — full method

Use this when the user is unsure about one or two assumptions, or wants a range of outcomes instead of one number.

Choose two variables to test, each one of: WACC, Terminal Growth Rate, or EBITDA Margin. For each of the two, choose a range of test values — for example, five WACC values from 8% to 12%, and five Terminal Growth Rate values from 1% to 5%.

Build a grid with one row per value in the first range and one column per value in the second range. For every cell in that grid (meaning every pairing of one value from range one with one value from range two):

1. Temporarily set the model's WACC, Terminal Growth Rate, or EBITDA Margin to the two test values for this cell (whichever two variables were chosen).
2. Redo the full cash flow projection from step 4 using these temporary values.
3. Redo the terminal value calculation from step 5 and the enterprise value calculation from step 6 using these temporary values.
4. Record the resulting Enterprise Value in this cell of the grid.

Once every cell has been filled in, reset WACC, Terminal Growth Rate, and EBITDA Margin back to their original values — don't leave the model sitting on the last test combination. The finished grid shows how much the Enterprise Value moves as the two chosen assumptions change.

## Optional helper calculations — full formulas

**Beta from historical returns.**

Beta = Covariance(Stock Returns, Market Returns) ÷ Variance(Market Returns)

Where Covariance measures how the stock's returns and the market's returns move together across the same time periods, and Variance measures how spread out the market's own returns are. If Variance of Market Returns is exactly zero, use Beta = 1.0 instead of dividing by zero.

**Free cash flow CAGR (compound annual growth rate).**

Requires at least two FCF data points, and both the first and the last FCF value in the series must be positive numbers. If either condition fails, CAGR = 0.

Otherwise, with n = the number of years between the first and last data point:

CAGR = (Last FCF ÷ First FCF) raised to the power of (1 ÷ n), then subtract 1.

Worked example, First FCF = 140.25, Last FCF (5 years later) = 220, n = 4 (four year-gaps across five data points):

CAGR = (220 ÷ 140.25) raised to the power of (1 ÷ 4), minus 1
CAGR = 1.5686 raised to the power of 0.25, minus 1
CAGR ≈ 1.1194 − 1
CAGR ≈ 0.1194, or about 11.9% per year

## Full worked example, start to finish

TechCorp. Historical years 2022–2024: Revenue 800, 900, 1,000. EBITDA 160, 189, 220. Capex 40, 45, 50. NWC 80, 90, 100.

Historical EBITDA Margins: 160÷800 = 0.20, 189÷900 = 0.21, 220÷1,000 = 0.22. Average = (0.20+0.21+0.22)÷3 = 0.21, or 21%.

Assumptions: 5-year projection. Revenue Growth Rate 15%, 12%, 10%, 8%, 6% for years 1 through 5. EBITDA Margin 23%, 24%, 25%, 25%, 25%. Tax Rate 25%. Terminal Growth Rate 3%.

WACC inputs: Risk-Free Rate 4%, Beta 1.2, Market Premium 7%, Cost of Debt 5%, Debt-to-Equity 0.5. As calculated in step 3: WACC ≈ 9.52%.

Year 1 cash flow, as calculated in step 4: Revenue 1,150, EBITDA 264.5, Depreciation 57.5, EBIT 207, Tax 51.75, NOPAT 155.25, Capex 57.5, NWC 115, Change in NWC 15, FCF 140.25.

Continue this same ten-line process for years 2 through 5, each time using that year's own Revenue Growth Rate and EBITDA Margin, and carrying forward the prior year's Revenue and NWC.

Once all five years of FCF are known, apply step 5 to get a terminal value from the year-5 FCF and the 3% terminal growth rate, apply step 6 to discount every year's FCF plus the terminal value at the ~9.52% WACC and sum them into an Enterprise Value, and, if desired, apply step 7 with Net Debt 200 and Shares Outstanding 50 to get an Equity Value and Value Per Share.

Use this example to check your own arithmetic line by line when running the model on a real company's numbers — every formula above should reproduce these same intermediate figures when fed these same inputs.
