
<h1 align="center">
  <a href="https://neurolifeblog.com">
    <img width="1267" height="365" alt="Screenshot 2026-03-08 222634" src="https://github.com/user-attachments/assets/376f26d1-4395-47a3-ad5a-f0429dd22906" /></a><br>
</h1>


<h1 align="center">
Namesake Neural Cellular Automata
</h1>
<p align="center">
  <a href="https://neurolifeblog.com">Paper</a>
</p>

</h1>
<p align="center">
  <a href="https://x.com/allankjensen">allankjensen</a>
</p>


## Abstract
While neural cellular automata (NCA) have proven effective for modeling morphogenesis and self-organizing processes, they are typically governed by a fixed, non-adaptive update rule shared across all cells. Each cell applies the same learned local transition function throughout its lifetime, resulting in static developmental dynamics once training is complete. We introduce Namesake Neural Cellular Automata (PD-NCA), a differentiable Artificial Life substrate that removes this constraint by allowing multiple, independent NCA agents to coexist, compete, and adapt within a shared environment. Unlike conventional NCA, each agent in PD-NCA continually updates its parameters via gradient descent during the simulation itself, enabling within-lifetime learning and open-ended behavioral change. This continual, multi-agent learning process transforms morphogenesis from a fixed developmental program into a dynamic ecosystem of interacting, adaptive entities. Through these interactions, PD-NCA exhibits emergent behaviors such as cyclic dynamics, cooperation, and persistent complexity growth, providing a promising new framework for studying open-endedness in differentiable systems.

<div style="display: flex; justify-content: space-between;">
  <img width="949" height="407" alt="Screenshot 2026-03-08 222634" src="https://github.com/user-attachments/assets/82dde020-4a98-4af7-a0e5-bbf26cbfd346" />

</div>


# Namesake NCA

What happens if we are able to put multiple different NCAs in a single substrate that compete for space?

## Setup 

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh         # Or install uv another way

git clone https://github.com/SakanaAI/petri-dish-nca
cd petri-dish-nca
uv sync
```

## Commands

- basic: `uv run python src/train.py --n-ncas 3 --epochs 1000 --device cpu`
- wandb logging: `uv run python src/train.py --n-ncas 3 --epochs 10000 --device cuda --wandb`
- run with config: `uv run python src/train.py --config configs/example.json`

## Configs

For additional configurations, you can load a JSON config file. Any parameters not specified in the config file will be set their default value in `src/config.py`

## Method
Petri Dish NCA continuously compete for space in a constrained 2D grid. Each different color represents an individual model. The simulation proceeds through 4 distinct phases: (1) processing, (2) competition, (3) normalization, and (4) a state update.
<img width="6000" height="1530" alt="image" src="https://github.com/user-attachments/assets/f6efafd8-525a-4946-9d13-250710ba64ad" />

## Experiments
We ran many experiments to explore PD-NCA, namely:

1. Analyzing the complex dynamics of PD-NCA over time by measuring information.
2. Inspecting the impact of learning to understand its role in ALife simulation.
3. Hyperparameter searches for noteworthy simulations, using video compression and the open-endedness score from the ASAL [9] framework.
4. Exploration of connection to hypercycles.

## Dynamics
One of our first findings when exploring PD-NCA was that scaling the grid size and number of NCA consistently led to richer collective behavior. This suggests that an avenue of exploration must involve engineering PD-NCA to run on much larger grids, support more NCA, and potentially run on many GPUs simultaneously. In order to measure the notion of ‘richness’ or ‘interesting behavior’, we cannot only rely on subjective assessment, as this precludes scaling hyperparameter searches. To this end, we propose to measure the amount of information stored on the grid as a proxy for complexity. Since NCA model size is fixed, the simulation behavior complexity is fully explained by input (the grid) complexity and the learned parameters of the individual NCA.

<img width="747" height="459" alt="image" src="https://github.com/user-attachments/assets/832d6d50-0828-42c0-80e8-bd5172d76c66" />
<img width="747" height="459" alt="image" src="https://github.com/user-attachments/assets/97cef719-9123-4ff7-bbfd-4554278d58a8" />

We see in simulations with just 1 NCA, information collapses to 0 as the NCA pushes each channel towards uniformity. With multiple NCA competing on the same grid, information increases over time. While bits/channel would be maximized with fully random channels, there is an optimization force against such an outcome: an NCA yielding pseudo-random updates can be trivially overwritten by an NCA with a strong attack vecto

<img width="3600" height="720" alt="image" src="https://github.com/user-attachments/assets/83cbe8f8-857b-44bd-bde4-18eafd7a5e9d" />


https://github.com/user-attachments/assets/8fe9591b-a6be-4534-857f-9eb96c430445




### Supervised Target & Open-Endedness
Use [main_opt.py](main_opt.py) with Sep-CMA-ES (from evosax):
```sh
# Search for a Lenia simulation matching a single prompt
python main_opt.py --seed=0 --save_dir="./data/supervised_0" --substrate='lenia' \
  --time_sampling=1 --prompts="a caterpillar" --coef_prompt=1. --coef_softmax=0. \
  --coef_oe=0. --bs=1 --pop_size=16 --n_iters=1000 --sigma=0.1

# Search for temporal targets (sequence of events)
python main_opt.py --seed=4 --save_dir="./data/supervised_temporal_4" --substrate='lenia' \
  --time_sampling=2 --prompts="a small biological cell;a large biological cell" \
  --coef_prompt=1. --coef_softmax=0.1 --coef_oe=0. --bs=1 --pop_size=16 --n_iters=1000 --sigma=0.1

