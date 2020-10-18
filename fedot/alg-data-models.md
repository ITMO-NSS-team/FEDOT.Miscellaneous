## The data preprocessing in FEDOT:

The data preparation for modelling implemented in following way in FEDOT. It is separated to transforming and preprocessing.

Transformer converts data formats, preprocessor - changes their "scale" through scaling/normalization, etc.

<img src="img/img-datamodel/datamodel.png" alt="drawing" width="300"/>

More complex operations with data are located in separate models. 
For example, data enrichment or dimensionality reduction can be done as:

<img src="img/img-datamodel/pca.png" alt="drawing" width="700"/>

The direct datamodel allow to enrich the model inputs with the raw data:

<img src="img/img-datamodel/datamodel-direct.png" alt="drawing" width="700"/>

The data prerocessing model for the table data can be added to the repository as follows:

```json
"pca_data_model": {
	  "meta": "dim_red_data_model",
	  "tags": ["linear"]
	}

"dim_red_data_model": {
	  "tasks": "[TaskTypesEnum.classification, TaskTypesEnum.regression]",
	  "input_type": "[DataTypesEnum.table]",
	  "output_type": "[DataTypesEnum.table]",
	  "strategies": ["core.models.evaluation.data_evaluation", "DataModellingStrategy"],
	  "tags": ["without_preprocessing", "data_model"],
	  "description": "Implementations of the models for the feature preprocessing (dimensionality reduction, etc)"
	}
```

The following python code can be used to create the chain with datamodel:

```python
node_first = PrimaryNode('pca_data_model')
    node_second = PrimaryNode('lda')
    node_final = SecondaryNode('rf', nodes_from=[node_first, node_second])

    chain = Chain(node_final)
```

The data prerocessing model for the time series:

<img src="img/img-datamodel/ts-datamodel.png" alt="drawing" width="700"/>

It can be created as follows:

```python
node_trend = Primary_node('trend_data_model')
node_lstm_trend = Secondary_node('lstm', nodes_from=[node_trend])

node_residual = Primary_node('residual_data_model')
node_ridge_residual = Secondary_node('ridge', nodes_from=[node_residual])

node_final = Secondary_node('linear', nodes_from=[node_ridge_residual, node_lstm_trend])
chain = Chain(node_final)
```