# Technical Assessment
## Objectives
* Assess your ability to solve practical problems through end-to-end pipeline development.
* Evaluate proficiency in Python by designing a modular, readable, and testable application.
* Test your understanding of workflow orchestration using **Prefect** to automate and schedule tasks.
* Evaluate your ability to design and interact with a **relational database** using **SQL**, including schema design and query optimization.
* Assess your experience in consuming and integrating **external APIs** into a data pipeline.
* Test your ability to transform raw product data using business logic and store it in a structured and queryable format.
## Submission Guidelines
1. **Fork** this repository to your own GitHub account.
2. Create a new **feature branch** off `main` in your fork. Use a descriptive name, e.g., `feature/solution-yourname`.
3. Implement your solution and **commit** your changes to your branch.
4. **Push** your branch to your forked repository.
5. Open a **Pull Request (PR)** from your feature branch to the `main` branch of this original repository.
6. **Do not merge** the PR yourself. We will review your submission and provide feedback directly on your PR.
## Folder Structure
```
ekz_technical_assessment/
├── api/
│   ├── __main__.py
│   └── README.md  # Setup & information about the provided local API
├── your_pipeline/
│   └── ...        # Your solution goes here. Build and deploy your pipeline here.
├── README.md      # Technical assessment instructions and guidelines
```
---

# Product Repricing Pipeline
## Problem
Our store is expanding its catalog. We want a **pipeline** that:

* Takes products from an **API endpoint**.
* Applies repricing based on **business logic**.
* Stores and updates product data in a **SQL database** (SQLite).
* Runs as a scheduled **Prefect flow**.

## Business Logic
1. **Default Target Profit Margin**
   * All products should aim for a **12% profit margin**, unless overridden by brand, vendor or category margin rules.
2. **Pricing Formula**
   * Price is calculated using:
     `price = total cost / (1 - target_margin)`
   * Total cost is calculated as the sum of product cost, shipping cost, and any applicable extra cost.
3. **Rounding Rule**
   * After calculating the raw price:
     * Adjust the whole number to the nearest lower odd number. E.g., 200 becomes 199.
     * If the decimal is greater than or equal `.50`, set the decimal to `.95` (e.g., 199 becomes 199.95).
     * If the decimal is less than `.50` or the number has no decimal, set the decimal to `.45` (e.g., 199 becomes 199.45).
4. **Target Margin & Extra Cost Rules**
   * If a **vendor**, **brand**, or **category** specifies a target margin, use that margin instead of the **default target margin**.
   * The **vendor's extra cost adjustment always applies**, but may be modified or waived in vendor category rules.
   * The **target margin** is determined as follows:
     * Use the **vendor category rules** when both the product's vendor and category match.
     * Otherwise, use the **default category rules** when the product's category applies.
     * If neither applies, use the **default vendor rules**.
   * If the shipping tier is **Null**, use the **brand rules** regardless of the rule condition.
5. **Shipping Tier Rule**:
   * Use the defined shipping cost unless the shipping tier is null, in which case assume $0.

### Default Vendor Rules
| Vendor               | Extra Cost | Target Margin |
| -------------------- | ---------- | ------------- |
| Evergreen Merchants  | +\$15      | 16%           |
| Titan Labs           | +\$20      | 20%           |
| Prime Distributors   | +\$10      | 15%           |
| Pacific Supply Group | +\$0       | 16%           |
| Brightside Trading   | +\$10      | 15%           |
| Global Supplies Ltd  | +\$0       | 14%           |

### Category Rules
#### Default Category Rules
| Category                | Target Margin |
| ----------------------- | ------------- |
| Automotive & Industrial | 30%           |
| Electronics             | 35%           |
| Toys & Kids             | 25%           |
| Beauty & Health         | 20%           |
| Furniture & Home        | 25%           |

#### Vendor Category Rules
| Vendor               | Category                | Target Margin | Extra Cost Adjustment      |
| -------------------- | ----------------------- | ------------- | -------------------------- |
| Evergreen Merchants  | Automotive & Industrial | 35%           | Extra cost **waived**      |
| Titan Labs           | Electronics             | 40%           | Extra cost reduced to +\$10|
| Prime Distributors   | Toys & Kids             | 30%           | Extra cost +\$10           |
| Pacific Supply Group | Furniture & Home        | 35%           | Extra cost +\$20           |
| Brightside Trading   | Beauty & Health         | 25%           | Extra cost **waived**      |

*Note: Vendor category rules overrides default category rules. If there's no vendor category rules, use the default category rules.*

#### Example
*Titan Labs:*
* **Product A** category is **Electronics**, apply the vendor extra cost of **+\$10** (reduced from +\$20) and the target margin of **40%** (vendor category rules override instead of the default category rules 35%).
* **Product B** category is **Toys & Kids** (no category override), apply the full vendor extra cost of **+\$20** and the default category rules of **25%**.

*Global Supplies Ltd:*
* Use the **default category rules** since this vendor doesn't have vendor category rules.

### Brand Rules
| Brand        | Target Margin |
| ------------ | ------------- |
| StoneBridge  | 60%           |
| BrightLeaf   | 60%           |
| Night Owl    | 50%           |
| SilverFox    | 45%           |
| Blue Horizon | 40%           |
| StormForge   | 35%           |
| RapidStream  | 50%           |

*Note: If the shipping tier is Null, brand rules overrides vendor and category rules.*

### Target Margin Rules Hierarchy
1. **Brand Target Margin Rules**  
   - Applies **only when `shipping_tier_cost` is Null**.  
   - Overrides all vendor and category target margins.
2. **Vendor Category Rules**  
   - Applies when a vendor has a target margin rule specific to the product's category.  
   - Includes custom extra cost adjustments (waived, reduced, or added).
3. **Default Category Rules**  
   - Applies if no vendor category rules exists for the product's category.  
   - Use the **default vendor's extra cost**.
4. **Default Vendor Rules** *(Fallback)*  
   - Applies only when the product's category is **not listed** in any margin rules.  
   - Use the **vendor's default rules and extra cost**.

## Task Breakdown
**You will build a mini-product pricing pipeline with these components:**

1. **API Consumer**
    * Fetch product data for each vendor using the local API, located in the `api/` directory.
    * For setup instructions and endpoint details, refer to the `README.md` file inside that directory.
2. **Build & Design a Database**
    * Design and create a database schema optimized for storing vendor product data.
3. **Pricing**
    * Apply the business logic to calculate product prices based on vendor, category, and brand rules.
4. **Prefect Flow**
    * Wrap the whole process into a Prefect flow:
      * Extract (API) -> Transform (Pricing logic) -> Load (DB)
5. **Exposed API (Optional/Bonus)**
    * Create a simple FastAPI service that allows querying repriced products with filters for brand, category, or vendor.

## Testing
Writing **unit tests** using `pytest` is a **mandatory** part of this technical assessment. Testing will be graded as part of your submission based on **coverage, quality, and clarity**.

The following points are provided as **examples**. You are expected to determine what to test and how to structure your tests, just as you would in a real-world scenario.

**Examples of aspects you may consider testing:**

* Correctness of **pricing calculations**.
* Business logic prioritization.
* Handling of vendor adjustments (waived/reduced/added extra cost/target margin).
* Verification of **database writes**.
* (Optional) **API endpoint tests**, if you implement the FastAPI bonus task:
  * Filtering by brand/category/vendor
  * Response structure and status codes
