已完成无侵入探查。跳板 `173.249.49.54` 可用，内层机器是 `autodl-container-dae84e9457-3808de12`。我只跑了只读命令，未改远端配置，未启动/停止任务，未读取密钥类内容。

**核心结论**
目标机上主要算法是基于 `junyanz/pytorch-CycleGAN-and-pix2pix` 的 **pix2pix 条件 GAN**，用于从 `real_A` 生成 `fake_B`，任务语境是道路/上下文栅格到建筑足迹栅格的图像到图像翻译。

**当前环境**
- OS: Ubuntu 22.04.3 LTS，容器环境
- Python: 3.12.3，Conda base
- PyTorch: `torch 2.3.0+cu121`，`torchvision 0.18.0+cu121`
- 当前 `nvidia-smi`: `No devices found`
- 但 2026-03 的训练日志显示训练当时用过 `cuda:0`
- 当前常驻服务：JupyterLab、TensorBoard、AutoDL 面板；没有发现正在跑的训练/推理进程

**项目位置**
- 根目录：`/root/autodl-tmp/EnvTrain`
- 代码：`/root/autodl-tmp/EnvTrain/02_code/pytorch-CycleGAN-and-pix2pix`
- 数据：`/root/autodl-tmp/EnvTrain/01_data`
- 权重：`/root/autodl-tmp/EnvTrain/06_ckpt`
- 日志：`/root/autodl-tmp/EnvTrain/04_logs`
- 导出结果：`/root/autodl-tmp/EnvTrain/07_export`
- Git 源：`https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix.git`
- Git HEAD: `2a7afba`

**算法配置**
- model: `pix2pix`
- direction: `AtoB`
- dataset_mode: `aligned`
- input/output: RGB, `input_nc=3`, `output_nc=3`
- image size: `512x512`
- generator: `unet_256`
- discriminator: `basic` PatchGAN / `n_layers_D=3`
- loss: GAN loss + `lambda_L1=100.0`
- GAN mode: `vanilla`
- optimizer: Adam, `lr=0.0002`, `beta1=0.5`
- epochs: `100 + 100` linear decay
- batch size: `1`
- normalization: batch norm
- checkpoints every 10 epochs, latest every 5000 iters

**数据集**
原始数据集 `pix2pix_realA_realB_v0`：
- train: 397 对
- val: 39 对
- test: 44 对
- total: 480 对
- FAR 标签：low 229，mid 188，high 63
- note: `A is OSM road/context raster; B is real existing building footprint raster`

FAR 分流：
- low FAR: 229 对，train 187 / val 26 / test 16
- high FAR: 63 对，train 54 / val 1 / test 8
- mid FAR 没看到单独训练分支

**训练产物**
已存在三类主要训练：
- 原始全量：`env_pix2pix_realA_realB_20260314_044038`
- low FAR full：`env_pix2pix_lowfar_full_20260316_fullsplit_020903_low`
- high FAR full：`env_pix2pix_highfar_full_20260316_fullsplit_020903_high`
- smoke 测试：low/high 各 1 epoch 小跑通过

最新 full 模型形状：
- Generator `latest_net_G.pth`: 217.7 MB，82 tensors，约 54.42M 参数
- Discriminator `latest_net_D.pth`: 11.1 MB，22 tensors，约 2.77M 参数
- low/high full 的模型结构一致，分别训练在 low/high FAR 数据上

**导出结果**
- `/root/autodl-tmp/EnvTrain/07_export` 下有 low/high 测试 HTML 和图像
- 主导出共 144 个 PNG、4 个 HTML、2 个 generator 权重
- 打包记录：`/root/autodl-tmp/far_split_results_20260316_fullsplit_020903.tar.gz`
- bundle: `/root/autodl-tmp/EnvTrain/07_export/far_split_bundle_20260316_fullsplit_020903`
- low/high 测试日志均显示成功加载 latest generator 并生成 `test_latest/index.html`

**状态判断**
这台机子当前像是“训练完成后的保留环境”，不是正在训练的状态。算法主体、数据分流、训练脚本、checkpoint、测试导出都齐全；唯一需要注意的是当前容器没有可见 GPU，若要继续训练或重新推理，需要重新分配/挂载 GPU 实例。
