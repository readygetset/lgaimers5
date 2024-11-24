# LG Aimers 5 Online Datathon
> 🏆 **LG Aimers 5 Online Datathon top 2% (15th/740)!**  
> This is the solution code for LG Aimers 5 Online Datathon hosted by LG AI Research.

## EDA

### ▶️ Feature Selection
- Removed columns with all null values (278 columns).
- Replaced null and outlier values ('OK') with 0 in certain columns (e.g., HEAD NORMAL COORDINATE X AXIS(Stage1) Collect Result_Dam).
- Deleted columns with only a single unique value (40 columns).
- Identified and removed redundant columns with high correlation, retaining only one, using a correlation coefficient with a threshold of 0.94.
  
### ▶️ Upsampling
- Class imbalance of train data with an AbNormal to Normal ratio of 0.06:1.
- Utilized SMOTENC to address this imbalance through upsampling.
- Upsampled AbNormal data to reach half the quantity of Normal data.
- Downsampled Normal data to equal the amount of augmented AbNormal data.

### ▶️ Rule Discovery
- Analyzed groups of common columns and discovered several rules:
  - If any Equipment_xxx differs, it is classified as AbNormal.
  - If any Production Qty Collect Result_xxx differs, it is classified as AbNormal.
  - If any Receip No Collect Result_xxx differs, it is classified as AbNormal.
- Considered two methods for handling these rules:
  - Add columns indicating rule satisfaction and proceed with training.
  - Post-training, hardcode to classify as AbNormal if rules are satisfied.
- Experimental results showed that the second method outperformed the first, leading to its final adoption.

## Model

### ▶️ Baseline
Established a baseline setting after surpassing 0.2 points, which was further developed individually.
- Features used: 47 features remaining after removing those with a correlation coefficient of 0.94 or higher.
- Sampling technique: Used SMOTENC to upsample the minority class to 50% of the majority class, then downsampled the majority class to match the minority class.
- Model: CatBoost
  - Employed K-Fold Cross Validation with K = 4.
  - Iteration: 1000, Learning Rate: 0.1, Depth: 6
- Rule handling: Hardcoded rules.

### ▶️ Model 1: Additional Feature Engineering
**Feature Engineering for X, Y, Z Coordinate**
- Identified clustering of coordinates at specific points.
- Defined clustering intervals for each column.
- Calculated the mean of values within each cluster and converted each coordinate to its distance from the cluster center.
- Applied appropriate scaling (x10) through experimentation.
- Compressed X, Y, Z coordinates of the same process and stage into a single column using 3D Euclidean Distance.
- Removed outliers not belonging to any cluster before application.  
**Feature Engineering for Resin**
- Focused on the ratio of (resin discharge speed resin discharge time) to the actual dispensed resin volume.
- Compressed three features into one for each stage of each process, removing the original features.
- For Fill1 process data, applied scaling (x100) and adjusted the mean for better classification.

### ▶️ Model 2: Stacking Ensemble
- Conducted training using CatBoostClassifier with 5-Fold Cross Validation (CV).
  - Saved predictions for validation and test data for each fold.
  - Stored the average prediction for the test data after training.
- Conducted training using LGBMClassifier with 5-Fold CV.
  - Saved predictions for validation and test data for each fold.
  - Stored the average prediction for the test data after training.
- Created a new feature `new_train` from the predictions of both models on the train data.
- Created a new feature `new_test` from the predictions of both models on the test data.
- Trained a new LGBMClassifier using `new_train`.
- Predicted the target for test data using `new_test`.

### ▶️ Model 3: Adjustment of Sampling Rate
- Used SMOTENC to upsample the minority class (AbNormal) to 30% of the majority class (Normal) and then downsampled the majority class to 1.2 times the minority class.
- Reduced the upsampling ratio to 30% as excessive upsampling was found to degrade performance.
- Considered the general prevalence of Normal over AbNormal, opting for a 1.2x downsampling instead of equalizing.
- Trained the model using CatBoost with 4 K-Fold splits, 800 iterations, a learning rate of 0.1, and a depth of 6.

