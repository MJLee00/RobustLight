## 项目说明

本仓库基于 **CityFlow** 交通仿真引擎，实现多种交通信号控制（TSC）方法（如 CoLight / MPLight / MaxPressure 及其变体），并提供带噪/缺失观测下的推理（inference）相关实验代码。

## 环境要求

- **操作系统**：Linux（推荐 Ubuntu 20.04+）
- **Python**：推荐 **3.9**（仓库内 `*.pyc` 显示以 3.9 运行过）
- **核心依赖**：
  - `cityflow`（CityFlow Python binding）
  - `numpy`、`pandas`
  - `torch`
  - `tensorflow`（多处使用 `tensorflow.keras`）
  - `einops`（推理模块使用）
  - `wandb`（可选，默认多数脚本关闭记录）

> 注意：部分脚本默认把 `device` 写成 `cuda:x`（如 `cuda:4` / `cuda:7`）。如果你没有 GPU 或 GPU 编号不同，需要在运行脚本里把 `device` 改成 `cpu` 或可用的 `cuda:0` 等。

## 安装步骤（推荐）

### 1) 创建虚拟环境

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip setuptools wheel
```

### 2) 安装 Python 依赖

仓库未提供 `requirements.txt`，可先按下面安装（足以覆盖仓库导入的三方包）：

```bash
pip install numpy pandas einops wandb
pip install torch
pip install tensorflow
```

说明：
- **PyTorch / TensorFlow** 与 CUDA 版本耦合较强；如需 GPU 版本，请按各自官方安装指引选择与你机器 CUDA 匹配的版本（可参考官方页面，地址可在搜索引擎中查询：`pytorch.org/get-started`、`tensorflow.org/install`）。
- 若你只需要 CPU：安装默认的 `torch`/`tensorflow` 通常即可；并将代码里 `device` 改为 `cpu`。

### 3) 安装 CityFlow（必须）

代码在 `utils/cityflow_env.py` 中直接 `import cityflow as engine`，因此必须安装 CityFlow 的 Python binding。

#### 方式 A：直接 pip 安装（若可用）

```bash
pip install cityflow
```

如果安装失败（常见于缺少对应平台 wheel），请用方式 B。

#### 方式 B：从源码编译安装（更稳）

1) 安装系统依赖（示例以 Ubuntu/Debian 为主）：

```bash
sudo apt-get update
sudo apt-get install -y build-essential cmake git libboost-all-dev
```

2) 获取 CityFlow 源码并编译安装 Python 包（不同 CityFlow 版本目录结构略有差异；原则是：先 `cmake && make` 编译，再把 Python binding 安装到当前 venv）：

```bash
git clone https://github.com/cityflow-project/CityFlow.git
cd CityFlow
mkdir -p build && cd build
cmake ..
make -j"$(nproc)"
```

随后在 CityFlow 仓库中找到 Python binding 的安装入口（常见为 `python/` 或 `pybind/` 等目录），在虚拟环境中执行：

```bash
pip install .
```

验证：

```bash
python -c "import cityflow; print('cityflow ok')"
```

## 数据准备

本仓库默认从 `data/<城市>/<路网>/` 读取 CityFlow 所需的 **路网文件** 与 **交通流文件**。仓库中已自带示例数据，结构类似：

```text
data/
  Hangzhou/4_4/
    roadnet_4_4.json
    anon_4_4_hangzhou_real_5816.json
  Jinan/3_4/
    roadnet_3_4.json
    anon_3_4_jinan_real.json
  newyork_28_7/28_7/
    roadnet_28_7.json
    anon_28_7_newyork_real_double.json
