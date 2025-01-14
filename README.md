[![gridai lightning optuna](https://github.com/robert-s-lee/grid-optuna/actions/workflows/unittest_ubuntu.yml/badge.svg)](https://github.com/robert-s-lee/grid-optuna/actions/workflows/unittest_ubuntu.yml)

[Grid.ai](https://www.grid.ai) can seamlessly train 100s of machine learning models on the cloud from your laptop, with zero code change.
In this example, we will run a model on a laptop, then run the unmodified model on the cloud.  On the cloud, we will run hyperparameter sweeps in parallel 8 ways.  The experiment will **complete 8x faster** with the parallel run.  The cost of the run will be **reduced by 70%** with the spot instance.  

# Overview

We will use familiar [MNIST](http://yann.lecun.com/exdb/mnist/).
Grid.ai is the creators of PyTorch Lightning.  Grid.ai is agnostics to Machine Learning frameworks and 3rd party tools.
The benefits of Grid.ai are available to other Machine Learning frameworks and tools.
To demonstrate this point, we will NOT use [PyTorch Lightning's Early Stop](https://medium.com/pytorch/pytorch-lightning-1-3-lightning-cli-pytorch-profiler-improved-early-stopping-6e0ffd8deb29).
Instead, we will use [Optuna](https://optuna.org) for early stopping.
We will track progress by viewing [PyTorch Lightning](https://www.pytorchlightning.ai)'s [Tensorboard](https://pytorch-lightning.readthedocs.io/en/stable/api/pytorch_lightning.loggers.tensorboard.html) in Grid.ai's [Tensorboard interface](https://docs.grid.ai/products/run-run-and-sweep-github-files/metrics-charts#tensorboard).

Grid.ai will launch experiments in parallel using [Grid Search](https://docs.grid.ai/products/run-run-and-sweep-github-files/sweep-syntax) strategy.  Grid.ai Hyperparameter sweep control `batchsize`, `epochs`, `pruning` -- whether Optuna is active or not. Optuna will control the number of layers, hidden units in each layer, and dropouts within each experiment.  The following combinations will result in 8 parallel experiments:

- batchsize=[32,128]
- epochs=[5,10]
- pruning=[0,1]

A single Grid.ai CLI command initiates the experiment.
 
``` bash
grid run --use_spot pytorch_lightning_simple.py --datadir grid:fashionmnist:1 --pruning="[0,1]"  --batchsize="[32,128]" --epochs="[5,10]"
```

# Step by Step Instruction

This instruction assumes access to a laptop with `bash` and `conda`.  For those with restricted local environment, please use `Jupyter` and click on `Terminal` on [Grid.ai Session](https://docs.grid.ai/products/sessions#start-a-session).

## Local python environment setup

```bash
# conda init bash 
conda init bash # exit and come back
# create conda env
conda create --name gridai python=3.8
conda activate gridai
# install packages
pip install lightning-grid
pip install optuna
pip install pytorch_lightning
pip install torchvision
# login to grid
grid login --username <username> --key <grid api key>
```

## Run locally

```bash
# retrieve the model
git clone https://github.com/robert-s-lee/grid-optuna
cd grid-optuna
mkdir data
# Run without Optuna pruning (takes a while)
python pytorch_lightning_simple.py --datadir ./data
# Run with Optuna pruning (takes a while)
python pytorch_lightning_simple.py --datadir ./data --pruning 1
```

## Prepare Grid.ai Datastore 

Setup [Grid.ai Datastore](https://docs.grid.ai/products/global-cli-configs/cli-api/grid-datastores) so that MNIST data is not downloaded on each run.  Note the **Version** number created.  Typically this will be **1**.

```bash
grid datastore create --source data --name fashionmnist 
grid datastore list # wait until the Status comes back with `Succeeded`
watch -n 10 grid datastore list  # refresh 
┏━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━┓
┃ Credential Id ┃              Name ┃ Version ┃     Size ┃          Created ┃    Status ┃
┡━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━┩
│ cc-qdfdk      │      fashionmnist │       1 │ 141.6 MB │ 2021-06-16 15:13 │ Succeeded │
└───────────────┴───────────────────┴─────────┴──────────┴──────────────────┴───────────┘
```
        
## Run on Grid

- Option 1: with Datastore option so that FashionMNIST is not downloaded again (use on your own or with sharable datastore)  
```bash
grid run --use_spot pytorch_lightning_simple.py --datadir grid:fashionmnist:1 --pruning="[0,1]"  --batchsize="[32,128]" --epochs="[5,10]"
```

- Option 2: without Datastore and can be shared freely without creating datastore 
```bash
grid run --use_spot pytorch_lightning_simple.py --pruning="[0,1]"  --batchsize="[32,128]" --epochs="[5,10]"
```

The above commands will show below (abbreviated)
  
```bash
Run submitted!
`grid status` to list all runs
`grid status smart-dragon-43` to see all experiments for this run
```

`grid status smart-dragon-43` shows experiments running in parallel
  
```bash
% grid status smart-dragon-43
┏━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━┳━━━━━━━━┓
┃ Experiment           ┃                     Command ┃  Status ┃    Duration ┃                  datadir ┃ pruning ┃ batchsize ┃ epochs ┃
┡━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━━╇━━━━━━━━┩
│ smart-dragon-43-exp7 │ pytorch_lightning_simple.py │ running │ 0d-00:07:24 │ /datastores/fashionmnist │       1 │        32 │     10 │
│ smart-dragon-43-exp6 │ pytorch_lightning_simple.py │ running │ 0d-00:07:27 │ /datastores/fashionmnist │       1 │        32 │      5 │
│ smart-dragon-43-exp5 │ pytorch_lightning_simple.py │ running │ 0d-00:07:14 │ /datastores/fashionmnist │       1 │       128 │      5 │
│ smart-dragon-43-exp4 │ pytorch_lightning_simple.py │ pending │ 0d-00:12:52 │ /datastores/fashionmnist │       0 │       128 │      5 │
│ smart-dragon-43-exp3 │ pytorch_lightning_simple.py │ running │ 0d-00:07:13 │ /datastores/fashionmnist │       0 │        32 │     10 │
│ smart-dragon-43-exp2 │ pytorch_lightning_simple.py │ running │ 0d-00:07:03 │ /datastores/fashionmnist │       0 │       128 │     10 │
│ smart-dragon-43-exp1 │ pytorch_lightning_simple.py │ running │ 0d-00:07:02 │ /datastores/fashionmnist │       1 │       128 │     10 │
│ smart-dragon-43-exp0 │ pytorch_lightning_simple.py │ pending │ 0d-00:12:52 │ /datastores/fashionmnist │       0 │        32 │      5 │
└──────────────────────┴─────────────────────────────┴─────────┴─────────────┴──────────────────────────┴─────────┴───────────┴────────┘
```

`grid logs smart-dragon-43-exp0` shows logs from that experiment

```bash
grid logs smart-dragon-43
```
## Simpler variations to run

```bash
grid run --use_spot pytorch_lightning_simple.py
grid run --use_spot pytorch_lightning_simple.py --datadir grid:fashionmnist:1"
```

## Use Grid.ai WebUI for Tensorboard graphs

Example of on-demand pricing (top at $0.09) and spot pricing (bottom at $0.03)

![](images/on-demand-spot-cost.png)

Example Metric from Grid.ai WebUI

![](images/grid-val-acc.png)

Example Metric from Tensorboard

![](images/tensorboard-parallel-coord.png)



