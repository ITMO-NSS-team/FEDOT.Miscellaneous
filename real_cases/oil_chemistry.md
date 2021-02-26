## Forecast of the stability of residual fuels for sedimentation during storage in tanks

At the facilities for storage of oil products are often in the production process there is a mix of various types of petroleum products. However, if the mixed oil products are incompatible, then there are qualitative losses, namely, the active form of the total sediment. The result of incompatibility:
* Deterioration of the quality of petroleum products;
* Promotes wear and clogging of technological systems;
* An increase in the intensity of accumulation of "dead" residue on the tanks, which leads to a decrease in the useful volume of the tanks;
* Economic losses due to the reduction in the cost of petroleum products;
* SO2 generation and atmospheric pollution;
* Corrosion of technological systems due to the SO3 formation;
* Formation of deposits on the heating surfaces;

<img src="img_oil_chemistry/fig_1.jpg" alt="drawing" width="431"/>

The International Maritime Organization (IMO) Committee for the Protection of the Marine Environment (MEPC) has approved a ban on the transport of marine fuels with a sulfur content of more than 0.5%, which will enter into force on March 1, 2020. Determining the composition of the fuel and the presence of sediment in it is a very expensive process(the most commonly used method is chromatography, the cost is about 1 million rubles per sample).
Using historical data containing information about the parameters of fuel and various mixtures, and the functionality of the automl framework "Fedot", it will be possible to model mixtures with the lowest sediment content.

**_Established approach:_**
* Xylitol/TThe toluene method
* A method based on state industry standards

**_The disadvantages of this approach:_**
* To spend more time on the solution to the problem
* The complexity of manual verification
* The high cost of the test

**_Solution:_**
Since the final goals are to predict the amount of sediment, 
as well as the classification of fuel mixtures into dangerous 
and non-dangerous, from the point of view of precipitation, 
this task can be simultaneously interpreted as both a regression 
task, and a classification task.The formulation of this problem in the form of a machine learning task is shown in Table 1.

Table 1. Example of a data format for machine learning problem statements.

| First sample for the mixture     | Second sample for the mixture    | Type of Machine learning task   | Physical interpretation of target variable    | Value of target variable |
|:--------------------------------:|:--------------------------------:|:-------------------------------:|:---------------------------------------------:|:------------------------:|
|       Number sample No. 13       |       Number sample No. 37       |           Regression            |     Percentage of sediment in the mixture     |          0.17            |
|       Number sample No. 13       |       Number sample No. 37       |          Classification         |      The class of mixture density sludge      |        1 class           |

For our training data set, we used information
about the physical and mechanical properties 
of the fuels in the mixture, information 
about their storage and transportation conditions, 
and information obtained during laboratory tests. 
In total, we managed to form 7 features 
for each fuel mixture. The format of the training data set 
for the regression task is shown in table 2.

Table 2. Example of a training data set for machine learning regression task.

| Number of mixture |   Feature 1  |   Feature 2  |   Feature 3   |  Feature 4 |  Feature 5  |  Feature 6  |   Feature 7  |
|:-----------------:|:------------:|:------------:|:-------------:|:----------:|:-----------:|:-----------:|:------------:|
|         13        |     583,6    |     973,0    |      2,74     |     0,3    |     0,03    |     20,0    |     110,0    |
|         37        |     472,3    |     965,0    |      2,48     |     0,2    |     0,03    |     19,0    |     110,0    |
|         22        |     18,80    |     851,0    |     0,0015    |     0,1    |     0,02    |     20,0    |     110,0    |

As can be seen from Table 3, the use of the Fedot 
framework allowed us to obtain higher values of quality 
metrics for both the regression problem and the classification problem. 
Thus, we can say that the use of the automl approach, using the
Fedot framework, for this task has a pronounced economic 
effect and expediency in terms of modeling sedimentation processes.

Table 3. Results of applying the Fedot framework, and the baseline XGB model to the generated data set.

|  Type of ML model     | ROC-AUC | F1-score | MSE   | R2    |
|:---------------------:|:-------:|:--------:|-------|-------|
| **Fedot composite model** | **0.861**   | **0.764**    | **0.168** | **0.761** |
|   Baseline XGB model  | 0.737   | 0.714    | 0.215 | 0.723 |