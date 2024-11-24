# LG Aimers 5 Online Datathon
> 🏆 **LG Aimers 5 Online Datathon top 2% (15th/740)!**  
> This is the solution code for LG Aimers 5 Online Datathon hosted by LG AI Research.
## EDA
### ▶️ Feature Selection
- Removed columns with all null values (278 columns).
- Replaced null and outlier values ('OK') with 0 in certain columns (e.g., HEAD NORMAL COORDINATE X AXIS(Stage1) Collect Result_Dam).
- Deleted columns with only a single unique value (40 columns).
- Identified and removed redundant columns with high correlation, retaining only one, using a correlation coefficient with a threshold of 0.94.
### ▶️ Upsampling
The train data exhibited a class imbalance with an AbNormal to Normal ratio of 0.06:1.
- utilized SMOTENC to address this imbalance through upsampling.
- upsampled AbNormal data to reach half the quantity of Normal data.
- downsampled Normal data to equal the amount of augmented AbNormal data.
