# L2: Guess the Fare and Feel the Metrics

- **Duration:** 45 to 60 minutes
- **Cloud required:** No
- **Outcome:** A plain-language comparison of fare errors using MAE and RMSE.

## Learning Objectives

1. Distinguish an actual fare from a predicted fare.
2. Calculate signed, absolute, and squared errors.
3. Explain why positive and negative errors can cancel.
4. Interpret MAE and RMSE in dollars.
5. Show why RMSE reacts more strongly to one large miss.

## Part A: Commit Fare Guesses

Complete `Your guess` and `Reason` before viewing Part B.

| Trip | Description | Your guess | Reason |
| --- | --- | ---: | --- |
| A | 1.2 miles, weekday morning, Midtown | $11.00 | Base entry fare + slow Midtown traffic |
| B | 2.5 miles, weekday afternoon, Manhattan | $16.00 | Standard daytime crosstown travel |
| C | 4 miles, evening, Manhattan to Brooklyn | $22.00 | Inter-borough trip via bridge with evening density |
| D | 8 miles, evening airport route | $32.00 | Highway mileage plus airport surcharge |
| E | 5.5 miles, late evening, Queens | $24.00 | Outer-borough late-night route |

## Part B: Calculate the Shared Errors

Use the convention:

```text
Error = prediction - actual
```

| Trip | Actual fare | Shared prediction | Error | Absolute error | Squared error |
| --- | ---: | ---: | ---: | ---: | ---: |
| A | $10 | $12 | +$2.00 | $2.00 | 4.00 |
| B | $15 | $13 | -$2.00 | $2.00 | 4.00 |
| C | $20 | $22 | +$2.00 | $2.00 | 4.00 |
| D | $30 | $40 | +$10.00 | $10.00 | 100.00 |
| E | $25 | $23 | -$2.00 | $2.00 | 4.00 |

Complete these calculations:

```text
Sum of signed errors = 2 + (-2) + 2 + 10 + (-2) = +10.00
Mean signed error = 10.00 / 5 = +$2.00

MAE = sum of absolute errors / 5 = (2 + 2 + 2 + 10 + 2) / 5 = 18.00 / 5 = $3.60

Mean squared error = sum of squared errors / 5 = (4 + 4 + 4 + 100 + 4) / 5 = 116.00 / 5 = 23.20
RMSE = square root of mean squared error = sqrt(23.20) = $4.82
```

## Part C: Interpret the Baseline

1. What information can the mean signed error hide?
   Positive (+2, +2, +10) and negative (-2, -2) errors cancel each other out algebraically, masking large individual prediction errors ($10 miss on Trip D) and giving a misleading appearance of near-zero overall bias.
2. What does MAE mean for these five predictions?
   On average, the model's prediction misses the actual fare by $3.60 in magnitude, treating over-predictions and under-predictions symmetrically.
3. Why is RMSE larger than MAE here?
   RMSE squares each error term prior to averaging, which disproportionately penalizes large misses. Trip D's $10 error squares to 100, heavily skewing the root mean square ($4.82) well above the linear mean absolute error ($3.60).
4. Which trip contributes most to RMSE, and why?
   Trip D contributes 100.00 out of the 116.00 total squared error points (86.2% of the total penalty) because its $10.00 error is five times larger than any other individual error.
5. Which operational groups should be checked separately before drawing a broad conclusion?
   Geographic slices (Manhattan vs. outer boroughs), trip types (airport routes vs. local street-hails), and temporal periods (overnight vs. peak congestion).

### Verify After Calculating

The recording verifies the hand calculation with the project's metric helpers.
Run this from the repository root after completing the baseline arithmetic:

```powershell
@'
import numpy as np
from src.models.common import mae, rmse

actual = np.array([10, 15, 20, 30, 25], dtype=float)
predicted = np.array([12, 13, 22, 40, 23], dtype=float)
print("MAE:", mae(actual, predicted))
print("RMSE:", rmse(actual, predicted))
'@ | .\.venv\Scripts\python.exe -
```

Compare full precision before rounding to cents. Inspect
[the metric helpers](../src/models/common.py) and
[their tests](../tests/unit/test_models.py). Reversing subtraction changes the
signed error, but not MAE or RMSE after absolute values or squaring.

## Part D: Extreme-Miss Comparison

Change only trip D's prediction from `$40` to `$60`.

| Metric | Baseline | Extreme case | Increase |
| --- | ---: | ---: | ---: |
| MAE | $3.60 | $7.60 | +$4.00 |
| RMSE | $4.82 | $13.54 | +$8.72 |

Record the changed trip-D values:

```text
Signed error = 60.00 - 30.00 = +$30.00
Absolute error = $30.00
Squared error = 900.00
```

After calculating the extreme case, rerun the verification block with only the
fourth prediction changed from `40` to `60`, as in the recording. The actuals
and the other four predictions must stay unchanged.

## Exit Ticket

Complete each statement:

1. Mean signed error can mislead because positive and negative errors cancel each other out, obscuring large individual misses.
2. MAE means the typical absolute distance in dollars between predicted fares and ground truth across all trips.
3. RMSE is larger in the baseline because squaring errors imposes a quadratic penalty on large outliers (such as Trip D's $10 miss).
4. MAE and RMSE should be reported with sample count (n), evaluation split context, and operational subgroup error slices.

## Completion Check

- All five personal fare guesses are recorded before the shared values appear.
- The shared error table contains signed, absolute, and squared errors.
- MAE and RMSE calculations show the working and retain dollar units.
- Repository code confirms the hand-calculated values.
- The extreme-miss comparison changes only trip D.
- The interpretations state sample size, context, and relevant subgroup checks.