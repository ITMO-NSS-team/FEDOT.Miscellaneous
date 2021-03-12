## State-of-the-art AutoML solutions: a brief overview

We cannot deny that there are a lot of well-implemented AutoML frameworks that are quite popular.

We tried to summarize the main advantages and disadvantages of the state-of-the-art solutions to the breif table:

|  Framework  | Pipeline | Optimisation algorithm |       Input data       | Ensembling |         Scaling         |  Additional features |                           Source                           |
|:-----------:|:--------:|:----------------------:|:----------------------:|:----------:|:-----------------------:|:--------------------:|:----------------------------------------------------------:|
|     TPOT    | Variable |           GP           |        Tabular         |      -     | Multiprocessing, Rapids |    Code generation   |            https://github.com/EpistasisLab/tpot            |
|     H2O     |   Fixed  |       Grid Search      |     Tabular, Texts     |      +     |          Hybrid         |           -          | https://docs.h2o.ai/h2o/latest-stable/h2o-docs/automl.html |
| AutoSklearn |   Fixed  |          SMAC          |         Tabular        |      +     |            -            |           -          |           https://github.com/automl/auto-sklearn           |
|     ATM     |   Fixed  |           BTB          |         Tabular        |      -     |          Hybrid         |           -          |             https://github.com/HDI-Project/ATM             |
|    FEDOT    | Variable |     GP + GridSearch    |   Tabular, Timeseries  |      +     |            -            |   Composite models   |             https://github.com/nccr-itmo/FEDOT             |
|  AutoGluon  |   Fixed  |     Fixed Defaults     | Tabular, Images, Texts |      +     |            -            | NAS, AWS integration |            https://github.com/awslabs/autogluon            |
|     LAMA    |   Fixed  |         Optuna         |         Tabular        |      +     |            -            |       Profiling      |       https://github.com/sberbank-ai-lab/LightAutoML       |
|     NNI     |   Fixed  |          Bayes         |     Tabular, Images    |      ?     |    Hybrid, Kubernetes   |      NAS, WebUI      |              https://github.com/microsoft/nni              |


## Why we want better AutoML?​

1. «Machine learning from scratch» is too hard – a lot of time and resources is required (https://ai.googleblog.com/2020/07/automl-zero-evolving-code-that-learns.html)​ Expert knowledge should be involved (i.e. as initial assumption);​
2. Pipelines with fixed structure – suitable for simple, routine tasks but not very effective for real problems;​
3. Raw data should be processed – preprocessing is not well-supported by modern AutoML;​
4. Modelling for multimodal data (i.e. table + images + time series + text) is not well-automated now for common tasks;​
5. Specialized for different modelling problems – current solutions are mostly over-specialised;​
6. Modular pipelines – composite structures instead of complex neural networks / End-To-End Learning for better controllability;