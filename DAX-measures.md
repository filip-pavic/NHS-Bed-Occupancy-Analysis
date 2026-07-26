# DAX Measures — GSTT Bed Occupancy Dashboard

Measures used for the Guy's and St Thomas' (GSTT) general & acute bed occupancy analysis

---

### Total Available Beds
```dax
Total Available Beds = SUM('GSTT_Bed_Occupancy_FullYear'[Available_GA])
```
Sums up all available general & acute beds across the dataset.

### Total Occupied Beds
```dax
Total Occupied Beds = SUM('GSTT_Bed_Occupancy_FullYear'[Occupied_GA])
```
Sums up all occupied general & acute beds.

### Occupancy Rate %
```dax
Occupancy Rate % = DIVIDE([Total Occupied Beds],[Total Available Beds], 0)
```
Core rate calculation. Uses `DIVIDE` instead of `/` so it doesn't error out on a zero denominator.

### GSTT Occupancy Rate %
```dax
GSTT Occupancy Rate % = CALCULATE([Occupancy Rate %], 'GSTT_Bed_Occupancy_FullYear'[Org Code] = "RJ1")
```
Filters the occupancy rate down to just GSTT (org code RJ1), so it can be shown alongside the peer average without being affected by slicers on other trusts.

### Peer Avg Occupancy Rate %
```dax
Peer Avg Occupancy Rate % = CALCULATE([Occupancy Rate %], 'GSTT_Bed_Occupancy_FullYear'[Org Code] <> "RJ1")
```
Same idea in reverse — everyone except GSTT, giving a peer benchmark to compare against.

### Headroom %
```dax
Headroom % = 0.85 - [Occupancy Rate %]
```
How much room is left before hitting the 85% safe threshold. Positive = under threshold, negative = over.

### Variance vs 85% Threshold
```dax
Variance vs 85% Threshold = [Occupancy Rate %] - 0.85
```
Same comparison as Headroom, just flipped — how far over (or under) the threshold the current rate sits. Kept both since some visuals read better one way round.

### Occupancy Status Color
```dax
Occupancy Status Color = 
VAR _Rate = [Occupancy Rate %]
VAR _NormalizedRate = IF(_Rate > 1, _Rate / 100, _Rate)
RETURN
IF(
    _NormalizedRate > 0.85,
    "#B05246", -- Soft Red / High Risk (Over 85%)
    "#689F38"  -- Soft Green / Safe Operational Level (Under 85%)
)
```
Returns a hex color for conditional formatting. The normalization step is a safety net in case the rate ever comes through as a whole number (e.g. 92) instead of a decimal (0.92).

### Org Name Short
```dax
Org Name Short = 
SWITCH(
    TRUE(),
    CONTAINSSTRING(GSTT_Bed_Occupancy_FullYear[Org Name], "Guy's and St Thomas'"), "GSTT",
    CONTAINSSTRING(GSTT_Bed_Occupancy_FullYear[Org Name], "King's College"), "KCH",
    CONTAINSSTRING(GSTT_Bed_Occupancy_FullYear[Org Name], "Imperial"), "Imperial",
    CONTAINSSTRING(GSTT_Bed_Occupancy_FullYear[Org Name], "University College"), "UCLH",
    CONTAINSSTRING(GSTT_Bed_Occupancy_FullYear[Org Name], "Royal Free"), "Royal Free",
    CONTAINSSTRING(GSTT_Bed_Occupancy_FullYear[Org Name], "St George's"), "St George's",
    CONTAINSSTRING(GSTT_Bed_Occupancy_FullYear[Org Name], "Lewisham"), "L&G",
    GSTT_Bed_Occupancy_FullYear[Org Name]
)
```
Shortens the full trust names into labels that actually fit on chart axes. Falls back to the original name if none of the trusts match.

### Quarter_Sort
```dax
Quarter_Sort = 
SWITCH(
    'GSTT_Bed_Occupancy_FullYear'[Quarter],
    "Q1", 1,
    "Q2", 2,
    "Q3", 3,
    "Q4", 4,
    5
)
```
Just gives the quarters a number so they sort Q1→Q4 instead of alphabetically.
