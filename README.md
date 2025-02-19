# Isaac Lab RL workspace

[![IsaacSim](https://img.shields.io/badge/IsaacSim-4.2.0-silver.svg)](https://docs.isaacsim.omniverse.nvidia.com/latest/index.html)
[![IsaacLab](https://img.shields.io/badge/IsaacLab-4.2.0-silver.svg)](https://docs.isaacsim.omniverse.nvidia.com/latest/index.html)

[![Python](https://img.shields.io/badge/python-3.10-blue.svg)](https://docs.python.org/3/whatsnew/3.10.html)
[![Linux platform](https://img.shields.io/badge/platform-linux--64-orange.svg)](https://releases.ubuntu.com/22.04/)


## System (verified by developer)

OS: Ubuntu 22.04

## Tool dependency

- [poetry](https://python-poetry.org/) (for virtual env)
- [vcs-tool](https://github.com/dirk-thomas/vcstool) (for cloning repositories)
- [git-lfs](https://docs.github.com/ja/repositories/working-with-files/managing-large-files/about-git-large-file-storage) (for cloning isaaclab extension repositories)

### Tool version verified by developer
- poetry: 1.7.1
- vcs-tool: 0.3.0
- git-lfs: 3.0.2

<details><summary>dependency installation</summary>

### poetry installation

This introduction follows [here](https://python-poetry.org/docs/#installation)

Install [pipx](https://pipx.pypa.io/stable/installation/)
```bash
python3 -m pip install --user pipx
python3 -m pipx ensurepath
```

Install poetry
```bash
pipx install poetry
```
Version 1.7.1 is checked to work well in my environment
```bash
# if you want to install specific version
pipx install poetry==1.7.1
```

To create venv folder in the project, I recommend to execute this.
```bash
poetry config virtualenvs.in-project true
```


### vcs-tool installation

```bash
sudo apt install python3-vcstool
```

### git-lfs installation

```bash
sudo apt install git-lfs
```


</details>


## Installation

1. Clone this repository
```bash
git clone git@github.com:fkfk21/isaaclab_rl_ws.git
```

2. Check the version of CUDA
- Current pyproject.toml is written for CUDA 12.1
- If you want to use different version, edit the versions of source url in pyproject.toml

3. Check the version of Isaac Sim and pytorch
- This repository is verified to work with the version 4.2.0.2 and 4.5.0.0.
- Edit pyproject.toml as your preference.
  - Note: if you use Isaac Sim >= 4.5.0.0, it needs pytorch >= 2.5.1.
  - Note: if you use Isaac Sim == 4.2.0.2, it needs pytorch == 2.4.1.
- pyproject.toml and poetry simplify the installation process of isaac sim
  - official installation process (https://docs.isaacsim.omniverse.nvidia.com/latest/installation/install_python.html)

4. Installation

Execute following commands.

```bash
cd isaaclab_rl_ws
vcs import . < isaaclab.repos  # git-lfs is necessary for this step.
cd isaaclab && git apply ../rsl_rl.patch  # this is needed due to version change of v2.0.2 in rsl-rl if you use isaac lab v1.4.1
poetry install --no-root
poetry run isaaclab/isaaclab.sh -i   # in this process, you have to type "Yes" to accept LICENSE
poetry run pip install -e mevius_isaac_lab/exts/mevius_isaac_lab
```


## Run

Activate virtual env
```bash
# for bash user
source `poetry env info --path`/bin/activate

# for fish shell user
source (poetry env info --path)/bin/activate.fish
```

### for training

```bash
python mevius_isaac_lab/scripts/rsl_rl/train.py --task=Isaac-Velocity-Flat-Mevius-v0 --num_envs 2048 --headless
```
### for resume training

```bash
python mevius_isaac_lab/scripts/rsl_rl/train.py --task=Isaac-Velocity-Flat-Mevius-v0 --resume True --load_run {date}--checkpoint model_{iteration_num}.pt
```

### for playing

```bash
python mevius_isaac_lab/scripts/rsl_rl/play.py --task=Isaac-Velocity-Flat-Mevius-Play-v0 --load_run {date} --checkpoint model_{iteration_num}.pt
```
example (to use the log data in logs/rsl_rl):
- name: mevius_flat
- date: 2024-12-21_02-19-31
- checkpoint: model_299.pt

### for training in the "rough" environment after learning in the "flat" environment

1. copy the folder of "logs/rsl_rl/mevius_flat/{date}" and paste it under "logs/rsl_rl/mevius_rough"
   1. date is the result which store the learning result to resume from
2. run the following code

```bash
python mevius_isaac_lab/scripts/rsl_rl/train.py --task=Isaac-Velocity-Rough-Mevius-v0 --resume True --load_run {date}
```