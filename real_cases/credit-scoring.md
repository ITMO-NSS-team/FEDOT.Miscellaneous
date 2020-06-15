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
The credit scoring task can be solved using a single model. The figure shows an example of scoring model quality metrics (ROC AUC, KS) on single classifiers:

<img src="img_scoring/single-classifiers.png" alt="drawing" width="700"/>

It is also possible to use an ensemble scoring model:

<img src="img_scoring/scoring-model.png" alt="drawing" width="700"/>

The ensembling make it possible to decrease the uncertainty in the model. 
However, the structure of such a model may be non-trivial.
Therefore, it is better to use the GPComp algorithm to create a chain.
Example of GPComp convergence for a simple chain of scoring models:

<img src="img_scoring/GPComp-simple.gif" alt="drawing" width="700"/>

Example of GPComp convergence for a more complex (multi-level) chain of scoring models:

<img src="img_scoring/GPComp-multi.gif" alt="drawing" width="700"/>

It can be seen that the GPComp algorithm converges more to a simpler tree, which is clear by the specifics of the problem to be solved (binary classification on a small number of features).

The best found model for the 30 minuter run is:

<img src="img_scoring/scoring_model_best.png" alt="drawing" width="700"/>
