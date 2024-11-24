# LG Aimers 5 Online Datathon
> 🏆 **LG Aimers 5 Online Datathon top 2% (15th/740)!**  
> This is the solution code for LG Aimers 5 Online Datathon hosted by LG AI Research.
## EDA
### ▶️ Feature Selection
**Preprocessing**
- Removed columns with all null values (278 columns).
- Replaced null and outlier values ('OK') with 0 in certain columns (e.g., HEAD NORMAL COORDINATE X AXIS(Stage1) Collect Result_Dam).
- Deleted columns with only a single unique value (40 columns).
145 columns remained
- Identified and removed redundant columns with high correlation, retaining only one.
- Used correlation coefficient for analysis, considering a threshold of 0.94 as appropriate due to:
  - Significant reduction in feature count.
  - Proximity to the commonly used threshold of 0.95.
  
