
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
While neural cellular automata (NCA) have proven effective for modeling morphogenesis and self-organizing processes, they are typically governed by a fixed, non-adaptive update rule shared across all cells. Each cell applies the same learned local transition function throughout its lifetime, resulting in static developmental dynamics once training is complete. We introduce Namesake Neural Cellular Automata (N-NCA), a differentiable Artificial Life substrate that removes this constraint by allowing multiple, independent NCA agents to coexist, compete, and adapt within a shared environment. Unlike conventional NCA, each agent in N-NCA continually updates its parameters via gradient descent during the simulation itself, enabling within-lifetime learning and open-ended behavioral change. This continual, multi-agent learning process transforms morphogenesis from a fixed developmental program into a dynamic ecosystem of interacting, adaptive entities. Through these interactions, N-NCA exhibits emergent behaviors such as cyclic dynamics, cooperation, and persistent complexity growth, providing a promising new framework for studying open-endedness in differentiable systems.

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
Namesake NCA continuously compete for space in a constrained 2D grid. Each different color represents an individual model. The simulation proceeds through 4 distinct phases: (1) processing, (2) competition, (3) normalization, and (4) a state update.
<img width="6000" height="1530" alt="image" src="https://github.com/user-attachments/assets/f6efafd8-525a-4946-9d13-250710ba64ad" />

## Experiments
We ran many experiments to explore N-NCA, namely:

1. Analyzing the complex dynamics of N-NCA over time by measuring information.
2. Inspecting the impact of learning to understand its role in ALife simulation.
3. Hyperparameter searches for noteworthy simulations, using video compression and the open-endedness score from the ASAL [9] framework.
4. Exploration of connection to hypercycles.

## Dynamics
One of our first findings when exploring N-NCA was that scaling the grid size and number of NCA consistently led to richer collective behavior. This suggests that an avenue of exploration must involve engineering N-NCA to run on much larger grids, support more NCA, and potentially run on many GPUs simultaneously. In order to measure the notion of ‘richness’ or ‘interesting behavior’, we cannot only rely on subjective assessment, as this precludes scaling hyperparameter searches. To this end, we propose to measure the amount of information stored on the grid as a proxy for complexity. Since NCA model size is fixed, the simulation behavior complexity is fully explained by input (the grid) complexity and the learned parameters of the individual NCA.

<img width="747" height="459" alt="image" src="https://github.com/user-attachments/assets/832d6d50-0828-42c0-80e8-bd5172d76c66" />
<img width="747" height="459" alt="image" src="https://github.com/user-attachments/assets/97cef719-9123-4ff7-bbfd-4554278d58a8" />

We see in simulations with just 1 NCA, information collapses to 0 as the NCA pushes each channel towards uniformity. With multiple NCA competing on the same grid, information increases over time. While bits/channel would be maximized with fully random channels, there is an optimization force against such an outcome: an NCA yielding pseudo-random updates can be trivially overwritten by an NCA with a strong attack vecto

https://github.com/user-attachments/assets/8fe9591b-a6be-4534-857f-9eb96c430445

Here we show the difference in behavior as we scale from 16 x 16 Grids to 196 x 196. Qualitatively, we see that cycles only emerge at the largest grid size and the grid size also supports the survival of more NCAs.
<img width="747" height="459" alt="image" src="https://github.com/user-attachments/assets/68cfb202-2b3a-4c87-a1d5-829af0285e20" />


## The impact of learning
The videos below explore whether learning has a notable impact on the N-NCA simulation. Without learning, the system eventually settles into a steady state with only minor fluctuation. With learning, however, we often observe interesting cyclic behavior and progression through various ‘states of interaction’. These demonstrations suggest that the number of NCA, grid size, and learning are necessary for the complex simulations that N-NCA can yield.

<img width="1434" height="720" alt="image" src="https://github.com/user-attachments/assets/2f57bfab-835a-4266-a4b7-c1ddce57c0b1" />


## Hyperparameter search
We searched the space of model configurations up to 15 layers, 128 channels (≈500K parameters). We ran up to a maximum of 15 NCA on grids up to 256 × 256 in size. We also included hyperparameters that control the ratio of steps with and without backpropagation as we found this to impact the simulation outcome. To guide this search we leveraged both a classical video compression score and neural score (i.e., the open-endedness score used in ASAL [9]).

<img width="960" height="640" alt="image" src="https://github.com/user-attachments/assets/941c9646-95d1-4d1e-8ed4-3b80b7c61419" />



## Conclusion
Summary We take the first step in seeking to understand whether end-to-end differentiable ALife systems can show signs of open-endedness. In our preliminary experiments, we see signs of cooperation, chemical waves, and the emergence of higher-order structure.

If you would like to discuss any issues or give feedback, please visit the GitHub repository of this page for more information.
