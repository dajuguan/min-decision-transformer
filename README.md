# Decision Transformer

## Install
```
# conda
conda install -c conda-forge gcc=12.1.0  # for 'GLIBCXX_3.4.30' not found
conda activate py3.8
pip install cython==3.0.0a8 # for Exception check on 'c_warning_callback' will always require the GIL to be acquired.

# mujoco_py
sudo apt install libosmesa6-dev libgl1-mesa-glx libglfw3 patchelf
wget https://roboti.us/download/mujoco200_linux.zip
wget https://roboti.us/file/mjkey.txt
mkdir ~/.mujoco
unzip mujoco200_linux.zip -d ~/.mujoco/
cp -r ~/.mujoco/mujoco200_linux ~/.mujoco/mujoco200
mv mjkey.txt ~/.mujoco/
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/root/.mujoco/mujoco200/bin
pip install mujoco_py==2.0.2.8

# 
git clone https://github.com/rail-berkeley/d4rl.git
sed -i "s;dm_control @ git+git://github.com/deepmind/dm_control@master#egg=dm_control;dm_control==0.0.364896371;g" ./d4rl/setup.py
sed -i "s;mjrl @ git+git://github.com/aravindr93/mjrl@master#egg=mjrl;mjrl @ git+https://github.com/aravindr93/mjrl@master#egg=mjrl;g" ./d4rl/setup.py
pip install -e d4rl/.

## ignore Warning: Flow/CARLA failed to import
pip install setuptools==65.5.0 # for gym compatibility
pip install --user wheel==0.38.0

# min-decision-transformer
cd min-decision-transformer
pip install -r requirements.txt
## download data
python data/download_d4rl_datasets.py
python scripts/test.py
# for cuda compatibility issue (cuda version > 12.1)
pip uninstall torch torchvision
pip cache purge
pip install torch==1.9.0+cu111 -f https://download.pytorch.org/whl/torch_stable.html
#Ref https://discuss.pytorch.org/t/pytorch-for-cuda-12/169447/38
```

## Test and Train
```
cd scripts
python test.py
python train.py
训练200 iters(20000 updates)所需时间: 
```

## Overview

Minimal code for [Decision Transformer: Reinforcement Learning via Sequence Modeling](https://arxiv.org/abs/2106.01345) for mujoco control tasks in OpenAI gym.
Notable difference from official implementation are:

- Simple GPT implementation (causal transformer)
- Uses PyTorch's Dataset and Dataloader class and removes redundant computations for calculating rewards to go and state normalization for efficient training
- Can be trained and the results can be visualized and rendered on google colab with the provided notebook

#### [Open `min_decision_transformer.ipynb` in Google Colab](https://colab.research.google.com/github/nikhilbarhate99/min-decision-transformer/blob/master/min_decision_transformer.ipynb) [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nikhilbarhate99/min-decision-transformer/blob/master/min_decision_transformer.ipynb)



## Results

**Note:** these results are mean and variance of 3 random seeds obtained after 20k updates (due to timelimits on GPU resources on colab) while the official results are obtained after 100k updates. So these numbers are not directly comparable, but they can be used as rough reference points along with their corresponding plots to measure the learning progress of the model. The variance in returns and scores should decrease as training reaches saturation.


| Dataset | Environment | DT (this repo) 20k updates | DT (official) 100k updates|
| :---: | :---: | :---: | :---: |
| Medium | HalfCheetah | 42.18 ± 00.59 | 42.60 ± 00.10 |
| Medium | Hopper | 69.43 ± 27.34 | 67.60 ± 01.00 |
| Medium | Walker | 75.47 ± 31.08 | 74.00 ± 01.40 |


| ![](https://github.com/nikhilbarhate99/min-decision-transformer/blob/master/media/halfcheetah-medium-v2.png)  | ![](https://github.com/nikhilbarhate99/min-decision-transformer/blob/master/media/halfcheetah-medium-v2.gif)  |
| :---:|:---: |


| ![](https://github.com/nikhilbarhate99/min-decision-transformer/blob/master/media/hopper-medium-v2.png)  | ![](https://github.com/nikhilbarhate99/min-decision-transformer/blob/master/media/hopper-medium-v2.gif)  |
| :---:|:---: |


| ![](https://github.com/nikhilbarhate99/min-decision-transformer/blob/master/media/walker2d-medium-v2.png)  | ![](https://github.com/nikhilbarhate99/min-decision-transformer/blob/master/media/walker2d-medium-v2.gif)  |
| :---:|:---: |



## Instructions

### Mujoco-py

Install `mujoco-py` library by following instructions on [mujoco-py repo](https://github.com/openai/mujoco-py)


### D4RL Data

Datasets are expected to be stored in the `data` directory. Install the [D4RL repo](https://github.com/rail-berkeley/d4rl). Then save formatted data in the `data` directory by running the following script:
```
python3 data/download_d4rl_datasets.py
```


### Running experiments

- Example command for training:
```
python3 scripts/train.py --env halfcheetah --dataset medium --device cuda
```


- Example command for testing with a pretrained model:
```
python3 scripts/test.py --env halfcheetah --dataset medium --device cpu --num_eval_ep 1 --chk_pt_name dt_halfcheetah-medium-v2_model_22-02-13-09-03-10_best.pt
```
The `dataset` needs to be specified for testing, to load the same state normalization statistics (mean and var) that is used for training.
An additional `--render` flag can be passed to the script for rendering the test episode.


- Example command for plotting graphs using logged data from the csv files:
```
python3 scripts/plot.py --env_d4rl_name halfcheetah-medium-v2 --smoothing_window 5
```
Additionally `--plot_avg` and `--save_fig` flags can be passed to the script to average all values in one plot and to save the figure.


### Note:
1. If you find it difficult to install `mujoco-py` and `d4rl` then you can refer to their installation in the colab notebook
2. Once the dataset is formatted and saved with `download_d4rl_datasets.py`, `d4rl` library is not required further for training.
3. The evaluation is done on `v3` control environments in `mujoco-py` so that the results are consistent with the decision transformer paper.


## Citing

Please use this bibtex if you want to cite this repository in your publications:

    @misc{minimal_decision_transformer,
        author = {Barhate, Nikhil},
        title = {Minimal Implementation of Decision Transformer},
        year = {2022},
        publisher = {GitHub},
        journal = {GitHub repository},
        howpublished = {\url{https://github.com/nikhilbarhate99/min-decision-transformer}},
    }



## References

- Official [code](https://github.com/kzl/decision-transformer) and [paper](https://arxiv.org/abs/2106.01345)
- Minimal GPT (causal transformer) [tweet](https://twitter.com/MishaLaskin/status/1481767788775628801?cxt=HHwWgoCzmYD9pZApAAAA) and [colab notebook](https://colab.research.google.com/drive/1NUBqyboDcGte5qAJKOl8gaJC28V_73Iv?usp=sharing)
