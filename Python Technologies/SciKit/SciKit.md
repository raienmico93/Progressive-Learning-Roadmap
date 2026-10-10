# Scikit-learn Comprehensive, Structured, and Progressive Learning Roadmap

## From Machine Learning Foundations to Advanced Modeling, Pipelines, Model Selection, and Production ML Engineering

Scikit-learn is best learned as more than "a library for machine learning models." The progression should cover **Python prerequisites → NumPy/Pandas prerequisites → ML fundamentals → estimators → data preprocessing → feature engineering → supervised learning → unsupervised learning → model evaluation → model selection → pipelines → ensembles → dimensionality reduction → text processing → time series → model persistence → deployment → production ML engineering**.

---

# I. Scikit-learn Foundations

- **1. What Scikit-learn Is**
  - Scikit-learn
  - Scikit-learn history
  - David Cournapeau
  - Google Summer of Code
  - Scikit-learn 0.x
  - Scikit-learn 1.0
  - Scikit-learn 1.3
  - Scikit-learn 1.4
  - Scikit-learn 1.5
  - Scikit-learn 1.6 (current)
  - Scikit-learn philosophy
    - Simple and efficient
    - Built on NumPy, SciPy, Matplotlib
    - Open source
    - Consistent API
    - Well documented
    - Production-ready
  - Scikit-learn vs TensorFlow
  - Scikit-learn vs PyTorch
  - Scikit-learn vs XGBoost
  - Scikit-learn vs LightGBM
  - Scikit-learn vs CatBoost
  - Scikit-learn vs statsmodels
  - Scikit-learn use cases
    - Classification
    - Regression
    - Clustering
    - Dimensionality reduction
    - Model selection
    - Preprocessing
    - Feature extraction
    - Pipeline construction
    - Anomaly detection
  - Scikit-learn in modern ML
  - Scikit-learn ecosystem
  - Scikit-learn API
  - Scikit-learn best practices

- **2. Prerequisites**
  - Python fundamentals
  - Variables
  - Data types
  - Control flow
  - Functions
  - Classes
  - Modules
  - NumPy
    - ndarray
    - Indexing
    - Slicing
    - Broadcasting
    - Universal functions
    - Aggregation
    - Linear algebra
    - Random
  - Pandas
    - Series
    - DataFrame
    - Index
    - Selection
    - Cleaning
    - Transformation
    - Grouping
    - Merging
    - Reshaping
  - Matplotlib
    - Plotting
    - Subplots
    - Styling
  - SciPy
    - Linear algebra
    - Statistics
    - Optimization
  - Jupyter
    - Notebooks
    - Cells
    - Markdown
  - Machine learning concepts
  - Prerequisite best practices

- **3. Machine Learning Foundations**
  - Machine learning
  - Supervised learning
  - Unsupervised learning
  - Semi-supervised learning
  - Reinforcement learning
  - Classification
  - Regression
  - Clustering
  - Dimensionality reduction
  - Anomaly detection
  - Features
  - Labels
  - Targets
  - Training data
  - Validation data
  - Test data
  - Overfitting
  - Underfitting
  - Bias-variance tradeoff
  - Regularization
  - Cross-validation
  - Model evaluation
  - Model selection
  - Hyperparameter tuning
  - ML workflow
  - ML best practices

- **4. Installing Scikit-learn**
  - Installation
    - pip
    - conda
    - mamba
    - uv
    - Poetry
  - `pip install scikit-learn`
  - `conda install scikit-learn`
  - Version checking
  - `sklearn.__version__`
  - Dependencies
    - NumPy
    - SciPy
    - joblib
    - threadpoolctl
  - Optional dependencies
    - Matplotlib
    - Pandas
    - Seaborn
    - Plotly
  - Pre-built wheels
  - Platform-specific installation
  - Installation best practices

