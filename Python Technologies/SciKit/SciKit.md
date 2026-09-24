# Scikit-learn Comprehensive, Structured, and Progressive Learning Roadmap

## From Foundational Machine Learning Concepts to Advanced Practical Mastery

The roadmap below is designed around the core Scikit-learn workflow: **estimators → preprocessing → pipelines → supervised/unsupervised learning → model evaluation → hyperparameter optimization → inspection → production practices**. The current stable Scikit-learn release is **1.9.1**, released in September 2026. ([Scikit-learn][1])

---

# I. Python and Mathematical Foundations

* **1. Python Prerequisites**

  * Python syntax

    * Variables
    * Data types
    * Operators
    * Conditional statements
    * Loops
  * Functions

    * Parameters
    * Return values
    * Lambda functions
  * Core data structures

    * Lists
    * Tuples
    * Dictionaries
    * Sets
  * Object-oriented basics

    * Classes
    * Objects
    * Methods
    * Attributes
  * Modules and packages
  * Exceptions
  * Virtual environments
  * Package management

* **2. NumPy Foundations**

  * Arrays
  * Dimensions
  * Shapes
  * Data types
  * Indexing
  * Slicing
  * Boolean masking
  * Broadcasting
  * Vectorized operations
  * Linear algebra

    * Dot products
    * Matrix multiplication
    * Norms

* **3. Pandas Foundations**

  * Series
  * DataFrames
  * Indexes
  * Loading datasets
  * Selecting columns
  * Filtering rows
  * Sorting
  * Grouping
  * Aggregation
  * Missing values
  * Merging
  * Joining
  * Concatenation
  * Reshaping
  * Data-type handling

* **4. Mathematical Foundations**

  * Linear algebra

    * Vectors
    * Matrices
    * Matrix operations
    * Eigenvalues
    * Eigenvectors
  * Probability

    * Random variables
    * Probability distributions
    * Conditional probability
    * Bayes' theorem
  * Statistics

    * Mean
    * Median
    * Variance
    * Standard deviation
    * Covariance
    * Correlation
  * Calculus

    * Derivatives
    * Partial derivatives
    * Gradients
    * Optimization
  * Optimization concepts

    * Objective functions
    * Loss functions
    * Regularization

---

# II. Machine Learning Foundations

* **5. Introduction to Machine Learning**

  * Machine learning definition
  * AI versus machine learning
  * Supervised learning
  * Unsupervised learning
  * Semi-supervised learning
  * Reinforcement learning
  * Features
  * Targets
  * Samples
  * Labels
  * Training data
  * Validation data
  * Test data

* **6. Core ML Concepts**

  * Model
  * Parameters
  * Hyperparameters
  * Training
  * Prediction
  * Generalization
  * Bias
  * Variance
  * Overfitting
  * Underfitting
  * Model complexity

* **7. The Standard ML Workflow**

  * Define the problem
  * Acquire data
  * Explore data
  * Clean data
  * Split data
  * Preprocess features
  * Select baseline model
  * Train
  * Validate
  * Tune
  * Evaluate
  * Interpret
  * Deploy

---

# III. Scikit-learn Fundamentals

Scikit-learn uses a consistent estimator-oriented API, where estimators commonly expose methods such as `fit`, `predict`, and, where appropriate, `transform` and `fit_transform`. Its current documentation organizes functionality around supervised learning, unsupervised learning, model selection/evaluation, inspection, preprocessing, computational utilities, model persistence, and common pitfalls. ([Scikit-learn][1])

* **8. Installation and Environment**

  * Installing Scikit-learn
  * Virtual environments
  * Dependency management
  * Version management
  * Importing modules
  * Checking installed versions

* **9. Estimator API**

  * Estimators
  * Transformers
  * Predictors
  * `fit()`
  * `predict()`
  * `transform()`
  * `fit_transform()`
  * `fit_predict()`
  * Model attributes
  * Hyperparameters

    * `get_params()`
    * `set_params()`

* **10. Dataset Handling**

  * NumPy arrays
  * Pandas DataFrames
  * Sparse matrices
  * Feature matrices `X`
  * Target vectors `y`
  * Dataset shape
  * Feature names
  * Target encoding

