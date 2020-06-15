# Прикладной пример (кредитный скоринг)
## Задача кредитного скоринга

Цель кредитного скоринга состоит в оценке и возможном уменьшении риска банка, ассоциированного с выдачей займа клиентам. Минимизация риска происходит за счет деления потенциальных заемщиков на кредитоспособных и некредитоспособных. Задача деления = задача классификации

Поведенческий скоринг предполагает оценку кредитоспособности на основе информации о заемщике, характеризующей его поведение и привычки и полученной из различных источников.

Иллюстрация задачи кредитного скоринга:

<img src="img_scoring/scoring-base.png" alt="drawing" width="700"/>

Для каждого из заемщиков оценивается вероятность просрочки, соотносимая со скоринговым баллом. Далее необходимо задать границу, разделяющую "добросовестных" и "недобросовестных" заемщиков.
## Описание данных

Входные данные получены из открытого источника: https://www.kaggle.com/c/GiveMeSomeCredit

| **№**       | **Параметр**           | **Описание**  |
| 1      | SeriousDlqin2yrs           | Отметка о наличии у заемщика опыта задолженности платежей на срок 90 дней и больше  |
| 2        | RevolvingUtilizationOfUnsecuredLines          | Итоговый баланс на кредитной карте и персональные кредитные линии за исключением недвижимости и первоначального взноса по задолженности (пр. кредит на машину, деленный на сумму кредитных лимитов)  |
| 3        | age           | Возраст заемщика (количество лет) |
| 4        | NumberOfTime30-59DaysPastDueNotWorse           | Количество раз, когда заемщик имел задолженности по платежам на срок 30-59 дней (не больше 1-2 месяцев) за последние 2 года|
| 5        | DebtRatio           | Отношение ежемесячных расходов (включая ежемесячные платежи по кредиту, алименты, бытовые расходы) к ежемесячному общему доходу |
| 6        | MonthlyIncome           |Ежемесячный доход|
| 7        |NumberOfOpenCreditLinesAndLoans      |Количество открытых кредитов (рассрочка, потребительский кредит (пр. кредит на машину), ипотека) и кредитных линий (пр. кредитные карты)|
| 8        | NumberOfTimes90DaysLate           |Количество раз, когда заемщик имел задолженности по платежам на срок 90 дней (3 месяца) и больше|
| 9        | NumberRealEstateLoansOrLines           |Количество ипотечных кредитов и кредитов на покупку недвижимости, включая кредиты, выданные под залог недвижимости|
| 10        | NumberOfTime60-89DaysPastDueNotWorse           |Количество раз, когда заемщик имел задолженность по платежам на срок 60-89 дней (не больше 2-3 месяцев) за последние 2 года|
| 11        | NumberOfDependents           |Количество человек на иждивении в семье, не включая самого заемщика (супруг/супруга, дети и т.д.)|

## Походы к решению задачи скоринга
Задача кредитного скоринга может решаться с помощью одиночной модели. На рисунке показан пример метрик качества скоринговой модели (ROC AUC, KS) на одиночных классификаторах:

<img src="img_scoring/score_models.png" alt="drawing" width="700"/>

Возможно и использование ансамблевой скоринговой модели:

<img src="img_scoring/score_ens_mod.png" alt="drawing" width="700"/>

Однако, структура такой модели может быть нетривиальной.

Поэтому для создания цепочки целесообразно использовать алгоритм GPComp.

Пример сходимости GPComp для простой цепочки скоринговых моделей:

<img src="img_scoring/simple_conv.gif" alt="drawing" width="700"/>

Пример сходимости GPComp для более сложной (многоуровевой) цепочки скоринговых моделей:

<img src="img_scoring/final_opt.gif" alt="drawing" width="700"/>

Видно, что алгоритм GPComp сходится скорее к более простому дереву, что обьясняется спецификой решаемой задачи (бинарная классификация на небольшом числе признаков)

## Руководство пользователя-программиста
Для практического применения GPComp целесообразно создать класс-обертку GPComposer. Для абстрагирования от конкретной реализации цепочек и моделей в рамках фреймворка в класс GPChainOptimiser, реализующий алгоритм GPComp, в качестве аргумента передаются только функции, обеспечивающие такое создание.

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

Требования к композированию могут быть заданным отдельным классом - GPComposerRequirements, унаследованным от общего ComposerRequirements.

```python
composer_requirements = GPComposerRequirements(
    primary=[LogRegression(), KNN(), LDA(), XGBoost(), DecisionTree(), RandomForest()],
    secondary=[LogRegression(), KNN(), LDA(), XGBoost(), DecisionTree(), RandomForest()], max_arity=3,
    max_depth=2, pop_size=20, num_of_generations=10,
    crossover_prob=0.4, mutation_prob=0.5)
```

Визуализация полученных цепочек осуществляется с помощью библиотеки networkx.

```python
graph, node_labels = _as_nx_graph(chain=deepcopy(chain))
            pos = node_positions(graph.to_undirected())
            nx.draw(graph, pos=pos,
                    with_labels=True, labels=node_labels,
                    font_size=12, font_family='calibri', font_weight='bold',
                    node_size=scaled_node_size(chain.length), width=2.0,
                    node_color=colors_by_node_labels(node_labels), cmap='Set3')
```
 
Для оценки качества скоринговой модели целесооразно применять метрику ROC AUC, реализацию которой может быть получена из библиотеки sklearn.metrics

```python
def calculate_validation_metric(chain: Chain, dataset_to_validate: InputData) -> float:
    # the execution of the obtained composite models
    predicted = chain.predict(dataset_to_validate)
    # the quality assessment for the simulation results
    roc_auc_value = roc_auc(y_true=dataset_to_validate.target,
                            y_score=predicted.predict)
    return roc_auc_value
```
Получение входных данных из файла следуюет производить следующим образом:

```python
def from_csv(file_path):
        data_frame = pd.read_csv(file_path)
        data_frame = _convert_dtypes(data_frame=data_frame)
        data_array = np.array(data_frame).T
        idx = data_array[0]
        features = data_array[1:-1].T
        target = data_array[-1].astype(np.float)
        return InputData(idx=idx, features=features, target=target)
Обьединение предсказаний с выходов нескольких моделей (для их передачи на вход следующей по цепочке модели) может быть выполнено так:

def from_predictions(outputs: List['OutputData'], target: np.array):
    idx = outputs[0].idx
    features = list()
    for elem in outputs:
        features.append(elem.predict)
    return InputData(idx=idx, features=np.array(features).T, target=target)
```
Обьединение нескольких моделей в цепочку осуществляется с помощью класса Chain. Добавление новых узлов производится с использованием метода chain_add. Для создания узла с заданным типом модели используются методы класса NodeGenerator.

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
Пример реализации и выполнения цепочки скоринговых моделей, оформленный в формате Jupyter Notebook, показан [здесь.](https://github.com/ITMO-NSS-team/FEDOT.Algs/blob/master/gp_comp/example/scoring_example_notebook.ipynb)
