# VERGE formal verification service

This repository contains the container image with the RL model verification service for [VERGE](https://www.verge-project.eu/) platform (SPT4AI). Verification uses [PRISM](https://github.com/prismmodelchecker/prism) probabilistic symbolic model checker to verify the safety properties of and RL model.

## Using the PRISM model checker directly

The image contains PRISM as an executable, compiled from version [v4.8](https://github.com/prismmodelchecker/prism/releases/tag/v4.8) of PRISM. To run it from the container use the following:

```bash
prism <prism model> -pf <safety properties>
```

## Using the PRISM model checker using the simple API

The API service expects to be able to fetch a DQN model from a model service running available from the service endpoint `{API_DKB}/models/{model_id}` configured with the `API_DKB` environment variable.

Available endpoints:

| Endpoint | Description | Request | Response |
| -------- | ----------- | ------- | -------- |
| `GET /`  | Hello world endpoint | - | `Model verification service` |
| `POST /verify` | Verification request for a stored model | ```{ "model_id": <model>, "properties": <verification properties> }``` | ```{ "verification_result": <prism output> }``` |


Example usage:
```bash
curl -X POST -H "Content-Type: application/json" -d '{"model_id": "DQNCartpole_1", "properties": "your_properties"}' https://<service>/verify
```

## Preparing the DQN model for verification

Preparing the model for verification includes sampling and discretizing the transitions of the environment and the trained agent, the outline of the process is below.
For more details consult [VERGE deliverable](https://www.verge-project.eu/dissemination/deliverables) 4.1:

```python
from verge_vf import to_prism

...

# Sample the transitions
transitions = to_prism.sample_and_process_continuous(environment, model, configuration, num_epochs)
# Discretize
disc_transitions = to_prism.discretize_continuous_transitions(transitions, NUM_BINS)
# Calculate transition probabilities
transition_probs = to_prism.get_transition_probabilities(disc_transitions)
# Store as intermediate CSV for inspection
to_prism.transitions_to_csv(transition_probs, save_path)

...

# Read transition probabilities
transition_probs = to_prism.read_transition_probabilities(save_path)
# Write initial states to prism file
to_prism.write_prism_model_initial_states(transition_probs, configuration, prism_path)
```

# VERGE Project

The ability to formally verify the safety of models contributes to VERGE *Objective 4: Develop tools that ensure the security, privacy and trustworthiness of the VERGE system.*
For more technical details, please refer to the [VERGE deliverable](https://www.verge-project.eu/dissemination/deliverables) 4.1 and 4.2.
