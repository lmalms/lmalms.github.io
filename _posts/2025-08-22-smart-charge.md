---
layout: post
title: "Smart Charge - Optimising Battery Charging Using MPC"
date: 2025-08-22
categories: blog
---

### Introduction

Smart Charge is a project that explores how linear programming and model predictive control (MPC) can be used to optimise battery charging. The aim of the project was to learn more about these techniques and their application to problems in the energy sector. The problem setup is intentionally simple and uses synthetic data so that I could focus on understanding the fundamentals before tackling more complex systems.

### Problem Setup

The energy system setup for this problem is illustrated below. It consists of a generation, storage, and consumption node, represented by the energy grid, battery, and house icons, respectively. In this system, energy can only flow from the grid to the battery, and from the battery to the house. Given some forecasts of future electricity prices and demand, the objective of the system is to minimise the cost of electricity consumed whilst meeting demand.

![system-diagram](/assets/smart-charge/system-diagram.png)
*Illustration of home energy system. Generated using ChatGPT.*

### MPC Formulation

#### Variables

- $c_k$ - *battery state of charge at time $k$*
- $s_k$ - *supply of energy drawn from electricity grid at time $k$*
- $d_k$ - *demand of energy drawn from battery at time $k$*
- $p_k$ - *price of electricity at time $k$*
- $t$ - *battery capacity*

#### Objective

The objective is to minimise the total cost of electricity consumed. Since in this simplified setup, electricity can only be consumed from the battery node, this is equivalent to minimising the cost of charging the battery.

$$
\underset{s_0:\tau}{\min} \sum_{k = 0}^{\tau} \gamma^k s_k p_k
$$

where $\tau$ is the optimisation horizon and $\gamma$ is a discount factor (0.95).

#### Constraints

Electricity flow has to be positive at all timesteps:

$$
s_k \ge 0 \quad \forall k \in \{0, 1, \ldots, \tau\}
$$

Demand has to be met at all timesteps:

$$
s_k \ge d_k - c_k \quad \forall k \in \{0, 1, \ldots, \tau\}
$$

Battery state of charge cannot exceed total capacity:

$$
s_k + c_k - d_k \le t \quad \forall k \in \{0, 1, \ldots, \tau\}
$$

Battery state of charge between subsequent timesteps has to evolve according to:

$$
c_{k + 1} = c_k + s_k - d_k \quad \forall k \in \{0, 1, \ldots, \tau - 1\}
$$

### Results

I ran the MPC agent on the following three scenarios and compared its performance against a greedy agent. The code for running both agents on this problem can be found [here](https://github.com/lmalms/linear-learners/tree/main/battery-charging).

- **Scenario 1:** Price and demand forecasts perfectly aligned, optimisation horizon $\tau$ of 15 timesteps.
- **Scenario 2:** Price and demand forecasts perfectly aligned, optimisation horizon $\tau$ of 8 timesteps.
- **Scenario 3:** Price and demand forecasts slightly misaligned and include noise.

#### Scenario 1

The results from the first scenario for both the greedy and MPC agents are shown below. The greedy agent optimises only for the current timestep, using only the current price and demand data. The optimal action for all timesteps is therefore to only draw from the grid whatever is currently being consumed. As a result, the grid flow for the greedy agent closely follows the demand curve. The exception are the first few timesteps where the agent discharges from the battery directly (initial state of charge is 20 kWh) before then drawing from the grid at later timesteps.

The MPC agent on the other hand, has access to future demand and electricity prices and optimises its charging behaviour accordingly. It charges the battery to maximum capacity when prices are lowest and then maintains that charge by drawing only enough to meet demand. During peak prices, it discharges the battery to avoid expensive grid electricity, effectively using the battery as a buffer. After 72 timesteps, the MPC agent reduces the total cost of charge by approximately 28% compared to the greedy agent.

![aligned-horizon-15](/assets/smart-charge/aligned_horizon_15.png)
*Results from scenario 1.*

#### Scenario 2

The results from the second scenario are similar to those from the first, the main difference being the MPC agent’s charging behaviour around the electricity price minima. The MPC agent again draws the majority of its charge at the lowest electricity prices, but in contrast to the first scenario it does not charge the battery completely (capacity is 50 kWh) when prices are at their minimum. This is because the agent only has access to the next 8 price and demand forecasts, compared to the next 15 timesteps in the first scenario. At the price minimum, the agent therefore only charges enough to cover demand for the next 8 timesteps which is less than the battery's total capacity.
Over the next few timesteps, the agent eventually charges the battery to full capacity but it cannot take full advantage of the price minimum as in the first scenario. As a result, the total cost of charge after 72 timesteps is slightly higher than in the first scenario.

![aligned-horizon-8](/assets/smart-charge/aligned_horizon_8.png)
*Results from scenario 2.*

#### Scenario 3

In the final scenario, the MPC agent again has access to forecasts for the next 15 timesteps. The price and demand curves are now slightly misaligned and include some noise, but still follow a broadly similar pattern. As a result, the MPC agent charges the battery to full capacity when electricity prices are lowest, but also takes advantage of local price minima by drawing extra electricity to meet future demand with cheaper energy. In this scenario the MPC agent outperforms the greedy agent by approximately 36% after 72 timesteps.

![misaligned](/assets/smart-charge/misaligned.png)
*Results from scenario 3.*
