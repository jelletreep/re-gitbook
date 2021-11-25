---
description: >-
  Below are the instructions and commands to quickly setup and server a jupyter
  notebook on an Ubuntu-18.04 with CUDA support
---

# Research Cloud - Setup Jupyter on Ubuntu 18.04 with CUDA

The official guide offered by Surfsara is available [here](https://servicedesk.surfsara.nl/wiki/display/WIKI/RC+plugin+CUDA)

## Step 1: Updates repository list and supporting software

Before starting the setup of this new VM, let's update the Ubuntu packages and add recommended dependencies.

```
# Update repo list
sudo apt update
# Use upgrade only if you can reboot your machine
# sudo apt -y upgrade

# Install supporting software
sudo apt-get update; sudo apt-get install --no-install-recommends make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
sudo apt install libffi-dev python-dev libssl-dev zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev wget
```

## Step 2: Install python 3.8 or 3.9 with Pyenv

Pyenv is a great way to install multiple versions of Python next to each other. You can also use the classical way of installing python  `apt get python3.8` but that is not recommended.

```
# Install pyenv installer
curl https://pyenv.run | bash

# Follow instructions to set pyenv init

# Install python version <X>
pyenv install 3.8.6 
pyenv install 3.9.0
```

Finally, tell pyenv to globally use a version of Python with&#x20;

```
pyenv global 3.8.6
```

Remember that you can even use `pyenv local <version>` to use specific versions of python (previously installed) on a folder/project level.

## Step 3: Set Up a Virtual Environment

One can setup a virtual environment with multiple tools as:

* [Poetry](https://python-poetry.org)
* [Pipenv](https://docs.python-guide.org/dev/virtualenvs/)
* [Anaconda](https://www.anaconda.com/products/individual)

### Using Poetry

```
# Get poetry
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python -

# Source poetry
source $HOME/.poetry/env

# Add poetry to bash-completion
poetry completions bash > /etc/bash_completion.d/poetry.bash-completion
```

### Using Venv

```
# Install
sudo apt install -y python3-venv
```

Now we are ready to create a folder in which we will store environments:&#x20;

```
cd to/folder/you/want/the/environments/to/be/in
mkdir environments
cd environments
```

Finally, we create the environment and activate it

```
python3 -m venv <name_env>

# Activate
source my_env/bin/activate
```

## Step 4: Install and setup Jupyter Lab

Install `jupyter lab`

```
pip install jupyterlab
```

Then let's open a new terminal window. In the first windows let's start a jupyterlab instance

```
jupyter lab --no-browser --port=5678
```

While in the second we connect to this instance (without SSHing in the VM, directly from your local machine)

```
ssh -CNL localhost:5678:localhost:5678 username@hostname
```

{% hint style="info" %}
Remember to change the naming of the machine to your Research Cloud username (usually  _namesurname_) and the hostname (i.e. IP) of the VM.
{% endhint %}

Now in your browser of choice navigate to:

```
localhost:5678
```

And use the token that appeared when creating the instance of Jupyter lab (everything after `token=`) to create a new password and connect to the instance from your browser.

## \[Optional] Step 5: Setup kernel with Poetry and Jupyter lab

If you selected Poetry as package manager, you will need to create a connection between the virtual environment created on Poetry and the kernel used by Jupyter lab.

If you didn't already, create the project as follows:

```
# Start the project
poetry init

# Add basic dependencies
poetry add ipykernel jupyter

# Start poetry
poetry shell
```

To create this connection, do the following:

```
# Within the directory with pyproject.toml
poetry shell

# Create a kernel from poetry
poetry run ipython kernel install --user --name=<NAME>
```

