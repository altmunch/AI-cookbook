# 深海

This project seeks to improve on the model architecture of AZR

**model architecture**:

1. *self-play loop*
- LLM agent
  - shared base LLM(Qwen)
  - proposer module
    - lightweight RL agent to increase difficulty to flow zone
  - solver module
    - lightweight RL agent to critique the question
  - orchestrator
    - guardrails

2. *multiple environments*
- physics simulation engines
- web/api systems