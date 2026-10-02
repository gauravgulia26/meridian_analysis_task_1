# Track A — Data Cleaning, Quality Issues, and Spend Classification

## 1. Objective

The cleaning and classification process was designed to satisfy two requirements:

1. **Flag data-quality issues and document how each issue was handled.**
2. **Clean and classify every purchased part into a small, sensible set of categories, with explicit reasoning for borderline cases.**

The approach therefore separates:

- **Data-quality remediation**: missing values, typos, inconsistent grade labels, and other structural issues.
- **Business classification**: determining what each part actually represents from its description and assigning it to a consistent category.

The original fields are retained wherever practical so that every correction remains traceable.

---

# 2. Data-Quality Issues Found and Treatment

| Issue found | Evidence / check | Treatment | Reason |
|---|---|---|---|
| **3 missing values in `category`** | Missing-value audit of the original category field | **Corrected / imputed** | The part descriptions provided enough semantic information to classify these parts without using a frequency-based imputation such as the mode |
| **`Farsteners` typo** | Category frequency and label inspection | **Corrected to `Fasteners`** | Deterministic spelling correction; otherwise the same business category would appear as two categories |
| **Inconsistent `gradetype` values** | Inspection of grade/material labels | **Standardized / corrected** | Equivalent grade representations should map to one consistent value before material-price calculations |
| **Category assigned inconsistently with description** | Cross-check between `category`, `description_bin`, and `part_description` | **Corrected where evidence was clear** | The original category is a historical label, so it was not treated as ground truth when it contradicted the actual part description |
| **Material type provides little category differentiation** | `type_of_material` reviewed across records | **Used as a consistency check, not a classification driver** | Material was consistently steel, so it did not distinguish the physical/component category |
| **Rare categories** | Category frequency review | **Reviewed individually; not automatically removed** | Low frequency does not by itself mean a category is incorrect |
| **Numerical fields used for costing** | Review of weight, volume, unit price and derived price-per-kg fields | **Validated before cost modeling** | Prevents unit or value inconsistencies from propagating into the should-cost calculations |

### Treatment Philosophy

The general rule was:

- **Correct** deterministic issues such as typos and inconsistent representations.
- **Impute** missing categorical values only when the part description provides sufficient evidence.
- **Reclassify** categories when the historical label conflicts with clear semantic evidence.
- **Retain and review** ambiguous cases instead of forcing unsupported classifications.
- **Flag for confirmation** when the available data cannot support a confident decision.

---

# 3. Category Classification Methodology

## 3.1 Primary source of classification

The **`part_description`** was treated as the primary source for understanding what the purchased part actually is.

Examples of useful component terms included:

- Bracket
- Mount
- Panel
- Plate
- Beam
- Nut
- Rail
- Pillar

The description often provides stronger evidence of the physical component type than the historical category field.

---

## 3.2 Description-bin construction

A `description_bin` column was created from `part_description` using regex-based extraction.

The broad bins were:

```text
Panel
Bracket
Plate
Mount
Beam
Nut
Rail
Pillar
```

The purpose of this field was **not to become the final business taxonomy automatically**. It was created as an intermediate, interpretable feature for:

- identifying the physical/component type,
- imputing the 3 missing categories,
- detecting inconsistencies between description and historical category,
- supporting manual review of borderline records.

---

# 4. Handling the 3 Missing Categories

The 3 missing category values were handled using a **description-driven imputation process**.

For each missing category:

1. Inspect the `part_description`.
2. Extract the relevant component type through `description_bin`.
3. Compare the component description with the surrounding cleaned category structure.
4. Assign the category that best represents the actual part.

This was preferred over:

```text
Missing category → most frequent category
```

because the mode would ignore the specific identity of the part.

The decision rule was:

> **Use the part's observable physical/component description rather than the overall frequency of historical labels.**

---

# 5. Detecting Incorrect Existing Categories

After constructing `description_bin`, the records were cross-examined to identify cases where:

```text
part_description
        ≠
existing category
```

Examples of the logic include:

- A description containing an explicit **Bracket** indication should not remain in an unrelated category without evidence.
- A description clearly identifying a **Panel** should not be categorized as a generic or unrelated structural type without justification.
- A description identifying a **Nut/Fastener** should be treated as hardware rather than as a broad formed-metal category.

The original category was therefore treated as **evidence**, not as unquestionable truth.

---

# 6. Borderline-Case Decision Rules

### Rule 1 — Prefer explicit component identity

If the description clearly identifies the part type, use that information first.