- **5. Importing Scikit-learn**
  - `import sklearn`
  - Subpackage imports
    - `from sklearn import ...`
    - `from sklearn.linear_model import ...`
    - `from sklearn.tree import ...`
    - `from sklearn.ensemble import ...`
    - `from sklearn.cluster import ...`
    - `from sklearn.preprocessing import ...`
    - `from sklearn.model_selection import ...`
    - `from sklearn.metrics import ...`
    - `from sklearn.pipeline import ...`
    - `from sklearn.decomposition import ...`
    - `from sklearn.feature_selection import ...`
    - `from sklearn.feature_extraction import ...`
    - `from sklearn.svm import ...`
    - `from sklearn.neighbors import ...`
    - `from sklearn.naive_bayes import ...`
    - `from sklearn.neural_network import ...`
  - Import best practices
  - Namespace conventions

- **6. Scikit-learn API**
  - Estimator API
  - `fit()`
  - `predict()`
  - `transform()`
  - `fit_transform()`
  - `predict_proba()`
  - `predict_log_proba()`
  - `decision_function()`
  - `score()`
  - `get_params()`
  - `set_params()`
  - Estimator types
    - Estimators
    - Transformers
    - Predictors
    - Meta-estimators
  - Consistency
  - API best practices

- **7. First Scikit-learn Model**
  - Dataset loading
  - Train-test split
  - Model creation
  - Model training
  - Model prediction
  - Model evaluation
  - First model best practices

---

# II. Data Preprocessing

- **8. Preprocessing Fundamentals**
  - Preprocessing
  - Data cleaning
  - Data transformation
  - Data scaling
  - Data encoding
  - Preprocessing best practices

- **9. Scaling**
  - Standardization
    - `StandardScaler`
  - Normalization
    - `MinMaxScaler`
    - `MaxAbsScaler`
  - Robust scaling
    - `RobustScaler`
  - Normalizer
    - `Normalizer`
  - Scaling best practices

- **10. Encoding Categorical Variables**
  - Label encoding
    - `LabelEncoder`
  - Ordinal encoding
    - `OrdinalEncoder`
  - One-hot encoding
    - `OneHotEncoder`
  - Target encoding
  - Binary encoding
  - Frequency encoding
  - Encoding best practices

- **11. Discretization**
  - Discretization
  - `KBinsDiscretizer`
  - Binning strategies
    - `uniform`
    - `quantile`
    - `kmeans`
  - Discretization best practices

- **12. Imputation**
  - Missing data
  - `SimpleImputer`
  - Imputation strategies
    - `mean`
    - `median`
    - `most_frequent`
    - `constant`
  - `KNNImputer`
  - `IterativeImputer`
  - Imputation best practices

- **13. Polynomial Features**
  - Polynomial features
  - `PolynomialFeatures`
  - Degree
  - Interaction terms
  - Polynomial feature best practices

- **14. Custom Transformers**
  - Custom transformers
  - `FunctionTransformer`
  - `TransformerMixin`
  - Custom transformer best practices

---

# III. Feature Engineering and Selection

- **15. Feature Engineering Fundamentals**
  - Feature engineering
  - Feature creation
  - Feature transformation
  - Feature interaction
  - Feature extraction
  - Feature engineering best practices

- **16. Feature Selection**
  - Feature selection
  - Filter methods
    - `VarianceThreshold`
    - `SelectKBest`
    - `SelectPercentile`
    - `SelectFpr`
    - `SelectFdr`
    - `SelectFwe`
    - `GenericUnivariateSelect`
  - Wrapper methods
    - `RFE`
    - `RFECV`
    - `SequentialFeatureSelector`
  - Embedded methods
    - `SelectFromModel`
  - Feature selection best practices

- **17. Feature Extraction**
  - Feature extraction
  - `PCA`
  - `TruncatedSVD`
  - `NMF`
  - `FastICA`
  - `FactorAnalysis`
  - `DictionaryLearning`
  - `LatentDirichletAllocation`
  - `MiniBatchDictionaryLearning`
  - `SparsePCA`
  - `MiniBatchSparsePCA`
  - `KernelPCA`
  - `IncrementalPCA`
  - Feature extraction best practices

- **18. Feature Importance**
  - Feature importance
  - Tree-based importance
  - Permutation importance
  - `permutation_importance`
  - SHAP
  - Feature importance best practices

---

# IV. Supervised Learning

- **19. Supervised Learning Fundamentals**
  - Supervised learning
  - Classification
  - Regression
  - Supervised learning best practices

