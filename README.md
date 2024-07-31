

<h1 align="center">Teaching Agents Open-World Skills</h1>


## Overview 

**Abstract:** Recent studies have delved into constructing generalist agents for open-world embodied environments like Minecraft. Despite the encouraging results, existing efforts mainly focus on solving basic programmatic tasks, e.g., material collection and tool-crafting following the Minecraft tech-tree, treating the ObtainDiamond task as the ultimate goal. This limitation stems from the narrowly defined set of actions available to agents, requiring them to learn effective long-horizon strategies from scratch. Consequently, discovering diverse gameplay opportunities in the open world becomes challenging. In this work, we introduce Odyssey, a new framework that empowers Large Language Model (LLM)-based agents with open-world skills to explore the vast Minecraft world. Odyssey comprises three key parts: 

- **(1) An interactive agent with an open-world skill library that consists of 40 primitive skills and 183 compositional skills.**
- **(2) A fine-tuned LLaMA-3 model trained on a large question-answering dataset with 390k+ instruction entries derived from the Minecraft Wiki.**
- **(3) A new open-world benchmark includes thousands of long-term planning tasks, tens of dynamic-immediate planning tasks, and one autonomous exploration task.**
  
Extensive experiments demonstrate that the proposed Odyssey framework can effectively evaluate the planning and exploration capabilities of agents. All datasets, model weights, and code are publicly available to motivate future research on more advanced autonomous agent solutions.


## Contents