* **11. Built-in Dataset Utilities**

  * Toy datasets
  * Real-world datasets
  * Synthetic datasets
  * Train/test splitting
  * Dataset generation

---

# IV. Exploratory Data Analysis for ML

* **12. Understanding the Dataset**

  * Dataset dimensions
  * Feature distributions
  * Target distribution
  * Feature types
  * Missing values
  * Duplicate observations
  * Outliers

* **13. Statistical Exploration**

  * Descriptive statistics
  * Distribution analysis
  * Correlation
  * Covariance
  * Group-level statistics
  * Class balance

* **14. Visualization**

  * Histograms
  * Box plots
  * Scatter plots
  * Pairwise relationships
  * Classification boundaries
  * Residual plots
  * Learning curves
  * Validation curves

---

# V. Data Preprocessing

Scikit-learn's transformers implement learned preprocessing through `fit` and apply it through `transform`; preprocessing can be composed through pipelines and column-wise transformations. ([Scikit-learn][2])

* **15. Feature Scaling**

  * Standardization

    * `StandardScaler`
  * Min-max scaling

    * `MinMaxScaler`
  * Robust scaling

    * `RobustScaler`
  * Maximum-absolute scaling

    * `MaxAbsScaler`
  * When scaling matters

    * Distance-based models
    * Linear models
    * Neural networks
    * Kernel methods

* **16. Categorical Encoding**

  * One-hot encoding

    * `OneHotEncoder`
  * Ordinal encoding

    * `OrdinalEncoder`
  * Encoding unknown categories
  * Handling high-cardinality features
  * Choosing appropriate encoding strategies

* **17. Missing-Value Handling**

  * Detecting missing data
  * Simple imputation

    * Mean
    * Median
    * Most frequent
    * Constant
  * Iterative imputation
  * Nearest-neighbor imputation
  * Missing indicators
  * Avoiding leakage during imputation

* **18. Feature Transformation**

  * Power transformations
  * Quantile transformations
  * Polynomial features
  * Function transformers
  * Distribution normalization

* **19. Outlier Handling**

  * Outlier detection
  * Robust scaling
  * Quantile-based approaches
  * Isolation-based approaches
  * Domain-specific treatment

---

# VI. Feature Engineering

* **20. Numerical Feature Engineering**

  * Ratios
  * Differences
  * Aggregates
  * Log transformations
  * Polynomial interactions
  * Binning
  * Ranking

* **21. Categorical Feature Engineering**

  * Category consolidation
  * Rare-category handling
  * Interaction features
  * Frequency-based representations

* **22. Date and Time Features**

  * Year
  * Month
  * Day
  * Week
  * Day of week
  * Time intervals
  * Cyclical representations

* **23. Feature Interaction**

  * Polynomial interaction terms
  * Domain-driven interactions
  * Interaction discovery
  * Feature crosses

---

# VII. Feature Selection

* **24. Why Feature Selection Matters**

  * Reduce dimensionality
  * Reduce noise
  * Improve interpretability
  * Reduce computational cost
  * Potentially improve generalization

* **25. Filter Methods**

  * Variance threshold
  * Univariate statistical tests
  * Correlation-based selection

* **26. Wrapper Methods**

  * Recursive feature elimination
  * Recursive feature elimination with cross-validation

* **27. Embedded Methods**

  * L1 regularization
  * Tree-based importance
  * Model-based selection

* **28. Selection Pitfalls**

  * Data leakage
  * Selection before splitting
  * Selection using test data
  * Unstable feature importance

---

# VIII. Pipelines and Composite Estimators

Pipelines are central to robust Scikit-learn workflows because preprocessing and estimation can be combined into a single composite estimator. `ColumnTransformer` is particularly useful when different feature columns require different transformations. ([Scikit-learn][2])

* **29. Pipeline Fundamentals**

  * `Pipeline`
  * Sequential transformations
  * Transformer → estimator
  * `fit`
  * `predict`
  * Accessing pipeline steps

* **30. ColumnTransformer**

  * Numerical preprocessing
  * Categorical preprocessing
  * Different transformations by column
  * Combining multiple preprocessing branches

