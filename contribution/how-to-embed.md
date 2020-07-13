## How to embed custom model into the Fedot pipeline

FEDOT operates with a lot of self-defined types of models, tasks and some more realised by the means of enum Python module and simple string definition. It uses model types described by JSON format. Here are some simple steps to extend FEDOT with a custom model.

First of all, you need to ‘register’ your model type in a .json repository placed at core/repository/data. This repository has two sections: “metadata” and “model”. Place the block with the custom model name and “meta”&”tag” in the “model” section as follows:

```python
"models": {
  "logit": {
    "meta": "sklearn_class",
    "tags": ["simple", "linear", "interpretable"]
},
  "lda": {
    "meta": "sklearn_class",
    "tags": ["discriminant", "linear"]
}
```
Most of the tags are used for composite models with the defined features, e.g. use an ‘interpretable’ tag to compose model from interpretable models only.
Some of the tags are used for setting certain rules like:
- ‘without_preprocessing’ means to use the model without preprocessing,
- ‘data_model’ for pseudo-models, used for nodes properties but not the modelling itself, 
- ‘expensive’ ones do not participate in composition, so you need to request them directly
end etc.

Think about what kind of problem this custom model solves, investigate the “metadata” section and choose the appropriate strategy to put it in the “meta” section of the new custom model. 
Find the appropriate strategy or create the custom one following the certain structure:
```python
"sklearn_class": {
	  "tasks": "[TaskTypesEnum.classification]",
	  "input_type": "[DataTypesEnum.table, DataTypesEnum.table]",
	  "output_type": "[DataTypesEnum.table, DataTypesEnum.table]",
	  "accepted_node_types": ["any"],
	  "forbidden_node_types": "[]",
	  "strategies": ["core.models.evaluation.evaluation", "SkLearnClassificationStrategy"],
	  "tags": ["ml", "sklearn"],
	  "description": "Implementations of the classification models from scikit-learn framework"
}
```

Hint! Check for all Enum-style types needed to create metadata at core.repository.
After choosing or creating the metadata block take into account the “strategy” field containing the location and name of the evaluation strategy. 

Before filling in this field you need to know that every model in Fedot has quite a familiar interface with *fit* and *predict* methods. Any model uses what is called a ‘strategy pattern’ for the model evaluation. In other words, it has _eval_strategy parameter defined by the model type (passed to the Model class constructor) and the task type. Check for it at core/models/model.py
```python
self._eval_strategy = _eval_strategy_for_task(self.model_type, task.task_type)
```
 
From that point, you need to define an evaluation strategy for your case in core/models/evaluation directory. 
Follow these steps to succeed:

1. Make a script file where the custom model implementation and its evaluation strategy will be located.
2. Implement your model and strategy in the created file. If the ‘model’ you have implemented is not a class with fit/predict methods, see the realisation of *StatsModelsAutoRegressionStrategy* uses kind of functional programming. 
3. Make a class with *CustomNameTaskNameStrategy* and **inherit** it from *EvaluationStrategy*. Here is an example of how *SkLearnEvaluationStrategy* is defined:

```python
from core.models.evaluation.evaluation import EvaluationStrategy


class SkLearnEvaluationStrategy(EvaluationStrategy):
    __model_by_types = {
        'xgboost': XGBClassifier,
        'xgbreg': XGBRegressor,
        'adareg': AdaBoostRegressor,
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
        # implementation
        return prediction


class SkLearnRegressionStrategy(SkLearnEvaluationStrategy):
    def predict(self, trained_model, predict_data: InputData):
        # implementation
        return prediction
```


! Notice that the sklearn prediction strategy is defined separately since the classification and regression prediction has some differences. If your strategy satisfies the skalern strategy requirements, you can just add its type to *__model_by_types* as shown above like in any of the implemented strategies. 
Take a look at how it works in case of functional types of *fit*/*predict* methods in core.models.evaluation.stats_models_eval:

```python
from core.models.evaluation.evaluation import EvaluationStrategy


class StatsModelsForecastingStrategy(EvaluationStrategy):
    __model_functions_by_types = {
        'arima': (fit_arima, predict_arima),
        'ar': (fit_ar, predict_ar)
    }
 
    def fit(self, train_data: InputData):
        'custom implementation'
        return <fitted_model_object>

    def predict(self, trained_model, predict_data: InputData) -> OutputData:
        return self._model_specific_predict(trained_model, predict_data)


def fit_arima():
    # implementation


def predict_arima():
    # implementation

```

Set aside fit_tuned method definition for that moment with raising NotImplementedError()

Finish line.
To give the Fedot an approach to evaluate your model fill in the mentioned ‘strategies’ field with the core.models.evaluation.custom_script_name and *CustomNameTaskNameStrategy* in the metadata block in .json repository where the model insertion has begun.

Now you can use your model within FEDOT architecture.
