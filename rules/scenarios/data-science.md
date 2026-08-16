# Data Science AI 编程规则
> Last updated: 2026 | Targets: Python 3.12+ / pandas 2.x / scikit-learn 1.5+

## 核心原则

- 使用 Jupyter Notebook 进行探索性分析
- 编写可复现的代码
- 文档化分析流程
- 使用版本控制管理代码和数据
- 遵循数据隐私和安全规范

## 代码风格

### 项目结构
```
project/
├── data/                   # 数据目录
│   ├── raw/               # 原始数据（只读）
│   ├── processed/         # 处理后的数据
│   └── external/          # 外部数据
├── notebooks/             # Jupyter Notebooks
│   ├── 01_exploration.ipynb
│   ├── 02_preprocessing.ipynb
│   └── 03_modeling.ipynb
├── src/                   # 源代码
│   ├── data/             # 数据处理
│   ├── features/         # 特征工程
│   ├── models/           # 模型
│   ├── visualization/    # 可视化
│   └── utils/            # 工具函数
├── models/                # 训练好的模型
├── reports/               # 分析报告
│   └── figures/          # 图表
├── tests/                 # 测试
├── requirements.txt       # 依赖
└── Makefile              # 自动化脚本
```

### 命名规范
- 文件名：`snake_case`：`data_preprocessing.py`, `train_model.py`
- 函数名：`snake_case`：`load_data`, `train_model`
- 类名：`PascalCase`：`DataLoader`, `ModelTrainer`
- 常量：`UPPER_SNAKE_CASE`：`RANDOM_STATE`, `TEST_SIZE`
- Notebook：数字前缀：`01_exploration.ipynb`

## 最佳实践

### 数据加载

```python
# ✅ 使用 pandas 进行数据加载
import pandas as pd
from pathlib import Path

def load_data(file_path: str, **kwargs) -> pd.DataFrame:
    """
    加载数据文件

    Args:
        file_path: 文件路径
        **kwargs: 传递给 pd.read_csv 的参数

    Returns:
        DataFrame
    """
    path = Path(file_path)

    if path.suffix == '.csv':
        return pd.read_csv(path, **kwargs)
    elif path.suffix == '.parquet':
        return pd.read_parquet(path, **kwargs)
    elif path.suffix == '.json':
        return pd.read_json(path, **kwargs)
    else:
        raise ValueError(f"Unsupported file format: {path.suffix}")

# ✅ 使用类型提示
from typing import Optional
import numpy as np

def preprocess_data(
    df: pd.DataFrame,
    numeric_columns: list[str],
    categorical_columns: list[str],
    target_column: str,
    test_size: float = 0.2,
    random_state: int = 42,
) -> tuple[pd.DataFrame, pd.DataFrame, pd.Series, pd.Series]:
    """
    预处理数据

    Returns:
        X_train, X_test, y_train, y_test
    """
    from sklearn.model_selection import train_test_split

    X = df[numeric_columns + categorical_columns]
    y = df[target_column]

    return train_test_split(
        X, y,
        test_size=test_size,
        random_state=random_state,
        stratify=y,
    )
```

### 特征工程

```python
# ✅ 使用 sklearn Pipeline
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer

def create_preprocessing_pipeline(
    numeric_features: list[str],
    categorical_features: list[str],
) -> ColumnTransformer:
    """创建预处理管道"""

    numeric_transformer = Pipeline(steps=[
        ('imputer', SimpleImputer(strategy='median')),
        ('scaler', StandardScaler()),
    ])

    categorical_transformer = Pipeline(steps=[
        ('imputer', SimpleImputer(strategy='most_frequent')),
        ('encoder', OneHotEncoder(handle_unknown='ignore', sparse_output=False)),
    ])

    preprocessor = ColumnTransformer(
        transformers=[
            ('num', numeric_transformer, numeric_features),
            ('cat', categorical_transformer, categorical_features),
        ],
        remainder='drop',
    )

    return preprocessor

# ✅ 自定义特征转换器
from sklearn.base import BaseEstimator, TransformerMixin

class DateTimeFeaturesExtractor(BaseEstimator, TransformerMixin):
    """从日期时间列提取特征"""

    def __init__(self, datetime_column: str):
        self.datetime_column = datetime_column

    def fit(self, X, y=None):
        return self

    def transform(self, X):
        X = X.copy()
        dt = pd.to_datetime(X[self.datetime_column])

        X[f'{self.datetime_column}_year'] = dt.dt.year
        X[f'{self.datetime_column}_month'] = dt.dt.month
        X[f'{self.datetime_column}_day'] = dt.dt.day
        X[f'{self.datetime_column}_dayofweek'] = dt.dt.dayofweek
        X[f'{self.datetime_column}_hour'] = dt.dt.hour

        X = X.drop(columns=[self.datetime_column])

        return X
```

