## 问题描述

运行 `./eval.sh` 时出现环境版本冲突，脚本无法正常执行。

## 根本原因分析

共发现 **3 个问题**，按出现顺序排列：

### 问题 1：安装了错误的 libero 包（主要问题）

**现象**：`ModuleNotFoundError: No module named 'libero'`

**原因**：
- 环境中以 editable 模式安装了原始的 `libero==0.1.0`（来源：`/mnt/data1/linjianqi/LIBERO`）
- 但 lerobot 项目实际需要的是 HuggingFace 维护的 `hf-libero>=0.1.3,<0.2.0`（见 `pyproject.toml` 第 165 行）
- 原始 `libero` 包的 editable install 机制已损坏：`__editable___libero_0_1_0_finder.py` 中的 `MAPPING` 字典为空，导致 Python 无法找到 `libero` 模块
- 原始 `libero` 包依赖极度过时的版本（numpy\==1.22.4, transformers\==4.21.1, gym\==0.25.2），与 lerobot 的依赖严重冲突

**解决方法**：
```bash
# 卸载错误的 libero 包
pip uninstall libero

# 安装正确的 hf-libero 包
# 注意：由于 robomimic==0.2.0 依赖 egl_probe，而 egl_probe 从源码编译有 cmake 兼容性问题，
# 但 hf-egl-probe 已提供等效模块，所以需要分步安装
pip install robosuite==1.4.0 --no-deps
pip install robomimic==0.2.0 --no-deps
pip install "hf-libero>=0.1.3,<0.2.0" --no-deps
pip install "hydra-core>=1.2,<1.4" "bddl==1.0.1" easydict thop "matplotlib>=3.5.3" "future>=0.18.2" h5py tensorboard tensorboardX
```

### 问题 2：numpy 版本过低

**现象**：`pip check` 报告 `opencv-python-headless` 和 `rerun-sdk` 需要 `numpy>=2`，但安装的是 `numpy==1.26.4`

**解决方法**：
```bash
pip install "numpy>=2,<2.3.0"
```

### 问题 3：robosuite 日志文件权限冲突

**现象**：`PermissionError: [Errno 13] Permission denied: '/tmp/robosuite.log'`

**原因**：`robosuite==1.4.0` 在 `log_utils.py` 中硬编码了日志路径 `/tmp/robosuite.log`，该文件已被其他用户 (`linmin`) 创建，当前用户 (`liufuweijia`) 无写入权限。

**解决方法**：
修改 `robosuite/utils/log_utils.py` 第 71 行，将日志路径改为用户独立的路径：
```python
# 修改前
fh = logging.FileHandler("/tmp/robosuite.log")

# 修改后
import os
fh = logging.FileHandler(os.path.join("/tmp", f"robosuite_{os.getuid()}.log"))
```

### 附加问题：hf-libero 缺少 assets 目录

**现象**：`FileNotFoundError: No such file or directory: '.../libero/libero/assets/scenes/libero_living_room_tabletop_base_style.xml'`

**原因**：`hf-libero` PyPI 包未包含场景资源文件（XML、模型等），需要从原始 LIBERO 仓库链接。

**解决方法**：
```bash
ln -s /mnt/data1/linjianqi/LIBERO/libero/libero/assets \
      /mnt/data1/linjianqi/conda/lerobot/lib/python3.10/site-packages/libero/libero/assets
```

## 修复后状态

所有环境版本冲突已解决。`eval.sh` 可以正常进入评估流程（成功创建 10 个 LIBERO 环境并加载策略）。

> **注意**：修复版本冲突后，eval.sh 还存在一个 **feature name mismatch** 配置问题（策略期望的观测键名与环境提供的不一致），这不是版本冲突问题，需要通过 `--rename_map` 参数解决。

## 经验教训

1. lerobot 的 `libero` extra 依赖 `hf-libero`（HuggingFace 的 fork），而非原始 `libero` 包。安装前应查看 `pyproject.toml` 确认正确的包名。
2. 多用户共享的 `/tmp` 目录下的固定路径文件容易产生权限冲突。
3. 使用 `pip install -e ".[libero]"` 是安装 lerobot libero 支持的推荐方式，可避免手动处理依赖。