### ▶️ Model 4: Feature Selection with LLM
- Utilized the paper "<LLM-Select: Feature Selection with Large Language Models>" to assign an importance score between 0 and 1 to each column using a prompt.
- Employed GPT-4 API for feature selection, with prompts similar to the LLM-Score prompt from the paper.
- Set an importance threshold of 0.6 through experimentation, resulting in 52 selected features.
- Applied SMOTENC to upsample the minority class to 30% of the majority class and downsampled the majority class to 1.2 times the minority class.
- Trained the model using CatBoost with 4 K-Fold splits, 1000 iterations, a learning rate of 0.1, and a depth of 9.

### ▶️ Model 5: Oversampling Techniques
- Conducted experiments with various oversampling techniques under the same conditions as the baseline SMOTENC, using average F1 score as the primary performance metric.
- SMOTEENN achieved the highest performance with an average F1 score of 0.9448.
 -BorderlineSMOTE and KMeansSMOTE also demonstrated strong performance with scores of 0.9382 and 0.9376, respectively.
- RandomOverSampler showed lower performance compared to other methods, with an average F1 score of 0.8849.
- Confirmed that complex sampling methods significantly enhance model performance.
- Applied SMOTEENN, which showed the best performance, to the baseline.

### ▶️ Model 6: Sampling Rate Ensemble
- Randomly selected pairs of features and performed arithmetic operations, then trained a Decision Tree on the resulting features.
- Added new features based on combinations that consistently recorded low loss, such as WorkMode Collect Result_Fill2 + CURE SPEED Collect Result_Fill2.
- Removed 28 columns that consistently recorded high loss.
- Discovered an accuracy-precision tradeoff depending on the undersample/upsample ratio.
- Consequently, sampled data at various ratios and trained models to create an ensemble.

### ▶️ Model 7: Deep Density Hybrid Learning
- Limitations of SMOTE:
  - SMOTE considers only minority samples, potentially disrupting learning balance by ignoring the influence of majority samples.
  - In high-dimensional spaces, Euclidean distance may inaccurately calculate distances, distorting data distribution.
- Proposed Method from "Learning From Imbalanced Data With Deep Density Hybrid Sampling":
  - Introduces a Hybrid Sampling technique using an AutoEncoder-based Embedding Network.
  - Combines three loss terms (Cross-Entropy, Center, Reconstruction) to maximize sampling performance.
  - Ensures high-quality data for the minority class through Density-based Filtering.
  - Applied the proposed method and hyperparameters (e.g., number of layers in the embedding layer, sampling ratio) as suggested in the paper.
  - Compared to SMOTENC and SMOTEENN used in Baseline and Model#5, the method showed comparable or slightly improved performance.
  - The DDHS technique achieved an average F1 score of 0.9496, outperforming SMOTE-based methods.

### ▶️ Model 8: AutoML
- Utilized the mljar-supervised library for model training.
- Initially attempted to address data imbalance through sampling, but results were unsatisfactory.
- Decided against applying sampling, as a study by Yotam Elo ("To SMOTE, or not to SMOTE?", 2022) indicated that strong learners may perform worse with oversampling.

### ▶️ Ensemble Model
- Performed Hard Voting using 8 submission files generated from different models.
- Hard Voting is an excellent method for enhancing performance by compensating for individual model weaknesses.
- Frequently achieves superior results in handling imbalanced data in various Kaggle Competitions.
- Experimented with different model combinations for Hard Voting; the combination of Model #1, Model #2, Model #5, and Model #6 yielded the best performance.
- Although combining AutoML showed significantly superior performance at times, it was excluded due to high variance, difficulty in reproducing results, and potential overfitting concerns.

## Improvement Suggestions
- Observed lower performance on Private compared to Public leaderboard, indicating model overfitting. Proposed improvements include:
  - Implementing more sophisticated data augmentation techniques.
  - Reducing the number of columns to 30 or fewer by deleting or synthesizing features.
  - Optimizing weights for each model during the Hard Voting process to achieve better predictive performance.