### 模型训练

```python
# ✅ 使用 sklearn 进行模型训练
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score, GridSearchCV
from sklearn.metrics import (
    accuracy_score,
    precision_score,
    recall_score,
    f1_score,
    roc_auc_score,
    classification_report,
)

class ModelTrainer:
    """模型训练器"""

    def __init__(self, random_state: int = 42):
        self.random_state = random_state
        self.models = {
            'logistic_regression': LogisticRegression(
                random_state=random_state,
                max_iter=1000,
            ),
            'random_forest': RandomForestClassifier(
                random_state=random_state,
                n_estimators=100,
            ),
            'gradient_boosting': GradientBoostingClassifier(
                random_state=random_state,
            ),
        }
        self.best_model = None
        self.best_model_name = None

    def train_and_evaluate(
        self,
        X_train: np.ndarray,
        y_train: np.ndarray,
        X_test: np.ndarray,
        y_test: np.ndarray,
    ) -> dict[str, dict[str, float]]:
        """训练并评估多个模型"""

        results = {}

        for name, model in self.models.items():
            # 交叉验证
            cv_scores = cross_val_score(
                model, X_train, y_train,
                cv=5,
                scoring='roc_auc',
            )

            # 训练模型
            model.fit(X_train, y_train)

            # 预测
            y_pred = model.predict(X_test)
            y_pred_proba = model.predict_proba(X_test)[:, 1]

            # 计算指标
            results[name] = {
                'cv_mean': cv_scores.mean(),
                'cv_std': cv_scores.std(),
                'accuracy': accuracy_score(y_test, y_pred),
                'precision': precision_score(y_test, y_pred),
                'recall': recall_score(y_test, y_pred),
                'f1': f1_score(y_test, y_pred),
                'roc_auc': roc_auc_score(y_test, y_pred_proba),
            }

            # 更新最佳模型
            if (
                self.best_model is None
                or results[name]['roc_auc'] > results[self.best_model_name]['roc_auc']
            ):
                self.best_model = model
                self.best_model_name = name

        return results

    def hyperparameter_tuning(
        self,
        model_name: str,
        X_train: np.ndarray,
        y_train: np.ndarray,
        param_grid: dict,
    ) -> GridSearchCV:
        """超参数调优"""

        model = self.models[model_name]

        grid_search = GridSearchCV(
            estimator=model,
            param_grid=param_grid,
            cv=5,
            scoring='roc_auc',
            n_jobs=-1,
            verbose=1,
        )

        grid_search.fit(X_train, y_train)

        return grid_search
```

### 可视化

```python
# ✅ 使用 matplotlib 和 seaborn
import matplotlib.pyplot as plt
import seaborn as sns
from typing import Optional

def plot_feature_importance(
    feature_importance: pd.DataFrame,
    top_n: int = 20,
    figsize: tuple[int, int] = (10, 8),
    save_path: Optional[str] = None,
) -> None:
    """绘制特征重要性图"""

    plt.figure(figsize=figsize)

    # 选择 top N 特征
    top_features = feature_importance.head(top_n)

    # 绘制水平条形图
    sns.barplot(
        data=top_features,
        x='importance',
        y='feature',
        palette='viridis',
    )

    plt.title(f'Top {top_n} Feature Importance', fontsize=14)
    plt.xlabel('Importance', fontsize=12)
    plt.ylabel('Feature', fontsize=12)
    plt.tight_layout()

    if save_path:
        plt.savefig(save_path, dpi=300, bbox_inches='tight')

    plt.show()

def plot_confusion_matrix(
    y_true: np.ndarray,
    y_pred: np.ndarray,
    labels: list[str],
    figsize: tuple[int, int] = (8, 6),
    save_path: Optional[str] = None,
) -> None:
    """绘制混淆矩阵"""

    from sklearn.metrics import confusion_matrix

    cm = confusion_matrix(y_true, y_pred)

    plt.figure(figsize=figsize)
    sns.heatmap(
        cm,
        annot=True,
        fmt='d',
        cmap='Blues',
        xticklabels=labels,
        yticklabels=labels,
    )

    plt.title('Confusion Matrix', fontsize=14)
    plt.xlabel('Predicted', fontsize=12)
    plt.ylabel('Actual', fontsize=12)
    plt.tight_layout()

    if save_path:
        plt.savefig(save_path, dpi=300, bbox_inches='tight')

    plt.show()

# ✅ 使用 plotly 进行交互式可视化
import plotly.express as px
import plotly.graph_objects as go

def plot_interactive_scatter(
    df: pd.DataFrame,
    x: str,
    y: str,
    color: Optional[str] = None,
    title: str = 'Interactive Scatter Plot',
) -> go.Figure:
    """绘制交互式散点图"""

    fig = px.scatter(
        df,
        x=x,
        y=y,
        color=color,
        title=title,
        hover_data=df.columns,
    )

    fig.update_layout(
        hovermode='closest',
        template='plotly_white',
    )

    return fig
```