```

运行时会发生两件事：
- 程序把 `data/...` 下的 `roadnet_*.json` / `anon_*.json` **复制**到本次实验的工作目录（在 `records/...` 下）
- 然后在工作目录里生成 `cityflow.config` 并启动仿真

如果你新增自己的数据，确保：
- `ROADNET_FILE` 与 `TRAFFIC_FILE` 文件名与脚本/配置一致
- `PATH_TO_DATA` 指向包含这两个文件的目录

## 运行方式（训练/评估）

### 输出目录说明

所有 `run_*.py` 脚本都会按时间戳创建输出目录：
- **训练日志/评估日志**：`records/<memo>/<traffic_file>_<timestamp>/`
- **模型文件**：`model/<memo>/<traffic_file>_<timestamp>/`
- 每个 round 的数据：`records/.../train_round/round_<k>/...`
- 每个 round 的测试：`records/.../test_round/round_<k>/...`
- 运行耗时统计：`records/.../running_time.csv`

### 1) 运行 CoLight（示例）

```bash
python run_colight.py -memo Colight_run -mod EfficientColight
```

可选参数（以脚本内 argparse 为准）：
- `-memo`：实验名（会体现在 `records/`、`model/` 路径中）
- `-mod`：算法名（会写入 `MODEL_NAME`），例如 `EfficientColight`
- `-eightphase`：启用 8 相位（如果脚本支持）
- `-gen`：每个 round 的 generator 数量
- `-multi_process` / `-workers`：是否启用多进程生成及并发数

### 2) 运行 MPLight（示例）

```bash
python run_mplight.py -memo MPLight_run -mod EfficientMPLight
```

### 3) 运行 Advanced 版本（示例）

```bash
python run_advanced_mplight.py -memo AdvMPLight_run -mod AdvancedMPLight
python run_advanced_maxpressure.py --memo AdvMP_run -model AdvancedMaxPressure
```

### 4) 推理/缺失观测相关（示例）

```bash
python run_advanced_colight_dsi.py -memo ColightDSI_run -mod ColightDSI
```

## 常见问题（必看）

### 1) `ModuleNotFoundError: No module named 'cityflow'`

- **原因**：未安装 CityFlow Python binding
- **解决**：按上面的“安装 CityFlow”完成安装，并用

```bash
python -c "import cityflow; print(cityflow)"
```

验证。

### 2) CUDA/显卡报错（例如 `cuda:7` 不存在、或没有 GPU）

- **现象**：`RuntimeError: CUDA error...`、或启动即报错找不到设备
- **解决**：
  - 打开你运行的脚本（例如 `run_colight.py` / `run_mplight.py` / `run_advanced_colight_dsi.py`）
  - 找到 `dic_traffic_env_conf_extra` 中的 `"device": "cuda:x"`，改为 `"device": "cpu"` 或 `"cuda:0"`

### 3) `FileNotFoundError` 找不到 `roadnet_*.json` / `anon_*.json`

- **原因**：`PATH_TO_DATA` 或文件名不匹配
- **解决**：
  - 检查 `data/<城市>/<路网>/` 下是否存在脚本里写的 `ROADNET_FILE`、`TRAFFIC_FILE`
  - 确认脚本里的 `template` 与 `road_net` 组合出的路径确实存在（例如 `data/Hangzhou/4_4`）

### 4) 选择城市不生效（一直跑 Hangzhou / 或脚本参数无法关闭）

部分脚本的 argparse 把 `-hangzhou/-jinan/-new_york` 设置为 `action="store_true"` 且 `default=True`，
再加上代码使用 `if ... elif ...`，会导致“多个城市同时为 True 时永远命中第一个分支”。

建议做法：
- **最简单**：直接在对应 `run_*.py` 里把不想跑的城市默认值改为 `False`
- 或者自己改 argparse，增加 `--city {hangzhou,jinan,new_york}` 这类互斥参数（如果你需要我也可以顺手改）

## 结果汇总（summary）

仓库提供 `summary.py` 用于汇总 `records/<memo>/...` 产生的测试结果。
使用前需要在 `summary.py` 里把 `memo = "..."` 改成你实际运行时的 `-memo` 值，然后执行：

```bash
python summary.py
```
