---
title: "01 Multi-armed Bandit"
author: "Kaushal Kishore"
layout: "post"
categories: rl
date: 2022-08-14
---
> Summary of Chapter 2 of Sutton and Barto

## Stationary k-armed Bandit Problem

![Multi-armed Bandit](/img/rl/01-multi-armed-bandit.png)

In any reinforcement learning problem, at any time step t, an agent selects an action $A_t$ from the list of all possible actions and receives a reward $R_t$ from the environment.

In case of stationary k-armed bandit problem, at any time step t, the agent must select one of the k-arms $A_t$, hence there are k-choices (actions) and receives a reward $R_t$ which is sampled from a stationary probability distribution $R(a)$.

The objective of multi-arm bandit problem is to maximize the expected reward over some given number of time steps T.

$$ \text{} \mathop{\max} \mathop{\mathbb{E}}[R_1 + R_2 + \cdots + R_T] $$

### Value of action

At time step t, let $A_t$ be the action taken and $R_t$ be corresponding reward. Then, the value of an arbitary action $a$ is:

$$ q_*(a) \doteq  \mathop{\mathbb{E}}[R_t | A_t = a] $$

$q_*(a)$ is an unknown quantity. We denote the estimate by $Q_t(a)$.

$$ \text{want } Q_t(a) \text{ close to } q_*(a) $$

This estimate is used to select an action which may give us better rewards.

### Action-value methods

Two important steps of any reinforcement learning problem is to:

1. Estimation of $Q_t(a)$
2. Selection of actions

A simple way to estimate $\forall a \in A, Q_t(a)$ is to progressively maintain average rewards received by action.

$$  Q_t(a) \doteq \frac{\sum_{t=1}^{T-1} R_t \cdot \mathop{\mathbb{I}\{A_t = a\}}}{\sum_{t=1}^{T-1}\cdot \mathop{\mathbb{I}\{A_t = a\}}} $$

Where numerator term is `total reward when 'a' taken prior to 't'` and denominator term is `number of times 'a' taken prior to 't'`. When denominator is zero we assign some default value.

Given an estimate $Q_t(a)$, a greedy selection can be made as follows:

$$ A_t \doteq \argmax_a Q_t(a) $$

> Exploration-and-Exploitation: For a better solution there has to be a balance between how much the agent exploits his current knowledge and how much it should explore. With only exploitation, it will not explore other options which can potentially give better rewards in long run. With only exploration, the agent is ever-exploring without using his accumulated knowledge to select an action which gives better reward compared to other actions.

> The balance between exploitation and exploration becomes prominent in a non-stationary action-reward distribution.

Unfortunately, the greedy approach often performs poor. It always **exploits** current knowledge to maximize immediate reward. It ignores exploring other actions which may possibly give better results in future. Hence, a better way is to have a parameter $0\lt\epsilon\lt1$ which is the probability of **exploration**.

Therefore, with probability $\epsilon$ the agents selects an action randomly (exploration) and with probability $1-\epsilon$ agent performs a **greedy choice**.

$$ 
 A_t \doteq \begin{cases}
       X & \text{w.p. $\epsilon$}  \\
       \argmax_a Q_t(a) & \text{w.p. $1-\epsilon$}
     \end{cases}
$$

where $X$ is random variable with a uniform probability over the action set.

### Incremental Implementation

$Q_n(a)$ is the estimate of value of action $a$ after it has been selected $n-1$ times.

$$
\begin{align*}
    Q_n(a) & \doteq \frac{R_1 + R_2 + \cdot + R_{n-1}}{n-1} \\
    Q_{n+1}(a) &= \frac{1}{n} \sum_{i=1}^{n}R_i \\
    &= \frac{1}{n} \left(R_n + \sum_{i=1}^{n-1}R_i \right) \\
    &= \frac{1}{n} \left(R_n + (n-1)Q_n \right)
\end{align*}
$$

$$
\begin{equation*}
\boxed{Q_{n+1}(a) = Q_n + \frac{1}{n} \left[R_n - Q_n \right]}
\end{equation*}
$$

This equation can be interpreted as:

$$
\begin{equation*}
NewEstimate \leftarrow OldEstimate + StepSize \left[Target - OldEstimate \right]
\end{equation*}
$$

where $Target - OldEstimate$ is the error in previous estimate.
