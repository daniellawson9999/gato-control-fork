We are aiming to provide  working and trainable [Gato](https://arxiv.org/abs/2205.06175) implementation for control tasks to be used for [Manifold's Neko](https://github.com/ManifoldRG) or other projects. This implementation is currently in progress.

If you use this project, we'd love for you to refer back to Manifold in your code!

# Setup

## Manual Setup
```bash
conda env create -f env.yml 
```

### Minari
We rely on [Minari](https://minari.farama.org/) to provide a standard for datasets. However, we are currently testing with MuJoCo locomotion and Atari tasks, which are not included in Minari by default. These datasets are generated by: https://github.com/daniellawson9999/data-tests/
and can be downloaded by: 



```bash
git clone https://github.com/Farama-Foundation/Minari.git
cd Minari
pip install -e .

cd ..
python ./gato/data/download_custom_datasets.py
```

## Docker

```bash
docker build -t gato-control -f ./docker/Dockerfile .
docker run -it --mount "type=bind,source=$(pwd),target=/app/gato-control" --entrypoint /bin/bash --gpus=all gato-control

```

# Training
Below are some example training commands. 

Training on 3 MuJoCo locomotion tasks:
```bash
python train.py --embed_dim=768 --layers=6 --heads=24 --training_steps=100000 --log_eval_freq=10000 --warmup_steps=10000 --batch_size=32 -k=240 --eval_episodes=10 --activation_fn=gelu --save_model --save_mode=checkpoint --datasets d4rl_halfcheetah-expert-v2 d4rl_hopper-expert-v2 d4rl_walker2d-expert-v2 -w
```
example run log: https://wandb.ai/daniellawson9999/gato-control/runs/j9u26q9p/overview?workspace=user-daniellawson9999


Atari (in progress):
```bash
python train.py --embed_dim=128 --layers=3 --heads=1 --training_steps=10000 --log_eval_freq=1 --warmup_steps=100 --batch_size=4 -k=512 --eval_episodes=1 --device=cuda --datasets Breakout-expert_s0-v0
```
example run log: https://wandb.ai/daniellawson9999/gato-control/runs/qagorj06/workspace?workspace=user-daniellawson9999\

In general, datasets can contain lists of any strings in download_custom_datasets.py or a dataset in https://minari.farama.org/ with Box or Discrete observation or action spaces, although these environments have not been tested yet. 
for example:
--datasets Breakout-expert_s0-v0 hammer-expert-v0

## Evaluation
```bash
python eval.py --model_path={model_path} --eval_episodes={n_episodes}
```

## Examples

```python
import torch
from gato.policy.gato_policy import GatoPolicy

model = GatoPolicy(
        device='cpu',
        embed_dim=128,
        layers=2,
        heads=4,
        dropout=0.1,
)

# This computes logits and (cross-entropy) loss over a batch of size three, where each diciontary is an episode in the batch

logits, loss = model([
    {
        'images': torch.randn(20, 3, 80, 64),
        'discrete_actions': torch.randint(0, 55, (20, 1)),
    },
    {
        'continuous_obs': torch.randn(15, 8),
        'continuous_actions': torch.randn(15, 4),
    },
    {
        'images': torch.randn(100, 3, 224, 224),
        'continuous_actions': torch.randn(100, 11),
    }
], compute_loss=True)

```

# Future
Our implementation does not directly mirror Gato. Features left out or planned to be added in the future can be found in [todo.md](https://github.com/ManifoldRG/gato-control/blob/master/misc/todo.md).  

# Credits

This implementation is influenced and uses components from:
- https://github.com/OrigamiDream/gato/tree/main
- https://github.com/LAS1520/Gato-A-Generalist-Agent
- [VIMA: General Robot Manipulation with Multimodal Prompts](https://github.com/vimalabs/VIMA)
- [Decision Transformer: Reinforcement Learning via Sequence Modeling](https://github.com/kzl/decision-transformer) 
- [Can Wikipedia Help Offline Reinforcement Learning?](https://github.com/machelreid/can-wikipedia-help-offline-rl)  