- **20. Linear Models**
  - Linear regression
    - `LinearRegression`
  - Ridge regression
    - `Ridge`
    - `RidgeCV`
  - Lasso regression
    - `Lasso`
    - `LassoCV`
    - `LassoLars`
    - `LassoLarsCV`
    - `LassoLarsIC`
  - Elastic Net
    - `ElasticNet`
    - `ElasticNetCV`
  - Orthogonal matching pursuit
    - `OrthogonalMatchingPursuit`
    - `OrthogonalMatchingPursuitCV`
  - Bayesian regression
    - `BayesianRidge`
    - `ARDRegression`
  - Logistic regression
    - `LogisticRegression`
    - `LogisticRegressionCV`
  - Perceptron
    - `Perceptron`
  - Passive aggressive
    - `PassiveAggressiveClassifier`
    - `PassiveAggressiveRegressor`
  - SGD
    - `SGDClassifier`
    - `SGDRegressor`
  - Huber regression
    - `HuberRegressor`
  - Quantile regression
    - `QuantileRegressor`
  - RANSAC
    - `RANSACRegressor`
  - Theil-Sen
    - `TheilSenRegressor`
  - Polynomial regression
  - Linear model best practices

- **21. Support Vector Machines**
  - SVM
  - `SVC`
  - `SVR`
  - `NuSVC`
  - `NuSVR`
  - `LinearSVC`
  - `LinearSVR`
  - `OneClassSVM`
  - Kernels
    - `linear`
    - `poly`
    - `rbf`
    - `sigmoid`
    - `precomputed`
  - SVM best practices

- **22. Nearest Neighbors**
  - KNN
  - `KNeighborsClassifier`
  - `KNeighborsRegressor`
  - `RadiusNeighborsClassifier`
  - `RadiusNeighborsRegressor`
  - `NearestCentroid`
  - `NearestNeighbors`
  - `KNeighborsTransformer`
  - `RadiusNeighborsTransformer`
  - `LocalOutlierFactor`
  - KNN best practices

- **23. Naive Bayes**
  - Naive Bayes
  - `GaussianNB`
  - `MultinomialNB`
  - `ComplementNB`
  - `BernoulliNB`
  - `CategoricalNB`
  - Naive Bayes best practices

- **24. Decision Trees**
  - Decision trees
  - `DecisionTreeClassifier`
  - `DecisionTreeRegressor`
  - `ExtraTreeClassifier`
  - `ExtraTreeRegressor`
  - Tree parameters
    - `criterion`
    - `splitter`
    - `max_depth`
    - `min_samples_split`
    - `min_samples_leaf`
    - `min_weight_fraction_leaf`
    - `max_features`
    - `random_state`
    - `max_leaf_nodes`
    - `min_impurity_decrease`
    - `class_weight`
    - `ccp_alpha`
  - Decision tree best practices

- **25. Ensemble Methods**
  - Ensembles
  - Bagging
    - `BaggingClassifier`
    - `BaggingRegressor`
  - Random forests
    - `RandomForestClassifier`
    - `RandomForestRegressor`
  - Extra trees
    - `ExtraTreesClassifier`
    - `ExtraTreesRegressor`
  - Boosting
    - `AdaBoostClassifier`
    - `AdaBoostRegressor`
    - `GradientBoostingClassifier`
    - `GradientBoostingRegressor`
    - `HistGradientBoostingClassifier`
    - `HistGradientBoostingRegressor`
  - Voting
    - `VotingClassifier`
    - `VotingRegressor`
  - Stacking
    - `StackingClassifier`
    - `StackingRegressor`
  - Ensemble best practices

- **26. Neural Networks**
  - Neural networks
  - `MLPClassifier`
  - `MLPRegressor`
  - `BernoulliRBM`
  - `MLPClassifier` parameters
  - `MLPRegressor` parameters
  - Neural network best practices

- **27. Gaussian Processes**
  - Gaussian processes
  - `GaussianProcessClassifier`
  - `GaussianProcessRegressor`
  - Kernels
  - Gaussian process best practices

- **28. Linear Discriminant Analysis**
  - LDA
  - `LinearDiscriminantAnalysis`
  - `QuadraticDiscriminantAnalysis`
  - LDA best practices

---

# V. Unsupervised Learning