### 实验追踪

```python
# ✅ 使用 MLflow 进行实验追踪
import mlflow
import mlflow.sklearn
from mlflow.models import infer_signature

class ExperimentTracker:
    """实验追踪器"""

    def __init__(self, experiment_name: str):
        mlflow.set_experiment(experiment_name)

    def log_experiment(
        self,
        model,
        params: dict,
        metrics: dict,
        X_train: np.ndarray,
        y_train: np.ndarray,
        model_name: str,
    ):
        """记录实验"""

        with mlflow.start_run(run_name=model_name):
            # 记录参数
            mlflow.log_params(params)

            # 记录指标
            mlflow.log_metrics(metrics)

            # 推断签名
            signature = infer_signature(X_train, y_train)

            # 记录模型
            mlflow.sklearn.log_model(
                model,
                artifact_path='model',
                signature=signature,
                registered_model_name=model_name,
            )

            # 记录特征重要性
            if hasattr(model, 'feature_importances_'):
                importance_df = pd.DataFrame({
                    'feature': range(len(model.feature_importances_)),
                    'importance': model.feature_importances_,
                }).sort_values('importance', ascending=False)

                importance_df.to_csv('feature_importance.csv', index=False)
                mlflow.log_artifact('feature_importance.csv')
```

### 数据验证

```python
# ✅ 使用 Great Expectations 进行数据验证
import great_expectations as gx

def validate_data(df: pd.DataFrame) -> bool:
    """验证数据质量"""

    context = gx.get_context()

    # 定义期望
    expectations = [
        gx.expectations.ExpectColumnValuesToNotBeNull(column='user_id'),
        gx.expectations.ExpectColumnValuesToBeUnique(column='user_id'),
        gx.expectations.ExpectColumnValuesToBeBetween(
            column='age',
            min_value=0,
            max_value=120,
        ),
        gx.expectations.ExpectColumnValuesToBeInSet(
            column='status',
            value_set=['active', 'inactive', 'pending'],
        ),
    ]

    # 验证数据
    validator = context.sources.pandas_default.read_dataframe(df)
    results = validator.validate(expectations)

    return results.success
```

## 测试

```python
# ✅ 数据处理函数测试
import pytest
import pandas as pd
import numpy as np

@pytest.fixture
def sample_data():
    """创建测试数据"""
    return pd.DataFrame({
        'user_id': [1, 2, 3, 4, 5],
        'age': [25, 30, 35, 40, 45],
        'income': [50000, 60000, 70000, 80000, 90000],
        'purchased': [0, 1, 0, 1, 1],
    })

def test_data_shape(sample_data):
    """测试数据形状"""
    assert sample_data.shape == (5, 4)

def test_data_types(sample_data):
    """测试数据类型"""
    assert sample_data['user_id'].dtype == np.int64
    assert sample_data['age'].dtype == np.int64

def test_no_missing_values(sample_data):
    """测试无缺失值"""
    assert sample_data.isnull().sum().sum() == 0

def test_data_range(sample_data):
    """测试数据范围"""
    assert sample_data['age'].between(0, 120).all()
    assert sample_data['income'].gt(0).all()
```

## 常见陷阱

### ❌ 避免
```python
# ❌ 硬编码随机种子
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

# ❌ 数据泄露
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)  # 在整个数据集上 fit
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y)

# ❌ 不处理缺失值
model.fit(X_train, y_train)  # 如果有 NaN 会报错
```

### ✅ 推荐
```python
# ✅ 使用常量定义随机种子
RANDOM_STATE = 42
X_train, X_test, y_train, y_test = train_test_split(
    X, y, random_state=RANDOM_STATE
)

# ✅ 在训练集上 fit，在测试集上 transform
X_train, X_test, y_train, y_test = train_test_split(X, y)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)  # 只在训练集上 fit
X_test_scaled = scaler.transform(X_test)  # 在测试集上 transform

# ✅ 使用 Pipeline 避免数据泄露
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression()),
])
pipeline.fit(X_train, y_train)
```

## 依赖推荐

- **数据处理**: pandas, numpy, polars
- **机器学习**: scikit-learn, xgboost, lightgbm
- **深度学习**: PyTorch, TensorFlow
- **可视化**: matplotlib, seaborn, plotly
- **实验追踪**: MLflow, Weights & Biases
- **数据验证**: Great Expectations, pandera
- **特征工程**: feature-engine, category_encoders
- **AutoML**: Auto-sklearn, TPOT

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- Python 版本：
- 主要框架：
- 数据源：
- 部署方式：
```
