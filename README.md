# Superstore EDA + ML Notebook — A World-Class Blueprint

> **Dataset**: Rohit Sahoo's `sales-forecasting/train.csv` — 9,800 rows, 18 columns (Order Date, Ship Date, Ship Mode, Customer ID/Name, Segment, Country, City, State, Postal Code, Region, Product ID, Category, Sub-Category, Product Name, Sales)
> **Goal**: Beginner-friendly comprehensive `.ipynb` covering full EDA → Regression → LSTM Forecasting

---

## ⚠️ Dataset Realities — Design Around These First

The user's file is **not** the common 9,994-row `Sample - Superstore.csv`. Three critical consequences:

- **No Profit or Discount columns.** Findings about margin or loss-making sub-categories require the extended version. Present those as optional sidebars.
- **Dates are `DD/MM/YYYY`** (European format). `pd.to_datetime('08/11/2017')` silently parses as August 11 instead of November 8. **Always pass `format='%d/%m/%Y'`**.
- **Only 11 null Postal Codes** — all Burlington, VT. No other nulls. Zero exact duplicates.

### ✅ Defensive Load Cell

```python
df = pd.read_csv('train.csv', encoding='latin-1')       # handles special chars
df.drop(columns='Row ID', inplace=True, errors='ignore')

for col in ['Order Date', 'Ship Date']:
    df[col] = pd.to_datetime(df[col], format='%d/%m/%Y')  # explicit, zero ambiguity

df.loc[df['Postal Code'].isna(), 'Postal Code'] = 5401    # Burlington VT primary ZIP
df['Postal Code'] = df['Postal Code'].astype('Int64').astype(str).str.zfill(5)
df.drop_duplicates(inplace=True)

assert (df['Ship Date'] >= df['Order Date']).all(), "Ship before order — bad parse!"
assert df['Country'].nunique() == 1                       # US-only; drop or ignore
```

> 💡 **One row = one order line, not one order.** Use `df['Order ID'].nunique()` for order counts, not `len(df)`.

---

## 📋 Known Insights to Validate in EDA

Use EDA as **hypothesis-confirmation**, not open-ended exploration. These findings are consistent across ~15 published analyses:

| Dimension | Established Finding |
|-----------|---------------------|
| Total Sales | ~$2.26M across ~5,000 orders, growing ~20% YoY 2015→2018 |
| Highest-Sales Category | **Technology (~37–38%)**, then Furniture (~32%), Office Supplies (~30%) |
| Highest-Order-Count Category | **Office Supplies (~60% of lines)** — cheap binders/paper dominate volume |
| Top Sub-Categories | **Phones, Chairs, Storage, Tables, Binders** |
| Regional Ranking | **West > East > Central > South** |
| Top States | **California (~$457K), New York (~$310K), Texas (~$170K)** |
| Bottom States | North Dakota, West Virginia, Maine, South Dakota, Wyoming |
| Segment Split | **Consumer ~51%, Corporate ~30%, Home Office ~19%** |
| Ship Mode Split | **Standard Class ~59%, Second Class ~20%, First Class ~15%, Same Day ~5%** |
| Delivery Time | Same Day ≈ 0d, First ≈ 2d, Second ≈ 3d, Standard ≈ 5d |
| Seasonality | **Strong Q4 peak, max in November** + secondary September peak |
| Weekday | Mon–Fri clustering (B2B/e-comm, not POS) |
| Pareto Customers | Top ~20% customers drive ~60–80% of revenue |

---

## 📁 Recommended 13-Section Notebook Structure

Keep **regression (per-order) and LSTM (aggregate daily) as two separate sequential stories**. Do not interleave — granularity, split strategy, and feature engineering all differ. Put Keras imports inside Section 11 only.

```
0  Title, abstract, business questions, what this notebook produces
1  Setup — imports, sns.set_theme, random seeds, dataset citation
2  Data loading — read_csv with latin-1, schema, column dictionary
3  Data cleaning — dates (DD/MM/YYYY!), duplicates, Burlington VT nulls, dtypes
4  Univariate analysis — Sales histogram (raw + log1p), Segment/Region/Category counts
5  Bivariate analysis — Sales by Category, Region, choropleth, correlation heatmap
6  Multivariate — treemap (Category → Sub-Category), sunburst, Region × Category heatmap
7  Time-series EDA — monthly/weekly resample, rolling mean, Month × Year heatmap,
     seasonal_decompose, YoY growth, weekday effect, Pareto of Sub-Categories
8  Feature engineering — date parts, ship_days, cyclic month, encoding plan per cardinality
9  Regression — Linear → Random Forest → LightGBM/XGBoost → CatBoost on log1p(Sales)
10 Bridge to forecasting — aggregate to daily, stationarity check, chronological split,
     scale on TRAIN ONLY
11 LSTM — tiny Keras 3 model, EarlyStopping, inverse-transform, compare vs naive/SARIMA/Prophet
12 Conclusions — findings, model takeaways, honest limitations
13 References
```

