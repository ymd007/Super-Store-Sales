# Complete Column Dictionary — After Feature Engineering
## What Each Column Means & How To Use It

---

## 📊 ORIGINAL COLUMNS (From Raw CSV)

### **Identifiers (SHOULD BE DROPPED BEFORE ML)**

| Column | Type | Values | What It Is | Use for ML? |
|--------|------|--------|-----------|-----------|
| **Row ID** | Integer | 1–9,800 (unique) | Sequential row counter | ✗ NO — every value unique |
| **Order ID** | String | CA-2018-104066 (unique) | Unique order identifier | ✗ NO — every value unique |
| **Customer ID** | String | CG-12520 (793 unique) | Unique customer identifier | ✗ NO — identifier, not feature |
| **Product ID** | String | FUR-BO-10001557 (1,861 unique) | Unique product SKU | ✗ NO — identifier, not feature |

---

### **Names (SHOULD BE DROPPED BEFORE ML)**

| Column | Type | Values | What It Is | Use for ML? |
|--------|------|--------|-----------|-----------|
| **Customer Name** | String | "Raymond White" (793 unique) | Full customer name | ✗ NO — free text, too many unique values |
| **Product Name** | String | "Staples" (1,849 unique) | Full product description | ✗ NO — free text, 1,849 unique values |

---

### **Dates (EXTRACTED → New Features, Drop Originals)**

| Column | Type | Format | Range | What It Is | Use for ML? |
|--------|------|--------|-------|-----------|-----------|
| **Order Date** | DateTime | DD/MM/YYYY → datetime64 | 2015-01-03 to 2018-12-30 | Date order was placed | ✗ KEEP EXTRACTED, DROP ORIGINAL |
| **Ship Date** | DateTime | DD/MM/YYYY → datetime64 | 2015-01-06 to 2019-01-02 | Date order was shipped | ✗ KEEP EXTRACTED, DROP ORIGINAL |

---

### **Geographic (Mix of Keep/Drop)**

| Column | Type | Values | What It Is | Use for ML? |
|--------|------|--------|-----------|-----------|
| **Country** | String | "United States" (1 only!) | Customer country | ✗ NO — only 1 value, no variance |
| **State** | String | "California" (49 unique) | US state abbreviation | ✓ YES — encode it |
| **City** | String | "San Francisco" (529 unique) | City name | ⚠️ MAYBE — too granular (529 values) |
| **Postal Code** | Numeric | 5401 (626 unique) | ZIP code | ⚠️ MAYBE — too granular (626 values) |

---

### **Main Numeric Features (TARGET & FEATURE)**

| Column | Type | Range | Mean | Median | What It Is | Use for ML? |
|--------|------|-------|------|--------|-----------|-----------|
| **Sales** | Float | $0.44 – $22,638.48 | $230.77 | $54.49 | **YOUR TARGET VARIABLE** | ✓ YES (target) |

---

## 🔧 NEW COLUMNS (Created During Feature Engineering)

### **log-Transformed Target**

| Column | Type | Range | Mean | Median | What It Is | Use for ML? |
|--------|------|-------|------|--------|-----------|-----------|
| **log_Sales** | Float | 0.3674 – 10.0275 | 4.1575 | 4.0162 | log(1 + Sales) | ✓ YES (better target) |

---

### **Date Features Extracted from Order Date**

| Column | Type | Values | What It Represents | Example | Use for ML? |
|--------|------|--------|-------------------|---------|-----------|
| **order_month** | Integer | 1–12 | Month of order placement | 1=January, 12=December | ✓ YES — seasonal patterns |
| **order_year** | Integer | 2015, 2016, 2017, 2018 | Year of order placement | 2015=2015, 2018=2018 | ✓ YES — growth trends |
| **order_quarter** | Integer | 1, 2, 3, 4 | Business quarter | 1=Q1 (Jan–Mar), 4=Q4 (Oct–Dec) | ✓ YES — quarterly cycles |
| **order_dayofweek** | Integer | 0–6 | Day of week when ordered | 0=Monday, 6=Sunday | ✓ YES — weekday patterns |
| **is_weekend** | Integer | 0, 1 | Was order placed on weekend? | 0=Weekday, 1=Weekend | ✓ YES — binary feature |
| **ship_days** | Integer | 0–7 | Days from order to shipment | 3=3 days to ship, 4=4 days, etc. | ✓ YES — logistics indicator |

---

### **Label Encoded Columns (Integer, 0 to N)**

**What is label encoding?**
Converts each category name to a number. Useful for tree-based models.