Example:

```text
Engine Mount Bracket
→ Bracket
```

rather than relying on an inconsistent historical label.

### Rule 2 — Use the functional/component noun rather than the application location

Words such as:

```text
Door
Hood
Seat
Floor
Engine
Battery
```

describe where the component is used.

Words such as:

```text
Bracket
Panel
Plate
Beam
Rail
Nut
```

more directly describe what the component is.

Therefore, the component type was prioritized when defining the category.

### Rule 3 — Use `type_of_material` as a validation field

The material field was reviewed together with the description.

Because the dataset was consistently identified as **steel**, material type did not provide enough information to distinguish the component categories.

Therefore:

```text
Description → primary classification evidence
Material → consistency check
```

### Rule 4 — Do not create unnecessary categories for every noun

The objective was not to create one category for every phrase appearing in a description.

For example, these can all remain in the broader structural grouping:

```text
Engine Mount Bracket
Chassis Mount Bracket
Steering Column Bracket
```

because their common component form is a bracket.

### Rule 5 — Preserve ambiguity

If a description does not contain enough evidence to assign a category confidently, it should be flagged for client confirmation rather than being assigned arbitrarily.

---

# 7. Final Cleaned Category Structure

After correcting inconsistencies, the detailed categories were consolidated into a smaller manufacturing-oriented taxonomy.

| Cleaned categories | Condensed category | Reason for grouping |
|---|---|---|
| **Brackets & Mounts** | **Structural Formed Components** | Similar structural/forming behavior and meaningful conversion effort |
| **Reinforcements** | **Structural Formed Components** | Structural components with forming/reinforcement requirements |
| **Rail** | **Structural Formed Components** | Structural formed component with similar broad manufacturing considerations |
| **Body Panels** | **Body Panels** | Kept separate because panel forming can have materially different characteristics |
| **Fasteners** | **Hardware** | Hardware/fastening component |
| **Nut** | **Hardware** | Same broad hardware family, while detailed category is retained |
| **Plate** | **Plates** | Generally simpler flat/formed sheet-metal component |

Final condensed groups:

```text
Structural Formed Components
Body Panels
Hardware
Plates
```

The detailed cleaned category is retained alongside the condensed category for traceability.

---

# 8. Why the Condensed Categories Are Used

The condensed categories are used for the **should-cost model**, not to erase useful information.

They provide a manageable set of manufacturing groups for assigning assumptions such as:

- relative conversion complexity,
- cycle-time ranges,
- manufacturing treatment.

This prevents the should-cost model from becoming overly fragmented.

---

# 9. Additional Cleaning and Validation Steps

The following checks were also performed before cost modeling:

### Missing-value audit

Reviewed missing values across the dataset rather than assuming that only category values required attention.

### Category-frequency audit

Examined category frequencies to identify:

- rare categories,
- duplicated spellings,
- inconsistent labels,
- potentially broad or suspicious classifications.

### Grade normalization

Standardized inconsistent `gradetype` representations before using grade information in material-price calculations.

### Numerical consistency

Reviewed:

- `unit_weight_kg`
- `annual_volume`
- `unit_price_local`
- `spend`
- `per_kg_price`

and the associated derived material-cost fields before using them in the should-cost model.

### Traceability

The original category information was not discarded. Cleaned and condensed fields were created separately so that changes can be audited.

---

# 10. Final Data-Cleaning Workflow

```text
Raw Dataset
    ↓
Missing-value audit
    ↓
Category-frequency and label audit
    ↓
Correct deterministic typo
    ↓
Standardize gradetype inconsistencies
    ↓
Create description_bin from part_description
    ↓
Impute 3 missing categories from description evidence
    ↓
Cross-examine existing categories against descriptions
    ↓
Correct clearly misplaced categories
    ↓
Use material type as a consistency check
    ↓
Review rare / borderline cases
    ↓
Create condensed manufacturing categories
    ↓
Validate numerical inputs for cost modeling
    ↓
Clean dataset ready for should-cost analysis
```

---

# 11. Final Outcome

The cleaning process was designed to produce a dataset in which:

- **Every part has a category.**
- Category labels are standardized.
- Obvious historical labeling errors are corrected.
- Material-grade inconsistencies are normalized.
- Category assignments are supported by the actual part description.
- Borderline classifications are handled explicitly rather than hidden behind an automated output.
- Detailed categories are retained for traceability.
- Condensed categories provide a small, sensible taxonomy for spend and should-cost analysis.