**Best practices per section:**
- Open every section with a markdown cell stating the question and expected takeaway
- Target **one markdown cell per 2–4 code cells**
- Use `<a id="sec7"></a>` HTML anchors for a clickable Table of Contents
- Wrap verbose diagnostics in `<details><summary>Click to expand</summary>…</details>`

---

## 📊 Visualization Strategy

Use **three libraries on purpose, not by accident**:
- **Seaborn** — statistical plots (classic API only, not objects interface)
- **Plotly Express** — geographic, hierarchical, and interactive plots
- **Matplotlib** — dual-axis Pareto charts and fine subplot control

### Global Theme (set once in Section 1)

```python
sns.set_theme(style='whitegrid', context='notebook', palette='colorblind')
plt.rcParams['figure.figsize'] = (10, 6)
```

### The Essential 10 Charts

| # | Chart | Library | Why It Earns Its Place |
|---|-------|---------|------------------------|
| 1 | Sales histogram (raw + `log1p`) | Matplotlib | Motivates the log transform visually |
| 2 | Box plot of Sales by Category (log scale) | Seaborn | Skew + category differences in one figure |
| 3 | Pareto chart of Sub-Categories | Matplotlib `twinx()` | Canonical 80/20 with cumulative line |
| 4 | US State Choropleth | Plotly Express | Most visually memorable chart in the notebook |
| 5 | Treemap Category → Sub-Category | Plotly Express | Hierarchical share at a glance |
| 6 | Month × Year Heatmap | Seaborn | Seasonality patterns instantly visible |
| 7 | `seasonal_decompose` (4-panel) | Statsmodels | Trend + season + residual decomposition |
| 8 | Correlation Heatmap | Seaborn | Feature relationships before modeling |
| 9 | Predicted-vs-Actual + Residual Plot | Matplotlib | Regression model diagnostic |
| 10 | Loss Curve + Actual-vs-Forecast Overlay | Matplotlib | LSTM training and evaluation |

### Color Palettes
- `viridis` / `rocket` — sequential data
- `vlag` / `coolwarm` — diverging (residuals, profits)
- `colorblind` — categorical default (2025 accessibility standard)

---

## 🤖 ML Paradigms Explained for Beginners

### Supervised Regression (per-order task)
Supervised learning is like studying flashcards where every card has the right answer on the back. You show the model many (features → answer) pairs, and it learns a function that produces answers for new cards. When the answer is a number — a dollar amount — it's called **regression**. Every row here is a flashcard: Ship Mode, Category, Region, shipping time, date on the front; the recorded Sales amount on the back.

### Supervised Time-Series Forecasting
Forecasting is still supervised learning — we just rearrange the flashcards so that the past is the front and the future is the back. For daily sales, we slide a 28-day window: features = last 28 days, label = day 29. **Order matters** — we can't shuffle or we'd let the model peek at the future (data leakage). That's why forecasting uses `TimeSeriesSplit`, not regular K-Fold, and why we fit the scaler only on training history.

### Unsupervised Clustering (RFM extension)
Unsupervised learning is what you do when the flashcards have blank backs — no labels. The algorithm discovers structure itself, grouping similar examples. The classic retail application is **RFM segmentation**: compute each customer's Recency, Frequency, and Monetary value, then feed those three numbers to K-Means. The algorithm discovers natural groups — "loyal high-spenders," "at-risk lapsers," "one-time buyers" — that no human labeled in advance.

### Other Paradigms (brief)
Semi-supervised doesn't apply here (Sales is recorded for every row). Reinforcement learning is relevant for discount policy setting, not historical prediction. Self-supervised / deep learning **rarely beats gradient boosting on 10k tabular rows** — which is why Kaggle winners through 2026 still reach for XGBoost, LightGBM, and CatBoost first.

---

## 🔬 Section 9 — Regression: Four Models in Ascending Sophistication

| Order | Model | Teaching Moment |
|-------|-------|-----------------|
| 1 | **Linear Regression** | Baseline. Shows why transforms matter: bad on raw Sales, respectable after `log1p` + target encoding |
| 2 | **Random Forest** | Bagging paradigm, handles non-linearity, native feature importance in one line |
| 3 | **LightGBM** (or XGBoost) | Modern boosting standard, 10× faster than RF, introduces early stopping + Optuna tuning |
| 4 | **CatBoost** ⭐ | **Featured model.** Pass `cat_features=[...]` directly — no one-hot needed. Wins on this categorical-heavy dataset |

Optional 5th: a **Keras MLP** that scores *worse* — teaches that deep learning doesn't auto-win on 10k tabular rows.

### Feature Engineering Recipe

```python
df['order_year']       = df['Order Date'].dt.year
df['order_month']      = df['Order Date'].dt.month
df['order_dayofweek']  = df['Order Date'].dt.dayofweek
df['order_quarter']    = df['Order Date'].dt.quarter
df['is_weekend']       = (df['order_dayofweek'] >= 5).astype(int)
df['is_month_end']     = df['Order Date'].dt.is_month_end.astype(int)
df['month_sin']        = np.sin(2 * np.pi * df['order_month'] / 12)   # cyclic encoding
df['month_cos']        = np.cos(2 * np.pi * df['order_month'] / 12)
df['ship_days']        = (df['Ship Date'] - df['Order Date']).dt.days
```

### Encoding Strategy by Cardinality

| Column | Unique Values | Strategy |
|--------|--------------|----------|
| Ship Mode | 4 | One-hot |
| Segment | 3 | One-hot |
| Region | 4 | One-hot |
| Category | 3 | One-hot |
| Sub-Category | 17 | One-hot or target encoding |
| State | 49 | Target encoding (sklearn 1.3+ `TargetEncoder`) |
| City | ~531 | Target encoding or pass to CatBoost as categorical |
| Product Name | ~1,850 | **Never one-hot** — pass to CatBoost or use target encoding |
| Country | 1 | Drop (zero variance) |

### Log-Transform the Target

```python
from sklearn.compose import TransformedTargetRegressor

model = TransformedTargetRegressor(
    regressor=RandomForestRegressor(n_estimators=500, random_state=42),
    func=np.log1p, inverse_func=np.expm1
)
# Trains on log1p(y), predicts in dollars automatically
```

> ⚠️ **Always report metrics on the original dollar scale**, not log scale.

### Metrics to Report: RMSE, MAE, R²
Skip MAPE — it explodes when Sales ≈ 0.

### Cross-Validation Contrast

```python
# Per-order regression — rows are iid, shuffling is fine
from sklearn.model_selection import KFold
cv = KFold(n_splits=5, shuffle=True, random_state=42)

# Time-series forecasting — NEVER shuffle
from sklearn.model_selection import TimeSeriesSplit
tscv = TimeSeriesSplit(n_splits=5)
```

> This contrast is one of the notebook's best teaching moments.

### Tuning: Optuna (2025 Standard)

```python
import optuna

def objective(trial):
    params = {
        'n_estimators': trial.suggest_int('n_estimators', 100, 1000),
        'learning_rate': trial.suggest_float('learning_rate', 1e-3, 0.3, log=True),
        'num_leaves': trial.suggest_int('num_leaves', 20, 300),
        'min_child_samples': trial.suggest_int('min_child_samples', 5, 100),
    }
    model = lgb.LGBMRegressor(**params)
    score = cross_val_score(model, X_tr, y_tr, cv=5, scoring='neg_root_mean_squared_error')
    return -score.mean()

study = optuna.create_study(direction='minimize')
study.optimize(objective, n_trials=50)
```

### Model Interpretation (3 Layers)

1. **Built-in feature importance** — quick first look (`model.feature_importances_`)
2. **Permutation importance** — `sklearn.inspection.permutation_importance` (more honest)
3. **SHAP** — `shap.plots.beeswarm()` and `shap.plots.waterfall()` for global + per-prediction explanations

---

## 📈 Section 11 — LSTM Forecasting: Be Honest About What Wins on Small Data

> 🎯 **Key research finding (arXiv 2601.00525, 2025)**: On ~1,460 daily retail points, a **64-unit 1-layer LSTM beats a 128-unit model by 47%** while being 73% smaller. Bigger is worse on small data. Also: SARIMA/Prophet often beats LSTM at this scale — **embrace that finding, don't hide it**.

### Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Granularity | Daily (~1,460 pts) | Monthly (~48 pts) is too small for any neural net |
| Keras version | Keras 3.x | `import keras` directly; backend-agnostic; save as `.keras` not `.h5` |
| LSTM size | 1 layer, 64 units, dropout 0.2 | Smaller beats larger on retail-scale data |
| Loss function | `Huber(delta=1.0)` | Robust to sales spikes, differentiable |
| Hardware | CPU | No GPU benefit on 1,460 points |

### The 5 Fatal Pitfalls (explain each explicitly in a markdown callout)

1. **Fitting scaler on all data before splitting** → leaks test statistics into training
2. **Shuffling during time-series split** → destroys temporal order
3. **Creating windows before splitting** → windows crossing the boundary leak future into training
4. **Over-parameterizing** → 128+ units and 3 layers on 1,460 points = memorization
5. **Skipping baselines** → LSTM RMSE without seasonal-naive is meaningless

### Reference Implementation

```python
import keras
from keras import layers, callbacks
from sklearn.preprocessing import MinMaxScaler

# --- Aggregate to daily ---
daily = df.set_index('Order Date')['Sales'].resample('D').sum().fillna(0)

# --- Chronological split (no shuffling!) ---
n = len(daily)
train_end, val_end = int(n * 0.80), int(n * 0.90)
train = daily.iloc[:train_end]
val   = daily.iloc[train_end:val_end]
test  = daily.iloc[val_end:]

# --- Scale on TRAIN ONLY ---
scaler = MinMaxScaler()
train_s = scaler.fit_transform(train.values.reshape(-1, 1))
val_s   = scaler.transform(val.values.reshape(-1, 1))
test_s  = scaler.transform(test.values.reshape(-1, 1))

# --- Create sliding windows ---
WINDOW = 28

def make_windows(s, w):
    X = np.array([s[i:i+w]   for i in range(len(s)-w)])
    y = np.array([s[i+w]     for i in range(len(s)-w)])
    return X, y

X_tr, y_tr = make_windows(train_s, WINDOW)
X_va, y_va = make_windows(val_s,   WINDOW)
X_te, y_te = make_windows(test_s,  WINDOW)

# --- Tiny model ---
model = keras.Sequential([
    keras.Input(shape=(WINDOW, 1)),
    layers.LSTM(64, dropout=0.2),
    layers.Dense(32, activation='relu'),
    layers.Dense(1),
])

model.compile(
    optimizer=keras.optimizers.Adam(1e-3),
    loss=keras.losses.Huber(1.0),
    metrics=['mae', keras.metrics.RootMeanSquaredError()]
)

cbs = [
    callbacks.EarlyStopping(patience=10, restore_best_weights=True),
    callbacks.ReduceLROnPlateau(factor=0.5, patience=5, min_lr=1e-5),
]

history = model.fit(
    X_tr, y_tr,
    validation_data=(X_va, y_va),
    epochs=100,
    batch_size=32,
    callbacks=cbs,
    shuffle=False,  # ← critical!
)

# --- Inverse-transform before metrics ---
preds  = scaler.inverse_transform(model.predict(X_te))
actual = scaler.inverse_transform(y_te.reshape(-1, 1))
```

### Mandatory Baselines Comparison Table

| Model | RMSE | MAE | SMAPE |
|-------|------|-----|-------|
| Seasonal Naive (y_t = y_{t-7}) | — | — | — |
| Naive (y_t = y_{t-1}) | — | — | — |
| SARIMA(1,1,1)(1,1,1,7) | — | — | — |
| Prophet | — | — | — |
| **LSTM (64 units)** | — | — | — |

> If SARIMA or Prophet wins, **that is the finding, not an embarrassment.** One paragraph can mention TFT, N-BEATS, N-HiTS as state-of-the-art for larger datasets (accessible via `darts` or `neuralforecast`).

---

## 🐼 20 Pandas/Python Concepts to Explain Inline

For each concept, add **1–2 sentences of markdown** just before its first use:

| # | Concept | One-Liner Explanation |
|---|---------|----------------------|
| 1 | `DataFrame` vs `Series` | A DataFrame is a spreadsheet; a Series is a single column |
| 2 | `.loc` vs `.iloc` vs boolean | `.loc` by label, `.iloc` by position, `df[mask]` by True/False filter |
| 3 | `groupby` | Sort into piles, calculate on each pile, stack results (= SQL GROUP BY) |
| 4 | `pivot_table` vs `groupby` | groupby → long; pivot_table → wide cross-tab for heatmaps |
| 5 | `apply` / `map` / `lambda` | Vectorized ops first; `lambda x: x*2` is a nameless tiny function |
| 6 | `pd.to_datetime` | Converts strings to real dates; always pass `format=` when ambiguous |
| 7 | `.dt` accessor | `.dt.year`, `.dt.month`, `.dt.day_name()` — like `.str` but for dates |
| 8 | `set_index` + `resample` | `resample('M').sum()` is a time-aware groupby that changes frequency |
| 9 | `rolling` vs `resample` | resample changes frequency; rolling smooths within same frequency |
| 10 | `merge` / `join` / `concat` | concat stacks; merge = SQL JOIN on key; join = merge on index |
| 11 | `isna` / `fillna` / `dropna` | Check, replace, remove missing values |
| 12 | `value_counts` / `unique` / `nunique` | Frequency table / distinct values / count of distinct values |
| 13 | `nlargest` / `nsmallest` | Top/bottom N without full sort — faster than `sort_values().head(N)` |
| 14 | NumPy `reshape(-1, 28, 1)` | `-1` means "figure this out"; LSTM expects `(samples, timesteps, features)` |
| 15 | `np.log1p` / `np.expm1` | `log(1+x)` handles zero safely; `expm1` inverts it. Use for skewed money |
| 16 | Chained indexing trap | Never `df[mask]['col'] = val`; always `df.loc[mask, 'col'] = val` |
| 17 | `train_test_split` + temporal data | Shuffle for iid (regression); never shuffle time series |
| 18 | Scaler fit on train only | `scaler.fit(X_train)` then `.transform(X_test)` — never fit on test |
| 19 | `axis=0` vs `axis=1` | axis=0 collapses rows (column aggregation); axis=1 collapses columns |
| 20 | f-strings | `f'Total = {total:,.2f}'` — cleaner than `%` or `.format()`, 2025 standard |

**Bonus micro-explanations**: `df.copy()` vs reference assignment, `select_dtypes('number')`, `as_index=False` in groupby.

---

## 📚 Reference Shelf

### EDA & Dataset-Specific
- Rohit Sahoo's own Kaggle notebook: `kaggle.com/code/rohitsahoo/eda-superstore-dataset`
- Katie Huang: `medium.com/analytics-vidhya/exploratory-data-analysis-super-store-cb91c37bcb06`
- James Charest (LinkedIn): "Delving into a Super-Messy Superstore Dataset" — best reference for DD/MM/YYYY and Burlington VT issues
- Analytics Vidhya: `analyticsvidhya.com/blog/2022/03/eda-on-superstore-dataset-using-python/`

### LSTM & Time Series
- Keras 3 time series examples: `keras.io/examples/timeseries/`
- TensorFlow WindowGenerator tutorial: `tensorflow.org/tutorials/structured_data/time_series`
- arXiv 2601.00525 (2025) — small-model superiority in retail LSTM forecasting
- Machine Learning Mastery — LSTM scaling and leakage guides

### Regression & Interpretation
- scikit-learn docs: `TransformedTargetRegressor`, `TargetEncoder`, `TimeSeriesSplit`
- CatBoost official docs — categorical feature handling
- LightGBM official docs — early stopping and Optuna integration
- SHAP docs: `shap.readthedocs.io` — TreeExplainer front-page example

### Notebook Structure & Pedagogy
- Neptune.ai: "How to Use Exploratory Notebooks"
- Real Python: pandas GroupBy guide (best single resource for teaching groupby)
- Chris Moffitt — Practical Business Python

---

## ✅ What Makes This Notebook World-Class

Quality comes from **three deliberate design choices**:

1. **Design around this dataset's actual quirks** — DD/MM/YYYY dates, 11 Burlington nulls, no Profit column, order lines ≠ orders — rather than generic retail templates.

2. **Let models progress in honest difficulty** — Linear → Random Forest → LightGBM → CatBoost — with CatBoost featured because Superstore's categorical-heavy schema is exactly its sweet spot, and with the Keras MLP's expected loss used to teach that deep learning isn't automatic.

3. **Present the LSTM section truthfully** — a small 1-layer 64-unit model, trained with train-only scaling and chronological windows, benchmarked against naive, SARIMA, and Prophet. If SARIMA wins, that is the finding.

> A notebook that teaches its reader to be suspicious of over-parameterized LSTMs, to log-transform skewed targets, to never one-hot 1,850-category columns, and to parse dates explicitly is worth more than one that reports a misleading 99% R².

---

*Blueprint generated April 2026 · Based on arXiv 2601.00525, Keras 3.x docs, scikit-learn 1.3+, and analysis of ~15 published Superstore notebooks*
