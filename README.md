# Qwen Spatial Grasp Demo

一个面向演示与展示的智能抓取仓库。

这个项目的重点不是论文复现实验，而是把“视觉语言模型如何理解指令、定位目标、生成抓取动作并驱动机械臂执行”这条链路讲清楚、跑起来、展示出来。仓库默认使用 `Qwen` 作为主模型，同时保留你自己的增强方法 `SVP` 作为可切换方案，方便做对照演示。

## 1. 项目在做什么

这个项目要解决的问题很直接：

给机器人一句自然语言指令，例如：

- `抓住香蕉中部`
- `把黄色小鸭放到杯子旁边`
- `把目标物体移动到桌面中央附近`

系统需要完成四件事：

1. 理解用户到底想做什么。
2. 在图像中找到对应的物体和部件。
3. 把语言目标转换成可执行的抓取/移动动作序列。
4. 把动作交给抓取候选生成与机械臂控制模块执行。

这个仓库就是围绕这四步组织的。

## 2. 为什么单独拆一个新仓库

原始仓库包含很多论文发布内容：

- 数据集
- 大量实验脚本
- 评估结果
- 表格与图表
- 论文复现相关目录

这些内容对“项目展示”帮助不大，反而会让仓库目标不清晰。

所以这个新仓库专门做了收缩，保留：

- Qwen 主线演示
- 你的 SVP 增强方法
- 抓取执行代码
- MuJoCo 机械臂环境
- 必要的后端服务
- 一个可以直接展示的项目页

现在这个仓库更适合：

- 做 GitHub 项目展示
- 给老师/同学/面试官演示
- 讲解 Qwen 在机器人抓取中的作用
- 作为后续继续整理产品化 demo 的基础

## 3. 项目核心亮点

### 3.1 Qwen 为主

这个版本默认就是 `Qwen-first`。

也就是说，主展示链路优先突出：

- Qwen 对自然语言指令的理解
- Qwen 对目标物体和目标部件的定位
- Qwen 输出结构化动作规划

相比原来把 Qwen 放在辅助位置，这个版本更适合直接展示“Qwen 如何驱动机器人抓取任务”。

### 3.2 保留你的方法

除了 Qwen 主线，仓库仍然保留 `SVP` 增强方案：

- `vision_agent_v2.py`
- `backend/` 中的 `SAM + DINO` 接口

这样你在展示时可以清楚地讲：

- `Qwen` 是主线方案，负责更自然、更直接的视觉语言推理
- `SVP` 是你自己的增强链路，强调结构化视觉提示与更强约束

这比只放一个方案更有说服力。

### 3.3 从语言到执行的完整闭环

这个项目不是只做图片理解，也不是只做抓取算法。

它的价值在于把以下几层串起来了：

- 自然语言指令
- 视觉 grounding
- 动作规划
- 抓取候选筛选
- 机械臂执行

所以它更像一个“VLM 驱动智能抓取”的端到端 demo。

## 4. 两条方法链路

## 4.1 Qwen-first 主链路

默认模式：

```powershell
$env:VLM_METHOD="qwen"
python main_vlm.py
```

主入口文件：

- [main_vlm.py](C:/Users/16551/Desktop/html_demo/qwen-spatial-grasp-demo/main_vlm.py)
- [vision_agent.py](C:/Users/16551/Desktop/html_demo/qwen-spatial-grasp-demo/vision_agent.py)

这条链路的职责分工：

1. 用户输入自然语言指令。
2. `vision_agent.py` 调用 Qwen，生成结构化动作计划。
3. 同一个模块继续定位目标物体部件，回写像素坐标。
4. `main_vlm.py` 根据像素位置调用抓取推理与机械臂执行。

这一条链路适合做“项目主展示”，因为逻辑更完整、叙事更顺。

## 4.2 SVP 增强链路

切换方式：

```powershell
$env:VLM_METHOD="svp"
python main_vlm.py
```

主入口文件：

- [vision_agent_v2.py](C:/Users/16551/Desktop/html_demo/qwen-spatial-grasp-demo/vision_agent_v2.py)
- [backend/server_core.py](C:/Users/16551/Desktop/html_demo/qwen-spatial-grasp-demo/backend/server_core.py)

这条链路的特点是：

- 用 `SAM` 做实例分割和编号
- 用 `DINO` 做目标 ID 的关键点定位
- 用结构化提示把动作约束到更明确的视觉锚点

这部分更适合用来介绍你的方法设计思想。

## 5. 系统架构

可以把这个项目理解成 4 层：

### Layer 1: 指令理解层

负责把用户的自然语言任务转换成明确的操作目标。

例如：

- 要抓哪个物体
- 抓哪个部位
- 是否需要移动到另一个参照物附近
- 是否需要释放、回零

### Layer 2: 视觉定位层

