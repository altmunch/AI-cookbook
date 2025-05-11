This is a research project on AZR

### **Absolute Zero Reasoner (AZR) Tutorial**

#### **Overview**
AZR is a self-play framework where a single language model (LLM) acts as both **task proposer** and **solver**, learning without external data. It uses code execution as a verifiable environment to generate and validate tasks.

---

### **Key Components**
1. **Self-Play Loop**:
   - **Proposer**: Generates tasks (code snippets + inputs/outputs).
   - **Solver**: Solves tasks (predicts missing elements in code triplets).
   - **Environment**: Validates tasks via code execution and provides rewards.

2. **Three Reasoning Modes**:
   - **Deduction**: Predict output `o` given program `p` and input `i`.
   - **Abduction**: Infer input `i` given program `p` and output `o`.
   - **Induction**: Synthesize program `p` from input-output examples.

3. **Reward Design**:
   - **Proposer Reward**: Maximizes task difficulty (neither too easy nor impossible).
     - \( r_{\text{propose}} = 1 - \bar{r}_{\text{solve}} \), where \( \bar{r}_{\text{solve}} \) is the solver’s average success rate.
   - **Solver Reward**: Binary reward for correctness (\( r_{\text{solve}} = \mathbb{I}_{(y=y^*)} \)).
   - **Format Penalty**: Penalizes syntax errors in responses.

---

### **Implementation Steps**
#### **1. Setup the Environment**
- **Code Executor**: Use a secure Python environment (e.g., E2B, restricted Docker containers) to validate generated code.
- **Forbidden Modules**: Block unsafe modules (e.g., `os`, `sys`).  
  Example:  
  ```python
  FORBIDDEN_MODULES = ["os", "sys", "shutil", ...]
  ```

#### **2. Initialize Buffers**
- **Seed Buffer**: Start with a simple program (e.g., identity function) to bootstrap training.
- **Task Buffers**: Maintain separate buffers for deduction, abduction, and induction tasks.

#### **3. Task Proposal**
- **Deduction/Abduction**:
  - Sample `K` reference examples from buffers.
  - Generate new `(p, i)` pairs. Validate via code execution to get `o`.
  - Example prompt:  
    ```python
    def f(numbers: list[int]) -> int:
        # Complex logic here...
    ```
- **Induction**:
  - Sample a program `p` from buffers.
  - Generate `N` inputs and a natural language description. Validate outputs.

#### **4. Task Solving**
- **Deduction**: Given `(p, i)`, predict `o`.
- **Abduction**: Given `(p, o)`, predict `i`.
- **Induction**: Given input-output pairs and a description, synthesize `p`.

#### **5. Reward Calculation**
- **Proposer**: Reward tasks with moderate difficulty (Equation 4).
- **Solver**: Binary reward for correctness (Equation 5).

#### **6. Training**
- **Algorithm**: Task-Relative REINFORCE++ (TRR++).
  - Compute advantage separately for each task type and role.
  - Update policy using PPO with task-specific baselines.
- **Hyperparameters**:
  - Batch size: 64 × 6 (2 roles × 3 tasks).
  - Learning rate: 1e-6.
  - Optimizer: AdamW.

---

### **Code Snippets**
#### **Task Validation**
```python
def validate_task(p: str, i: str) -> Tuple[bool, str]:
    # 1. Check syntax
    # 2. Check forbidden modules
    # 3. Ensure determinism (run twice, compare outputs)
    try:
        exec(p, {"__builtins__": None}, locals())
        output_1 = locals()['f'](*i)
        output_2 = locals()['f'](*i)
        return (output_1 == output_2), output_1
    except:
        return False, None
```

#### **Reward Calculation**
```python
def proposer_reward(success_rate: float) -> float:
    if success_rate in {0.0, 1.0}:
        return 0.0
    return 1.0 - success_rate

def solver_reward(pred: Any, gold: Any) -> float:
    return 1.0 if pred == gold else -0.5  # Format penalty included
```

---

### **Strengths**
1. **No Human Data**: Trains entirely on self-generated tasks.
2. **Cross-Domain Generalization**: Coding skills improve math reasoning.
3. **Scalability**: Performance improves with model size (3B → 14B).

### **Weaknesses**
1. **Safety Risks**: May generate unsafe code or reasoning (e.g., "uh-oh moments").
2. **Determinism Limitation**: Only handles deterministic programs.
3. **Token Length Growth**: Longer responses over time, especially for abduction.

---

### **Key Insights**
- **Code as a Medium**: Code’s expressiveness and verifiability enable grounded learning.
- **Emergent Behaviors**: Models naturally use step-by-step comments (ReAct-style).
- **Curriculum Learning**: Tasks evolve from simple to complex through self-play.

---

### **Tips for Implementation**
1. **Security**: Sandbox code execution to prevent malicious code.
2. **Diversity**: Use in-context examples to avoid task repetition.
3. **Monitoring**: Track token lengths and safety flags during training.

By following this guide, you can replicate AZR’s self-play loop and achieve state-of-the-art reasoning without human data. For full details, refer to the paper’s [code](https://github.com/LeapLabTHU/Absolute-Zero-Reasoner) and appendix.