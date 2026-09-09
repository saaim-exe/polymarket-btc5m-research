
# 1. Problem Definition
# 2. Load Data
# 3. Understand Data
# 4. Define Target
# 5. Initial Features
# 6. Chronological Split
# 7. Baseline
# 8. Logistic Regression
# 9. Validation Metrics
# 10. Compare With Market
# 11. What Failed / What Next?

## Mistakes to avoid early on 
# Dont use future information in a feature 
# Dont preprocess the full dataset before splitting 
# Dont repeatedly tune based on test performance 
# Dont randomly shuffle observations (for this time series dataset ; CRUCIAL)
# Always compare your results against a dumb baseline 


MAIN QUESTION? 
CLASSIFICATION PROBLEM 

Can we estimate the P(UP resolves = 1 | X_t) {where X denotes some market info at time t} better than the market price of the contract? 

PREDICTION TARGET (y) 

 y = resolution 

 1 = UP 
 0 = DOWN 

 FEATURES (x) 

 BTC Returns 
 Current Market Price 
 # Prices where t_prediction = end_ts - 5minutes (we want at the 10th minute)
 Volatility 

 SPLIT 

Chronological (Time Series) 
Dont Randomly Shuffle... 

BASELINE 

Market Implied Probability 
# the current contract price of UP/DOWN  = market implied probability 
# market proability proxy ~ (bid + ask) / 2 


MODELS 

# initial -> LOGISTIC REGRESSION 

METRICS 
- BRIER SCORE 
- LOG LOSS
- ACCURACY 

VALIDATION (tune / experiment)

TEST (do not touch until final evaluation)









