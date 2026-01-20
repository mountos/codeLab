---
title: "Managing AI Agent State with a Flexible TrajectoryManager"
description: "A deep dive into a Python class designed to manage the state and history of an AI agent's operations, with support for multiple storage backends."
pubDate: 2026-01-21T00:00:00Z
categories: ["AI Agentics", "Python"]
tags: ["State Management", "AI", "Architecture", "Storage"]
---

### LOG_ENTRY: 20260121

### SUBJECT: TrajectoryManager Implementation

In the development of autonomous AI agents, managing state and history—what we call a 'trajectory'—is a critical architectural challenge. An agent's trajectory is the complete, ordered sequence of its operations, observations, and state changes. For debugging, analysis, and even for the agent's own long-term planning, a robust trajectory management system is essential.

Today, we're open-sourcing a core component of our agent framework: the `TrajectoryManager`.

### // Core Design Principles

The `TrajectoryManager` was designed with two primary goals:

1.  **Flexibility:** It should not be tied to a single storage mechanism. Local files are great for simple agents, but production systems might require a more robust backend like Redis.
2.  **Simplicity:** The interface for the agent to interact with its own history should be simple and intuitive (`add_step`, `get_trajectory`).

### // The Code

Below is the implementation. It uses a base `BaseStorage` class to define the interface and `FileStorage` for a concrete, file-based implementation.

> The system is initialized with a `trajectory_id` and a storage backend. If no backend is provided, it defaults to `FileStorage`. This allows for easy testing and deployment.

```python
import json
from pathlib import Path
from typing import Any, Dict, List, Optional, Union

# --- Storage Backend Interfaces ---

class BaseStorage:
    """Base class for trajectory storage."""
    def save(self, trajectory: List[Dict[str, Any]]) -> None:
        raise NotImplementedError

    def load(self) -> List[Dict[str, Any]]:
        raise NotImplementedError

class FileStorage(BaseStorage):
    """File-based storage for a single trajectory."""
    def __init__(self, file_path: Union[str, Path]):
        self.file_path = Path(file_path)
        # Ensure the directory exists
        self.file_path.parent.mkdir(parents=True, exist_ok=True)

    def save(self, trajectory: List[Dict[str, Any]]) -> None:
        with open(self.file_path, 'w') as f:
            json.dump(trajectory, f, indent=4)

    def load(self) -> List[Dict[str, Any]]:
        if not self.file_path.exists():
            return []
        with open(self.file_path, 'r') as f:
            return json.load(f)

# --- Trajectory Manager ---

class TrajectoryManager:
    """
    Manages an agent's trajectory, handling storage and retrieval.
    """
    def __init__(self, trajectory_id: str, storage: Optional[BaseStorage] = None):
        self.trajectory_id = trajectory_id
        
        if storage is None:
            # Default to a file-based storage in a local '.trajectories' directory
            file_path = Path(".trajectories") / f"{self.trajectory_id}.json"
            self.storage = FileStorage(file_path)
        else:
            self.storage = storage
            
        self.trajectory = self.storage.load()

    def add_step(self, step_name: str, details: Dict[str, Any]) -> None:
        """
        Adds a new step to the trajectory and saves it.

        Args:
            step_name: The name or type of the step (e.g., 'thought', 'command').
            details: A dictionary containing the content of the step.
        """
        step = {"step": step_name, **details}
        self.trajectory.append(step)
        self.storage.save(self.trajectory)

    def get_trajectory(self) -> List[Dict[str, Any]]:
        """
        Returns the full trajectory.
        """
        return self.trajectory

    def get_last_step(self) -> Optional[Dict[str, Any]]:
        """
        Returns the most recent step in the trajectory.
        """
        if not self.trajectory:
            return None
        return self.trajectory[-1]

    def clear(self) -> None:
        """
        Clears the trajectory from memory and storage.
        """
        self.trajectory = []
        self.storage.save(self.trajectory)
```

### // Usage Example

Using the manager is straightforward:

```python
# 1. Initialize the manager for a specific agent run
session_id = "agent-run-001"
traj_manager = TrajectoryManager(session_id)

# 2. Add steps as the agent operates
traj_manager.add_step("thought", {"text": "I need to find out the capital of France."})
traj_manager.add_step("command", {"name": "web_search", "query": "capital of France"})
traj_manager.add_step("observation", {"result": "The capital of France is Paris."})

# 3. Retrieve the history at any point
full_history = traj_manager.get_trajectory()
print(json.dumps(full_history, indent=2))
```

This component provides a solid foundation for more complex agent behaviors. By swapping the `storage` backend, it can be adapted for different production environments without changing the agent's core logic.

### END_OF_LOG