⚠️ **WARNING:** Don't use label-encoded columns in linear models! Use one-hot encoding instead.

| Column | Type | Values | Mapping | What It Is | Use for ML? |
|--------|------|--------|---------|-----------|-----------|
| **segment_encoded** | Integer | 0, 1, 2 | 0=Consumer, 1=Corporate, 2=Home Office | Customer segment as number | ✓ YES (for trees) |
| **region_encoded** | Integer | 0, 1, 2, 3 | 0=Central, 1=East, 2=South, 3=West | US region as number | ✓ YES (for trees) |
| **category_encoded** | Integer | 0, 1, 2 | 0=Furniture, 1=Office Supplies, 2=Technology | Product category as number | ✓ YES (for trees) |
| **sub_category_encoded** | Integer | 0–16 | 0=Accessories, ..., 16=Tables | Product sub-category as number (17 values) | ⚠️ YES but high cardinality |
| **ship_mode_encoded** | Integer | 0, 1, 2, 3 | 0=First Class, 1=Same Day, 2=Second Class, 3=Standard Class | Shipping method as number | ✓ YES (for trees) |

---

### **One-Hot Encoded Columns (Binary, 0 or 1)**

**What is one-hot encoding?**
Creates a separate binary column for each category value.
- 0 = this category value is NOT present for this row
- 1 = this category value IS present for this row

⚠️ **WARNING:** Don't use ALL one-hot columns in linear models! Drop one to avoid multicollinearity.

#### **Segment (3 columns)**

| Column | Type | Values | What It Is | Example |
|--------|------|--------|-----------|---------|
| **Segment_Consumer** | Integer | 0, 1 | Is this a Consumer order? | 1 = Consumer, 0 = Not Consumer |
| **Segment_Corporate** | Integer | 0, 1 | Is this a Corporate order? | 1 = Corporate, 0 = Not Corporate |
| **Segment_Home Office** | Integer | 0, 1 | Is this a Home Office order? | 1 = Home Office, 0 = Not Home Office |

**Key:** Exactly ONE of these three is 1 for each row. The other two are 0.

#### **Region (4 columns)**

| Column | Type | Values | What It Is | Example |
|--------|------|--------|-----------|---------|
| **Region_Central** | Integer | 0, 1 | Is customer in Central region? | 1 = Central, 0 = Not Central |
| **Region_East** | Integer | 0, 1 | Is customer in East region? | 1 = East, 0 = Not East |
| **Region_South** | Integer | 0, 1 | Is customer in South region? | 1 = South, 0 = Not South |
| **Region_West** | Integer | 0, 1 | Is customer in West region? | 1 = West, 0 = Not West |

**Key:** Exactly ONE of these four is 1 for each row.

#### **Category (3 columns)**

| Column | Type | Values | What It Is | Example |
|--------|------|--------|-----------|---------|
| **Category_Furniture** | Integer | 0, 1 | Is product a Furniture item? | 1 = Furniture, 0 = Not Furniture |
| **Category_Office Supplies** | Integer | 0, 1 | Is product Office Supplies? | 1 = Office Supplies, 0 = Not Office Supplies |
| **Category_Technology** | Integer | 0, 1 | Is product Technology? | 1 = Technology, 0 = Not Technology |

**Key:** Exactly ONE of these three is 1 for each row.

#### **Ship Mode (4 columns)**

| Column | Type | Values | What It Is | Example |
|--------|------|--------|-----------|---------|
| **Ship Mode_First Class** | Integer | 0, 1 | Was this shipped First Class? | 1 = First Class, 0 = Not First Class |
| **Ship Mode_Same Day** | Integer | 0, 1 | Was this shipped Same Day? | 1 = Same Day, 0 = Not Same Day |
| **Ship Mode_Second Class** | Integer | 0, 1 | Was this shipped Second Class? | 1 = Second Class, 0 = Not Second Class |
| **Ship Mode_Standard Class** | Integer | 0, 1 | Was this shipped Standard Class? | 1 = Standard Class, 0 = Not Standard Class |

**Key:** Exactly ONE of these four is 1 for each row.

---

## 📋 WHICH COLUMNS TO KEEP FOR ML?

### **✓ KEEP THESE (Ready for Models)**

**Numeric Features:**
- `order_month`, `order_year`, `order_quarter`
- `order_dayofweek`, `is_weekend`
- `ship_days`
- `segment_encoded`, `region_encoded`, `category_encoded`, `ship_mode_encoded`
- `sub_category_encoded` (but watch cardinality)
- All one-hot encoded columns (Segment_*, Region_*, Category_*, Ship Mode_*)