- [Directory Description](#Directory-Description)
- [Odyssey Installation](#Odyssey-Installation)
  - [Python Install](#Python-Install)
  - [Node.js Install](#Node.js-Install)
  - [Minecraft Server](#Minecraft-Server)
  - [Embedding Model](#Embedding-Model)
- [Config](#Config)
- [Odyssey Tasks](#Odyssey-Tasks)
  - [Subgoal](#Subgoal)
  - [Long-term Planning Task](#Long-term-Planning-Task)
  - [Dynamic-Immediate Planning Task](#Dynamic-Immediate-Planning-Task)
  - [Autonomous Exploration Task](#Autonomous-Exploration-Task)



## Directory Description

1. **LLM-Backend**

   Code to deploy LLM backend.

2. **MC-Crawler**

   Crawling Minecraft game information from Minecraft Wiki, and store data in markdown format.

3. **MineMA-Model-Fine-Tuning**

     Code to fine-tune the LLaMa model and generate training and test datasets.

4. **Odyssey**

     Code for Minecaft agents based on a large language model and skill library.

## Odyssey Installation

We use Python ≥ 3.9 and Node.js ≥ 16.13.0. We have tested on Ubuntu 20.04, Windows 10, and macOS.

### Python Install

```bash
cd Odyssey
pip install -e .
pip install -r requirements.txt
```

### Node.js Install

```bash
npm install -g yarn
cd Odyssey/odyssey/env/mineflayer
yarn install
cd Odyssey/odyssey/env/mineflayer/mineflayer-collectblock
npx tsc
cd Odyssey/odyssey/env/mineflayer
yarn install
cd Odyssey/odyssey/env/mineflayer/node_modules/mineflayer-collectblock
npx tsc
```

### Minecraft Server

You can deploy a Minecraft server using docker. See [here](./Odyssey/docs/run_using_docker.md).

### Embedding Model

1. Need to install [git-lfs](https://git-lfs.com) first.

2. Download embedding model repository

   ```bash
   git lfs install
   git clone https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2.git
   ```

3. The directory where you clone the repository is then used to set `embedding_dir`.

## Config

You need to create `config.json` according to the format of `conf/config.json.keep.this` in `conf` directory.

- `server_host`: LLaMa backend server ip.
- `server_port`: LLaMa backend server port.
- `NODE_SERVER_PORT`: Node service port.
- `SENTENT_EMBEDDING_DIR`: Path to your embedding model.
- `MC_SERVER_HOST`: Minecraft server ip.
- `MC_SERVER_PORT`: Minecraft server port.

## Odyssey Tasks

### Subgoal

```python
def test_subgoal():
    odyssey_l3_8b = Odyssey(
        mc_port=mc_port,
        mc_host=mc_host,
        env_wait_ticks=env_wait_ticks,
        skill_library_dir="./skill_library",
        reload=True, # set to True if the skill_json updated
        embedding_dir=embedding_dir, # your model path
        environment='subgoal',
        resume=False,
        server_port=node_port,
        critic_agent_model_name = ModelType.LLAMA3_8B_V3,
        comment_agent_model_name = ModelType.LLAMA3_8B_V3,
        curriculum_agent_qa_model_name = ModelType.LLAMA3_8B_V3,
        curriculum_agent_model_name = ModelType.LLAMA3_8B_V3,
        action_agent_model_name = ModelType.LLAMA3_8B_V3,
    )
    # 5 classic MC tasks
    test_sub_goals = ["craft crafting table", "craft wooden pickaxe", "craft stone pickaxe", "craft iron pickaxe", "mine diamond"]
    try:
        odyssey_l3_8b.inference_sub_goal(task="subgoal_llama3_8b_v3", sub_goals=test_sub_goals)
    except Exception as e:
        print(e)
```

| Model                          | For what                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| action_agent_model_name        | Choose one of the k retrieved skills to execute              |
| curriculum_agent_model_name    | Propose tasks for farming and explore                        |
| curriculum_agent_qa_model_name | Schedule subtasks for combat, generate QA context, and rank the order to kill monsters |
| critic_agent_model_name        | Action critic                                                |
| comment_agent_model_name       | Give the critic about the last combat result, in order to reschedule subtasks for combat |

### Long-term Planning Task

```python
def test_combat():
    odyssey_l3_70b = Odyssey(
        mc_port=mc_port,
        mc_host=mc_host,
        env_wait_ticks=env_wait_ticks,
        skill_library_dir="./skill_library",
        reload=True, # set to True if the skill_json updated
        embedding_dir=embedding_dir, # your model path
        environment='combat',
        resume=False,
        server_port=node_port,
        critic_agent_model_name = ModelType.LLAMA3_70B_V1,
        comment_agent_model_name = ModelType.LLAMA3_70B_V1,
        curriculum_agent_qa_model_name = ModelType.LLAMA3_70B_V1,
        curriculum_agent_model_name = ModelType.LLAMA3_70B_V1,
        action_agent_model_name = ModelType.LLAMA3_70B_V1,
    )
    
    multi_rounds_tasks = ["1 enderman", "3 zombie"]
    l70_v1_combat_benchmark = [
                        # Single-mob tasks
                         "1 skeleton",  "1 spider", "1 zombified_piglin", "1 zombie",
                        # Multi-mob tasks
                        "1 zombie, 1 skeleton", "1 zombie, 1 spider", "1 zombie, 1 skeleton, 1 spider"
                        ]
    for task in l70_v1_combat_benchmark:
        odyssey_l3_70b.inference(task=task, reset_env=False, feedback_rounds=1)
    for task in multi_rounds_tasks:
        odyssey_l3_70b.inference(task=task, reset_env=False, feedback_rounds=3)
```

### Dynamic-Immediate Planning Task

```python
def test_farming():
    odyssey_l3_8b = Odyssey(
        mc_port=mc_port,
        mc_host=mc_host,
        env_wait_ticks=env_wait_ticks,
        skill_library_dir="./skill_library",
        reload=True, # set to True if the skill_json updated
        embedding_dir=embedding_dir, # your model path
        environment='farming',
        resume=False,
        server_port=node_port,
        critic_agent_model_name = ModelType.LLAMA3_8B_V3,
        comment_agent_model_name = ModelType.LLAMA3_8B_V3,
        curriculum_agent_qa_model_name = ModelType.LLAMA3_8B_V3,
        curriculum_agent_model_name = ModelType.LLAMA3_8B_V3,
        action_agent_model_name = ModelType.LLAMA3_8B_V3,
    )

    farming_benchmark = [
                    # Single-goal tasks
                    "collect 1 wool by shearing 1 sheep",
                    "collect 1 bucket of milk",
                    "cook 1 meat (beef or mutton or pork or chicken)",
                    # Multi-goal tasks
                    "collect and plant 1 seed (wheat or melon or pumpkin)"
                    ]
   	for goal in farming_benchmark:
	      odyssey_l3_8b.learn(goals=goal, reset_env=False)
```

### Autonomous Exploration Task

```python
def explore():
    odyssey_l3_8b = Odyssey(
        mc_port=mc_port,
        mc_host=mc_host,
        env_wait_ticks=env_wait_ticks,
        skill_library_dir="./skill_library",
        reload=True, # set to True if the skill_json updated
        embedding_dir=embedding_dir, # your model path
        environment='explore',
        resume=False,
        server_port=node_port,
        critic_agent_model_name = ModelType.LLAMA3_8B,
        comment_agent_model_name = ModelType.LLAMA3_8B,
        curriculum_agent_qa_model_name = ModelType.LLAMA3_8B,
        curriculum_agent_model_name = ModelType.LLAMA3_8B,
        action_agent_model_name = ModelType.LLAMA3_8B,
        username='bot1_8b'
    )
    odyssey_l3_8b.learn()
```




