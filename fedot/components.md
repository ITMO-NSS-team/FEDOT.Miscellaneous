## Main logical and structural blocks of the FEDOT

In the framework, the representation of the composite model consist of:

...

There is the main classes of the FEDOT:

**Chain** - encapsulates graph of composite model from operations; allows to calculate the whole chain of models recursively;

**Node** - chain component that contains the model, references to parent and child nodes, and cached model object;

**NodeFactory** - a factory for creating a necessary Node object;

**EvaluationStrategy** - strategy of model learning and calculation of predicted values.
