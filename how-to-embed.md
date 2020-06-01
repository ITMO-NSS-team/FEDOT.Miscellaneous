### [Автоматическое МО](https://yashchenkoanastasia.github.io/github-pages-with-jekyll/autoML)
### [Фреймворк FEDOT](https://yashchenkoanastasia.github.io/github-pages-with-jekyll/about-FEDOT)
### [Как нас процитировать](https://yashchenkoanastasia.github.io/github-pages-with-jekyll/citation)
### [How to embed custom model into the FEDOT pipeline](https://yashchenkoanastasia.github.io/github-pages-with-jekyll/how-to-embed)
### [Руководство по работе с репозиторием](https://yashchenkoanastasia.github.io/github-pages-with-jekyll/rukovodstvo)

# How to embed custom model into the FEDOT pipeline
FEDOT operates with a lot of self-defined types of models, tasks and some more have been realised by the means of enum Python module. FEDOT uses model types described by enumerations to operate with integrated models. Here are some simple steps to extend FEDOT with a custom model.

First of all, you need to 'register' your model type in a repository placed at core/repository/model_types_repository.py by adding a line with a model name in ModelTypesIdsEnum class as follows:
