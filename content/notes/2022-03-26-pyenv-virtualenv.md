---
layout: post
title:  "Python virtual environments with pyenv"
date:   2022-03-26 17:55:00 +0530
categories: workflow
tags:
    - python
    - pyenv
---
[//]: # (The body)

Virtual environments isolate a set of executables, libraries and related files from another such set. From the **python** [docs][1]:
> A virtual environment is a Python environment such that the Python interpreter, libraries and scripts installed into it are isolated from those installed in other virtual environments, and (by default) any libraries installed in a “system” Python, i.e., one which is installed as part of your operating system.

There are many options to do this: **venv** (comes installed by default with Python 3.3+), **virtualenv**, **conda**, **poetry** and so on... For people like us who use these languages mostly for scientific work, **conda** is a great option (and for other applications too - it handles dependency management well and has its own package manager `conda`), but it irks me to great measure because to install conda you'd have to install a **python** version (the *base* version of conda) again (?!) and the install size, even for [**miniconda**][2], is ~400MB. It is however a good option if that's the *first python installation one starts with* - I happened to have a working python installation (v3.9.7 on Ubuntu 21.10 on WSL) with many packages and I was unwilling to get rid of it and consequently break many applications.

Also, many applications demand a different python version altogether. If one does not use **conda**, separate python versions must be installed which can also break functionality if the path to the appropriate executable is not set in `PATH` and scripts do not properly select the right version. So this demands proper management. This is what we achieve with [**pyenv**][3].

**pyenv** helps manage various python installations on the system easily and also provides the plugin [**pyenv-virtualenv**][pyvnv] (installed separately, based on **venv**/**virtualenv**: [more on this][pyvnv-method]). 

### Installation

To start with, we configure a [proper build environment][buildenv] for building Python distributions with **pyenv** on-the-fly:
```bash
sudo apt-get update; \
sudo apt-get install make build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```
Some of these packages would probably already be installed, but be prepared to install a bulk of packages. For me the download size was ~100MB and installation size ~500MB. [^sizeissue]

Once this is done, we install **pyenv** via a Github checkout (of course, `git` needs to be installed). To do so, we first clone the repo:
```bash
 git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```
where the location has been set as *~/.pyenv/*. This can be changed. Next, we optionally *try* to compile a *dynamic Bash extension* (without this step, **pyenv** works just fine, this should just speed up **pyenv**):
```bash
 cd ~/.pyenv && src/configure && make -C src
```
Next, we configure the shell's environment to work with **pyenv**. For bash on Ubuntu with a *.profile* file that sources *.bashrc*, 
```bash
sed -Ei -e '/^([^#]|$)/ {a \
export PYENV_ROOT="$HOME/.pyenv"
a \
export PATH="$PYENV_ROOT/bin:$PATH"
a \
' -e ':a' -e '$!{n;ba};}' ~/.profile
```
This puts the two `export` lines at the beginning (which is why we do all the gymnastics with `sed`) of *.profile* to 
  - Create the variable *$PYENV_ROOT* which stores the path to the folder we cloned the repo to, and
  - Add this variable to (the beginning of) *$PATH*.
```bash
echo 'eval "$(pyenv init --path)"' >>~/.profile
```
This puts the pyenv *shims* into *$PATH*. The *shims* redirect calls to the python executable to the right one. Details on the working [here][shimswork].
```bash
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
```
and this modifies *.bashrc*. For other setups, see [here][setup].

Finally, we restart the shell - and we're done installing pyenv!

Next, we install **pyenv-virtualenv**. This simply requires checking out the repo to the *.pyenv/plugins/* directory:
```bash
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
```
and we then run
```bash
exec <shell>
```
(in our case, `<shell>` is just `bash`) to restart the shell.

If installed with this method, upgrading is super simple - we just go the *.pyenv* directory and pull from the repo.
```bash
cd $(pyenv root)
git pull
```
and a similar procedure follows for **pyenv-virtualenv**.

### Usage

When using **pyenv**, we have a *system* version of python that is present by default (python was installed by default, of course). We can install more versions by running
```bash
pyenv install <version>
```
where we `<version>` may be replaced by `3.8.1`, for instance. We can list all available versions by typing
```bash
pyenv install --list
```
and choose the appropriate one. After installation, this goes under *.pyenv/versions/{version}/* and all packages/virtual-environments concerning this version go under this directory. We can list the currently installed versions by
```bash
pyenv versions
```
The currently active version is marked by an asterisk (\*). This can also be checked by running `pyenv version` instead. The output of the command will depend upon the current session or the current location. This is how **pyenv** chooses the python version (from the docs):
>  1. The *PYENV_VERSION* environment variable (if specified). You can use the `pyenv shell` command to set this environment variable in your current shell session.
>
>  2. The application-specific *.python-version* file in the current directory (if present). You can modify the current directory's *.python-version* file with the `pyenv local` command.
>
>  3. The first *.python-version* file found (if any) by searching each parent directory, until reaching the root of your filesystem.
>
>  4. The global *$(pyenv root)/version* file. You can modify this file using the `pyenv global` command. If the global version file is not present, pyenv assumes you want to use the *system* Python. (In other words, whatever version would run if pyenv weren't in your *PATH*.)

In other words, there are two ways to specify a python version to use:
  - Change the version being used for the current session by running `pyenv shell <version>`. Running `pyenv shell` or `pyenv version` would now show `<version>` as output. For this session, until changed, this version will be used for running python scripts. We can run `pyenv shell --unset` to revert to the shell being originally used before any such commands were executed. *This choice has the higest precedence.*
  - Create a *.python-version* file by running `pyenv local <version>` in a project directory. Whenever scripts are run from this directory, or *any* subdirectories, the chosen version will always be used, considering no shell version has been configured for the session. One can set multiple versions in decreasing order of precedence by running `pyenv local <version-1> <version-2> ...`  in a directory. Again, running `pyenv local` or `pyenv version` would now show the active versions as output. To unset this file/config, run `pyenv local --unset`.

To uninstall a python version, we can either run `pyenv uninstall <version>` or remove the entire *{version}/* directory in *.pyenv/versions/*.

To create virtual environments (our original concern!), we run the command
```bash
pyenv virtualenv <version> <env-name>
```
i.e. we select a `<version>` and specify the name `<env-name>` of the virtual environment we want to create. We can omit `<version>`: in that case, the version currently set will be used to create the virtual environment, as determined by our configuration. The packages installed under this environment will be listed under the directory *.pyenv/versions/{version}/envs/{env-name}*. To activate this environment, we run
```bash
pyenv activate <env-name>
```
and `pyenv deactivate` to - well, deactivate the environment. To list created virtual environments, we run `pyenv virtualenvs`. It is possible to specify a virtual environment in a local *.python-version* file by running 
```bash
pyenv local <env-name>
```
As mentioned before, we can list multiple python versions, environments etc. separated by spaces.

To remove an environment altogether, we can run
```bash
pyenv virtualenv-delete <env-name>
```
or just delete the *{env-name}* directory in *.pyenv/versions/{version}/envs/*. 

After activating, we might want to install necessary packages. One could do this using a *requirements* text file which lists specific versions of packages (perhaps a natural use-case in virtual environments), passed to `pip`:
```bash
pip install -r requirements.txt
```
It could be helpful to use a `--no-cache-dir` option if `pip` uses cached versions which do not match the required version.[^cache]

### Installing ipython kernel

Finally, we could install an **ipython kernel** for a virtual environment, if we use **jupyter notebook** installed for the *system* version. Installing multiple jupyter instances may in general not make sense (?). So we run
```bash
pyenv activate <env-name>
pip install ipykernel ipython_genutils
```
**Note:** Depending upon the project and/or the python version, a specific version of ipykernel might be required. By installing **ipython_genutils** for the environment we can get away without installing **ipython** itself, since it will be installed for the *system* version.

After the installation, we may run
```bash
python -m ipykernel install --user --name=<env-kernel-name>
```
where we type in a name for this kernel. It need not be identical to `<env-name>`. Now when we fire Jupyter Lab/notebook, this kernel should be available. We wouldn't need to activate the virtual environment for this purpose.








[//]: # (Footnotes, if any)

[^sizeissue]: Of course, one can ask here if this defeats the purpose of *not* installing **conda** - but it is probably not necessary to install all of these. It is *suggested* by the devs - the question of potential failure would probably need to be answered on a case-by-case basis. In any case, this step can probably be optimized. *TO BE INVESTIGATED*

[^cache]: This may likely happen when the unlisted dependencies of the packages listed are installed from cache. However, using `--no-cache-dir` might significantly increase install times. Some details [here][pipnocache].





[//]: # (Links, if any)

[1]: <https://docs.python.org/3/library/venv.html>
[2]: <https://docs.conda.io/en/latest/miniconda.html>
[3]: <https://github.com/pyenv/pyenv>
[pyvnv]: <https://github.com/pyenv/pyenv-virtualenv>
[pyvnv-method]: <https://github.com/pyenv/pyenv-virtualenv#virtualenv-and-venv>
[buildenv]: <https://github.com/pyenv/pyenv/wiki#suggested-build-environment>
[shimswork]: <https://github.com/pyenv/pyenv#how-it-works>
[setup]: <https://github.com/pyenv/pyenv#basic-github-checkout>
[pipnocache]: <https://stackoverflow.com/questions/9510474/pip-uses-incorrect-cached-package-version-instead-of-the-user-specified-version/61762308#61762308>