* **31. FeatureUnion and Feature Composition**

  * Parallel feature transformations
  * Combining feature spaces
  * Complex preprocessing architectures

* **32. Pipeline Hyperparameters**

  * Nested parameter names
  * Tuning preprocessing
  * Tuning the final estimator
  * Pipeline-aware cross-validation

* **33. Pipeline Best Practices**

  * Preventing leakage
  * Reproducibility
  * Consistent training/inference transformations
  * Encapsulating preprocessing
  * Building reusable workflows

---

# IX. Regression

* **34. Linear Regression**

  * Ordinary least squares
  * Assumptions
  * Coefficients
  * Predictions
  * Residuals

* **35. Regularized Linear Regression**

  * Ridge
  * Lasso
  * Elastic Net
  * Regularization strength
  * Feature shrinkage
  * Sparse coefficients

* **36. Polynomial Regression**

  * Polynomial features
  * Nonlinear relationships
  * Degree selection
  * Overfitting

* **37. Robust Regression**

  * Handling outliers
  * RANSAC
  * Huber regression
  * Theil-Sen regression

* **38. Generalized Linear Models**

  * Link functions
  * Distribution assumptions
  * Poisson-style regression
  * Gamma-style regression
  * Tweedie models

* **39. Regression Evaluation**

  * MAE
  * MSE
  * RMSE
  * R²
  * Adjusted-analysis concepts
  * Explained variance
  * Quantile-based metrics

---

# X. Classification

* **40. Logistic Regression**

  * Binary classification
  * Multiclass classification
  * Decision scores
  * Probabilities
  * Regularization

* **41. Nearest Neighbors**

  * K-nearest neighbors
  * Distance metrics
  * Choosing `k`
  * Classification versus regression

* **42. Naive Bayes**

  * Gaussian Naive Bayes
  * Multinomial Naive Bayes
  * Bernoulli Naive Bayes
  * Probabilistic assumptions

* **43. Decision Trees**

  * Tree construction
  * Splits
  * Impurity

    * Gini
    * Entropy
  * Tree depth
  * Pruning concepts
  * Feature importance

* **44. Support Vector Machines**

  * Linear SVM
  * Kernel SVM
  * Margin
  * Support vectors
  * Kernel functions
  * Regularization
  * Parameter tuning

---

# XI. Tree Ensembles

* **45. Random Forests**

  * Bagging
  * Random feature selection
  * Ensemble averaging
  * Classification
  * Regression
  * Feature importance

* **46. Extremely Randomized Trees**

  * Extra Trees
  * Randomized splits
  * Bias/variance behavior

* **47. Gradient Boosting**

  * Sequential learning
  * Weak learners
  * Learning rate
  * Number of estimators
  * Tree depth

* **48. HistGradientBoosting**

  * Histogram-based learning
  * Computational efficiency
  * Large tabular datasets
  * Classification
  * Regression

* **49. Ensemble Strategy**

  * Bagging versus boosting
  * Bias/variance trade-offs
  * Diversity among estimators
  * Ensemble interpretability

---

# XII. Model Evaluation

Scikit-learn provides dedicated model-selection and evaluation facilities covering cross-validation, hyperparameter tuning, metrics/scoring, and validation/learning curves. ([Scikit-learn][3])

* **50. Train/Validation/Test Splits**

  * Training set
  * Validation set
  * Test set
  * Holdout evaluation
  * Repeated evaluation

* **51. Cross-Validation**

  * K-fold cross-validation
  * Stratified K-fold
  * Repeated cross-validation
  * Leave-one-out
  * Group-based CV
  * Time-series-aware splitting

* **52. Classification Metrics**

  * Accuracy
  * Precision
  * Recall
  * F1
  * ROC-AUC
  * PR-AUC
  * Log loss
  * Confusion matrix
  * Balanced accuracy
  * Matthews correlation coefficient

* **53. Regression Metrics**

  * MAE
  * MSE
  * RMSE
  * R²
  * Mean absolute percentage concepts
  * Median absolute error
  * Quantile-related metrics

* **54. Metric Selection**

  * Matching metric to business objective
  * Imbalanced classification
  * Cost-sensitive evaluation
  * Ranking metrics
  * Probability-quality evaluation

