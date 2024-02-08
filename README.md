
# Training LocalRetro using HPE Machine Learning Development Environment (MLDE)
Implementation of experiment tracking, checkpointing, distributed training and hyperparameter search for [LocalRetro model](https://github.com/kaist-amsg/LocalRetro) on HPE MLDE.

# What is LocalRetro?
In chemistry, retrosynthesis focuses on designing reaction pathways and intermediates for a target compound. AI/ML is applied to automate retrosynthesis process by learning from the previous chemical reactions to make new predictions. LocalRetro is a local retrosynthesis framework, [proposed by Shuan Chen and Yousung Jung at KAIST](https://pubs.acs.org/doi/10.1021/jacsau.1c00246) who are motivated by the chemical intuition that the molecular changes occur mostly locally during the chemical reactions, demonstrated promising 89.5 and 99.2% round-trip accuracy at top-1 and top-5 predictions for the USPTO-50K dataset containing 50 016 reactions and round-trip top-1 and top-5 accuracy of 87.0 and 97.4%, respectively, on UTPTO-MIT containing 479 035 reactions. LocalRetro could predict synthesis pathways of five drug candidate molecules from various literature.

## Step 0 - Train LocalRetro using MLDE shell (without model porting)
Advantages of running training/inference processes on MLDE:

- Simplified environment setting: Set up a shell config file shell.yaml that provides a readily built docker image by Determined AI (aka MLDE) and available on DockerHub. These docker images are frequently released and users often just need to pick and choose an existing Docker image published by Determined AI.
- Containerised environment: Training/inference eorkload environment is isolated from the host OS, so users do have constraint with upgrading/downgrading packages
- Automated GPU resource procurement by defining slots number in a config file
- Automated SSH to the launched shell environment without the need to manually setup and manage credentials when connecting to a remote cluster

On local machine, create a MLDE shell/open an existing Determined shell, and install dependencies & packages 
```bash 
git clone https://github.com/kaist-amsg/LocalRetro.git
cd LocalRetro-main
det -m <master ip address> shell create -c .
det -m <master ip address> shell open shell_id

cd /
cd run/determined/workdir

conda env create --file=LocalRetro.yml
conda init bash

exit

det -m <master ip address> shell open <shell-id>
conda activate LocalRetro

apt-get install libxrender1 -y
pip install opencv-python
apt update && apt install -y libsm6 libxext6

cd /
cd run/determined/workdir

cd data/configs
nano default_config.json
cd ..
cd ..
```
After installation is successful, you can start training.
```bash 
cd scripts
python Train.py -d USPTO_50K
```
*Note*: For changes to Conda environment to take effect, you might be asked to close and re-open your current shell. You can also create a Docker image that constain the above dependencies & packages. In such case, you just need to change the link to DockerHub in shell.yaml.

So far, you have run LocalRetro model training on MLDE shell which provides a **simplified environment setting, containerisation, automated GPU resource allocation, automated SSH** benefits.
Furthermore, you can also use MLDE for **experiment tracking, ex., visualising training metrics on a web UI, automated checkpointing, hyperparameter search using MLDE's original implementation of Adaptive ASHA, and distributed training on multiple GPUs (up to 8 GPUs) in a compute node**. In order to leverage these capabilities of MLDE, execute steps 1-4 below using the LocalRetro script ported to MLDE using the MLDE's [CoreAPI](https://hpe-mlde.determined.ai/latest/model-dev-guide/api-guides/apis-howto/api-core-ug-basic.html#get-started-with-core-api).

## Step 1 - Visualize metrics on WebUI 
Run the following command to run the experiment:<br>
```bash
cd step1_metrics
det -m <master_address> e create step1-metrics.yaml .
```

## Step 2 - Automated checkpointing
Run the following command to run the experiment:<br>
```bash
cd step2_checkpoint
det -m <master_address> e create step2-checkpoint.yaml .
```

## Step 3 - Hyperparameter search using Adaptive ASHA 
Run the following command to run the experiment:<br>
```bash
cd step3_hpsearch
det -m <master_address> e create step3-hpsearch.yaml .
```

## Step 4 - Distributed training on multiple GPUs on a single compute node 
Run the following command to run the experiment:<br>
```bash
cd step4_distributed
det -m <master_address> e create step4-distributed.yaml .
```

*Note*: You should consider to load the dgl graph instead of building from scratch to reduce running time for each experiment.
```bash
Processing dgl graphs from scratch... 
Processing molecule 1000/50016
Processing molecule 2000/50016 
... 
Processing molecule 50000/50016 
```