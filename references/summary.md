# Meridian Auto Components — Spend & Should-Cost Summary

The analysis focused on cleaning the spend data, creating a consistent part classification, and building a transparent should-cost benchmark for stamped-metal parts. The three main opportunities and risks are:

- **1. Parts priced above the modeled should-cost range — potential savings opportunity**  
  The should-cost model combines material cost with estimated press, labor, tooling, overhead, and secondary-operation costs.
  - Parts where the supplier price is above the high end of the modeled cost range should be prioritized for supplier review and negotiation. These parts represent the clearest potential cost-reduction opportunities because the observed price is higher than what the available cost drivers suggest is reasonable.

- **2. Category and specification inconsistencies — spend visibility risk**  
  - Several records had missing or inconsistent categories, a category typo, misplaced category labels, and inconsistent grade-type values.
  - These issues were corrected using the part description, description-derived component bins, and material information.
  - Cleaning the data matters because inconsistent labels can hide where spend is actually concentrated and can lead to the wrong should-cost assumptions.

- **3. Conversion-cost assumptions need validation — model risk and next-step opportunity**  
  The current should-cost is a transparent benchmark because the dataset does not contain actual cycle times, machine rates, labor rates, tooling costs, or supplier conversion costs.
  - The model therefore uses low/base/high assumptions. Before using the benchmark for commercial decisions, Meridian should validate the key process assumptions with plant or supplier data. This will reduce uncertainty and make future savings estimates more reliable.

## Bottom Line

The analysis provides a practical benchmark for identifying parts that may deserve procurement attention while clearly separating **observed data** from **model assumptions**. The strongest next step is to validate the highest-value price gaps and replace the current conversion assumptions with actual manufacturing data where available.