---

# XIII. Hyperparameter Optimization

* **55. Hyperparameter Fundamentals**

  * Parameters versus hyperparameters
  * Manual tuning
  * Search spaces
  * Validation strategy

* **56. Grid Search**

  * `GridSearchCV`
  * Parameter grids
  * Cross-validation
  * Best estimator
  * Best score

* **57. Randomized Search**

  * `RandomizedSearchCV`
  * Probability distributions
  * Search-budget control
  * Large hyperparameter spaces

* **58. Advanced Search Concepts**

  * Successive halving
  * Resource allocation
  * Multi-metric scoring
  * Custom scoring
  * Nested cross-validation

* **59. Hyperparameter Tuning Strategy**

  * Establish baseline
  * Tune high-impact parameters
  * Avoid unnecessary search
  * Compare models fairly
  * Preserve a final untouched test set

---

# XIV. Imbalanced Learning

* **60. Class Imbalance**

  * Imbalanced targets
  * Rare-event classification
  * Majority/minority classes
  * Accuracy pitfalls

* **61. Evaluation**

  * Precision
  * Recall
  * F1
  * PR-AUC
  * ROC-AUC
  * Confusion matrices

* **62. Model-Level Strategies**

  * Class weights
  * Threshold adjustment
  * Cost-sensitive learning

* **63. Sampling Concepts**

  * Oversampling
  * Undersampling
  * Synthetic sampling concepts
  * Leakage-aware resampling

---

# XV. Model Calibration and Decision Thresholds

* **64. Probability Calibration**

  * Predicted probabilities
  * Calibration concepts
  * Reliability diagrams
  * Calibration curves

* **65. Calibration Methods**

  * Sigmoid calibration
  * Isotonic calibration
  * Cross-validation-aware calibration

* **66. Threshold Optimization**

  * Default threshold
  * Precision/recall trade-offs
  * Cost-based threshold selection
  * Business-rule thresholds

---

# XVI. Unsupervised Learning

* **67. Clustering**

  * K-means
  * Mini-batch K-means
  * Hierarchical clustering
  * Agglomerative clustering
  * DBSCAN
  * OPTICS
  * Spectral clustering
  * Gaussian mixture models

* **68. Clustering Concepts**

  * Distance metrics
  * Cluster initialization
  * Number of clusters
  * Density-based clustering
  * Cluster validation

* **69. Clustering Evaluation**

  * Silhouette score
  * Calinski-Harabasz concepts
  * Davies-Bouldin concepts
  * External validation when labels exist

---

# XVII. Dimensionality Reduction

* **70. Principal Component Analysis**

  * PCA fundamentals
  * Variance maximization
  * Components
  * Explained variance
  * Whitening

* **71. Truncated SVD**

  * Sparse matrices
  * Latent representations
  * Text feature reduction

* **72. Other Dimensionality Reduction Methods**

  * Random projection
  * Non-negative matrix factorization
  * Manifold learning

    * Isomap
    * Locally Linear Embedding
    * Spectral Embedding
    * t-SNE

* **73. Practical Applications**

  * Visualization
  * Noise reduction
  * Compression
  * Feature preprocessing

---

# XVIII. Mixture Models and Probabilistic Learning

* **74. Gaussian Mixture Models**

  * Mixture components
  * Covariance structures
  * Soft clustering
  * Density estimation

* **75. Expectation-Maximization**

  * Latent variables
  * E-step
  * M-step
  * Convergence

* **76. Density Estimation**

  * Kernel density estimation
  * Probability density modeling
  * Anomaly-oriented applications

---

# XIX. Anomaly and Novelty Detection

* **77. Outlier Detection**

  * Statistical outliers
  * Distribution-based approaches
  * Model-based approaches

* **78. Scikit-learn Methods**

  * Isolation Forest
  * Local Outlier Factor
  * One-Class SVM
  * Elliptic Envelope

* **79. Practical Considerations**

  * Contamination assumptions
  * Novelty detection
  * Outlier scoring
  * Threshold selection

---

# XX. Semi-Supervised Learning

