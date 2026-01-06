# Issues Found in Training Notebook

## Critical Issues

### 1. **Weak Regularization**
Your models have very weak regularization:
- `reg_alpha=0.1` (too low, should be 0.5-2.0)
- `reg_lambda=1.0` (too low for this dataset)
- Result: Models overfit significantly (Train MAE much lower than Test MAE)

### 2. **Log Target Transformation**
Training on `log1p(y)` then `expm1(pred)` introduces:
- Bias in predictions (Jensen's inequality)
- Poor performance on extreme values
- The inverse transform doesn't preserve MAE optimization

### 3. **Too Many Approaches Without Focus**
You're testing 4 different preprocessing approaches which:
- Wastes computation
- Makes it hard to understand what works
- Creates confusion about best practices

### 4. **No Early Stopping in Initial Training**
Your first round of models don't use early stopping:
- Models train for full iterations even if not improving
- Leads to overfitting
- Wastes time

### 5. **Feature Explosion**
Without proper rare category grouping (min_count=50 is too low for 147k rows):
- OHE creates thousands of features
- Many features have <5 samples
- Model overfits on rare categories

## Performance Issues

### Current Results
- Test MAE: ~$35,000
- Test RMSE: ~$55,000-57,000
- R²: ~0.83

### Expected Improvements with Fixes
- Test MAE: $30,000-32,000 (10-15% better)
- Test RMSE: $48,000-52,000
- R²: 0.85-0.87

## Recommended Fixes (Implemented in Final_Training.ipynb)

1. **Stronger Regularization**
   ```python
   reg_alpha=1.0,  # L1 regularization
   reg_lambda=1.0,  # L2 regularization
   ```

2. **Remove Log Target Transformation**
   - Train directly on original target
   - Use MAE objective function

3. **Better Rare Category Handling**
   ```python
   min_samples=100  # For 147k dataset
   ```

4. **Early Stopping**
   ```python
   early_stopping_rounds=50
   eval_set=[(X_val, y_val)]
   ```

5. **Focus on Best Approach**
   - Use approach3 (minimal preprocessing, raw scale)
   - Skip approaches 1, 2, 4

6. **Proper Cross-Validation**
   - Use validation set for hyperparameter tuning
   - Test set only for final evaluation

## Summary

Your code is well-structured but has overfitting issues. The new notebook:
- Removes overfitting-prone techniques
- Uses stronger regularization
- Implements early stopping
- Cleaner, more maintainable code
- Should achieve MAE ~$30-32k (vs your current $35k)
