# 多场景 GNSS 数据集与基准测试

> **本仓库用途（`tian1714522/citypdp`）**：发布该项目的**可下载打包版本**——完整数据分卷压缩包与源码快照，见 [Release `dataset-v1`](https://github.com/tian1714522/citypdp/releases/tag/dataset-v1)。
> 项目的**原始 Git 仓库**（含 Git LFS 数据、完整提交历史）为 [Laanyz/multi-scene-gnss-benchmark](https://github.com/Laanyz/multi-scene-gnss-benchmark)。

面向复杂环境下的全球导航卫星系统（GNSS）研究，本项目整理了软件园2期、湖边万达、五缘湾和厦禾路四个场景的数据，并提供定位评估、应用场景分类、非视距（NLOS）/多径分类，以及 SAGE 多径估计相关代码。

项目采用 Python 与 MATLAB，覆盖接收机观测处理、特征构建、模型训练及实验结果分析。各模块拥有独立的数据入口和输出目录，可按研究任务分别运行。

> **数据说明：** 本仓库通过 Git LFS 发布四个场景的 Xsens 结果、KL6 软件接收机处理结果、定位基准测试文件和 NovAtel 定位结果转换文件，合计约 3.67 GiB；另提供软件园2期相机帧的 4 个分卷，合计约 5.5 GiB。其他场景的相机数据和原始中频 IQ 不随仓库分发。

## 主要内容

| 模块 | 功能 |
| --- | --- |
| 多场景数据集 | 整理 KL6 软件接收机处理结果、NovAtel 定位结果、事后 RTK 观测文件、Xsens 结果及影像等资料；具体内容以各场景目录为准 |
| 定位基准测试 | 计算 LSQ/WLS 定位解，将可用的 SPP、SPP-RAIM、LSQ 和 WLS 结果与 RTK 真值对齐比较，生成误差统计与轨迹图 |
| 应用场景分类 | 以历元为样本，使用跟踪记录数和平均载噪比训练四分类 MLP |
| NLOS/多径分类 | 提供 MLP、1D CNN、1D ResNet，支持 3、101、104 维输入，按场景独立训练与评估 |
| 多径估计与定位扩展 | 提供基于相关器 I/Q 包络的 MATLAB SAGE 路径估计，以及主径/副径特征的神经网络定位实验 |

## 场景与类别

| 场景代码 | 数据目录 | 场景类别 | 场景分类标签 |
| --- | --- | --- | ---: |
| `r2` | 软件园2期 | 典型园区 | 0 |
| `wd` | 湖边万达 | 商业综合体 | 1 |
| `wyw` | 五缘湾 | 滨海开阔区 | 2 |
| `xhl` | 厦禾路 | 城市街谷主干道 | 3 |

场景类别映射见 [label.csv](应用场景分类基准测试代码/label.csv)。NLOS/多径任务使用另一套标签：`0 = Multipath`、`1 = NLOS`，两套标签不可混用。

## 目录结构

以下仅列出主要目录与入口，省略缓存、历史快照和大部分实验产物。

```text
.
├── README.md
├── 数据集/                            # 下列四类数据通过 Git LFS 分发
│   ├── 软件园2期/
│   ├── 湖边万达/
│   ├── 五缘湾/
│   └── 厦禾路/
├── 定位基准测试代码/
│   ├── configs/                       # CSV 字段映射示例
│   ├── input/                         # 可自行放入定位输入数据
│   ├── output/                        # 定位结果、统计报告与图像
│   └── scripts/
│       ├── run_all.py                 # 定位、评估与绘图入口
│       ├── run_benchmark.py           # 定位结果对比入口
│       ├── run_filter.py              # 离群点处理
│       └── run_plot.py                # 结果绘图
├── 应用场景分类基准测试代码/
│   ├── input/                         # 四个场景的跟踪观测 CSV
│   ├── label.csv
│   ├── run_training.py               # 完整训练与结果导出
│   ├── mlp_scene_classification.py    # 特征处理、划分与训练
│   ├── model_mlp111.py
│   ├── requirements.txt
│   ├── README_run.md
│   └── training_result.md
├── NLOS_multipath分类基准测试基准代码/
│   ├── input/                         # 跟踪观测及 NLOS/多径标签
│   ├── mlp_nlos_multipath.py
│   ├── cnn_nlos_multipath.py
│   ├── resnet_nlos_multipath.py
│   ├── model_mlp_nlos.py
│   ├── model_1dcnn_nlos.py
│   ├── model_1dresnet_nlos.py
│   ├── outputs/
│   └── 三种输入维度基准测试结果汇总.md
└── MultiPathEsiti/
    ├── external_driver/               # 原始相关器数据的外部 SAGE 驱动
    ├── MultiPathEsiti/                # MATLAB 多径估计相关代码
    ├── sage_test/                     # SAGE 扩展及神经网络定位实验
    └── sage_wls_solver/
```

## 环境准备

已有分类实验记录使用 Python 3.12.10。下面以 Windows PowerShell 为例，所有命令均从仓库根目录执行。新建独立环境后，通过完整解释器路径运行，无需激活环境。

仓库包含 Git LFS 数据。完整克隆前先安装并初始化 Git LFS：

```powershell
git lfs install
git clone https://github.com/Laanyz/multi-scene-gnss-benchmark.git
```

只需要代码时，可跳过 LFS 数据下载：

```powershell
$env:GIT_LFS_SKIP_SMUDGE = '1'
git clone https://github.com/Laanyz/multi-scene-gnss-benchmark.git
Remove-Item Env:GIT_LFS_SKIP_SMUDGE
```

克隆后也可只下载某个场景或某类数据，例如：

```powershell
cd multi-scene-gnss-benchmark
git lfs pull --include="数据集/软件园2期/Xsens结果文件/**"
```

软件园2期相机帧以 Git LFS 分卷提供。下载 4 个分卷后，在 PowerShell 中合并并解包：

```powershell
cd multi-scene-gnss-benchmark
cmd /c copy /b "数据集\软件园2期\相机数据分卷\software_camera.tar.part01"+"数据集\软件园2期\相机数据分卷\software_camera.tar.part02"+"数据集\软件园2期\相机数据分卷\software_camera.tar.part03"+"数据集\软件园2期\相机数据分卷\software_camera.tar.part04" "software_camera.tar"
tar -xf software_camera.tar -C "数据集\软件园2期"
```

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
```

按需安装所运行模块的依赖：

```powershell
# 定位评估与绘图
.\.venv\Scripts\python.exe -m pip install numpy pandas matplotlib pillow

# NLOS/多径分类
.\.venv\Scripts\python.exe -m pip install torch numpy pandas

# 场景分类额外依赖
.\.venv\Scripts\python.exe -m pip install scikit-learn joblib tensorboard
```

以上为按代码导入整理的基础安装命令，未锁定版本。场景分类目录另提供记录原实验直接依赖版本的 [requirements.txt](应用场景分类基准测试代码/requirements.txt)：

```powershell
.\.venv\Scripts\python.exe -m pip install -r "应用场景分类基准测试代码/requirements.txt"
```

该文件指定了 CUDA 12.6 版 PyTorch，使用前需确认与目标机器环境匹配。分类代码支持 CPU；需要 GPU 时，应安装与本机驱动兼容的 PyTorch 构建。MATLAB 模块需另行准备 MATLAB，并按入口脚本配置数据路径及算法参数。

## 快速开始

### 1. 应用场景分类

#### 输入文件

下载数据后，在 `应用场景分类基准测试代码/input/` 中准备四个场景的跟踪观测文件；场景映射文件保存在代码目录根部：

```text
应用场景分类基准测试代码/
├── label.csv
└── input/
    ├── track_data_alls_r2.csv
    ├── track_data_alls_wd.csv
    ├── track_data_alls_wyw.csv
    └── track_data_alls_xhl.csv
```

| 输入 | 必需字段 | 说明 |
| --- | --- | --- |
| `track_data_alls_<scene>.csv` | `signalId`, `prn`, `readMsCnt`, `chState`, `CN0` | `<scene>` 分别为 `r2`、`wd`、`wyw`、`xhl`；每行是一条卫星/信号跟踪记录 |
| `label.csv` | 前两列为场景代码、场景名称 | 必须包含 `r2`、`wd`、`wyw`、`xhl` 四类；仓库已提供该文件 |

上述 CSV 字段名两侧的空格会被清理，五个观测字段必须可转换为数值。程序只保留 `chState == 3` 的记录，要求 `signalId`、`prn` 和 `readMsCnt` 为有效非负整数，并拒绝同一场景、同一历元下重复的 `(signalId, prn)`。

当前预处理以 `(scene_code, readMsCnt)` 标识历元。每个样本包含两个特征：

- `n_sat`：`chState == 3` 且身份字段合法的跟踪记录数。同一 PRN 的不同信号分别计数，因此不是去重后的物理卫星数。
- `mean_cn0`：上述记录中有限且大于零的 CN0 的算术平均值。没有有效 CN0 的历元跳过。

运行训练：

```powershell
$env:OPENBLAS_NUM_THREADS = '1'
$env:OMP_NUM_THREADS = '1'
$env:MKL_NUM_THREADS = '1'
.\.venv\Scripts\python.exe -X utf8 -u "应用场景分类基准测试代码/run_training.py"
```

四个场景的有效历元合并后，先随机打乱，再按 70% / 15% / 15% 划分训练、验证和测试集，不按标签分层，随机种子为 `1995`。同一历元的观测保留在同一集合。标准化器只在训练集拟合，模型按最低验证损失选择。

默认最多训练 300 轮，批量大小为 1024，初始学习率为 0.001。结果按运行时间分别写入该模块的 `Model1/<时间戳>/` 和 `Log1/<时间戳>/`，包括最佳权重、标准化器、特征配置、划分清单、训练曲线及测试预测。

```powershell
.\.venv\Scripts\python.exe -m tensorboard.main --logdir "应用场景分类基准测试代码/Log1"
```

详细说明见 [训练运行说明](应用场景分类基准测试代码/README_run.md)。其中历史本机路径应替换为自己的仓库路径。

### 2. NLOS/多径分类

#### 输入文件

默认读取 `NLOS_multipath分类基准测试基准代码/input/`。每个场景需要一份跟踪观测文件和一份 NLOS/多径标签文件：

```text
NLOS_multipath分类基准测试基准代码/input/
├── track_data_alls_r2.csv
├── r2_卫星历元_NLOS多径分类标签.csv
├── track_data_alls_wd.csv
├── wd_卫星历元_NLOS多径分类标签.csv
├── track_data_alls_wyw.csv
├── wyw_卫星历元_NLOS多径分类标签.csv
├── track_data_alls_xhl.csv
└── xhl_卫星历元_NLOS多径分类标签.csv
```

不同输入维度读取的字段如下：

| 输入维度 | 跟踪观测 CSV 的必需字段 |
| ---: | --- |
| 3 | `signalId`, `prn`, `readMsCnt`, `week`, `tow`, `chState`, `CN0`, `pseudorange`, `carrierFreq` |
| 101 | `signalId`, `prn`, `readMsCnt`, `week`, `tow`, `chState`，以及 `I_E_50`…`I_E_1`, `I_P`, `I_L_1`…`I_L_50` 和对应的全部 Q 列 |
| 104 | 同时包含上述 3 维与 101 维所需的全部字段 |

| 标签字段 | 格式与含义 |
| --- | --- |
| `gps_week` | GPS 周，数值 |
| `gps_tow_s` | GPS 周内秒，数值；程序按毫秒取整匹配 |
| `satellite_prn` | 带星座前缀的卫星编号，例如 `G01`、`C07`、`E12` |
| `label_id` | `0 = Multipath`，`1 = NLOS` |

标签按 GPS 毫秒时间和 `satellite_prn` 精确匹配，并且同一时间、同一卫星只能有一条标签。跟踪文件中的 `signalId` 必须在训练脚本的 `SIGNAL_INFO` 中有定义。没有匹配标签的观测不参与监督损失或评价指标；模型可用且特征有效时，可为其输出预测。

| 输入维度 | 特征 |
| ---: | --- |
| 3 | 伪距变化与多普勒的一致性残差 `D`、当前 `CN0`、相邻观测的 `ΔCN0` |
| 101 | 按 Prompt 幅值归一化的 101 点 I/Q 幅值，顺序为 `E50…E1, P, L1…L50` |
| 104 | 3 维物理特征与 101 维 I/Q 幅值拼接 |

物理特征使用跟踪记录中的 `pseudorange`、`carrierFreq`、`CN0` 及时间字段；I/Q 输入使用 `I_E_50`…`I_E_1`、`I_P`、`I_L_1`…`I_L_50` 及对应 Q 列。具体字段与信号枚举见各训练脚本。

以下示例分别训练三种网络的 104 维输入版本：

```powershell
.\.venv\Scripts\python.exe -X utf8 -u "NLOS_multipath分类基准测试基准代码/mlp_nlos_multipath.py" --input-dim 104 --device auto --output-dir "NLOS_multipath分类基准测试基准代码/outputs/readme_mlp_104d"
.\.venv\Scripts\python.exe -X utf8 -u "NLOS_multipath分类基准测试基准代码/cnn_nlos_multipath.py" --input-dim 104 --device auto --output-dir "NLOS_multipath分类基准测试基准代码/outputs/readme_cnn_104d"
.\.venv\Scripts\python.exe -X utf8 -u "NLOS_multipath分类基准测试基准代码/resnet_nlos_multipath.py" --input-dim 104 --device auto --output-dir "NLOS_multipath分类基准测试基准代码/outputs/readme_resnet_104d"
```

将 `--input-dim` 改为 `3` 或 `101` 可运行其他特征设置，并应使用新的输出目录。重复实验也应另设输出目录，以保留已有结果。

常用参数包括 `--scenes r2`、`--epochs 200`、`--patience 25`、`--batch-size 256`、`--device cpu`、`--device cuda:0`、`--input-dir` 和 `--label-dir`。三个入口均支持 `--help`。

此任务按场景分别训练：先将全部 `chState == 3` 的观测逐条随机划分为 70% / 15% / 15%，再筛选各集合中有标签且特征有效的样本。它与场景分类的历元划分不同，同一历元的不同卫星或信号可以分入不同集合。

输出包含实验配置、数据审计、分场景模型与标准化器、划分清单、训练历史、分类结果及指标。现有数据中，`xhl` 精确匹配后的有效监督样本缺少 NLOS 类，训练状态为 `skipped_single_class`，不能将跳过的结果视作零分。

### 3. 定位基准测试

#### 输入文件

完整入口为 `定位基准测试代码/scripts/run_all.py`，执行 LSQ/WLS 求解、RTK 真值对比、离群点处理和绘图。当前 `定位基准测试代码/input/` 为空，可直接指定数据集中的输入目录：

| 输入 | 是否必需 | 默认字段或格式 | 作用 |
| --- | --- | --- | --- |
| 观测 CSV | 是 | `epoch_id`, `gps_week`, `sow`, `sat_sys`, `signal_id`, `signal_name`, `sat_prn`, `is_valid`, `converged`, `main_path_pseudorange_m`, `sat_x_m`, `sat_y_m`, `sat_z_m`, `el_deg` | 计算 LSQ/WLS 定位解 |
| 改正数 CSV | 否 | `signalId`, `prn`, `readMsCnt`, `week`, `tow`, `pseudorange`, `Clk`, `Iono`, `Trop` | 按信号、卫星和历元匹配钟差、电离层及对流层改正；使用 `--no-corrections` 可跳过 |
| RTK 真值 | 是 | 含校验和正确的 NMEA `RMC` 和有效 `GGA` 语句 | 提供日期、时间、经纬度、高程和定位质量，用于真值对齐与误差计算 |
| SPP-RAIM 文件 | 是 | NMEA 文本，或包含 NMEA 文本的 UBX 混合流；需有 `RMC` 和有效 `GGA` | 与 RTK 真值比较 SPP-RAIM 定位结果 |
| SPP 文件 | 否 | 含 `RMC` 和有效 `GGA` 的 NMEA 文件 | 存在时执行四算法比较；缺失时执行三算法比较 |

观测 CSV 中仅保留 `is_valid == 1`、`converged == 1`，且主径伪距和卫星 ECEF 坐标有效的记录。默认字段名可通过 `--observation-schema` 和 `--correction-schema` 指定 JSON 映射进行适配。`sat_sys` 使用 `BDS`、`GPS`、`GAL` 等系统标识，`signal_name` 应与配置中的参考信号名称一致。

一个可自动识别的输入目录示例：

```text
定位基准测试代码/input/
├── 主径伪距分量.csv                 # 观测 CSV
├── KL6软件接收机处理结果.csv        # 可选改正数 CSV
├── 诺瓦泰RTK定位结果.gps            # RTK 真值
├── SPP-RAIM.ubx                     # SPP-RAIM
└── SPP.gps                          # 可选 SPP
```

```powershell
.\.venv\Scripts\python.exe -X utf8 "定位基准测试代码/scripts/run_all.py" --input-dir "../数据集/软件园2期/定位基准测试文件/输入" --output-dir "output/readme_r2"
```

**该入口的相对路径以 `定位基准测试代码/` 为基准解析**，不以启动命令时的工作目录为基准。上述示例使用现有软件园2期输入中的主径伪距 CSV、KL6 改正数据、RTK 定位结果、`SPP-RAIM.ubx` 和 `SPP.gps`。

输入文件会按列结构或文件名识别。使用其他数据时，可通过 `--observations`、`--correction-table`、`--rtk`、`--spp`、`--spp-raim` 显式指定；字段映射可参考 [配置示例](定位基准测试代码/configs/pseudorange_lsq_wls_schema.example.json)。SPP 文件缺失时仅比较其余三种算法。

```powershell
.\.venv\Scripts\python.exe "定位基准测试代码/scripts/run_all.py" --help
```

输出包括 `positions_lsq.csv`、`positions_wls.csv`、观测残差、逐历元误差、成功率与误差统计、Markdown 报告及轨迹/误差图。默认三维误差阈值为 300 m，可通过 `--threshold-m` 修改；超阈值历元计为定位失败，从误差统计中剔除，解读结果时应同时查看成功率。

默认初始 ECEF 坐标来自 RTK 真值位置的中位数。若需要评估不使用真值初始化的定位流程，应通过 `--initial-ecef X Y Z` 提供独立的粗略初始位置，并在报告中记录这一设置。

### 4. SAGE 多径估计与定位扩展

`MultiPathEsiti/` 保存 MATLAB 多径估计代码及相关实验。主要过程为：读取相关器 I/Q、构造幅值包络、估计路径时延与幅度、选择主径/次径，再导出路径伪距、残差和定位特征。

#### 输入文件

SAGE 模块包含两类入口，所需输入如下：

| 入口 | 输入 | 必需字段 |
| --- | --- | --- |
| `external_driver/RunSageFromRawCIR.m` | 原始相关器 CSV | `prn`, `readMsCnt`，以及至少三组延迟位置相同的 I/Q 抽头；抽头命名为 `I_E_n`/`Q_E_n`、`I_P`/`Q_P`、`I_L_n`/`Q_L_n` |
| `main_sage_iq_main_sub_delay.m` | 完整相关器 CSV | 默认需要 201 个 I 抽头和 201 个 Q 抽头：`E_100…E_1, P, L_1…L_100` |
| `main_sage_iq_main_sub_delay_large_csv.m` | 大型相关器 CSV | 与全量入口相同；通过 datastore 分块读取，默认每侧 100 个抽头 |

`signalId` 是外部驱动的可选筛选字段；当 `UseOnlySignalRows=true` 时用于只保留指定信号。常用可选元数据包括 `week`、`tow`/`sow`、`sys`、`CN0`、`pseudorange`、`Xp`、`Yp`、`Zp`、`Clk`、`elevation`/`el_deg` 和 `corSpace`。若要生成带 ENU/LOS 信息的定位特征，应提供卫星 ECEF 坐标 `Xp`、`Yp`、`Zp`，并在入口脚本中设置与数据一致的接收机参考坐标。

SAGE 输出继续用于主径/副径神经网络定位时，Python 训练入口需要：

| 文件 | 必需字段 |
| --- | --- |
| 特征宽表 CSV | `gps_week`, `sow`, `epoch_id`；每个卫星/信号槽位至少包含 `<slot>_E`, `<slot>_N`, `<slot>_U`, `<slot>_main_diff_pr`；`main_sub` 模式还需要 `<slot>_sub1_diff_pr`, `<slot>_sub2_diff_pr` |
| ENU 真值 CSV | 至少包含 `gps_week`, `sow`，以及 `truth_e_m/truth_n_m/truth_u_m`、`e_m/n_m/u_m` 或 `enu_e_m/enu_n_m/enu_u_m` 三组字段中的一组；可带 `epoch_id` 共同匹配 |

例如：

```powershell
.\.venv\Scripts\python.exe "MultiPathEsiti/sage_test/MultiPathEsiti/MultiPathEsiti/python_nn_main_path/train_main_path.py" --input-csv "数据下载目录/epoch_sat_diff_pseudorange_multi_signal_matrix_los.csv" --label-csv "数据下载目录/truth_enu_aligned.csv" --feature-mode main_sub --output-dir "MultiPathEsiti/sage_test/MultiPathEsiti/MultiPathEsiti/python_nn_main_path/outputs/main_sub_run"
```

由于该目录包含多个实验版本，运行前需按所选入口配置输入文件、项目路径和输出目录。相关说明：

- [外部 SAGE 驱动](MultiPathEsiti/external_driver/README_external_driver.md)：从原始相关器数据调用已有 SAGE 核心。
- [MATLAB SAGE 模块说明](MultiPathEsiti/sage_test/MultiPathEsiti/README.md)：全量读取与大 CSV 分块处理的结构和入口。
- [主径/副径神经网络定位](MultiPathEsiti/sage_test/MultiPathEsiti/MultiPathEsiti/python_nn_main_path/README.md)：使用主径或主径加副径特征预测 ENU 坐标。

## 已有实验结果

| 实验 | 当前记录 |
| --- | --- |
| 两维场景分类 MLP | 共 6,990 个有效历元；训练/验证/测试为 4,893 / 1,048 / 1,049；测试准确率 **65.3956%**，宏平均 F1 **0.617958** |
| NLOS/多径分类 | 已有 MLP、CNN、ResNet × 3 种输入维度 × 3 个可训练场景，共 27 组 GPU 训练结果；`xhl` 各配置因单类别跳过 |

上述为仓库已有实验记录，不是新环境运行后的性能保证。详细配置、指标和结果解释见：

- [场景分类训练结果](应用场景分类基准测试代码/training_result.md)
- [NLOS/多径三种输入维度结果汇总](NLOS_multipath分类基准测试基准代码/三种输入维度基准测试结果汇总.md)

分类实验采用同一批采集数据内的随机留出测试。相邻历元相关性、采集文件差异，以及 NLOS 任务中同历元观测跨集合分配，都可能影响指标，现有结果不能代表独立日期、路线或采集批次上的泛化能力。NLOS 类样本较少，比较模型时应结合平衡准确率、NLOS 精确率/召回率和 F1，不宜只看总体准确率。

## 数据与发布说明

本仓库当前通过 Git LFS 提供以下数据子集：

| 数据类别 | 场景 | 内容 |
| --- | --- | --- |
| Xsens 结果文件 | 四个场景 | Xsens 导航/惯导导出结果及相关记录 |
| KL6 软件接收机处理结果 | 四个场景 | 跟踪、定位、RINEX 及汇总结果 |
| 定位基准测试文件 | 四个场景 | 定位输入、LSQ/WLS 输出、统计报告与图表 |
| NovAtel 定位结果转换文件 | 四个场景 | ASCII、RINEX 等格式转换结果 |
| 软件园2期相机分卷 | 软件园2期 | `软二影像数据/` 下的相机 PNG 帧，下载后需先合并 4 个分卷 |

湖边万达、五缘湾和厦禾路的相机数据，以及原始中频 IQ，未包含在 Git 仓库中。应用场景分类和 NLOS/多径分类所需的专用 `input/` 文件也没有重复存储；可从上述 KL6 结果和配套标签整理后放入各模块的 `input/` 目录。

虚拟环境 `.venv/`、下载缓存 `.downloads/`、`__pycache__/`、临时目录、重复备份和模型权重不随仓库分发。Git LFS 文件会消耗仓库所有者的 LFS 存储与下载流量；仅需代码的用户应使用上方的跳过 LFS 命令。

仓库根目录当前未提供统一的 `LICENSE` 或正式论文引用信息。代码和数据的授权范围、第三方模块归属及引用方式，应在正式发布时由维护者明确。


---

## 本仓库 Release 的下载与使用

本仓库通过 [Release `dataset-v1`](https://github.com/tian1714522/citypdp/releases/tag/dataset-v1) 提供打包好的数据与代码，**无需 Git LFS** 即可完整获取。

### 1. 完整数据（分卷压缩包）

- 共 **21 个分卷**（`dataset.7z.001` … `dataset.7z.021`），每卷 2000 MiB，归档总大小 **40.04 GB**
- 原始内容：53,164,138,359 字节 / 176,175 个文件 / 939 个目录（打包时已排除 `.git`、`.venv`、`.downloads`、`__pycache__`）
- 使用方法：
  1. 将**全部分卷**下载到同一目录（文件名不要修改）
  2. 用 [7-Zip](https://www.7-zip.org/) 打开 `dataset.7z.001` 解压
- 完整性校验：`checksums.sha256.txt` 列出各分卷 SHA-256，例如 `certutil -hashfile dataset.7z.001 SHA256`

### 2. 代码

- `source-code.zip` — 源码快照（Python / MATLAB 源码、Markdown 文档、配置 JSON，以及小于 5 MB 的样本 CSV 与图片），共 4822 个文件
- `repo-main.bundle` — 完整 Git 历史（4 个提交），可用 `git clone repo-main.bundle` 还原仓库；大型数据文件在其中为 Git LFS 指针

> 大型数据文件（如 `software_camera.tar`、超过 5 MB 的 CSV / 图片）未重复打包，均已包含在上述数据分卷中。
> 需要 Git LFS 原始数据与提交历史时，请克隆 [Laanyz/multi-scene-gnss-benchmark](https://github.com/Laanyz/multi-scene-gnss-benchmark)。