* **80. Semi-Supervised Fundamentals**

  * Labeled data
  * Unlabeled data
  * Mixed training sets
  * Why semi-supervised learning is useful

* **81. Algorithms**

  * Label propagation
  * Label spreading
  * Self-training concepts

* **82. Practical Considerations**

  * Quality of pseudo-labels
  * Label noise
  * Validation strategies

---

# XXI. Feature Extraction

* **83. Text Feature Extraction**

  * Bag of words
  * `CountVectorizer`
  * TF-IDF
  * `TfidfVectorizer`
  * N-grams
  * Vocabulary management

* **84. Text Classification**

  * Sparse matrices
  * Naive Bayes
  * Logistic regression
  * Linear SVM
  * Text pipelines

* **85. Feature Hashing**

  * Hashing trick
  * Memory efficiency
  * Streaming-style feature construction

* **86. Dictionary-Based Features**

  * Feature extraction from mappings
  * Mixed structured inputs

---

# XXII. Multiclass and Multioutput Learning

* **87. Multiclass Classification**

  * One-vs-rest
  * One-vs-one
  * Native multiclass estimators

* **88. Multioutput Learning**

  * Multioutput regression
  * Multi-label classification
  * Multiple targets

* **89. Problem Transformation**

  * Binary decomposition
  * Label powerset concepts
  * Independent estimators

---

# XXIII. Ensemble Learning

* **90. Voting**

  * Hard voting
  * Soft voting
  * Regression voting

* **91. Bagging**

  * Bootstrap aggregation
  * Base estimator diversity

* **92. Randomized Ensembles**

  * Random forests
  * Extra trees

* **93. Boosting**

  * AdaBoost
  * Gradient boosting
  * Histogram gradient boosting

* **94. Stacking**

  * Base estimators
  * Meta-estimator
  * Cross-validated stacking
  * Avoiding leakage

---

# XXIV. Model Inspection and Interpretability

Scikit-learn provides inspection functionality for understanding how trained models use features, including tools such as permutation importance and partial-dependence-style analysis. ([Scikit-learn][4])

* **95. Feature Importance**

  * Tree-based importance
  * Permutation importance
  * Importance instability
  * Correlated-feature problems

* **96. Partial Dependence**

  * Partial dependence plots
  * Marginal effects
  * Feature interactions

* **97. Individual Conditional Expectation**

  * Per-sample prediction behavior
  * Heterogeneous effects

* **98. Model Inspection**

  * Decision boundaries
  * Residual analysis
  * Prediction errors
  * Calibration
  * Error segmentation

* **99. Explainability Limitations**

  * Correlation versus causation
  * Feature leakage
  * Extrapolation
  * Model-specific versus model-agnostic interpretation

---

# XXV. Statistical Learning and Regularization

* **100. Bias-Variance Trade-off**

  * High bias
  * High variance
  * Model complexity
  * Generalization

* **101. Regularization**

  * L1
  * L2
  * Elastic Net
  * Complexity control

* **102. Feature Selection Through Regularization**

  * Sparse solutions
  * Coefficient shrinkage
  * Interpretability

* **103. Resampling-Based Reasoning**

  * Cross-validation
  * Bootstrap concepts
  * Stability analysis

---

# XXVI. Advanced Preprocessing

* **104. Heterogeneous Data**

  * Numeric columns
  * Categorical columns
  * Text columns
  * Sparse features
  * Multiple transformation branches

* **105. Custom Transformers**

  * Transformer API
  * `fit`
  * `transform`
  * Custom feature logic
  * Reusable preprocessing

* **106. Target Transformation**

  * Transforming regression targets
  * Log-scale targets
  * Inverse transformations
  * Prediction interpretation

---

# XXVII. Advanced Model Selection

* **107. Nested Cross-Validation**

  * Inner loop
  * Outer loop
  * Hyperparameter selection
  * Unbiased performance estimation

* **108. Group-Aware Validation**

  * Group K-fold
  * Related observations
  * Patient/customer/entity-level leakage prevention

* **109. Time-Aware Validation**

  * Temporal ordering
  * Rolling evaluation
  * Time-series splits
  * Future-information leakage

