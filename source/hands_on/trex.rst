TREX
^^^^^^^

Overview
---------
TREX (Trajectory-ranked Reward Extrapolation) was first proposed in `Extrapolating Beyond Suboptimal Demonstrations via Inverse Reinforcement Learning from Observations <https://arxiv.org/abs/1904.06387>`_, which uses ranked demonstrations to inference reward function. 
Different from previous methods, TREX seeks a reward function that explains the ranking over demonstrations, rather than justifying the demonstrations. This approach is proved practically to have the potential of learning good policy from highly suboptimal demostrations. In addition, the predicted reward only depends on observations, without actions.

Quick Facts
-------------
1. TREX is an Inverse Reinforcement Learning (IRL) algorithm and can be combined with any RL algorithm.

2. Demonstrations used in TREX require **ranking** information.

3. The reward function optimization can be viewed as a simple binary classification problem.

4. TREX is able to infer a meaningful reward function even when noisy, time-based rankings are provided.

Pseudo-code
---------------
.. image:: images/TREX.png
   :align: center
   :scale: 110%
In the pseudo code, theta denotes the parameters of reward function and ti means a trajectory sampled in demonstrations. The functon L is the loss for training reward function, which is explained below.

Key Equations or Key Graphs
---------------------------
The loss for reward function:

.. math::

   \mathcal{L}(\theta)=\mathbb{E}_{\tau_i, \tau_j \sim \Pi}[\xi(\mathcal{P}(\hat J_\theta(\tau_i) < \hat J_\theta(\tau_j)),\tau_i \prec \tau_j)]

where ti, tj is trajectories in demonstrations. In fact, this loss function is only a **binary classification** loss function. Its goal is to figure out the better one of two trajectories. 
The function P is defined as follow:

.. math::

   \mathcal{P}(\hat J_\theta(\tau_i) < \hat J_\theta(\tau_j)) \approx \frac {exp \sum_{s \in \tau_j} \hat r_\theta(s)} {exp \sum_{s \in \tau_i} \hat r_\theta(s) + exp \sum_{s \in \tau_j} \hat r_\theta(s)}

The final loss function is in cross entropy form:

.. math::
   \mathcal{L}(\theta) = - \sum_{\tau_i \prec \tau_j} log \frac {exp \sum_{s \in \tau_j} \hat r_\theta(s)} {exp \sum_{s \in \tau_i} \hat r_\theta(s) + exp \sum_{s \in \tau_j} \hat r_\theta(s)}

Extensions
----------
- TREX can work with noisy ranked demonstrations and is relatively robust to noise of up to around 15% pairwise errors.
- TREX can work with time-based rankings. We can just use data generated from training a certain RL agent. The rankings can be estimated by the time when the trajectory is generated.

Implementations
----------------
TREX can be combined with the following methods:

    - PPO `Proximal Policy Optimization <https://arxiv.org/pdf/1707.06347.pdf>`_

    - SAC `Soft Actor-Critic <https://arxiv.org/pdf/1801.01290>`_

    Given demonstrations generated from RL algorithms or human knowledge (only **observations** and **rankings** are needed), TREX will infer the reward function of the environment. Then the reward function can be applied to RL algorithms like PPO or SAC to estimate rewards while training.

In practical, demonstrations are generated by pretrained policies in our implementation. First, we need to train a RL agent and save the checkpoints every 10000 iterations. Then, we use the different agents to generate demonstrations and rank them according to the final rewards.
According to the paper, we also implement several data augmentation techniques. In the default setting, we sample a part of the trajectories randomly and generate new demonstrations, whose number is equal to the origin ones. The final dataset for training reward function contains both origin demonstrations and generated demonstrations. Note that later states in a trajectory is usually better than the beginning ones, so we ensure that :

.. math::
   for all \tau_i \prec \tau_j, \hat \tau_i, \hat \tau_j is randomly a part sampled from \tau_i and \tau_j. Then the beginning index of \hat \tau_i is no larger than that of \hat \tau_j.
   

The input of the reward model is observations and its output is the predicted reward value. The default reward model is defined as follows:

.. autoclass:: ding.reward_model.TrexRewardModel
   :noindex:


实验 Benchmark
------------------
+---------------------+-----------------+-----------------------------------------------------+----------------------------------------------------+--------------------------+
| environment         |best mean reward | PPO                                                 |                   TREX+PPO                         |    config link           |
+=====================+=================+=====================================================+====================================================+==========================+
|                     |                 |                                                     |                                                    |`config_link_l <https://  |
|                     |                 |                                                     |                                                    |github.com/opendilab/     |
|                     |                 |                                                     |                                                    |DI-engine/tree/main/dizoo/|
|Lunarlander          |  2M env_step,   |.. image:: images/benchmark/lunar_lander_ppo.png     |.. image:: images/benchmark/lunarlander_trex_ppo.png|box2d/lunarlander/config/ |
|                     |  reward 200     |                                                     |                                                    |lunarlander_trex_dqn_     |
|                     |                 |                                                     |                                                    |config.py>`_              |
+---------------------+-----------------+-----------------------------------------------------+----------------------------------------------------+--------------------------+

We use the agents every 1000 iterations(1000-9000) in PPO training process to generate demonstrations. Although the reward of the best demonstration used is about -50 (suboptimal), the agent trained by trex can reach a much better score (200). 

Reference
----------

Daniel S. Brown, Wonjoon Goo, Prabhat Nagarajan, Scott Niekum: "Extrapolating Beyond Suboptimal Demonstrations via Inverse Reinforcement Learning from Observations", 2019; arXiv:1904.06387. https://arxiv.org/abs/1904.06387
