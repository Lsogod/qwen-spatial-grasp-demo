# Qwen Spatial Grasp Demo

一个面向智能抓取展示的新仓库，主线突出 `Qwen`，同时保留你的增强方法 `SVP` 作为可切换方案。

这个版本从原始论文发布仓库中拆出，只保留演示相关代码：

- `qwen` 模式：默认入口，使用 `vision_agent.py`
- `svp` 模式：你的增强方法，使用 `vision_agent_v2.py`
- `main_vlm.py`：MuJoCo + 抓取执行主入口
- `backend/`：`svp` 模式需要的 `/vision/sam` 和 `/vision/dino` 服务
- `graspnet-baseline/` 与 `manipulator_grasp/`：抓取候选生成与机械臂环境

## Project Focus

这个仓库主要用于展示两条智能抓取链路：

1. `Qwen-first`
   使用 Qwen 做视觉理解、部件定位和动作规划，适合做 demo、交互展示和快速说明。
2. `Your method: SVP`
   保留你原来的结构化视觉提示链路，作为增强方案和对照方案。

默认已经切到 `Qwen-first`：

```powershell
$env:VLM_METHOD="qwen"
python main_vlm.py
```

如果要切到你的方法：

```powershell
$env:VLM_METHOD="svp"
python main_vlm.py
```

## Repo Layout

```text
.
|- main_vlm.py
|- vision_agent.py
|- vision_agent_v2.py
|- grasp_process.py
|- vlm_process.py
|- backend/
|- graspnet-baseline/
|- manipulator_grasp/
|- docs/
```

## Quick Start

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

还需要安装 GraspNet 的扩展：

```bash
cd graspnet-baseline/pointnet2
python setup.py install

cd ../knn
python setup.py install

cd ../graspnetAPI
pip install .
```

### 2. Prepare env vars

复制 `.env.example` 到 `.env`，至少填好这些变量：

- `MODELSCOPE_API_KEY`
- `MODELSCOPE_MODEL_ID`
- `VLM_METHOD=qwen`
- `ACTION_MODEL_PROVIDER=qwen`

如果你要跑 `svp` 模式，再额外配置：

- `BACKEND_URL`
- `AZURE_ENDPOINT`
- `AZURE_DEPLOYMENT`
- `AZURE_API_VERSION`
- `AZURE_KEY`

### 3. Prepare local weights

这个仓库不会上传大权重文件。你需要手动放置：

- `sam_b.pt`
- `logs/log_rs/checkpoint-rs.tar`

其中：

- `sam_b.pt` 供 `vlm_process.py` 使用
- `logs/log_rs/checkpoint-rs.tar` 供 `grasp_process.py` 使用

### 4. Run Qwen-only planning and grounding

```bash
python vision_agent.py --img docs/assets/scene-1.png --ins "抓住黄色小鸭并移动到杯子旁边" --save logs/qwen_demo
```

### 5. Run the full grasp demo

```bash
python main_vlm.py
```

程序启动后输入自然语言指令，例如：

- `抓住香蕉中部`
- `把黄色小鸭放到杯子旁边`
- `把目标物体移动到桌面中心附近`

## Two Pipelines

### Qwen-first

`vision_agent.py` 现在支持把动作规划也切到 Qwen：

- `ACTION_MODEL_PROVIDER=qwen` 时，Qwen 同时负责动作规划和部件定位
- `ACTION_MODEL_PROVIDER=azure` 时，保留原先 Azure-compatible planner 行为

这让“Qwen 模式”真正成为一个自洽的主链路，而不是只做 grounding。

### SVP

`vision_agent_v2.py` 保留你的增强方法：

- 后端 SAM 做实例编号
- DINO 做目标 ID 的关键点定位
- 结构化提示把动作序列回写到像素点

适合展示“Qwen 方案 + 你的方法方案”的差异。

## Backend

`svp` 模式需要先启动后端服务：

```bash
cd backend
uvicorn app:app --host 0.0.0.0 --port 8000
```

接口包括：

- `POST /vision/sam`
- `POST /vision/dino`

## Notes

- 这个仓库已经移除了原始论文数据、实验表格和大批量结果文件。
- 当前默认目标是“展示”和“交互演示”，不是复现实验论文结果。
- 如果只做展示，优先使用 `qwen` 模式。