负责把语言中的目标映射到图像中的具体位置。

Qwen 模式下：

- 直接让模型返回目标物体/部件对应的点或框

SVP 模式下：

- 先做实例分割编号
- 再做目标关键点定位
- 最后把编号映射回像素

### Layer 3: 抓取推理层

负责根据目标像素附近的视觉上下文生成抓取候选，并结合约束进行筛选。

关键文件：

- [grasp_process.py](C:/Users/16551/Desktop/html_demo/qwen-spatial-grasp-demo/grasp_process.py)
- [graspnet-baseline/](C:/Users/16551/Desktop/html_demo/qwen-spatial-grasp-demo/graspnet-baseline)

### Layer 4: 执行控制层

负责驱动 MuJoCo 中的机械臂、夹爪和场景对象执行动作。

关键目录：

- [manipulator_grasp/](C:/Users/16551/Desktop/html_demo/qwen-spatial-grasp-demo/manipulator_grasp)

## 6. 目录说明

```text
.
|- main_vlm.py              # 端到端抓取演示主入口
|- vision_agent.py          # Qwen 主线：动作规划 + 部件定位
|- vision_agent_v2.py       # 你的增强方法：SVP
|- grasp_process.py         # GraspNet 推理与抓取候选筛选
|- vlm_process.py           # SAM 等视觉处理辅助逻辑
|- backend/                 # SVP 需要的 FastAPI 后端
|- graspnet-baseline/       # 抓取候选模型与依赖
|- manipulator_grasp/       # MuJoCo 机械臂与场景环境
|- docs/                    # 项目展示页
```

## 7. 推荐展示方式

如果你是要拿这个项目做展示，建议这样讲：

### 先讲一句话定位

“这是一个基于 Qwen 的智能抓取演示系统，用户输入自然语言指令，系统自动完成目标理解、部件定位、动作规划和机械臂抓取执行。与此同时，我还保留了自己设计的结构化视觉提示方法作为增强方案。”

### 再讲两条路线

- `Qwen`：主展示链路，突出大模型理解和控制能力
- `SVP`：你的方法，突出你对结构化视觉提示和执行约束的设计

### 最后讲工程价值

- 不只是识别目标
- 不只是输出坐标
- 而是把语言、视觉、抓取和执行串成一个闭环

## 8. 快速开始

### 8.1 安装依赖

```bash
pip install -r requirements.txt
```

另外还需要安装 GraspNet 扩展：

```bash
cd graspnet-baseline/pointnet2
python setup.py install

cd ../knn
python setup.py install

cd ../graspnetAPI
pip install .
```

### 8.2 环境变量

复制 `.env.example` 到 `.env`。

如果只跑主展示链路，至少配置：

- `VLM_METHOD=qwen`
- `ACTION_MODEL_PROVIDER=qwen`
- `MODELSCOPE_API_KEY`
- `MODELSCOPE_MODEL_ID`

如果要跑你的 `SVP` 方法，还要额外配置：

- `BACKEND_URL`
- `AZURE_ENDPOINT`
- `AZURE_DEPLOYMENT`
- `AZURE_API_VERSION`
- `AZURE_KEY`

## 9. 模型与权重

这个仓库不会上传大权重文件，需要本地自行放置：

- `sam_b.pt`
- `logs/log_rs/checkpoint-rs.tar`

它们分别被这些模块使用：

- [vlm_process.py](C:/Users/16551/Desktop/html_demo/qwen-spatial-grasp-demo/vlm_process.py)
- [grasp_process.py](C:/Users/16551/Desktop/html_demo/qwen-spatial-grasp-demo/grasp_process.py)

## 10. 运行方式

### 10.1 只运行 Qwen 规划与定位

```bash
python vision_agent.py --img docs/assets/scene-1.png --ins "抓住黄色小鸭并移动到杯子旁边" --save logs/qwen_demo
```

### 10.2 运行完整抓取演示

```bash
python main_vlm.py
```

### 10.3 运行 SVP 后端

```bash
cd backend
uvicorn app:app --host 0.0.0.0 --port 8000
```

## 11. 项目当前定位

这个仓库当前最适合的定位是：

- 一个以 `Qwen` 为主的机器人抓取展示项目
- 一个可解释你自己方法设计的技术作品集仓库
- 一个后续可以继续做前端可视化、视频演示、在线页面扩展的基础版本

它不再强调：

- 论文数据集发布
- 批量实验复现
- 大规模结果评估表

而是强调：

- 展示清晰
- 方法主线明确
- 代码能讲清楚
- 结构适合对外展示

## 12. 一句话总结

这是一个把 `Qwen` 放到主位、把你的 `SVP` 方法作为增强方案保留下来的智能抓取演示仓库，重点展示“自然语言 -> 视觉定位 -> 动作规划 -> 抓取执行”的完整闭环。