* **110. Validation Strategy Design**

  * Match split strategy to deployment
  * Preserve independence
  * Avoid preprocessing leakage
  * Avoid target leakage

---

# XXVIII. Computational Efficiency

* **111. Dataset Size Considerations**

  * Small datasets
  * Medium datasets
  * Large datasets
  * Sparse datasets

* **112. Efficiency Techniques**

  * Efficient feature representations
  * Sparse matrices
  * Batching
  * Parallel processing
  * Caching
  * Appropriate algorithm choice

* **113. Randomness and Reproducibility**

  * `random_state`
  * Deterministic experimentation
  * Seed management
  * Reproducible benchmarks

---

# XXIX. Advanced Scikit-learn API Concepts

* **114. Estimator Conventions**

  * Constructor parameters
  * `fit`
  * Learned attributes
  * Parameter introspection

* **115. Tags and Metadata**

  * Estimator capabilities
  * Metadata-aware workflows
  * Routing of auxiliary data
  * Advanced estimator composition

* **116. Custom Estimator Development**

  * Building estimators
  * Building transformers
  * Parameter validation
  * Compatibility with model-selection tools
  * Estimator testing

Recent Scikit-learn development has continued expanding metadata-routing capabilities and array-API support, so these are useful advanced topics once the core estimator/pipeline model is well understood. ([scikit-learn Blog][5])

---

# XXX. Model Persistence and Deployment

* **117. Saving Models**

  * Model serialization
  * Pipeline serialization
  * Version compatibility
  * Security considerations

* **118. Deployment Pipeline**

  * Train
  * Validate
  * Serialize
  * Load
  * Predict

* **119. Production Preprocessing**

  * Same transformations during training and inference
  * Schema validation
  * Feature-order consistency
  * Missing-feature handling

* **120. Model Lifecycle**

  * Model versioning
  * Dataset versioning
  * Experiment tracking
  * Retraining
  * Rollback

---

# XXXI. Machine Learning Production Engineering

* **121. Reproducibility**

  * Fixed seeds
  * Environment specification
  * Dependency locking
  * Dataset versioning
  * Configuration management

* **122. Data Validation**

  * Schema checks
  * Range checks
  * Category checks
  * Missing-value checks
  * Distribution checks

* **123. Training Pipelines**

  * Data ingestion
  * Validation
  * Preprocessing
  * Training
  * Evaluation
  * Serialization

* **124. Inference**

  * Batch inference
  * Online inference
  * Latency considerations
  * Throughput
  * Error handling

* **125. Monitoring**

  * Prediction distributions
  * Input drift
  * Concept drift
  * Performance degradation
  * Calibration drift

---

# XXXII. ML System Design with Scikit-learn

* **126. Problem Definition**

  * Business objective
  * ML objective
  * Prediction unit
  * Prediction horizon
  * Success criteria

* **127. Dataset Design**

  * Training examples
  * Label generation
  * Sampling
  * Leakage prevention
  * Temporal consistency

* **128. Model Architecture**

  * Baseline
  * Candidate models
  * Ensemble options
  * Preprocessing architecture

* **129. Evaluation Architecture**

  * Offline metrics
  * Validation design
  * Test methodology
  * Error analysis

* **130. Deployment Architecture**

  * Batch
  * Online
  * Hybrid
  * Retraining schedule
  * Monitoring

---

# XXXIII. End-to-End Projects

* **131. Beginner Projects**

  * Iris classification

    * Train/test split
    * Scaling
    * Logistic regression
    * Accuracy
  * House-price regression

    * Missing values
    * Numerical features
    * Linear regression
    * RMSE

* **132. Intermediate Projects**

  * Customer churn prediction

    * Mixed feature types
    * `ColumnTransformer`
    * Classification pipeline
    * Cross-validation
    * ROC-AUC
  * Credit-risk-style classification

    * Imbalanced data
    * Class weighting
    * Precision/recall
    * Threshold analysis
  * Customer segmentation

    * Scaling
    * K-means
    * Cluster evaluation
    * Visualization

