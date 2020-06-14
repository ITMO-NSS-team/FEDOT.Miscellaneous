## How to embed custom model into the Fedot pipeline

Fedot operates with a lot of self-defined types of models, tasks and some more have been realised by the means of enum Python module. Fedot uses **model types** described by enumerations to operate with integrated models. Here are some simple steps to extend Fedot with a custom model.

First of all, you need to 'register' your model type in a repository placed at core/repository/model_types_repository.py by adding a line with a model name in *ModelTypesIdsEnum* class as follows:

```python
class ModelTypesIdsEnum(Enum):
    xgboost = 'xgboost',
    knn = 'knn',
    logit = 'logit',
    . . .
```
At the same script file add your model type to the corresponding model group of *ModelTypesRepository* _initialise_tree method. For example, if your model can solve the classification task, add the model type to the corresponding model group initialisation or initialise your own (with is_initial=False, is_secondary=False):

```python
self._initialise_models_group(models=[ModelTypesIdsEnum.rf,
                                      ModelTypesIdsEnum.dt,
                                      ModelTypesIdsEnum.mlp,
                                      ModelTypesIdsEnum.lda,
                                      ModelTypesIdsEnum.qda,
                                      ModelTypesIdsEnum.logit,
                                      ModelTypesIdsEnum.knn,
                                      ModelTypesIdsEnum.xgboost],
                              task_type=[MachineLearningTasksEnum.classification],
                              parent=ml)
```
! Notice the *parent* parameter values are available in *ModelGroupsIdsEnum* class. You may also check for available task types in *MachineLearningTasksEnum* class described in core/repository/task_type.py.

Every model in Fedot has quite a familiar interface with *fit* and *predict* methods. Any model uses what is called a ‘strategy pattern’ for the model evaluation. In other words, it has _eval_strategy parameter defined by the model type (passed to the Model class constructor) and the task type. Check for it at core/models/model.py

```python
self._eval_strategy = _eval_strategy_for_task(self.model_type, task)
```
From that point, you need to define an evaluation strategy for your case in core/models/evaluation/evaluation.py. Follow these steps to succeed:

1. Import the model you want to use. For example:

```python
from sklearn.linear_model import LogisticRegression as SklearnLogReg
```

2. Make a class with CustomNameTaskNameStrategy and inherit it from *EvaluationStrategy*. Here is an example of how *SkLearnEvaluationStrategy* is defined:

```python
class SkLearnEvaluationStrategy(EvaluationStrategy):
    __model_by_types = {
        ModelTypesIdsEnum.xgboost: XGBClassifier,
        ModelTypesIdsEnum.logit: SklearnLogReg,
        ModelTypesIdsEnum.knn: SklearnKNN,
    }
    def fit(self, train_data: InputData):
        sklearn_model = self._sklearn_model_impl()
        sklearn_model.fit(train_data.features,
        train_data.target.ravel())
        return sklearn_model

    def predict(self, trained_model, predict_data: InputData) -> OutputData:
        raise NotImplementedError()


class SkLearnClassificationStrategy(SkLearnEvaluationStrategy):
    def predict(self, trained_model, predict_data: InputData):
        prediction = trained_model.predict_proba(predict_data.features)[:, 1]
        return prediction
```
! Notice that the sklearn prediction strategy is defined separately since the classification and regression prediction has some differences. If your strategy satisfies the skalern strategy requirements, you can just add its type to __model_by_types as shown above.

Hint: If the ‘model’ you have implemented is not a class with fit/predict methods, see the realisation of *StatsModelsAutoRegressionStrategy* uses kind of functional programming. Set aside fit_tuned method definition for that moment with raising NotImplementedError().

Finish line. Some more steps to allow Fedot using your model.

Once the Model gets the information of the task_type (integrated parameter into InputData) it needs to decide what strategy to use.

1. Go back to core/models/model.py and add your strategy to corresponding task list in _eval_strategy_for_task function:
strategies_for_tasks = {
    MachineLearningTasksEnum.classification: [SkLearnClassificationStrategy, AutoMLEvaluationStrategy],
}

```python
strategies_for_tasks = {
    MachineLearningTasksEnum.classification: [SkLearnClassificationStrategy, AutoMLEvaluationStrategy],
}
```

2. Add your model_type to the corresponding strategy list at the same place.

```python
models_for_strategies = {
    SkLearnClassificationStrategy: [ModelTypesIdsEnum.xgboost, ModelTypesIdsEnum.knn, ModelTypesIdsEnum.logit]
} 
```
