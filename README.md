# Trip Anomaly Detection Pipeline

Consolidated, automated system to identify suspicious, fraudulent, and inconsistent trip records in driver telematics data using **rule-based anomaly flags**.

**Status:** ✅ Complete & Operational | **Method:** Rule-Based Detection | **Output:** Excel (2 sheets)

## 📥 Latest Updates (2026-01-08)
- Added [anomaly_detection_v6.ipynb](anomaly_detection_v6.ipynb) with **user-level aggregation and infection status analysis**.
- User statistics including anomaly rates, risk categorization (GREEN/YELLOW/ORANGE/RED), and infection status (NOT_INFECTED/INFECTED).
- Advanced visualizations: bubble charts for 3D risk analysis, user trip distribution, and serious anomalies breakdown.
- Full CSV exports of user statistics, infected users, and RED risk users for investigation.
- Previous versions: [v5](anomaly_detection_v5.ipynb) (trip-level filtering), [v4](anomaly_detection_v4.ipynb), [v3](anomaly_detection_v3.ipynb), [v2](anomaly_detection_v2.ipynb), [v1](anomaly_detection_v1.ipynb).

## 🎯 Objectives

Detect and classify anomalous trip behavior to identify:
- **Odometer Fraud:** Recorded distance inflated beyond actual GPS movement (distance > 200km & haversine < 20km)
- **Data Integrity Issues:** Incomplete or corrupted trip records (zero events + EOT=Y but no points)
- **Unrealistic Behavior:** Impossible driving patterns (zero sudden events on substantial trips > 400km or > 1hr)
- **High-Risk Users:** Identify users with multiple suspicious trips for investigation

---

## 📊 Simplified Data Pipeline

```
TRIP DATA (CSV)
      ↓
FEATURE ENGINEERING (Haversine, distance_diff, duration, rates)
      ↓
RULE-BASED FLAGGING (4 suspicious flags)
      ↓
EXPLORATORY DATA ANALYSIS (Statistics, boxplots, correlations)
      ↓
ANALYSIS & VISUALIZATION (Scatter plots, flag distribution)
      ↓
USER-LEVEL PROFILING (Aggregate suspicious trips by user)
      ↓
EXPORT TO EXCEL (2 sheets: Flagged trips + User IDs)
```

**Pipeline Location:** `anomaly_detection.ipynb` (9 sections, fully consolidated)

---

## 📌 Rule-Based Anomaly Flags (Trip Level)

### Flag 1: `flag_suspicious_distance`
**Detects:** Odometer fraud or GPS spoofing
```
(distance > 200) & (distance_haversine < 20)
```

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Recorded distance | > 200 km | Substantial trips |
| Haversine (GPS) | < 20 km | Stayed local |
| **Interpretation** | Driver reports 200+ km but GPS shows <20km movement → odometer inflated |
| **Rate in data** | 0.4% (1 trip) | High specificity, rare |

**Example:** 250km recorded but only 15km straight-line distance = fraud signal.

---

### Flag 2: `flag_data_integrity_issue`
**Detects:** System corruption or incomplete recording
```
zero_sudden & eot_zero_point
where:
  zero_sudden = (all sudden_*_count = 0) & ((distance > 400) | (time > 3600))
  eot_zero_point = (eot = 'Y') & (trip_point = 0)
```

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Sudden events | = 0 | No maneuvers detected |
| Distance OR Time | > 400km OR > 1hr | Substantial trip |
| EOT | = 'Y' | Trip completed |
| Points | = 0 | No reward given |
| **Interpretation** | Safe drive (0 events) + completion but no points → data mismatch |
| **Rate in data** | 7.2% (18 trips) | System errors |

**Example:** 500km in 5 hours, zero events, EOT=Y but 0 points → integrity issue.

---

### Flag 3: `flag_zero_sudden_only`
**Detects:** Unrealistic driving behavior
```
zero_sudden & ~eot_zero_point
where:
  zero_sudden = same as Flag 2
  ~eot_zero_point = (eot != 'Y') | (trip_point > 0)
```

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Sudden events | = 0 | Over substantial trip |
| EOT or Points | ≠ integrity pattern | Not matching Flag 2 |
| **Interpretation** | Zero sudden events over 400km/1hr trip but not data issue → unrealistic |
| **Rate in data** | 92.8% (233 trips) | Human drivers always have some events |

**Example:** 500km in 5 hours with absolutely zero sudden events = inhuman behavior.

---

### Flag 4: `flag_any_suspicious`
**Combines:** Any of the three flags above
```
flag_suspicious_distance | flag_data_integrity_issue | flag_zero_sudden_only
```

---

## 🔍 Previous Experiments: Why KMeans Failed

We tested KMeans clustering (k=2, 3, 4) to automatically separate anomalies, but it failed because:

**Results:**
- k=2: 99.3% in Cluster 0 (25.6% suspicious), only 0.7% separated
- k=3/k=4: Similar - 97.4% of anomalies remained mixed in large cluster