- **29. Unsupervised Learning Fundamentals**
  - Unsupervised learning
  - Clustering
  - Dimensionality reduction
  - Anomaly detection
  - Unsupervised learning best practices

- **30. Clustering**
  - K-Means
    - `KMeans`
    - `MiniBatchKMeans`
  - Affinity Propagation
    - `AffinityPropagation`
  - Mean Shift
    - `MeanShift`
  - Spectral Clustering
    - `SpectralClustering`
  - Hierarchical Clustering
    - `AgglomerativeClustering`
    - `FeatureAgglomeration`
  - DBSCAN
    - `DBSCAN`
  - HDBSCAN
  - OPTICS
    - `OPTICS`
  - Birch
    - `Birch`
  - Gaussian Mixture Models
    - `GaussianMixture`
    - `BayesianGaussianMixture`
  - Clustering best practices

- **31. Dimensionality Reduction**
  - PCA
    - `PCA`
    - `IncrementalPCA`
    - `KernelPCA`
    - `SparsePCA`
    - `MiniBatchSparsePCA`
  - Truncated SVD
    - `TruncatedSVD`
  - NMF
    - `NMF`
  - ICA
    - `FastICA`
  - Factor Analysis
    - `FactorAnalysis`
  - Dictionary Learning
    - `DictionaryLearning`
    - `MiniBatchDictionaryLearning`
  - LDA
    - `LatentDirichletAllocation`
  - Manifold learning
    - `TSNE`
    - `Isomap`
    - `LocallyLinearEmbedding`
    - `SpectralEmbedding`
    - `MDS`
  - Dimensionality reduction best practices

- **32. Anomaly Detection**
  - Anomaly detection
  - `IsolationForest`
  - `LocalOutlierFactor`
  - `OneClassSVM`
  - `EllipticEnvelope`
  - `Covariance`
  - Anomaly detection best practices

- **33. Novelty Detection**
  - Novelty detection
  - `OneClassSVM`
  - `LocalOutlierFactor` with `novelty=True`
  - `IsolationForest` with `novelty=True`
  - Novelty detection best practices

---

# VI. Model Evaluation

- **34. Model Evaluation Fundamentals**
  - Model evaluation
  - Evaluation metrics
  - Cross-validation
  - Holdout validation
  - Evaluation best practices

- **35. Classification Metrics**
  - Accuracy
    - `accuracy_score`
  - Precision
    - `precision_score`
  - Recall
    - `recall_score`
  - F1 score
    - `f1_score`
  - F-beta score
    - `fbeta_score`
  - ROC AUC
    - `roc_auc_score`
    - `roc_curve`
  - Precision-Recall AUC
    - `precision_recall_curve`
    - `auc`
  - Confusion matrix
    - `confusion_matrix`
    - `ConfusionMatrixDisplay`
  - Classification report
    - `classification_report`
  - Balanced accuracy
    - `balanced_accuracy_score`
  - Matthews correlation
    - `matthews_corrcoef`
  - Cohen's kappa
    - `cohen_kappa_score`
  - Jaccard similarity
    - `jaccard_score`
  - Hamming loss
    - `hamming_loss`
  - Log loss
    - `log_loss`
  - Brier score
    - `brier_score_loss`
  - Classification metric best practices

- **36. Regression Metrics**
  - Mean squared error
    - `mean_squared_error`
  - Root mean squared error
    - `root_mean_squared_error`
  - Mean absolute error
    - `mean_absolute_error`
  - Mean absolute percentage error
    - `mean_absolute_percentage_error`
  - R² score
    - `r2_score`
  - Adjusted R²
  - Explained variance
    - `explained_variance_score`
  - Median absolute error
    - `median_absolute_error`
  - Max error
    - `max_error`
  - Mean squared log error
    - `mean_squared_log_error`
  - Regression metric best practices