**Target Variable:**
- `log_Sales` (for ML) or `Sales` (for reference)

---

### **✗ DROP THESE (Before Training)**

| Column | Reason |
|--------|--------|
| Row ID | Unique identifier — no predictive value |
| Order ID | Unique identifier — no predictive value |
| Customer ID | Unique identifier — no predictive value |
| Product ID | Unique identifier — no predictive value |
| Customer Name | Free text, 793 unique values |
| Product Name | Free text, 1,849 unique values |
| Country | Only 1 value ("United States") |
| Order Date | Already extracted (month, year, etc.) |
| Ship Date | Already extracted (ship_days) |

**Questionable:**
- `City` — 529 unique values (too granular for ML)
- `State` — 49 unique values (consider encoding or dropping)
- `Postal Code` — 626 unique values (too granular)

---

## 🎯 QUICK REFERENCE: What to Use for Each Model Type

### **For Tree-Based Models (Random Forest, LightGBM, XGBoost)**
✓ Use: `segment_encoded`, `region_encoded`, `category_encoded`, `ship_mode_encoded`, `sub_category_encoded`
✗ Skip: One-hot encoded columns (unnecessary, slower)
✓ Use: All date features
✓ Use: `ship_days`

### **For Linear Models (Linear Regression, Logistic Regression)**
✓ Use: One-hot encoded columns (Segment_*, Region_*, Category_*, Ship Mode_*)
✗ Skip: Label-encoded columns (implies order)
⚠️ Drop ONE category per feature to avoid multicollinearity (e.g., drop Segment_Consumer)
✓ Use: All date features
✓ Use: `ship_days`, `is_weekend`

### **For Deep Learning (Neural Networks)**
✓ Use: Normalized numeric features
✓ Use: One-hot encoded columns
✗ Skip: Label-encoded columns
✓ Use: All date features

---

## 📊 EXAMPLE ROW

Let's say you have this row in the original data:

```
Row ID       : 1
Order ID     : CA-2015-001234
Order Date   : 01/01/2015
Ship Date    : 01/04/2015
Customer ID  : AB-12345
Segment      : Consumer
Region       : West
Category     : Technology
Ship Mode    : Standard Class
Sales        : $150.00
```

After feature engineering, that row becomes:

```
Original columns (keep these):
  Sales                          : 150.00
  Ship Mode                      : "Standard Class"
  Segment                        : "Consumer"
  Region                         : "West"
  Category                       : "Technology"

New date features (extracted):
  order_month                    : 1
  order_year                     : 2015
  order_quarter                  : 1
  order_dayofweek                : 2 (Wednesday)
  is_weekend                     : 0
  ship_days                      : 3

Log-transformed target:
  log_Sales                      : log(1 + 150) = 5.0173

Label-encoded (for trees):
  segment_encoded                : 0 (Consumer=0)
  region_encoded                 : 3 (West=3)
  category_encoded               : 2 (Technology=2)
  ship_mode_encoded              : 3 (Standard Class=3)

One-hot encoded (for linear):
  Segment_Consumer               : 1
  Segment_Corporate              : 0
  Segment_Home Office            : 0
  Region_Central                 : 0
  Region_East                    : 0
  Region_South                   : 0
  Region_West                    : 1
  Category_Furniture             : 0
  Category_Office Supplies       : 0
  Category_Technology            : 1
  Ship Mode_First Class          : 0
  Ship Mode_Same Day             : 0
  Ship Mode_Second Class         : 0
  Ship Mode_Standard Class       : 1
```

---

## 📝 SUMMARY TABLE

| Type | Count | Purpose | Use in ML? |
|------|-------|---------|-----------|
| **Numeric Features** | 6 | Temporal patterns | ✓ Always |
| **Label-Encoded** | 5 | Tree models | ✓ For trees |
| **One-Hot Encoded** | 14 | Linear models | ✓ For linear |
| **Target Variables** | 2 (Sales + log_Sales) | Prediction target | ✓ Always |
| **To Drop** | 11 | Identifiers, free text | ✗ Never use |

**Total columns after engineering: 38+**
**Ready for ML: ~30 columns**
**To drop before training: ~8-10 columns**

---

## 🚀 NEXT STEP

Ready to use these for ML? Keep:
- All numeric features (6)
- All one-hot columns if using linear models (14)
- OR label-encoded columns if using tree models (5)
- Target: `log_Sales`

Drop everything else before training!
