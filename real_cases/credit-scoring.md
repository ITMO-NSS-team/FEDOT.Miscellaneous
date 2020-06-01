## Credit Scoring problem

## Credit Scoring Task
The purpose of credit scoring is to assess and possibly reduce the risk of a bank associated with lending clients. Risk minimization happens due to dividing potential borrowers into creditworthy and non-creditworthy borrowers. Division task = classification task.
Behavioural scoring involves an assessment of creditworthiness based on information about the borrower, characterizing his behaviour and habits and obtained from various sources.

Illustration of the credit scoring task:

<img src="img_scoring/task.png" alt="drawing" width="700"/>

For each of the borrowers, the probability of delay, related to the scoring score, is estimated. Then it is necessary to set a border separating "conscientious" and "unconscientious" borrowers.
## Data description

Input data is obtained from open source: https://www.kaggle.com/c/GiveMeSomeCredit

| **№**       | **Parameter**           | **Description**  |
| 1      | SeriousDlqin2yrs           | A label that the borrower has experience of payment delay for 90 days or more  |
| 2        | RevolvingUtilizationOfUnsecuredLines          | The total balance on credit card and personal credit lines excluding real estate and initial debt contribution (for instance, car credit divided by credit limits)  |
| 3        | age           | Age of borrower (number of years)  |
| 4        | NumberOfTime30-59DaysPastDueNotWorse           | Number of times the borrower had payment delays for 30-59 days (not more than 1-2 months) for the last 2 years  |
| 5        | DebtRatio           | The ratio of monthly expenses (including monthly loan payments, alimony, household expenses) to monthly total income  |
| 6        | MonthlyIncome           | Monthly total income  |
| 7        |NumberOfOpenCreditLinesAndLoans      |Number of open loans (instalments, consumer credit (etc. car credit), mortgage) and credit lines (etc. credit cards) |
| 8        | NumberOfTimes90DaysLate           | Number of times the borrower has payment delays for 90 days (3 months) or more |
| 9        | NumberRealEstateLoansOrLines           |  Number of mortgages and loans to purchase real estate, including real estate loans  |
| 10        | NumberOfTime60-89DaysPastDueNotWorse           | Number of times the borrower had payment delays for 60-89 days (not more than 2-3 months) for the last 2 years |
| 11        | NumberOfDependents           | Number of dependants in the family, not including the borrower himself\herself (spouse, children, etc.)  |

## Approaches to solving scoring task
The credit scoring task can be solved using a single model. The figure shows an example of Scoring Model Quality Metrics (ROC AAA, KS) on single classifiers:

<img src="img_scoring/single-classifiers.png" alt="drawing" width="700"/>

It is also possible to use an ensemble scoring model:

<img src="img_scoring/scoring-model.png" alt="drawing" width="700"/>

However, the structure of such a model may be non-trivial.
Therefore, it is better to use the GPComp algorithm to create a chain.
Example of GPComp convergence for a simple chain of scoring models:

<img src="img_scoring/GPComp-simple.gif" alt="drawing" width="700"/>

Example of GPComp convergence for a more complex (multi-level) chain of scoring models:

<img src="img_scoring/GPComp-multi.gif" alt="drawing" width="700"/>

It can be seen that the GPComp algorithm converges more to a simpler tree, which is clear by the specifics of the problem to be solved (binary classification on a small number of features)
## Guide for the programmer user 
For practical application of GPComp, it is useful to create a GPComposer wrapper class. For abstraction from a particular implementation of chains and models within a framework, only the functions providing such creation are passed to the GPChainOptimiser class implementing the GPComp algorithm as an argument.

```python
class GPComposer(Composer):
    def compose_chain(self, data: InputData, initial_chain: Optional[Chain],
                      composer_requirements: Optional[GPComposerRequirements],
                      metrics: Optional[Callable]) -> Chain:
        metric_function_for_nodes = partial(self._metric_for_nodes,
                                            metrics, data)
        optimiser = GPChainOptimiser(initial_chain=initial_chain,
                                     requirements=composer_requirements,
                                     primary_node_func=GP_NodeGenerator.primary_node,
                                     secondary_node_func=GP_NodeGenerator.secondary_node)
 
        best_found, history = optimiser.optimise(metric_function_for_nodes)
 
        ComposerVisualiser.visualise_history(history)
 
        best_chain = GPComposer._tree_to_chain(tree_root=best_found, data=data)
        return best_chain
```

Compositional requirements can be specified as particular class-GPComposerRequirements inherited from the general ComposerRequirements.

```python
composer_requirements = GPComposerRequirements(
    primary=[LogRegression(), KNN(), LDA(), XGBoost(), DecisionTree(), RandomForest()],
    secondary=[LogRegression(), KNN(), LDA(), XGBoost(), DecisionTree(), RandomForest()], max_arity=3,
    max_depth=2, pop_size=20, num_of_generations=10,
    crossover_prob=0.4, mutation_prob=0.5)
```

The obtained chains are visualized using the networking library:

```python
graph, node_labels = _as_nx_graph(chain=deepcopy(chain))
            pos = node_positions(graph.to_undirected())
            nx.draw(graph, pos=pos,
                    with_labels=True, labels=node_labels,
                    font_size=12, font_family='calibri', font_weight='bold',
                    node_size=scaled_node_size(chain.length), width=2.0,
                    node_color=colors_by_node_labels(node_labels), cmap='Set3')
```

To evaluate the quality of the scoring model, use the ROC ACC metric, the implementation of which can be obtained from the sklearn.metrics library.

```python
def calculate_validation_metric(chain: Chain, dataset_to_validate: InputData) -> float:
    # the execution of the obtained composite models
    predicted = chain.predict(dataset_to_validate)
    # the quality assessment for the simulation results
    roc_auc_value = roc_auc(y_true=dataset_to_validate.target,
                            y_score=predicted.predict)
    return roc_auc_value
```

One should obtain input data from the file as follows:

```python
def from_csv(file_path):
        data_frame = pd.read_csv(file_path)
        data_frame = _convert_dtypes(data_frame=data_frame)
        data_array = np.array(data_frame).T
        idx = data_array[0]
        features = data_array[1:-1].T
        target = data_array[-1].astype(np.float)
        return InputData(idx=idx, features=features, target=target)
```

The combination of predictions from the outputs of several models (to transmit them to the input of the next model in the chain) can be done as follows:

```python
def from_predictions(outputs: List['OutputData'], target: np.array):
    idx = outputs[0].idx
    features = list()
    for elem in outputs:
        features.append(elem.predict)
    return InputData(idx=idx, features=np.array(features).T, target=target)
```

One can use the Chain class to integrate multiple models into a chain. New nodes are added using the chain_add method. Use the NodeGenerator class methods to create a node with the specified model type.

```python
complex_chain = Chain()
 
last_node = NodeGenerator.secondary_node(MLP())
 
y1 = NodeGenerator.primary_node(XGBoost(), dataset_to_train)
complex_chain.add_node(y1)
 
y2 = NodeGenerator.primary_node(LDA(), dataset_to_train)
complex_chain.add_node(y2)
 
y3 = NodeGenerator.secondary_node(RandomForest(), [y1, y2])
complex_chain.add_node(y3)
 
y4 = NodeGenerator.primary_node(KNN(), dataset_to_train)
complex_chain.add_node(y4)
y5 = NodeGenerator.primary_node(DecisionTree(), dataset_to_train)
complex_chain.add_node(y5)
 
y6 = NodeGenerator.secondary_node(QDA(), [y4, y5])
complex_chain.add_node(y6)
 
last_node.nodes_from = [y3, y6]
complex_chain.add_node(last_node)
```