- **37. Clustering Metrics**
  - Adjusted Rand Index
    - `adjusted_rand_score`
  - Rand Index
    - `rand_score`
  - Adjusted Mutual Information
    - `adjusted_mutual_info_score`
  - Normalized Mutual Information
    - `normalized_mutual_info_score`
  - Mutual Information
    - `mutual_info_score`
  - Homogeneity
    - `homogeneity_score`
  - Completeness
    - `completeness_score`
  - V-measure
    - `v_measure_score`
  - Fowlkes-Mallows
    - `fowlkes_mallows_score`
  - Silhouette coefficient
    - `silhouette_score`
    - `silhouette_samples`
  - Calinski-Harabasz
    - `calinski_harabasz_score`
  - Davies-Bouldin
    - `davies_bouldin_score`
  - Clustering metric best practices

- **38. Cross-Validation**
  - Cross-validation
  - `cross_val_score`
  - `cross_validate`
  - `cross_val_predict`
  - K-Fold
    - `KFold`
  - Stratified K-Fold
    - `StratifiedKFold`
  - Group K-Fold
    - `GroupKFold`
  - Stratified Group K-Fold
    - `StratifiedGroupKFold`
  - Leave-One-Out
    - `LeaveOneOut`
  - Leave-P-Out
    - `LeavePOut`
  - Shuffle Split
    - `ShuffleSplit`
  - Stratified Shuffle Split
    - `StratifiedShuffleSplit`
  - Group Shuffle Split
    - `GroupShuffleSplit`
  - Time Series Split
    - `TimeSeriesSplit`
  - Repeated K-Fold
    - `RepeatedKFold`
  - Repeated Stratified K-Fold
    - `RepeatedStratifiedKFold`
  - Cross-validation best practices

- **39. Learning Curves**
  - Learning curves
  - `learning_curve`
  - Validation curves
  - `validation_curve`
  - Learning curve best practices

- **40. Evaluation Visualization**
  - `ConfusionMatrixDisplay`
  - `RocCurveDisplay`
  - `PrecisionRecallDisplay`
  - `DetCurveDisplay`
  - `PredictionErrorDisplay`
  - `LearningCurveDisplay`
  - `ValidationCurveDisplay`
  - Evaluation visualization best practices

---

# VII. Model Selection and Hyperparameter Tuning

- **41. Model Selection Fundamentals**
  - Model selection
  - Hyperparameter tuning
  - Grid search
  - Random search
  - Bayesian optimization
  - Model selection best practices

- **42. Grid Search**
  - `GridSearchCV`
  - Parameter grid
  - Cross-validation
  - Scoring
  - Grid search best practices

- **43. Random Search**
  - `RandomizedSearchCV`
  - Parameter distributions
  - Number of iterations
  - Random search best practices

- **44. Halving Search**
  - `HalvingGridSearchCV`
  - `HalvingRandomSearchCV`
  - Halving search best practices

- **45. Bayesian Optimization**
  - Bayesian optimization
  - `BayesSearchCV` (scikit-optimize)
  - Optuna
  - Hyperopt
  - Bayesian optimization best practices

- **46. Model Selection Utilities**
  - `cross_val_score`
  - `cross_validate`
  - `cross_val_predict`
  - `learning_curve`
  - `validation_curve`
  - `permutation_test_score`
  - Model selection utility best practices

- **47. Model Persistence**
  - Model persistence
  - `joblib.dump`
  - `joblib.load`
  - `pickle`
  - ONNX
  - Model persistence best practices

---

# VIII. Pipelines

- **48. Pipeline Fundamentals**
  - Pipeline
  - `Pipeline`
  - `make_pipeline`
  - Pipeline steps
  - Pipeline naming
  - Pipeline best practices

- **49. FeatureUnion**
  - `FeatureUnion`
  - Feature composition
  - Feature union best practices

- **50. ColumnTransformer**
  - `ColumnTransformer`
  - Column selection
  - Column transformation
  - Column transformer best practices

- **51. Pipeline with Preprocessing**
  - Preprocessing pipeline
  - Scaling
  - Encoding
  - Imputation
  - Pipeline preprocessing best practices

- **52. Pipeline with Model**
  - Model pipeline
  - Preprocessing + model
  - Pipeline model best practices

- **53. Pipeline with Cross-Validation**
  - Pipeline cross-validation
  - Grid search with pipeline
  - Pipeline cross-validation best practices

- **54. Pipeline Persistence**
  - Pipeline persistence
  - `joblib.dump`
  - `joblib.load`
  - Pipeline persistence best practices