* **133. Advanced Projects**

  * Fraud/anomaly detection

    * Imbalanced classification
    * Anomaly detection
    * Precision-recall analysis
    * Threshold selection
  * Text classification system

    * TF-IDF
    * Linear model
    * Pipeline
    * Hyperparameter search
  * Recommendation-oriented clustering

    * Feature engineering
    * Dimensionality reduction
    * Clustering

* **134. Expert Projects**

  * Production-grade tabular ML system

    * Data validation
    * Feature engineering
    * `ColumnTransformer`
    * Multiple candidate models
    * Cross-validation
    * Hyperparameter optimization
    * Model inspection
    * Persistence
    * Monitoring design
  * End-to-end forecasting-style system

    * Time-aware validation
    * Feature generation
    * Regression
    * Error analysis
  * Enterprise classification platform

    * Group-aware validation
    * Imbalance handling
    * Calibration
    * Explainability
    * Deployment architecture

---

# XXXIV. Progressive Learning Sequence

## Level 1 — Python + ML Foundations

* Learn:

  * Python
  * NumPy
  * Pandas
  * Statistics
  * Basic linear algebra
  * ML terminology
* Master:

  * `X`
  * `y`
  * train/test split
  * `fit`
  * `predict`

## Level 2 — Core Scikit-learn

* Learn:

  * Estimator API
  * Preprocessing
  * Regression
  * Classification
  * Metrics
* Master:

  * `StandardScaler`
  * `OneHotEncoder`
  * `LogisticRegression`
  * `LinearRegression`
  * `DecisionTreeClassifier`
  * `RandomForestClassifier`

## Level 3 — Practical ML

* Learn:

  * Feature engineering
  * Feature selection
  * Pipelines
  * `ColumnTransformer`
  * Cross-validation
* Master:

  * Leakage-free preprocessing
  * Reproducible experiments
  * End-to-end pipelines

## Level 4 — Model Selection

* Learn:

  * Hyperparameters
  * Grid search
  * Randomized search
  * Custom metrics
  * Validation strategies
* Master:

  * `GridSearchCV`
  * `RandomizedSearchCV`
  * Stratified CV
  * Group-aware CV
  * Time-aware CV

## Level 5 — Advanced ML

* Learn:

  * Ensemble methods
  * Gradient boosting
  * Clustering
  * Dimensionality reduction
  * Anomaly detection
  * Text processing
* Master:

  * End-to-end modeling workflows
  * Model comparison
  * Advanced feature representations

## Level 6 — Interpretability and Reliability

* Learn:

  * Permutation importance
  * Partial dependence
  * Calibration
  * Error analysis
  * Imbalanced classification
* Master:

  * Understanding model behavior
  * Diagnosing failure modes
  * Selecting decision thresholds

## Level 7 — Production ML

* Learn:

  * Model persistence
  * Reproducibility
  * Validation
  * Monitoring
  * Deployment
* Master:

  * Training/inference consistency
  * Model lifecycle management
  * Production-ready pipelines

## Level 8 — Expert Scikit-learn Engineering

* Learn:

  * Custom transformers
  * Custom estimators
  * Advanced composition
  * Metadata routing
  * Computational optimization
* Master:

  * Designing reusable ML systems
  * Building estimator-compatible components
  * Engineering robust, maintainable pipelines

---

# XXXV. What to Master at Each Stage

* **Beginner**

  * Understand:

    * What features and targets are
    * How estimators work
    * How `fit()` and `predict()` work
  * Build:

    * Simple regression models
    * Simple classification models

* **Intermediate**

  * Understand:

    * Preprocessing
    * Cross-validation
    * Bias/variance
    * Overfitting
  * Build:

    * Pipelines
    * Mixed-type preprocessing
    * Tuned models

* **Advanced**

  * Understand:

    * Ensembles
    * Feature selection
    * Model inspection
    * Calibration
    * Unsupervised learning
  * Build:

    * Complete modeling systems
    * Robust evaluation workflows

* **Expert**

  * Understand:

    * Validation design
    * Production constraints
    * Model lifecycle
    * Computational trade-offs
    * Estimator architecture
  * Build:

    * Reusable Scikit-learn components
    * Production-grade pipelines
    * Large-scale tabular ML systems

---

# XXXVI. Essential Scikit-learn API Map