# Search for open-ended simulations
python main_opt.py --seed=3 --save_dir="./data/open_endedness_3" --substrate='lenia' \
  --time_sampling=32 --prompts="" --coef_prompt=0. --coef_softmax=0. --coef_oe=1. \
  --bs=1 --pop_size=16 --n_iters=1000 --sigma=0.1

# Combine supervised target + open-endedness
python main_opt.py --seed=0 --save_dir="./data/st_oe_0" --substrate='lenia' \
  --time_sampling=32 --prompts="a diverse ecosystem of cells" --coef_prompt=1. \
  --coef_softmax=0. --coef_oe=1. --bs=1 --pop_size=16 --n_iters=1000 --sigma=0.1
```

### Illumination
Use [main_illuminate.py](main_illuminate.py) with a custom genetic algorithm:
```sh
python main_illuminate.py --seed=0 --save_dir="./data/illuminate_0" --substrate='lenia' \
  --n_child=32 --pop_size=256 --n_iters=1000 --sigma=0.1
```

### Game of Life Sweep
Use [main_sweep_gol.py](main_sweep_gol.py) with brute force search (discrete search space):
```sh
python main_sweep_gol.py
```

[neurolife.ipynb](neurolife.ipynb) goes through everything you need to know in detail, including how to visualize results.

## Tips for Getting Good Results

Some tips on getting the results you want from a given substrate:
- **Run for many seeds.** Meta optimizing chaotic systems is quite tough and noisy. More seeds can significantly increase the chances of getting what you want.
- **Play around with different hyperparameters.**
    - When doing the supervised temporal targets with multiple prompts, `coef_softmax` needs to be tuned.
    - Pay attention to the `time_sampling` variable and tune it. For open-endedness, try setting `time_sampling` to a large value (32 works well).
- **Run for more search iterations.** This can get you quite far sometimes.
- **Pick good expressive substrates.** Some substrates aren't capable of expressing certain lifeforms. Check out the discussion section of the paper.
- **Use better search algorithms!** We used Sep-CMA-ES, SGD, GAs, and brute force search in the paper, but other search algorithms may be better for certain substrates you choose.
- **CLIP seems to be a good choice of foundation model.** I don't think this is the bottleneck. But newer foundation models may be better than CLIP.

Particle Life++ has lots of potential and is an extremely untapped substrate so far, so experimenting with that would be very cool, although optimization is hard in that, due to how chaotic it is.

If you get bored, it would be amazing to apply NeuroLife to newer substrates like [ALIEN](https://www.youtube.com/watch?v=qwbMGPkoJmg) and [JaxLife](https://github.com/luchris429/jaxlife)!

## Running Locally
### Installation

To run this project locally, you can start by cloning this repo.
```sh
git clone https://github.com/neurolife-org/NeuroLife.git
```
Then, set up the python environment with conda:
```sh
conda create --name neurolife python=3.10.13
conda activate neurolife
```

Then, install the necessary python libraries:
```sh
python -m pip install -r requirements.txt
```
However, if you want GPU acceleration (trust me, you do), please [manually install jax](https://github.com/jax-ml/jax?tab=readme-ov-file#installation) according to your system's CUDA version.

### Loading Our Dataset of Simulations

We provide pre-computed datasets from our large scale searches:

- The Lenia and Boids dataset contains **8192 simulations** found in a large illumination search.
- The GoL dataset contains the **262,144 simulations** ranked in order of most open-ended to least open-ended.

You can view our dataset of simulations at:
- [Lenia Dataset](https://pub.sakana.ai/asal/data/illumination_poster_lenia.png)
- [Boids Dataset](https://pub.sakana.ai/asal/data/illumination_poster_boids.png)

You can download the datasets from:
```sh
wget -P ./data https://pub.sakana.ai/asal/data/illumination_lenia.npz
wget -P ./data https://pub.sakana.ai/asal/data/illumination_boids.npz
wget -P ./data https://pub.sakana.ai/asal/data/illumination_plife.npz
wget -P ./data https://pub.sakana.ai/asal/data/sweep_gol.npz
```

Here's how to load and visualize a specific simulation from the dataset:
```python
import numpy as np
from functools import partial
import jax
import substrates
from rollout import rollout_simulation

params_all = np.load("./data/illumination_lenia.npz", allow_pickle=True)['params']
print(params_all.shape) # shape: (number of simulations, parameter dimension)
params = params_all[6198] # visualize simulation #6198

substrate = substrates.create_substrate('lenia')
substrate = substrates.FlattenSubstrateParameters(substrate)

rollout_fn = partial(rollout_simulation, s0=None, substrate=substrate, fm=None,
                     rollout_steps=substrate.rollout_steps, time_sampling=8,
                     img_size=224, return_state=False)
rollout_fn = jax.jit(rollout_fn)

rng = jax.random.PRNGKey(0)
rollout_data = rollout_fn(rng, params)
# rollout_data['rgb'] has shape (8, 224, 224, 3)
```

Directions on how to visualize and load these simulations in more detail are shown in [neurolife.ipynb](neurolife.ipynb).

## Reproducing Results from the Paper
Everything you need is already in this repo.

## Bibtex Citation
To cite our work, you can use the following:
```
@article{chase2024neurolife,
  title = {Automating the Search for Artificial Life with Foundation Models},
  author = {Lee Chase},
  year = {2024},
  url = {https://neurolifeblog.com}
}
```
