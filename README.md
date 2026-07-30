# Interactive Robotics with SmolVLA

Real-time instruction following with a Vision-Language-Action model on the low-cost SO-101 robotic arm.

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Robot](https://img.shields.io/badge/Robot-SO--101-informational)](#system-setup)
[![Policy](https://img.shields.io/badge/Policy-SmolVLA-orange)](https://huggingface.co/lerobot/smolvla_base)
[![Framework](https://img.shields.io/badge/Framework-LeRobot-yellow)](https://github.com/huggingface/lerobot)

## Overview

This repository documents a real-robot study of **SmolVLA fine-tuning and deployment on the SO-101 manipulation platform**. The project covers the complete workflow from leader-follower teleoperation and multi-view RGB data collection to task-specific fine-tuning, real-robot inference, quantitative evaluation, and failure analysis.

The work was completed as a Forschungspraxis project at the **Institute for Cognitive Systems, Technical University of Munich**.

Three manipulation tasks were investigated:

1. Pick up a brick and place it in a target box.
2. Stack a blue block on a red block.
3. Pick up a sponge and wipe a stain from paper.

The experiments show that compact VLA models can learn useful real-robot behaviors from relatively small task-specific datasets. They also reveal clear limitations in spatial generalization, language grounding, multi-target selection, stage recovery, and contact-rich manipulation.

> **Repository status:** The project summary, three training datasets, corresponding fine-tuned models, and representative data-collection videos are available. Training, inference, evaluation scripts, real-robot evaluation videos, and supporting files are being organized for release.

## Highlights

- End-to-end real-robot pipeline using **SO-101**, **LeRobot**, and **SmolVLA**.
- Leader-follower teleoperation for collecting **401 task demonstrations** across three datasets.
- Two RGB camera views with no wrist camera.
- Task-specific fine-tuning under limited local GPU resources.
- Real-robot evaluation with quantitative success rates and staged task analysis.
- Generalization experiments involving new colors, multiple objects, and changed language instructions.
- Failure analysis covering spatial bias, weak language grounding, residual-stain loops, and stage hallucination.

## System Setup

| Component | Configuration |
|---|---|
| Robot | SO-101 leader-follower system |
| Cameras | Two Intel RealSense RGB cameras |
| Camera views | Side view and upper view |
| Image observations | 640 × 480 at 15 FPS |
| Robot state | 6 joint dimensions |
| Robot action | 6 joint dimensions |
| Policy | SmolVLA |
| Framework | Hugging Face LeRobot |
| Local GPU | NVIDIA RTX 4060 Laptop GPU, 8 GB |

## Tasks and Resources

| Task | Language instruction | Episodes | Dataset | Model | Video |
|---|---|---:|---|---|---|
| Pick and place | `pick up the red lego brick into the green box` | 98 | [Hugging Face Dataset](https://huggingface.co/datasets/shuowenLLL/redbrick-pick-place_20260602_153911) | [Fine-tuned SmolVLA](https://huggingface.co/shuowenLLL/smolvla-redbrick-pick-place-v2) | [Dataset sample](#task-1-pick-and-place-demonstration) |
| Block stacking | `put the blue block on the red block` | 102 | [Hugging Face Dataset](https://huggingface.co/datasets/shuowenLLL/blue-on-red-stack) | [Fine-tuned SmolVLA](https://huggingface.co/shuowenLLL/smolvla-blue-on-red-stack-30k-10) | [Dataset sample](#task-2-block-stacking-demonstration) |
| Wiping | `wipe the coffee stain off the paper with the sponge` | 201 | [Hugging Face Dataset](https://huggingface.co/datasets/shuowenLLL/wipe-coffee-with-sponge_20260616_164822) | [40k-step checkpoint](https://huggingface.co/shuowenLLL/smolvla-wipe-coffee-with-sponge-40k-200) | [Dataset sample](#task-3-sponge-wiping-demonstration) |

Datasets and model weights are hosted separately on the Hugging Face Hub rather than committed directly to this Git repository.

## Dataset Demonstration Samples

The following videos show representative demonstrations used to build the three task-specific LeRobot datasets. They are **data-collection samples recorded through SO-101 leader-follower teleoperation**, not autonomous policy evaluation videos.

Each demonstration contains synchronized side-view and upper-view RGB observations, the 6-dimensional robot state, the 6-dimensional action, and the corresponding language instruction.

### Task 1: Pick-and-Place Demonstration

**Instruction:** `pick up the red lego brick into the green box`

The operator controls the SO-101 follower arm to approach the red brick, grasp it, transport it to the green target box, release it, and return the arm toward its initial configuration.

https://github.com/user-attachments/assets/c99151bc-a8dd-4aa8-a44d-fe1a33609180

### Task 2: Block Stacking Demonstration

**Instruction:** `put the blue block on the red block`

The operator guides the arm to locate and grasp the blue block, move it above the red block, and release it to complete the stacking sequence.

https://github.com/user-attachments/assets/d64b16f9-06c2-4976-b90c-07593423dd68

### Task 3: Sponge Wiping Demonstration

**Instruction:** `wipe the coffee stain off the paper with the sponge`

The operator guides the arm through a longer tool-use sequence: grasping the sponge, moving it to the stained paper, executing repeated wiping motions, returning the sponge, and moving the arm back toward its final configuration.

https://github.com/user-attachments/assets/8b58875a-b1c0-45b5-93c0-db2f9f7026dd

## Data Collection and Fine-tuning Pipeline

1. Configure the SO-101 leader and follower arms.
2. Record demonstrations through direct leader-follower teleoperation.
3. Capture synchronized side-view and upper-view RGB observations.
4. Store robot state, action, image, and language-task features in the LeRobot dataset format.
5. Fine-tune `lerobot/smolvla_base` on each task-specific dataset.
6. Deploy the fine-tuned policy using synchronous or asynchronous action-chunk inference.
7. Evaluate successful execution, recovery behavior, generalization, and failure patterns on the physical robot.

## Experimental Results

### Task 1: Pick and Place

The brick was evaluated at three fixed workspace positions with 30 trials per position.

| Brick position | Successful trials | Success rate |
|---|---:|---:|
| Center | 25 / 30 | 83.3% |
| Left | 27 / 30 | 90.0% |
| Right | 0 / 30 | 0.0% |

The policy could successfully approach, grasp, transport, and release the brick at positions represented well in the training distribution. The complete failure at the right-side position indicates strong spatial bias and limited workspace generalization.

Recovery behavior was observed in some trials: after dropping or pushing the brick, the policy could follow its new position and attempt another grasp.

### Task 2: Block Stacking

The policy learned the visual routine of approaching one block and attempting to place it on another. However, changing the instruction from “blue on red” to “red on blue” did not reliably reverse the role of the two blocks.

When both blocks were separated across the left and right sides, the arm often moved toward the middle and repeatedly opened and closed the gripper. These results suggest that the learned visual stacking pattern was stronger than the language-specified color order.

### Task 3: Sponge Wiping

The wiping task was evaluated in stages to separate tool acquisition from the wiping behavior.

| Evaluation stage | Result |
|---|---|
| Stage 1: approach, grasp, and lift the sponge | 4 / 20 successful |
| Stage 2: wipe with the sponge already grasped | 7 / 10 successful |
| Stage 3: execute the wiping phase | 10 / 10 produced wiping motions; completion quality varied |

The policy could move toward visible stains, perform repeated wiping motions, and enter the final sponge-return stage after the stain disappeared. However, light residual stains often caused repetitive wiping without effective coverage or termination.

## Generalization and Failure Patterns

### Spatial generalization

Performance depended strongly on where objects appeared in the workspace. Positions underrepresented during training produced large approach and grasp errors.

### Weak language grounding

When the requested color was changed, the robot often continued to follow the visual routine learned during training. In multi-object scenes, target selection was frequently ambiguous.

### Visual completion cues

Task-completion detection was more reliable than target selection. For example, when an object was manually placed in the target box or a stain was manually removed, the robot often transitioned to its learned completion and reset behavior.

### Stage hallucination

If the initial sponge grasp failed, the policy sometimes continued directly toward the stain as though the sponge had already been acquired. This indicates that stage progression was not conditioned on an explicit verification of tool possession.

### Contact-rich manipulation

RGB observations and joint states alone provided limited information about contact force, slip, sponge pressure, and wiping effectiveness. Tactile sensing is therefore a promising direction for future work.

## Repository Structure

```text
interactive-robotics-smolvla/
├── README.md
├── LICENSE
├── NOTICE
├── CITATION.cff
├── .gitignore
├── assets/        # Images, short GIFs, and diagrams
├── configs/       # Camera, robot, and training configurations
├── docs/          # Report, presentation, and setup notes
├── results/       # Evaluation tables and result summaries
└── scripts/       # Data collection, training, inference, and evaluation
```

## Related Resources

- [Hugging Face LeRobot](https://github.com/huggingface/lerobot)
- [SmolVLA paper](https://arxiv.org/abs/2506.01844)
- [SmolVLA base model](https://huggingface.co/lerobot/smolvla_base)
- [Robot Learning: A Tutorial](https://github.com/fracapuano/robot-learning-tutorial)

## Citation

A citation file is provided in [`CITATION.cff`](CITATION.cff). Until an archival release is available, this repository may be cited as:

```bibtex
@software{li2026interactive,
  author = {Shuowen Li},
  title = {Interactive Robotics with SmolVLA: Real-time Instruction Following with Vision-Language-Action},
  year = {2026},
  url = {https://github.com/shuowenLLL/interactive-robotics-smolvla}
}
```

## Acknowledgements

This project builds on the open-source work of the Hugging Face **LeRobot** and **SmolVLA** teams. The research practice project was carried out at the Institute for Cognitive Systems, Technical University of Munich.

## License

The original source code and configuration files in this repository are licensed under the [Apache License 2.0](LICENSE), unless stated otherwise.

Third-party source files retain their original copyright and license notices. Datasets, model weights, reports, figures, and videos may be distributed under separate terms specified in their corresponding files or repository cards.