- **55. Advanced Pipelines**
  - Nested pipelines
  - Pipeline caching
  - Pipeline memory
  - Pipeline visualization
  - `set_config(display='diagram')`
  - Advanced pipeline best practices

---

# IX. Text Processing

- **56. Text Processing Fundamentals**
  - Text processing
  - Text preprocessing
  - Tokenization
  - Vectorization
  - Text processing best practices

- **57. Count Vectorization**
  - `CountVectorizer`
  - Tokenization
  - N-grams
  - Stop words
  - Vocabulary
  - Count vectorization best practices

- **58. TF-IDF Vectorization**
  - `TfidfVectorizer`
  - TF-IDF
  - `TfidfTransformer`
  - TF-IDF best practices

- **59. Hashing Vectorization**
  - `HashingVectorizer`
  - Feature hashing
  - Hashing vectorization best practices

- **60. Text Classification**
  - Text classification
  - Pipelines with text
  - Text classification best practices

- **61. Topic Modeling**
  - Topic modeling
  - `LatentDirichletAllocation`
  - `NMF`
  - Topic modeling best practices

- **62. Text Similarity**
  - Text similarity
  - Cosine similarity
  - `cosine_similarity`
  - Text similarity best practices

---

# X. Advanced Topics

- **63. Multiclass and Multioutput**
  - Multiclass classification
  - Multioutput classification
  - Multioutput regression
  - `OneVsRestClassifier`
  - `OneVsOneClassifier`
  - `OutputCodeClassifier`
  - `MultiOutputClassifier`
  - `MultiOutputRegressor`
  - `RegressorChain`
  - `ClassifierChain`
  - Multiclass and multioutput best practices

- **64. Imbalanced Data**
  - Imbalanced data
  - Class weights
  - Resampling
  - SMOTE (imbalanced-learn)
  - ADASYN
  - Random undersampling
  - Random oversampling
  - Imbalanced data best practices

- **65. Calibration**
  - Probability calibration
  - `CalibratedClassifierCV`
  - `calibration_curve`
  - Calibration best practices

- **66. Partial Dependence**
  - Partial dependence
  - `partial_dependence`
  - `PartialDependenceDisplay`
  - `permutation_importance`
  - Partial dependence best practices

- **67. Incremental Learning**
  - Incremental learning
  - `partial_fit`
  - `MiniBatchKMeans`
  - `SGDClassifier`
  - `SGDRegressor`
  - `IncrementalPCA`
  - Incremental learning best practices

- **68. Out-of-Core Learning**
  - Out-of-core learning
  - Streaming data
  - Chunked processing
  - Out-of-core learning best practices

- **69. Random State and Reproducibility**
  - Random state
  - Reproducibility
  - `random_state`
  - `np.random.seed`
  - Reproducibility best practices

- **70. Custom Estimators**
  - Custom estimators
  - `BaseEstimator`
  - `ClassifierMixin`
  - `RegressorMixin`
  - `TransformerMixin`
  - Custom estimator best practices

- **71. Metadata Routing**
  - Metadata routing
  - `set_config(enable_metadata_routing=True)`
  - Metadata routing best practices

---

# XI. Scikit-learn Ecosystem

- **72. Scikit-learn and NumPy**
  - NumPy arrays
  - NumPy operations
  - NumPy integration best practices

- **73. Scikit-learn and Pandas**
  - Pandas DataFrames
  - Pandas Series
  - Pandas integration best practices

- **74. Scikit-learn and Matplotlib**
  - Matplotlib
  - Visualization
  - Matplotlib integration best practices

- **75. Scikit-learn and Seaborn**
  - Seaborn
  - Statistical visualization
  - Seaborn integration best practices

- **76. Scikit-learn and XGBoost**
  - XGBoost
  - XGBoost integration
  - XGBoost best practices

- **77. Scikit-learn and LightGBM**
  - LightGBM
  - LightGBM integration
  - LightGBM best practices

- **78. Scikit-learn and CatBoost**
  - CatBoost
  - CatBoost integration
  - CatBoost best practices

- **79. Scikit-learn and imbalanced-learn**
  - imbalanced-learn
  - Resampling
  - Imbalanced learn best practices

