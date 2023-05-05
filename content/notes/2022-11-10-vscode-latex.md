---
title: 2022-11-10-vscode-latex
date: 2022-11-10 17:37
categories: workflow
tags:
    - vscode
    - LaTeX
---
[//]: # (The body)

When it comes to installing software, my primary goal is optimization between saving storage space and having a convenient (enough) workflow. Installing an entire **LaTeX** distribution can consume >7 GB worth of storage space. Even though small TeX distributions may not be very useful, there is still space for optimisation. 

In this workflow, we first install **TeXLive** on WSL (Ubuntu 22.04). There are many ways to do this, but we stick to the one advised on [the official website][tug-offcl] for Unix-like systems:
First, we get the installer:
```shell
wget https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz 
```














[//]: # (Footnotes, if any)

[^fn]: Footnote





[//]: # (Links, if any)

[tug-offcl]: https://tug.org/texlive/quickinstall.html