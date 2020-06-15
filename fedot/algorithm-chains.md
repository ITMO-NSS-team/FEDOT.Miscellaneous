## Description of the algorithm for creation of machine learning models' chains

## Purpose of the algorithm
Creation of a chain of machine learning (ML) models to solve application tasks.
For each problem to be solved, the modified genetic programming algorithm forms a chain of several ML models contained in a given set. A chain can contain two types of nodes:

1. Secondary nodes. Nodes that combine multiple models (input characteristics are generated from model predictions merged by this node)

2. Primary nodes. The nodes, representing independent models (model of the first level which input features are obtained from solving task) 

The data transfer between nodes can be represented as follows:
<img src="img/img-algorithm-chains/purpose.png" alt="drawing" width="700"/>

## Input data

### 1. Algorithm resources  (composer_requirements)

* primary и secondary - two sets of models that can be used as Primary nodes and Secondary nodes, respectively, taken from the generated model repository.
* num_of_generations - a number of generations of the evolutionary algorithm.
* pop_size - the size of trees population.
* max_depth - maximum depth of a tree.
* max_arity - maximum arity of a Secondary node (maximum amount of models, which can be merged by a node).
* crossover_prob - breeding probability in a generic algorithm.
* mutation_prob - mutation probability in a generic algorithm.

### 2. Database of the solving task.

### 3. A quality metric for a task solution (taken from an existing quality metrics repository quality_metrics_repository.py).

As an example, if you want to apply the ROC-AUCs quality metric to the algorithm, you need to initialize it as follows:

``` python
metric_function = MetricsRepository().metric_by_id(ClassificationMetricsEnum.ROCAUC)
```

## Output data
The output is the ML-model tree found by an evolutionary algorithm and converted using _tree_to_chain method into the ML-model chain.
## Description of the operation principle
Solutions in genetic programming are presented as trees. The trees may be n-ary, i.e. the number of under-nodes of each tree node may not be greater than n. Usually, in genetic programming, tree nodes are elements of one of two sets: terminal (T) or functional (F). A functional set includes functions that are used to form a solution and a terminal set includes constants and variables. Within this framework, the equivalent of the function nodes is Secondary nodes, and Primary nodes stand for terminal ones.
### Let's have a look at the detailed description of the algorithm:
The evolution process according to which the genetic programming algorithm is functioning can be described as follows: a population of several chains is accidentally initialized, and then covers the best regions of the search space through tree selection, mutation, and recombination random processes.
Scheme of the genetic programming algorithm for model chaining:

<img src="img/img-algorithm-chains/algorithm-scheme.png" alt="drawing" width="700"/>

### 1. Initialization
In the first stage, a population of trees is generated. A growing method is used for initialization in genetic programming. With this method, primary nodes can be located at any depth (except zero, since it always has a functional node), but the maximum depth of the node in the tree is d (user-defined parameter).
 
### 2. Calculation of suitability of individuals
Based on the quality metric selected from the repository.
### 3. Selection
The developed implementation uses tournament selection of individuals realized in the tournament_selection method. According to this strategy, a group of n population individuals (n > 2) selected randomly is created to select one individual who will participate in the crossing as parents. The most suitable individual is selected from this group and the remaining individuals are discarded. The number n is called tournament size, usually, the size of tournaments is n = 2, n = 5 and n = 9. Advantages of this type of selection: simplicity, absence of the requirement to rank individuals, which requires much more computational resources.

<img src="img/img-algorithm-chains/tournament_selection.png" alt="drawing" width="700"/>

Selection is implemented as follows:

```python
def tournament_selection(fitnesses, group_size=5):
    selected = []
    pair_num = 0
    for j in range(len(fitnesses) * 2):
        if not j % 2:
            selected.append([])
            if j > 1:
                pair_num += 1

        tournir = [randint(0, len(fitnesses) - 1) for _ in range(group_size)]
        fitness_obj_from_tour = [fitnesses[tournir[i]] for i in range(group_size)]

        selected[pair_num].append(tournir[np.argmin(fitness_obj_from_tour)])

    return selected
```

### 4. Breeding
The standard breeding type implemented in the standard_crossover method was used. After selecting the parents in the selection step for each parent pair:
1. Breeding point is chosen from every available parent.
2. Parents exchange subtrees below the breeding point.

<img src="img/img-algorithm-chains/tree-breeding.gif" alt="drawing" width="700"/>

Breeding is implemented as follows:

```python
def standard_crossover(tree1, tree2, max_depth, 
                       crossover_prob, pair_num=None, 
                       pop_num=None):
    if tree1 is tree2 or random.random() > crossover_prob:
        return deepcopy(tree1)
    tree1_copy = deepcopy(tree1)
    rn_layer = randint(0, tree1_copy.get_depth_down() - 1)
    rn_self_layer = randint(0, tree2.get_depth_down() - 1)
    if rn_layer == 0 and rn_self_layer == 0:
        return deepcopy(tree2)

    changed_node = choice(tree1_copy.get_nodes_from_layer(rn_layer))
    node_for_change = choice(tree2.get_nodes_from_layer(rn_self_layer))

    if rn_layer == 0:
        return tree1_copy

    if changed_node.get_depth_up() + node_for_change.get_depth_down() - \
       node_for_change.get_depth_up() < max_depth + 1:
        changed_node.swap_nodes(node_for_change)
        return tree1_copy
    else:
        return tree1_copy
   ```
### 5. Mutation
In the standard_mutation method, a point mutation scheme was implemented. A randomly selected node in the tree changes to a randomly selected element of the same type. I.e. the selected node is replaced by a randomly selected member of a set of valid models for primary nodes if that node belongs to that type, or a member of a set of valid secondary nodes.

<img src="img/img-algorithm-chains/mutation-scheme.png" alt="drawing" width="700"/>

Mutation is implemented as follows:

```python
def standard_mutation(root_node, secondary_requirements, 
                      primary_requirements, probability=None):
    if not probability:
        probability = 1.0 / root_node.get_depth_down()

    def _node_mutate(node):
        if node.nodes_from:
            if random.random() < probability:
                node.eval_strategy.model = random.choice(secondary_requirements)
            for child in node.nodes_from:
                _node_mutate(child)
        else:
            if random.random() < probability:
                node.eval_strategy.model = random.choice(primary_requirements)

    result = deepcopy(root_node)
    _node_mutate(node=result)

    return result
```
## How to use the algorithm for applications
First, you must define the model classes for which you want to chain. Their start and calculation is given to the external software complex in which the algorithm is embedded.
Binding with external functions is implemented through callback mechanics - that is, an object/function is passed to the algorithm optimizer as an external parameter, which, for example, can track changes in the population. 

At the end of each iteration of the algorithm the function sellf.history_callback () is called, where the required parameters (fitness function values, best/worst individuals, etc.) are transmitted. After that, any wanted functionality is implemented in callback - console output, visualization and so on. An example of this implementation can be found in the Keras [documentation](https://www.tensorflow.org/guide/keras/custom_callback).
Evolutionary operators are preferably separated into separate scripts and divided into selection.py, crossover.py, mutation.py, etc.

## Guide for the advanced developer
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