- **80. Scikit-learn and scikit-optimize**
  - scikit-optimize
  - Bayesian optimization
  - scikit-optimize best practices

- **81. Scikit-learn and Optuna**
  - Optuna
  - Hyperparameter optimization
  - Optuna best practices

- **82. Scikit-learn and MLflow**
  - MLflow
  - Experiment tracking
  - Model registry
  - MLflow best practices

- **83. Scikit-learn and FastAPI**
  - FastAPI
  - Model serving
  - FastAPI best practices

- **84. Scikit-learn and ONNX**
  - ONNX
  - Model export
  - ONNX best practices

---

# XII. Scikit-learn Projects by Difficulty

## Beginner Projects

- **1. Iris Classification**
  - Dataset loading
  - Train-test split
  - Model training
  - Model evaluation

- **2. Boston Housing Regression**
  - Dataset loading
  - Linear regression
  - Model evaluation
  - Visualization

- **3. Wine Classification**
  - Dataset loading
  - Preprocessing
  - Classification
  - Evaluation

- **4. Customer Segmentation**
  - K-Means
  - Elbow method
  - Visualization
  - Interpretation

- **5. Titanic Survival Prediction**
  - Data cleaning
  - Feature engineering
  - Classification
  - Evaluation

---

## Intermediate Projects

- **6. End-to-End ML Pipeline**
  - Data preprocessing
  - Pipeline
  - Model training
  - Cross-validation
  - Evaluation

- **7. Text Classification**
  - Text preprocessing
  - TF-IDF
  - Classification
  - Evaluation

- **8. House Price Prediction**
  - Feature engineering
  - Regression
  - Ensemble methods
  - Evaluation

- **9. Image Classification**
  - Feature extraction
  - Classification
  - Evaluation
  - Visualization

- **10. Fraud Detection**
  - Imbalanced data
  - Resampling
  - Classification
  - Evaluation

---

## Advanced Projects

- **11. Automated ML Pipeline**
  - Pipeline
  - Grid search
  - Cross-validation
  - Model selection
  - Persistence

- **12. Recommendation System**
  - Collaborative filtering
  - Content-based filtering
  - Evaluation
  - Deployment

- **13. Anomaly Detection System**
  - Anomaly detection
  - Isolation Forest
  - Evaluation
  - Visualization

- **14. Time Series Forecasting**
  - Time series
  - Feature engineering
  - Regression
  - Evaluation

- **15. Model Deployment**
  - Model training
  - Model persistence
  - FastAPI
  - Docker
  - Monitoring

---

## Expert Projects

- **16. Production ML Platform**
  - Data pipelines
  - Feature store
  - Model training
  - Model serving
  - Monitoring
  - MLOps

- **17. AutoML System**
  - Automated preprocessing
  - Automated feature engineering
  - Automated model selection
  - Automated hyperparameter tuning
  - Deployment

- **18. Real-Time ML System**
  - Streaming data
  - Incremental learning
  - Real-time predictions
  - Monitoring

- **19. Multi-Model Ensemble System**
  - Multiple models
  - Stacking
  - Voting
  - Blending
  - Deployment

- **20. End-to-End MLOps Pipeline**
  - Data versioning
  - Experiment tracking
  - Model registry
  - CI/CD
  - Monitoring
  - Governance

---

# XIII. Progressive Scikit-learn Learning Sequence

## Level 1 — Scikit-learn Fundamentals

- Master:
  - Installation
  - Import
  - API
  - First model
  - Estimators

## Level 2 — Data Preprocessing

- Master:
  - Scaling
  - Encoding
  - Discretization
  - Imputation
  - Polynomial features
  - Custom transformers

## Level 3 — Feature Engineering

- Master:
  - Feature engineering fundamentals
  - Feature selection
  - Feature extraction
  - Feature importance

## Level 4 — Supervised Learning

- Master:
  - Linear models
  - SVM
  - Nearest neighbors
  - Naive Bayes
  - Decision trees
  - Ensemble methods
  - Neural networks
  - Gaussian processes
  - LDA

## Level 5 — Unsupervised Learning

- Master:
  - Clustering
  - Dimensionality reduction
  - Anomaly detection
  - Novelty detection

## Level 6 — Model Evaluation

- Master:
  - Classification metrics
  - Regression metrics
  - Clustering metrics
  - Cross-validation
  - Learning curves
  - Evaluation visualization

## Level 7 — Model Selection

- Master:
  - Model selection fundamentals
  - Grid search
  - Random search
  - Halving search
  - Bayesian optimization
  - Model selection utilities
  - Model persistence

## Level 8 — Pipelines

- Master:
  - Pipeline fundamentals
  - FeatureUnion
  - ColumnTransformer
  - Pipeline with preprocessing
  - Pipeline with model
  - Pipeline with cross-validation
  - Pipeline persistence
  - Advanced pipelines

## Level 9 — Text Processing

- Master:
  - Text processing fundamentals
  - Count vectorization
  - TF-IDF vectorization
  - Hashing vectorization
  - Text classification
  - Topic modeling
  - Text similarity

## Level 10 — Advanced Topics

- Master:
  - Multiclass and multioutput
  - Imbalanced data
  - Calibration
  - Partial dependence
  - Incremental learning
  - Out-of-core learning
  - Random state and reproducibility
  - Custom estimators
  - Metadata routing

## Level 11 — Ecosystem

- Master:
  - NumPy
  - Pandas
  - Matplotlib
  - Seaborn
  - XGBoost
  - LightGBM
  - CatBoost
  - imbalanced-learn
  - scikit-optimize
  - Optuna
  - MLflow
  - FastAPI
  - ONNX

## Level 12 — Production Engineering

- Master:
  - Data pipelines
  - Feature engineering
  - Model training
  - Model evaluation
  - Model selection
  - Model deployment
  - Monitoring
  - MLOps
  - Production best practices

---

# XIV. Final Scikit-learn Competency Map

- **Foundations**

  - Installation
  - Import
  - API
  - Estimators
  - First model

- **Preprocessing**

  - Scaling
  - Encoding
  - Discretization
  - Imputation
  - Polynomial features
  - Custom transformers

- **Feature Engineering**

  - Feature engineering fundamentals
  - Feature selection
  - Feature extraction
  - Feature importance

- **Supervised Learning**

  - Linear models
  - SVM
  - Nearest neighbors
  - Naive Bayes
  - Decision trees
  - Ensemble methods
  - Neural networks
  - Gaussian processes
  - LDA

- **Unsupervised Learning**

  - Clustering
  - Dimensionality reduction
  - Anomaly detection
  - Novelty detection

- **Model Evaluation**

  - Classification metrics
  - Regression metrics
  - Clustering metrics
  - Cross-validation
  - Learning curves
  - Evaluation visualization

- **Model Selection**

  - Grid search
  - Random search
  - Halving search
  - Bayesian optimization
  - Model selection utilities
  - Model persistence

- **Pipelines**

  - Pipeline fundamentals
  - FeatureUnion
  - ColumnTransformer
  - Pipeline with preprocessing
  - Pipeline with model
  - Pipeline with cross-validation
  - Pipeline persistence
  - Advanced pipelines

- **Text Processing**

  - Count vectorization
  - TF-IDF vectorization
  - Hashing vectorization
  - Text classification
  - Topic modeling
  - Text similarity

- **Advanced Topics**

  - Multiclass and multioutput
  - Imbalanced data
  - Calibration
  - Partial dependence
  - Incremental learning
  - Out-of-core learning
  - Random state and reproducibility
  - Custom estimators
  - Metadata routing

- **Ecosystem**

  - NumPy
  - Pandas
  - Matplotlib
  - Seaborn
  - XGBoost
  - LightGBM
  - CatBoost
  - imbalanced-learn
  - scikit-optimize
  - Optuna
  - MLflow
  - FastAPI
  - ONNX

- **Production**

  - Data pipelines
  - Feature engineering
  - Model training
  - Model evaluation
  - Model selection
  - Model deployment
  - Monitoring
  - MLOps

---

## Recommended Overall Progression

**Scikit-learn Fundamentals → Data Preprocessing → Feature Engineering → Supervised Learning → Unsupervised Learning → Model Evaluation → Model Selection → Pipelines → Text Processing → Advanced Topics → Ecosystem → Production Engineering**
