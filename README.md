


```
# Intelligent-System-ERP-Mapping-and-Cataloging
![OMOTAYO ALEXANDER OLANUSI Technischer Bericht_ Funktionsweise meines Recordlinkage-Codes für das Projekt](https://github.com/user-attachments/assets/45648263-ffd7-4421-a104-4b4d806128b3)

## Overview
This Python script uses the **recordlinkage** library to match technical systems from a customer file with item numbers from an EP catalog.  
It generates a structured Excel output for **inventory** and **categorization**, optimizing performance through indexing and Jaro-Winkler similarity.

Compared to **fuzzywuzzy**, this approach is significantly faster for large datasets.

---

## Features
- Efficient record linkage with **SortedNeighbourhood indexing**
- High-confidence matches using **Jaro-Winkler similarity**
- Configurable similarity threshold and indexing window size
- Filters EP catalog entries for actual items only
- Generates Excel reports with:
  - **Systems** sheet (inventory-ready)
  - **Attributes** sheet (detailed feature breakdown)

---

## Installation
```bash
pip install pandas recordlinkage openpyxl
````

---

## How It Works

### 1. Load and Filter Data

**Function:** `load_data`

* Loads:

  * `Kundendatei.xlsx` (customer file)
  * `EP_Katalog.xlsx` (EP catalog)
* Keeps only rows where:

  ```python
  Art == 'N - Normalposition'
  ```

  This excludes category rows (`NG - Normalgruppe`), as pandas cannot read Excel formatting.
* Outputs:

  * `customer_df` (all systems)
  * `ep_catalog_items` (filtered items)

---

### 2. Index and Compare

**Function:** `match_systems`

* Uses **SortedNeighbourhood** indexing (window size = 5) to reduce comparisons from `O(n*m)` to far fewer.
* Compares using **Jaro-Winkler similarity** with a **0.85** threshold.
* Outputs:

  * DataFrame of candidate pairs with similarity scores.

---

### 3. Assign Item Numbers

**Function:** `assign_item_numbers`

* Selects the highest-scoring match per system.
* Assigns EP catalog `Item Number` to matched systems.
* Unmatched systems get `"Not Found"`.

---

### 4. Prepare Systems Sheet

**Function:** `prepare_systems_df`

* Creates `Systems` DataFrame with columns:

  | Output Column      | Source Column         |
  | ------------------ | --------------------- |
  | Building ID        | WirtEinh              |
  | System Type        | Anlage                |
  | System Component   | EQ\_übergeordnet      |
  | System Name        | EQ-Bezeichnung        |
  | Subsystem ID       | EQ-Klasse             |
  | Subsystem          | EQ-Klasse-Bezeichnung |
  | AKS Name           | Gewerk                |
  | Quantity           | EQ-Menge              |
  | Association Number | Item Number           |

---

### 5. Prepare Attributes Sheet

**Function:** `prepare_attributes_df`

* Parses columns `EQ-Merkmal_001` to `EQ-Merkmal_052`:

  * Splits at `":"` into attribute and value (e.g., `"Cooling Capacity:109 kW"`).
* Output columns:

  * Building ID, System Type, System ID, System, Subsystem, AKS Name, Attribute ID, Attribute, Value, Unit (blank), Price Relevant (`TRUE`).

---

### 6. Generate Excel Report

**Function:** `generate_excel_report`

* Writes:

  * **Systems** sheet
  * **Attributes** sheet
* File: `System_Matching_Results.xlsx` (via **openpyxl**).

---

### 7. Main Execution

**Function:** `main`

* Orchestrates:

  * Loading
  * Matching
  * Assigning item numbers
  * Preparing sheets
  * Exporting Excel file
* Configurable:

  * Threshold (default: `0.85`)
  * Window size (default: `5`)

---

## Why Not Fuzzywuzzy?

* **fuzzywuzzy** compares *every* system with *every* catalog entry → **O(n\*m)** complexity.
* Example:

  * 1,000 systems × 10,000 items = **10 million** comparisons.
  * recordlinkage indexing reduces this to **thousands**.
* Jaro-Winkler similarity preserves matching accuracy while saving time.

---

## Technical Specs

* **Libraries:** `pandas`, `recordlinkage`, `openpyxl`, `logging`
* **Assumptions:**

  * Required columns exist
  * Attributes follow `"Attribute: Value"` format
* **Parameters:**

  * `threshold`: 0.85 (adjustable)
  * `window_size`: 5 (adjustable)

---

## Example Usage

```python
if __name__ == "__main__":
    main(
        customer_file="Kundendatei.xlsx",
        ep_catalog_file="EP_Katalog.xlsx",
        output_file="System_Matching_Results.xlsx",
        threshold=0.85,
        window_size=5
    )
```

---

## License

This project is licensed under the MIT License.

```

---

I can also add **flow diagrams** showing the matching pipeline and **performance comparison graphs** to make your GitHub README visually appealing and easier for visitors to understand. That would make it look like a polished, professional open-source project.  

Do you want me to add those diagrams next?
```
