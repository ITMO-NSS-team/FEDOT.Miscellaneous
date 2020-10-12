## Robotics problem tutorial

## Grasp stability prediction

The selected robotics case which was dedicated to the problem of manipulator grasp stability binary classification at the Kaggle platform. The main goal was to predict grasp
stability based on several features like positions, angular velocities, efforts of each kinematic joint. The
proposed manipulator consisted of one hand with three fingers with three joints per the last one. Total
amount of joints was equal to 9 and therefore total amount of features was equal to 27.

The visualisation of the robotics simulator used in this study:
<img src="https://github.com/shadow-robot/smart_grasping_sandbox/raw/master/doc/img/smart_grasping_sandbox.png" alt="drawing" width="600"/>


It should be mentioned that existing framework functionality is not enough because of the majority cases are excellently decided by various neural
networks in robotics. Firstly, it takes in place in a computer vision tasks which decisions are hard to
imagine without convolutional neural networks implementation. But it’s fair to say that it is possible to
organize neural architecture search for a specific cases based on Fedot functionality like it was done in
the [nas-fedot](https://github.com/ITMO-NSS-team/nas-fedot) module.


## Dataset description

The grasping dataset obtained in simulation using the Smart Grasping Sandbox (https://github.com/shadow-robot/smart_grasping_sandbox). An experiment
consists of grasping the ball, shaking it for a while, while computing a grasp robustness (which is the
variation of the distance between the palm and the ball during the shake). Multiple measurements are
taken during a given experiment. Only one robustness value is associated though. The obtained dataset
is balanced and has 50/50 stable and unstable grasps respectively.


## Optimal chains generation and tuning

At the beginning, genetic algorithm configuration was initialized by default values which
represented in the table: 

| p_crossover | p_mutation | d_max | a_max | n_generation | s_population |
| --- |--- |--- |--- |--- |--- |
|0.8 | 0.8 | 3 | 3 | 20 | 20 ||

where Pcrossover is a crossover probability, Pmutation is a mutation probability,
dmax is a depth limit, amax is a arity limit, ngeneration is a number of generations, spopulation is a
population size. It was used the following genetic programming (GP) operators: tournament
selection, one-point mutation, standard crossover.


The first problem was time execution of composing process. The reason of why such problem
occurred was the enormous size of the dataset and lack of opportunity to compute in parallel flows
despite the simple chain generation. In order to overcome this problem and get data to analyze it was
proposed to decrease dataset size to 5% with respect to the original dataset size. Such value was
empirically obtained to restrict execution time to a reasonable value.


The model’s variability was based on framework models repository with respect to classification
task. After several executions, it was detected that the process steadily leading to chains which consisted
of the following components: logistic model (logit), random forest model (rf), latent dirichlet allocation
model (lda), support vector clustering model (svc).


Evolution convergence for some obtained chains is shown in the picture:

<img src="/FEDOT.Docs/real_cases/img_robotics/robotics_evo.png" alt="drawing" width="600"/>

As a result of multiple experiments, there is no need to set more than 100 generation for this
specific case. 


However, the chains were composed and fitted by only 5% of the original dataset. Taking into
account the most time-consuming composing, it was proposed to use 5% to compose and 100% to fit
composed chain to improve estimation quality and restrict composition time to a reasonable value. In
this case, the original data were initially divided into data to validate for experiment purity and the other
from which the aforementioned percents were taken. It was used 5 quality metrics such as accuracy, roc
auc, f1, precision, recall to evaluate classification quality.

## Performance models

The most time-consuming part of the process is a composing. Suggested approach of decreasing
dataset size to compose gave a good results for a reasonable time. But the following tuning was based
on the much more data size. And it was interesting to create performance models for some chains
which were obtained during compositing. The aim was to extrapolate the performance model of each
individual node to chain performance model which consisted of the first ones.

<img src="/FEDOT.Docs/real_cases/img_robotics/rf_perf.png" alt="drawing" width="600"/>

There is and further, it was used least squared errors (LSE) algorithm to approximate origin
points of a single model performance. As you can see, some of the original points were excluded
because of having anomaly values which were statistically checked. The approximation model was
chosen in terms of compliance with the two conditions: monotonicity and the best value of the root
mean square error (RMSE) metric.

It could be observed that rf performance model approximation is good enough in terms of
RMSE, little and balanced with respect to zero value residuals over both directions. And likely, there isn’t
overfitting because of reasonable monotonicity condition of the pattern function.

One of the potential reasons of anomalies is a partly non-compliance with the main experiment condition which can be expressed as a not isolated
environment of experiment conduction. For instance, it could be occurred when the parallel process has
a big weight and a higher priority or because of the operating system features. But such anomalies have
a point character and are easy to detect.

There is a systematic error which is concluded in the local
monotonicity increasing of residual over num_lines direction but the total value of RMSE metric is small
enough with respect to the range of time variance for lda model. Moreover, this problem has a local
character and don’t spread on the rest majority part of the surface.

## Conclusion

Firstly, the optimal in terms of a grasp stability prediction chains with enhanced performance
with respect to existing methods have been proposed. It should be mentioned that just a single rf
model give an excellent results but metrics values can be improved by including genetic algorithm to compose not single model chain.

Secondly, it was proposed chain performance model extrapolation procedure with a good
enough validation metrics values. It can be used to detect problems with chain fitting algorithms like
time redundancy of a chain fit process or to intelligently manage of automate models generation.
Further, it can be conducted some experiments to mix obtained best chains into a single chain to
potentially improve quality. Besides, the dataset can be extended by new simulations with new grasping
objects of different shapes. Moreover, it could be excellent to implement self-tuning genetic algorithm
to this task and extend number of experiments with chain performance models extrapolation to get
statistically good results and implement this in an appropriate way.


Author: Mikhail Yachmenkov