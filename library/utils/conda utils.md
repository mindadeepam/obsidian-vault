# conda utils

## install conda

[link](https://docs.anaconda.com/free/miniconda/index.html#quick-command-line-install)
These four commands quickly and quietly install the latest 64-bit version of the installer and then clean up after themselves. To install a different version or architecture of Miniconda for Linux, change the name of the `.sh` installer in the `wget` command.

```
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
```

After installing, initialize your newly-installed Miniconda. The following commands initialize for bash and zsh shells:

```
~/miniconda3/bin/conda init bash
~/miniconda3/bin/conda init zsh
```

`source ~/.bashrc` to apply changes. now run `conda` to see if c

## list all environments

`conda env list`

## list packages in an environment

`conda list`                                           # if env is active

`conda list -n name_of_env`             # any env

## save environment to a yml file

`conda env export > environment.yml`

## create environment using a yml file

`conda env create --name envname --file=environments.yml`

## update env using yml file

`conda env update --file local.yml --prune`

- *Attention:*

  If there is a `name` tag with a name other than that of your environment in `local.yml`, the command above will create a new environment with that name. To avoid this, use (thanks @NumesSanguis):

  ```
  conda env update --name myenv --file local.yml --prune
  ```

## remove environment

`conda env remove -n <your environment name>`

## conda in jupyter

- **setup**

  Since everyone works on different projects, they might need to have different Python working environments. It is easy to create new conda environments in the notebooks

  ```
  # create a new conda env:
  # The below command creates a new environemnt named deepak_python39 with python 3.9
  $ conda create -n $env_name python=3.9

  # Activate your new environment: (Name of the environment is important)
  $ conda activate $env_name

  #install ipykernel when logged in the new env:
  # This is required for the environment to be shown in the notebook interface
  (python39)$ conda install ipykernel

  # install the ipython kernel at user level
  # Continuation of the above step
  (python39)$ ipython kernel install --user --name=$env_name
  # or if no module named platformdirs error
  (python39)$ python -m ipykernel install --user --name=qc

  # Deactivate the new environment
  (python39)$ conda deactivate
  ```
- **other commands for conda X jupyter**

  ### list all kernels in jupyer

  `jupyter kernelspec list`

  ### remove jupyter kernel

  `jupyter kernelspec uninstall unwanted-kernel`
