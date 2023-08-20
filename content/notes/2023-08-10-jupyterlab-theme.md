---
title: "Designing a Jupyterlab theme"
date: 2023-08-10 11:23
categories: workflow
tags:
    - jupyterlab
    - theme
---
[//]: # (The body)

Designing a Jupyterlab theme and publishing it involves a fairly straightforward workflow. This workflow guide targets Jupyterlab >= 4.0.0, but it's not necessary to do so. However, when it comes to extensions, there are API-breaking changes between major versions of Jupyterlab, which one must be aware of. ==insert more details==

This guide has been written with a Linux environment in mind. Specifically, I'm using Ubuntu on WSL on Windows. A smarter way to do this would perhaps be to run a docker container. 

Most of this guide draws from this [official extension tutorial][jup-extut].
## Getting Started

To get started, we first need to create a development environment, which we will do using `conda`. This could of course be done with [[notes/2022-03-26-pyenv-virtualenv|pyenv-virtualenv]] or simply `venv` instead, while being mindful of the dependencies.
```shell
conda create -n jupyter-ext-dev --override-channels --strict-channel-priority -c conda-forge -c nodefaults jupyterlab=4 nodejs=18 git copier=8 jinja2-time
```
It is possible to skip `git` if we already have it installed. Now we activate `jupyter-ext-dev`, our newly created environment:
```shell
conda activate jupyter-ext-dev
```
Now we create a folder we will house our extension in. We name it **juptheme**:
```shell
mkdir juptheme
cd juptheme
```
*In this folder*, we will use `copier` to generate our project from a copier template[^copier-unsafe], housed in the github repo [jupyterlab/extension-template][extemp-link]. 
```shell
copier copy --UNSAFE https://github.com/jupyterlab/extension-template .
```
Once we execute this command, we will be required to answer a few questions about our extension for the initial setup. This includes details like
- Extension type - *theme* in our case
- Author details
- Package details
- Misc - user settings, Binder example, setup tests
- github repo, if applicable. 

With those questions answered, the skeleton for our extension is now set up.

## Development
















[//]: # (Footnotes, if any)

[^copier-unsafe]: The `--UNSAFE` flag is required for `copier` v8+.





[//]: # (Links, if any)

[jup-extut]: <https://jupyterlab.readthedocs.io/en/stable/extension/extension_tutorial.html>
[extemp-link]: <https://github.com/jupyterlab/extension-template>