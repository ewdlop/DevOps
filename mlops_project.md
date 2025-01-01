# Project Structure
mlops_project/
├── .github/
│   └── workflows/
│       ├── ci_pipeline.yml
│       └── cd_pipeline.yml
├── src/
│   ├── data/
│   │   ├── __init__.py
│   │   ├── data_ingestion.py
│   │   └── data_validation.py
│   ├── features/
│   │   ├── __init__.py
│   │   └── feature_engineering.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── model.py
│   │   └── hyperparameter_tuning.py
│   └── monitoring/
│       ├── __init__.py
│       ├── model_monitoring.py
│       └── drift_detection.py
├── tests/
│   ├── test_data_ingestion.py
│   ├── test_feature_engineering.py
│   └── test_model.py
├── configs/
│   ├── config.yaml
│   └── model_params.json
├── notebooks/
│   └── experiment_tracking.ipynb
├── requirements.txt
└── Dockerfile

# Data Ingestion Component
from typing import Dict, Any
import pandas as pd
from great_expectations.dataset import PandasDataset

class DataIngestion:
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = self._setup_logging()

    def ingest_data(self, source_path: str) -> pd.DataFrame:
        """
        Ingest data from various sources with proper error handling
        and logging
        """
        try:
            if source_path.endswith('.csv'):
                data = pd.read_csv(source_path)
            elif source_path.endswith('.parquet'):
                data = pd.read_parquet(source_path)
            else:
                raise ValueError(f"Unsupported file format: {source_path}")
            
            self.logger.info(f"Successfully ingested data from {source_path}")
            return data
        except Exception as e:
            self.logger.error(f"Error ingesting data: {str(e)}")
            raise

    def validate_data(self, data: pd.DataFrame) -> bool:
        """
        Validate data quality using Great Expectations
        """
        ge_dataset = PandasDataset(data)
        
        # Define expectations
        expectations = [
            ge_dataset.expect_column_values_to_not_be_null("customer_id"),
            ge_dataset.expect_column_values_to_be_between("age", 18, 100),
            ge_dataset.expect_column_values_to_be_in_set("subscription_type", 
                ["basic", "premium", "enterprise"])
        ]
        
        return all(exp["success"] for exp in expectations)

# Feature Engineering Pipeline
import numpy as np
from sklearn.preprocessing import StandardScaler

class FeatureEngineering:
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.scaler = StandardScaler()

    def create_features(self, data: pd.DataFrame) -> pd.DataFrame:
        """
        Create features with versioning and tracking
        """
        features = data.copy()
        
        # Feature engineering steps
        features['account_age_days'] = (
            pd.to_datetime('now') - pd.to_datetime(features['account_created'])
        ).dt.days
        
        features['total_spend'] = features['monthly_spend'] * features['tenure_months']
        
        # Create interaction features
        features['spend_per_service'] = (
            features['total_spend'] / features['num_services']
        ).fillna(0)

        return features

    def preprocess_features(self, features: pd.DataFrame) -> pd.DataFrame:
        """
        Preprocess features with proper scaling and encoding
        """
        numeric_columns = features.select_dtypes(include=[np.number]).columns
        features[numeric_columns] = self.scaler.fit_transform(features[numeric_columns])
        
        # One-hot encode categorical variables
        categorical_columns = ['subscription_type', 'customer_segment']
        features = pd.get_dummies(features, columns=categorical_columns)
        
        return features

# Model Training and Deployment
from sklearn.ensemble import RandomForestClassifier
import mlflow
import joblib

class ChurnModel:
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.model = RandomForestClassifier(**config['model_params'])
        mlflow.set_tracking_uri(config['mlflow_tracking_uri'])

    def train(self, X_train: pd.DataFrame, y_train: pd.Series):
        """
        Train model with experiment tracking
        """
        with mlflow.start_run():
            # Log parameters
            mlflow.log_params(self.config['model_params'])
            
            # Train model
            self.model.fit(X_train, y_train)
            
            # Log metrics
            train_score = self.model.score(X_train, y_train)
            mlflow.log_metric("train_score", train_score)
            
            # Save model
            mlflow.sklearn.log_model(self.model, "model")

    def predict(self, X: pd.DataFrame) -> np.ndarray:
        """
        Make predictions with monitoring
        """
        predictions = self.model.predict_proba(X)
        
        # Log prediction metrics
        self._log_prediction_metrics(X, predictions)
        
        return predictions

    def _log_prediction_metrics(self, X: pd.DataFrame, predictions: np.ndarray):
        """
        Log metrics for monitoring prediction quality
        """
        prediction_time = pd.Timestamp.now()
        
        metrics = {
            "prediction_count": len(predictions),
            "average_confidence": np.mean(np.max(predictions, axis=1)),
            "timestamp": prediction_time
        }
        
        # Log to monitoring system
        self._log_to_monitoring_system(metrics)

# Model Monitoring
from typing import List
import numpy as np
from scipy import stats

class ModelMonitoring:
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.baseline_statistics = None

    def calculate_drift(self, 
                       current_data: pd.DataFrame, 
                       reference_data: pd.DataFrame) -> Dict[str, float]:
        """
        Calculate statistical drift between current and reference data
        """
        drift_metrics = {}
        
        for column in current_data.columns:
            if current_data[column].dtype in [np.number]:
                # KS test for numerical features
                ks_statistic, p_value = stats.ks_2samp(
                    current_data[column],
                    reference_data[column]
                )
                drift_metrics[f"{column}_ks_statistic"] = ks_statistic
                drift_metrics[f"{column}_p_value"] = p_value
            else:
                # Chi-square test for categorical features
                chi2, p_value = stats.chi2_contingency(
                    pd.crosstab(current_data[column], reference_data[column])
                )[:2]
                drift_metrics[f"{column}_chi2"] = chi2
                drift_metrics[f"{column}_p_value"] = p_value
        
        return drift_metrics

    def check_performance_degradation(self, 
                                    current_metrics: Dict[str, float],
                                    threshold: float = 0.05) -> bool:
        """
        Check if model performance has degraded
        """
        if self.baseline_statistics is None:
            return False
            
        degradation_detected = False
        
        for metric_name, current_value in current_metrics.items():
            baseline_value = self.baseline_statistics.get(metric_name)
            if baseline_value is not None:
                relative_change = abs(current_value - baseline_value) / baseline_value
                if relative_change > threshold:
                    degradation_detected = True
                    break
        
        return degradation_detected

# CI/CD Pipeline Configuration (GitHub Actions)
name: MLOps Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.8'
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Run tests
      run: |
        pytest tests/

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'
      run: |
        echo "Deploying to production"
        # Add deployment steps here

# Docker Configuration
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "src/main.py"]
