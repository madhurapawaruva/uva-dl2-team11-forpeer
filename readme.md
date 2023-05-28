# LRGB: Long Range Graph Benchmark

<img src="https://i.imgur.com/2LKoGbu.png" align="right" width="275"/>

In this repo, we provide the source code to train various GNN models on the proposed LRGB datasets. We also provide scripts to run baseline and exploratory experiments.


### Python environment setup with Conda

```bash
conda create -n lrgb python=3.9
conda activate lrgb

pip install torch torchvision torchaudio
pip install torch_geometric==2.0.2 performer-pytorch torchmetrics==0.7.2
pip install ogb wandb pytorch_lightning yacs 
pip install torch_scatter torch_sparse 
pip install tensorboardX
pip install numba
pip install e3nn
```
All our important model-checkpoints and results generated are stored in the folder: https://drive.google.com/drive/folders/1-WCfFcvqB2uVr12NtNb7XcX-oUdyx6sF?usp=drive_link.  

### Training the Graph models

The configuration for training graph model on a particular LRGB dataset is provided in src/configs directory. The training process can be started by running the following script,

#### For GCN and Transformers+LapPE

```bash
python src/main.py --cfg src/configs/GCN/vocsuperpixels-GCN.yaml device cuda:0 wandb.use False

python src/main.py --cfg src/configs/GT/vocsuperpixels-Transformer+LapPE.yaml device cuda:0 wandb.use False
```

#### For E(n)-Invariant (EGNN) and E(n)-Equivariant (EGNN2)

```bash
python src/main_egnn.py --cfg src/configs/EGNN/vocsuperpixels-EGNN.yaml device cuda:0 wandb.use False

python src/main_egnn.py --cfg src/configs/EGNN/vocsuperpixels-EGNN2.yaml device cuda:0 wandb.use False
```

### W&B logging
To use W&B logging, set `wandb.use True` and have a `gtransformers` entity set-up in your W&B account (or change it to whatever else you like by setting `wandb.entity`).

### Calculate Influence Scores
After training, the model checkpoints could be loaded up directly by providing the checkpoint path in the configuration at `cfg.train.finetune` and set `cfg.train.freeze_pretrained` to True. To compute influence scores, run src/model_inference.py and provide the relevant configuration file,

```bash
python src/model_inference.py --cfg src/configs/EGNN/vocsuperpixels-EGNN2.yaml device cuda:0 wandb.use False
```
## Accuracy and F1 experiments

To run the experiments that relate accuracy and f1 score to  experiments:
* navigate to the directory containing noising_experiment.py.
* Set up the config files, as was required for model training.
* Run the following terminal command to generate the raw data

``` python noising_experiment.py  --cfg <path_to_config_file> --output_file <path_to_file_to_save_results_to> --device <cuda_device> --num_graphs <number_of_graphs_to_generate_results_for>```

* Upon completion, the program will dump a pickled dataframe containing the results.
* Load that dataframe in the notebook 'noiser_results.ipynb' using
```
df = pre_process_df(output_file_path)
ra = get_relative_accuracy(df)
f1 = get_relative_f1s(df)  
```

There are two pre-prepared pickle files available for use. They are found in the Drive folder. 

## Influence Score experiments

To obtain run the influence score experiments:
* navigate to the directory containing model_inference.py.
* Set up the config files, as was required for model training.
* Run the following terminal command to generate the raw gradient information:

```
python model_inference.py --cfg <path_to_config_file> device 'cuda_device'
```

* This will output a pickle file containing the gradients to "inf_scores_{model_type}.pkl".
* Now run the following code to plot the influence score for a given model:

```
from src.influence import process_all_graphs, plot_mean_influence_by_distance
import matplotlib.pyplot as plt

file_name = './path/to/pickle/
influence_df_gcn = process_all_graphs('inf_scores_gcn_with_adj.pkl', normalise=True)

fig, ax = plt.subplots()
plot_mean_influence_by_distance(influence_df_gcn, ax, 'GCN')

ax.set_xlabel('Shortest path distance from target node')
ax.set_ylabel('Proportion of total gradient')
ax.legend()
```

Alternatively, the notebook 'influence.ipynb' contains an example of this pipeline.

pickle files containing the influence scores of 3 models are avaiable in the Drive folder. 

```assets/influence_experiments/<model_name>.pkl```


### Calculate cheeger constant, diameter and normalized shortest path

```bash
python src/metrices_main.py --cfg src/configs/GCN/vocsuperpixels-GCN.yaml device cuda:0 wandb.use False
```
