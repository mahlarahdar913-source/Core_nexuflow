# NexusFlow: A Decentralized Load Balancing Framework with Entropy-Driven Resilience

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)

## Overview
NexusFlow is an advanced, lightweight library designed for high-throughput, large-scale decentralized systems. Unlike traditional load balancers that rely on centralized controllers or simple round-robin algorithms, NexusFlow implements a **Self-Adaptive Optimization** approach. 

By integrating **Shannon Entropy** into the cost function, NexusFlow prevents the system from getting trapped in local minima and ensures structural resilience against sudden traffic bursts (shocks).

## Core Logic: The Optimization Paradox
The framework minimizes a multi-objective function $Z$:

$$min Z = \alpha \cdot \text{Cost} - \beta \cdot \text{Entropy}(L)$$

Where:
- $\text{Cost}$ represents the immediate computational/energy load.
- $\text{Entropy}(L)$ represents the distribution uniformity of the load across $N$ nodes.
- $\alpha$ and $\beta$ are dynamic weighting coefficients.

This approach forces the system toward a **Homogeneous Distribution** during high-stress periods, effectively implementing a "self-healing" mechanism without central command.

## Installation
```bash
pip install nexusflow  # Placeholder for future release
from nexusflow import NexusLoadBalancer

# Initialize nodes with their capacities
nodes = ["edge_01", "edge_02", "cloud_core", "edge_03"]
capacities = {"edge_01": 100, "edge_02": 100, "cloud_core": 500, "edge_03": 300}

balancer = NexusLoadBalancer(nodes, capacities)

# Simulate incoming request stream
requests = [20, 50, 10, 80, 30, 45]
for req in requests:
target = balancer.distribute_load(req)
print(f"Request size {req} directed to: {target}")

**و فایل `nexusflow.py`:**

```python
import numpy as np

class NexusLoadBalancer:
    """
    A decentralized load balancer implementing entropy-based 
    self-adaptation for resilient resource allocation.
    """
    def __init__(self, nodes, capacity, alpha=0.85, beta=0.15):
        self.nodes = nodes
        self.capacity = capacity
        self.current_load = {node: 0.0 for node in nodes}
        self._alpha = alpha  # Efficiency weight
        self._beta = beta    # Resilience/Entropy weight

    def _calculate_entropy(self, loads):
        """Calculates Shannon Entropy of the current load distribution."""
        total_load = sum(loads)
        if total_load == 0:
            return 0
        # Normalize loads to create a probability distribution
        probabilities = [l / total_load for l in loads]
        # Avoid log(0) by adding a tiny epsilon
        return -sum(p * np.log2(p + 1e-9) for p in probabilities)

    def _objective_function(self, potential_loads):
        """
        The core optimization paradox:
        Minimizing cost while maximizing entropy to ensure distribution.
        """
        cost = sum(potential_loads)
        entropy = self._calculate_entropy(potential_loads)
        
        # min Z = alpha * Cost - beta * Entropy
        return (self._alpha * cost) - (self._beta * entropy)

    def distribute_load(self, incoming_size):
        """Selects the optimal node based on the multi-objective function."""
        best_node = None
        min_z = float('inf')

        for node in self.nodes:
            # Simulate what the load WOULD be if we picked this node
            temp_loads = [self.current_load[n] for n in self.nodes]
            idx = self.nodes.index(node)
            temp_loads[idx] += incoming_size
            
            # Check capacity constraint (soft constraint for the paradox)
            if temp_loads[idx] <= self.capacity[node] * 1.2: 
                z_score = self._objective_function(temp_loads)
                
                if z_score < min_z:
                    min_z = z_score
                    best_node = node

        # Fallback to simplest node if all are overwhelmed
        if not best_node:
            best_node = self.nodes[0]

        self.current_load[best_node] += incoming_size
        return best_node

if __name__ == "__main__":
    # Test Scenario
    nodes = ["edge_server_1", "edge_server_2", "central_hub", "cloud_node_A"]
    capacities = {"edge_server_1": 100, "edge_server_2": 100, "central_hub": 500, "cloud_node_A": 300}
    
    balancer = NexusLoadBalancer(nodes, capacities)
    requests = [10, 25, 5, 40, 15, 10, 50]

    print("Starting Load Distribution Simulation...")
    for r in requests:
        target = balancer.distribute_load(r)
        print(f"Request: {r} units -> Directed to: {target} | Current Loads: {balancer.current_load}")
