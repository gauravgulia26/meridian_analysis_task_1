# Should-Cost Model: Methodology and Calculation

## 1. Objective

The objective of the should-cost analysis is to estimate a reasonable manufacturing cost for each purchased part and compare it with the supplier's observed price.

The model decomposes should cost into:

1. Material cost
2. Manufacturing / conversion cost

The supplier price is **not** assumed to equal manufacturing cost. The difference between supplier price and modeled should cost is treated as a potential price gap, not automatically as supplier margin or overcharging.

---

## 2. Available Data

The analysis uses the available part-level information:

- `unit_weight_kg`: weight of one part
- `annual_volume`: expected annual quantity
- `unit_price_local`: observed supplier unit price
- Material-price assumptions represented by low, base, and high price-per-kg estimates
- Part descriptions and cleaned category information

The dataset does not directly provide:

- Actual supplier conversion cost
- Actual press cycle time
- Actual machine rate
- Actual labor rate
- Actual tooling investment
- Actual factory overhead
- Actual secondary-operation cost

Therefore, the conversion-cost component is constructed using explicit manufacturing assumptions rather than being learned from supplier price.

---

# 3. Category Cleaning and Manufacturing Grouping

The original categories were consolidated into broader manufacturing-oriented groups.

| Original categories | Condensed category |
|---|---|
| Brackets & Mounts, Reinforcements, Rail | Structural Formed Components |
| Body Panels | Body Panels |
| Fasteners, Nut | Hardware |
| Plate | Plates |

The purpose of this consolidation is to create groups that are more meaningful for manufacturing-complexity assumptions.

The original category is retained for detailed analysis, while the condensed category is used for broader manufacturing assumptions.

---

# 4. Material Cost

For a part with weight \(W\) and material price \(P_m\) per kilogram:

\[
C_{material} = W \times P_m
\]

Three scenarios are calculated:

\[
C_{material}^{low}=W\times P_m^{low}
\]

\[
C_{material}^{base}=W\times P_m^{base}
\]

\[
C_{material}^{high}=W\times P_m^{high}
\]

### Why

Material is directly related to the physical quantity of material required to produce the component. Low, base, and high material-price scenarios avoid presenting an uncertain commodity price as an exact value.

---

# 5. Manufacturing Complexity

The dataset does not contain actual process-level information such as cycle time or number of manufacturing operations.

Therefore, the condensed categories are used to establish relative manufacturing complexity.

The analysis retains:

- `Complexity_Weight`
- `Relative_Complexity`

These are **analytical assumptions**, not observed supplier costs.

Complexity is represented through category-level cycle-time assumptions in the final cost calculation. It is not multiplied again on top of those cycle times, which avoids double-counting complexity.

---

# 6. Cycle-Time Assumptions

Because actual cycle times are unavailable, category-level cycle-time ranges are assigned.

| Condensed category | Low | Base | High |
|---|---:|---:|---:|
| Hardware | 2 sec | 4 sec | 6 sec |
| Plates | 3 sec | 5 sec | 8 sec |
| Body Panels | 6 sec | 10 sec | 15 sec |
| Structural Formed Components | 6 sec | 12 sec | 20 sec |

These values are **analytical assumptions**, not measured production cycle times.

Three scenarios are retained to reflect uncertainty in the absence of plant-level process data.

---

# 7. Press / Machine Cost

Press cost represents the machine-related cost of running the manufacturing process.

\[
C_{press}
=
\frac{T_{cycle}}{3600}
\times
\frac{R_{press}}{E}
\]

where:

- \(T_{cycle}\) = cycle time in seconds
- \(R_{press}\) = press/machine rate in INR per hour
- \(E\) = effective utilization
- \(3600\) = seconds per hour

For the base example:

\[
T_{cycle}=10\text{ sec},\quad
R_{press}=₹225/\text{hour},\quad
E=0.825
\]

Therefore:

\[
C_{press}
=
\frac{10}{3600}
\times
\frac{225}{0.825}
\approx ₹0.76/\text{part}
\]

### Why

The machine incurs cost while processing the part. Cycle time converts the hourly machine rate into a per-part cost, while effective utilization accounts for non-productive machine time.

---

# 8. Direct Labor Cost

Direct labor is calculated from the same production cycle time:

\[
C_{labor}
=
\frac{T_{cycle}}{3600}
\times
\frac{R_{labor}}{E}
\]

where:

- \(T_{cycle}\) = cycle time in seconds
- \(R_{labor}\) = labor rate in INR per hour
- \(E\) = effective utilization

### Why

Operator time is a variable manufacturing cost associated with producing the part. Low, base, and high scenarios are retained because the actual labor rate is not present in the dataset.

---

# 9. Tooling Cost

Tooling is treated as a fixed manufacturing investment allocated across expected production volume:

\[
C_{tooling}
=
\frac{I_{tooling}}{V_{annual}}
\]

where:

- \(I_{tooling}\) = tooling investment
- \(V_{annual}\) = annual production volume

For example:

\[
I_{tooling}=₹20,000,\quad
V_{annual}=100,000
\]

gives:

\[
C_{tooling}
=
\frac{20,000}{100,000}
=
₹0.20/\text{part}
\]

### Why

Tooling investment is distributed over expected production rather than charged entirely to one unit. This also makes annual volume relevant to the cost model.

---

# 10. Factory Overhead

Manufacturing also incurs indirect costs such as utilities, maintenance, depreciation, supervision, and facility costs.

These are represented using an overhead rate:

\[
C_{overhead}
=
(C_{press}+C_{labor})\times R_{overhead}
\]

The current scenarios use:

| Scenario | Overhead rate |
|---|---:|
| Low | 10% |
| Base | 15% |
| High | 20% |

### Why

Ignoring factory overhead would underestimate manufacturing cost because machine and direct labor do not capture the entire factory burden.

---

# 11. Secondary Operations

Some parts may require additional activities beyond the primary forming operation.

Examples identified from part descriptions include:

- Welding
- Assembly
- Riveting
- Bolting
- Coating or painting

The cost is represented as:

\[
C_{secondary}
=
\begin{cases}
R_{secondary}, & \text{if a secondary operation is required}\\
0, & \text{otherwise}
\end{cases}
\]

A description-based flag is used to identify potential secondary operations.

These rates are assumptions and should be replaced by plant-specific benchmarks if available.

---

# 12. Total Conversion Cost

The manufacturing components are combined:

\[
\boxed{
C_{conversion}
=
C_{press}
+
C_{labor}
+
C_{tooling}
+
C_{overhead}
+
C_{secondary}
}
\]

This represents the estimated cost of converting the raw material into the finished component.

---

# 13. Final Should-Cost Function

The complete should-cost function is:

\[
\boxed{
C_{should}
=
C_{material}
+
C_{press}
+
C_{labor}
+
C_{tooling}
+
C_{overhead}
+
C_{secondary}
}
\]

Substituting the component equations:

\[
\boxed{
C_{should}
=
W P_m
+
\frac{T_{cycle}}{3600}\frac{R_{press}}{E}
+
\frac{T_{cycle}}{3600}\frac{R_{labor}}{E}
+
\frac{I_{tooling}}{V_{annual}}
+
(C_{press}+C_{labor})R_{overhead}
+
C_{secondary}
}
\]

The calculation is performed independently for low, base, and high assumptions:

\[
C_{should}^{low},\qquad
C_{should}^{base},\qquad
C_{should}^{high}
\]

This produces a **should-cost range** instead of a falsely precise single estimate.

---

# 14. Comparing Supplier Price With Should Cost

The observed supplier price is:

\[
P_{supplier}=\texttt{unit\_price\_local}
\]

The absolute gap is:

\[
Gap=P_{supplier}-C_{should}
\]

The relative gap is:

\[
Gap\%
=
\frac{P_{supplier}-C_{should}}
{C_{should}}
\times100
\]

The supplier price is interpreted relative to the modeled range:

| Supplier price position | Interpretation |
|---|---|
| Below the low estimate | Below the modeled cost range |
| Within the modeled range | Broadly consistent with the model |
| Above the high estimate | Above the modeled cost range |

A price above the modeled range represents a **potential cost-reduction opportunity**, not proof that the supplier is overcharging.

---

# 15. Key Limitations

This is a **bottom-up analytical should-cost model**, not a machine-learning prediction model.

The main limitation is the absence of actual process-level manufacturing data.

Therefore:

1. Cycle times are assumed by component category.
2. Machine and labor rates are assumed.
3. Tooling investment is assumed.
4. Overhead is represented using an assumed rate.
5. Secondary-operation costs are estimated from description-based flags.
6. Supplier margin is not directly estimated.
7. The model does not claim to recover the supplier's actual internal cost.

The purpose is to create a **transparent and auditable cost benchmark** against which supplier prices can be evaluated.

---

# 16. Final Model Flow

```text
Part Description
       ↓
Category Cleaning
       ↓
Condensed Manufacturing Category
       ↓
Manufacturing Complexity
       ↓
Cycle-Time Assumption
       ↓
┌──────────────────────────────┐
│ Press Cost                   │
│ Direct Labor Cost            │
│ Tooling Amortization         │
│ Factory Overhead             │
│ Secondary Operations         │
└──────────────────────────────┘
       ↓
Conversion Cost
       ↓
       + Material Cost
       ↓
Should Cost
       ↓
Low / Base / High Range
       ↓
Compare With Supplier Price
       ↓
Potential Price Gap
```

---

# 17. Conclusion

The model estimates what a part could reasonably cost to manufacture using the available physical and commercial information.

The core principle is:

\[
\boxed{
\text{Should Cost}
=
\text{Material Cost}
+
\text{Manufacturing Conversion Cost}
}
\]

The conversion cost is decomposed into manufacturing components that are either calculated from available data or explicitly stated assumptions.

The supplier price is used only at the final comparison stage. It is **not used to derive conversion cost**, which prevents circularity in the model.
