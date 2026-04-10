---
title: "How to publish a PYPI package on CONDA-FORGE"
datePublished: 2026-04-10T09:37:37.212Z
cuid: cmnsppxo7004d29kaftcochlw
slug: how-to-publish-a-pypi-package-on-conda-forge

---

I am maintainer of some [PyPi](https://pypi.org/) libraries and had the wish to publish them also on [CONDA-FORGE](https://conda-forge.org).

I won’t go into why CONDA-FORGE and what’s great about Anaconda and the alternative package manager.

There are several ways, here is described the easiest and most modern way with `grayskull` .

Basically publishing a package only takes about 15 minutes if you know how!

#### First, some quick info:

Anyone can publish a PyPi package on CONDA-FORGE, you don’t have to be connected in any way to the project you want to publish.

You need a [GitHub](https://github.com/) account and an installed Python.

The PyPi package must be available as a source package (SDIST) and not as a wheel on PyPi.

![](https://cdn-images-1.medium.com/max/800/1*rqDwnZ72IqcVCDAK-hd7YQ.png)

#### Here’s how:

1\. The goal is to create a recipe. This is a file (meta.yaml) that contains all the information for CONDA-FORGE to automatically build a package. This recipe can be created automatically by the tool `grayskull`. So we start with the installation of `grayskull` either with pip or conda as described here: [https://github.com/conda-incubator/grayskull#installation](https://github.com/conda-incubator/grayskull#installation).

2\. Once `grayskull` is installed, the command `grayskull pypi <pypi-package-name>` creates a recipe from the PyPi package, which can be submitted to CONDA-FORGE.  
If this command throws the error `AttributeError: Hash information for sdis was not found on PyPi metadata.`, then the PyPi package is not available as source distribution but probably only as wheel. A SDIST package can be created with the command `python3 setup.py sdist` and can be combined with the creation of a wheel: `python3 setup.py bdist_wheel sdist`

3\. Once the recipe (meta.yaml) is created, please open and check it:

The maintainer name must match your github username.

In the `about:`sector change the url in `home:` to the project website and add below:  
_dev_url: https://github.com/you/repo  
doc_url: https://you.github.io/repo_

4\. Create a fork of [https://github.com/conda-forge/staged-recipes](https://github.com/conda-forge/staged-recipes)

5\. Create a separate branch of the `staged-recipes` repository on Github for each recipe you want to submit.

6\. Add a folder with the name of your package to the respective branch in the folder `recipes` and add the created `meta.yaml`and start a pull request.  
Read the comments for writing the pull request, check all the checkboxes and do what they ask for.

8\. All CI checks of the PR should be successful. If so, create a new post with the content `@conda-forge/staged-recipes, ready for review` and `@conda-forge-admin, please ping team`. This triggers the editing process by the CONDA-FORGE team.

9\. Wait for email messages and join the team on Github.

10\. On each new PyPi release a bot opens a PR on the feedstock repository and you have to merge it manually to trigger the creation of a new package for conda. To make this automatically just add `@conda-forge-admin, please add bot automerge`.

Chat for support: [https://gitter.im/conda-forge/conda-forge.github.io](https://gitter.im/conda-forge/conda-forge.github.io)

----------

I hope you found this tutorial informative and enjoyable! Don’t forget to follow me on [Medium](https://medium.com/@oliverzehentleitner/about), [Twitter](https://twitter.com/unicorn_oz), [GitHub](https://github.com/oliver-zehentleitner), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases and insights. If you found this article helpful, please hit that applause 👏 button to show your support! Your constructive feedback is always appreciated as it helps me improve the quality of my content.

Thank you for reading, and happy coding!