**Why It Failed:**
1. Subtle anomalies have similar feature patterns to normal trips
2. KMeans assumes spherical, well-separated clusters (anomalies don't follow this)
3. Rule-based flags are more interpretable and effective

**Decision:** Use **rule-based detection** instead of clustering.

---

## 📊 Features Used

| Category | Features |
|----------|----------|
| **Distance** | distance, distance_haversine, distance_diff |
| **Time** | driving_time, driving_duration_days, distance_per_hour |
| **Sudden Events** | sudden_{start\|stop\|acceleration\|deceleration}_count, sudden_sum, sudden_per_hour |
| **Scoring** | trip_point, trip_safety_score, trip_seq |

---

## 📁 File Structure

```
repo/
├── README.md                              (this file)
├── requirements.txt                       (dependencies)
├── anomaly_detection.ipynb                (consolidated pipeline)
├── data/
│   └── trip_data.csv                      (input data ~65k trips)
└── outputs/
    └── anomaly_trips.xlsx                 (results: 2 sheets)
        ├── Sheet 1: Flagged_Trips         (all flagged trips + all columns)
        └── Sheet 2: User_IDs              (users with anomalies)
```

---

## 📋 Export Output Format

### Sheet 1: Flagged_Trips
Contains all trips flagged as suspicious with all original columns plus:
- `flag_suspicious_distance` - Odometer fraud indicator
- `flag_data_integrity_issue` - Data quality issue
- `flag_zero_sudden_only` - Unrealistic behavior
- `flag_any_suspicious` - Any of above 3 flags

**Rows:** Number of flagged trips  
**Columns:** All original + 4 flag columns

### Sheet 2: User_IDs
Contains summary of users with >= 1 suspicious trip:
- `user_id` - User identifier
- `total_trips` - Total trips by user
- `suspicious_trips` - Count of flagged trips
- `pct_suspicious` - Percentage of trips flagged

**Rows:** High-risk users  
**Columns:** 4 (user_id, total_trips, suspicious_trips, pct_suspicious)

---

## 🛠️ Notebook Structure

## 🛠️ Notebook Sections (9 Total)

| # | Section | Purpose | Output |
|---|---------|---------|--------|
| 1️⃣ | **Config & Imports** | Set thresholds, load libraries | Configuration |
| 2️⃣ | **Data Loading** | Load CSV or BigQuery | DataFrame (~65k rows) |
| 3️⃣ | **Feature Engineering** | Calculate distances, duration, rates | 9 new features |
| 4️⃣ | **Rule-Based Flagging** | Apply 4 suspicious flags | 8 flag columns |
| 5️⃣ | **EDA** | Statistics, boxplots, correlations | Visualizations |
| 6️⃣ | **Analysis & Visualization** | Scatter plots, flag distributions | 2 plots |
| 7️⃣ | **User Profiling** | Aggregate by user, identify high-risk | User summary |
| 8️⃣ | **Export Results** | Save to Excel (2 sheets) | anomaly_trips.xlsx |
| 9️⃣ | **Summary** | Final statistics and insights | Console output |

---

## 📈 Expected Results

After running the full pipeline, you'll get:

**Trip-Level Analysis:**
- Valid trips identified and separated from invalid ones
- Flagged trips breakdown by anomaly type
- Top anomalous trips with highest distance differences

**User-Level Analysis:**
- High-risk users ranked by suspicious trip count
- Percentage of anomalies per user
- Identification of repeat offenders

**Output Files:**
- **anomaly_trips.xlsx** (2 sheets with all flagged trips and user IDs)
- **Visualizations** (boxplots, heatmaps, scatter plots)
- **Console summary** (statistics, top users)

---

## 🎯 Configurable Thresholds

Edit **Section 1** of notebook to adjust detection sensitivity:

```python
# Trip Validity
MIN_DRIVING_TIME_SEC = 60      # Minimum seconds
MIN_DISTANCE_KM = 0.5           # Minimum kilometers

# Odometer Fraud Detection
SUSPICIOUS_DISTANCE_KM = 200    # Recorded distance threshold
SUSPICIOUS_HAVERSINE_KM = 20    # GPS distance threshold

# Unrealistic Behavior Detection
SUBSTANTIAL_DISTANCE_KM = 400   # Distance for zero-event check
SUBSTANTIAL_TIME_SEC = 3600     # Time for zero-event check (1 hour)
```

---

## ❓ How to Use

### Run the Pipeline
1. Open `anomaly_detection.ipynb` in Jupyter
2. Press `Ctrl+A` to select all cells
3. Press `Ctrl+Enter` to run all
4. Check `outputs/anomaly_trips.xlsx` for results

### Adjust Sensitivity (Optional)
1. Edit thresholds in Section 1
2. Rerun cells 1-9
3. Compare results with different settings

### BigQuery Integration (Optional)
1. Uncomment **Section 2, Option 2**
2. Authenticate with GCP
3. Run extraction cell
4. CSV file auto-saved for future use

---

## 📦 Dependencies

```
pandas >= 1.0.0
numpy >= 1.18.0
matplotlib >= 3.1.0
seaborn >= 0.10.0
openpyxl >= 3.0.0        (for Excel export)
pydata-google-auth >= 0.10.0  (optional, for BigQuery)
pandas-gbq >= 0.13.0          (optional, for BigQuery)
```

Install via:
```bash
pip install -r requirements.txt
```

---

## 📝 Next Steps

1. ✅ **Consolidated notebook:** `anomaly_detection.ipynb` (ready to use)
2. ✅ **Rule-based detection:** 4 configurable suspicious flags
3. ✅ **Excel export:** 2 sheets (flagged trips + user IDs)
4. ✅ **Visualizations:** EDA plots and anomaly distribution charts
5. ⏭️ **Release tagging:** Create a version tag after validating v5 outputs

---

## 📞 Support

For questions or to adjust thresholds:
1. Edit Section 1 of `anomaly_detection.ipynb`
2. Rerun pipeline
3. Review `outputs/anomaly_trips.xlsx` for new results