* **Data preprocessing**

  * `sklearn.preprocessing`
  * `sklearn.impute`
  * `sklearn.feature_extraction`

* **Feature selection**

  * `sklearn.feature_selection`

* **Pipelines**

  * `sklearn.pipeline`
  * `sklearn.compose`

* **Regression**

  * `sklearn.linear_model`
  * `sklearn.tree`
  * `sklearn.ensemble`
  * `sklearn.neighbors`
  * `sklearn.svm`

* **Classification**

  * `sklearn.linear_model`
  * `sklearn.naive_bayes`
  * `sklearn.neighbors`
  * `sklearn.svm`
  * `sklearn.tree`
  * `sklearn.ensemble`

* **Clustering**

  * `sklearn.cluster`

* **Dimensionality reduction**

  * `sklearn.decomposition`
  * `sklearn.manifold`
  * `sklearn.random_projection`

* **Model evaluation**

  * `sklearn.metrics`

* **Model selection**

  * `sklearn.model_selection`

* **Inspection**

  * `sklearn.inspection`

* **Datasets**

  * `sklearn.datasets`

* **Model persistence**

  * Scikit-learn persistence mechanisms and compatible serialization workflows

---

# XXXVII. Final Scikit-learn Mastery Map

* **Foundations**

  * Python
  * NumPy
  * Pandas
  * Statistics
  * Linear algebra

* **Core Scikit-learn**

  * Estimators
  * Transformers
  * Predictors
  * `fit`
  * `predict`
  * `transform`

* **Data Preparation**

  * Imputation
  * Scaling
  * Encoding
  * Feature engineering
  * Feature selection

* **Core Machine Learning**

  * Regression
  * Classification
  * Trees
  * Nearest neighbors
  * SVM
  * Naive Bayes

* **Advanced Machine Learning**

  * Random forests
  * Gradient boosting
  * Ensembles
  * Clustering
  * Dimensionality reduction
  * Anomaly detection
  * Semi-supervised learning

* **ML Evaluation**

  * Metrics
  * Cross-validation
  * Hyperparameter tuning
  * Threshold selection
  * Calibration

* **ML Engineering**

  * Pipelines
  * `ColumnTransformer`
  * Custom transformers
  * Reproducibility
  * Persistence

* **ML Interpretability**

  * Feature importance
  * Permutation importance
  * Partial dependence
  * Error analysis

* **Production**

  * Validation
  * Deployment
  * Monitoring
  * Drift
  * Retraining
  * Lifecycle management

* **Expert Mastery**

  * Estimator architecture
  * Metadata routing
  * Advanced composition
  * Computational optimization
  * Production ML system design

### The overall progression

**Python → NumPy → Pandas → Statistics/Linear Algebra → ML Fundamentals → Scikit-learn Estimator API → Preprocessing → Feature Engineering → Pipelines → Regression → Classification → Tree Ensembles → Cross-Validation → Hyperparameter Tuning → Imbalanced Learning → Calibration → Clustering → Dimensionality Reduction → Anomaly Detection → Text ML → Model Inspection → Custom Transformers → Model Persistence → Production ML → Advanced Scikit-learn Engineering.**

This sequence aligns closely with Scikit-learn's current organization around supervised/unsupervised learning, preprocessing and feature transformations, model selection/evaluation, inspection, and production-oriented utilities. ([Scikit-learn][1])

[1]: https://scikit-learn.org/?utm_source=chatgpt.com "scikit-learn: machine learning in Python — scikit-learn 1.9.1 documentation"
[2]: https://scikit-learn.org/stable/data_transforms.html?utm_source=chatgpt.com "8. Dataset transformations — scikit-learn 1.9.1 documentation"
[3]: https://scikit-learn.org/1.4/model_selection.html?utm_source=chatgpt.com "3. Model selection and evaluation — scikit-learn 1.4.2 documentation"
[4]: https://scikit-learn.org/1.0/user_guide.html?utm_source=chatgpt.com "User guide: contents — scikit-learn 1.0.2 documentation"
[5]: https://blog.scikit-learn.org/year/2026?utm_source=chatgpt.com "2026 - scikit-learn Blog"
