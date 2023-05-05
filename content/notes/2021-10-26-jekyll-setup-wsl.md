---
layout: post
title:  "Setting up Jekyll on WSL"
date:   2021-10-26 23:55:00 +0530
categories: setup
tags:
    - jekyll
    - wsl
---
[//]: # (The body)

This guide is mainly an accumulation of information from two sources, both from the *Jekyll Docs*:
  - [Windows Installation][1]
  - [Troubleshooting][2]

### Installation:

We plan to install Jekyll[^vernote] on **Ubuntu 20.04 on WSL**. For this we have to install **Ruby**. The Ruby *gems*, however, we want to install only for the user (instead of a system-wide installation). For this purpose we edit the *~/.bashrc* file and add the following:
```shell
# Ruby exports

export GEM_HOME=$HOME/gems
export PATH=$HOME/gems/bin:$PATH
```
which basically sets the path where the *gems* will be installed. We need to source the *~/.bashrc* file now...
```shell
source ~/.bashrc
```

Now we start with the **Ruby** installation. We install this from the **BrightBox** PPA (Personal Package Archive):
> which hosts optimized versions of Ruby for Ubuntu. [(ref)][1]

```shell
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
sudo apt-get install ruby2.5 ruby2.5-dev build-essential dh-autoreconf
```
Update our gems:[^failnote1]
```shell
gem update
```
and finally install **Jekyll** and **Bundler**[^failnote2]:
```shell
gem install jekyll bundler
```
And then we can check the version using `jekyll -v`.

### New blog setup:

A blog can be quickly set up with
```shell
jekyll new sitename
```
which will create a folder named *sitename* with all the relevant files. After that, we can `cd sitename` and start a server at that location:
```shell
bundle exec jekyll serve --force-polling
```
*(As of 26-Oct-2021)* Without the `--force-polling` option, changes made to files in the site are not reflected upon reload in the website. The command itself displays this warning:
```
Auto-regeneration may not work on some Windows versions.
Please see: https://github.com/Microsoft/BashOnWindows/issues/216
If it does not work, please upgrade Bash on Windows or run Jekyll with --no-watch.
```
One can also add the option `--livereload` to have the website reload automatically when the files are changed. Solution from issue [#216][3] in the *WSL repo*.







[//]: # (Footnotes, if any)

[^vernote]: We use a fairly old version of **Ruby** (2.5.8p224) here, following the tutorial. It's not necessary to do this, but various gems have to have versions whicha re consistent with each other for the whole thing to work. So this is a safe bet.
[^failnote1]: In my case, a few gems failed to update properly.
[^failnote2]: By default, this fails (version conflict between **RubyGems** and **Ruby**?).
    ```shell
    Fetching: public_suffix-4.0.6.gem (100%)
ERROR:  While executing gem ... (ArgumentError)
    wrong number of arguments (given 4, expected 1)
    ```
    however, `gem uninstall psych` fixes the issue and installation completes successfully ([source][4]). <span style="color:red">*TO BE INVESTIGATED.*</span>





[//]: # (Links, if any)

[1]: <https://jekyllrb.com/docs/installation/windows/>
[2]: <https://jekyllrb.com/docs/troubleshooting/#no-sudo>
[3]: <https://github.com/microsoft/WSL/issues/216#issuecomment-716047269>
[4]: <https://stackguides.com/questions/68899508/gem-install-wrong-number-of-arguments-given-4-expected-1>