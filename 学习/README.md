# Safety Island (安全岛) 模块

> **符合 ISO 26262 ASIL-D 汽车电子功能安全要求的硬件安全监控模块**

## 项目概述

本工程实现了符合 ASIL-D 等级的"安全岛"（Safety Island）RTL 设计及完整 UVM 验证环境。模块通过 AXI4 总线接口周期性读取外部安全寄存器，进行掩码和按位 OR 处理，判定并输出故障信号。

### 关键指标
- **SPFM**: ≥99% (ASIL-D 单点故障度量)
- **LFM**: ≥90% (ASIL-D 潜伏故障度量)
- **故障检测延迟**: ≤10 clock cycles
- **仿真工具**: Synopsys VCS + Verdi

---

## 目录结构

```
safety_island/
├── rtl/                   # 可综合 SystemVerilog RTL 设计
│   ├── safety_island_top.sv       # 顶层模块
│   ├── axi_slave_cfg.sv           # AXI4 Slave 配置接口
│   ├── cfg_register_file.sv       # ECC 保护配置寄存器组
│   ├── read_scheduler.sv          # TMR 保护周期读取调度器
│   ├── axi_master_read_engine.sv  # AXI4 Master 读引擎 (×5)
│   ├── data_processor.sv          # 掩码 + OR 累加处理器
│   ├── fault_detector.sv          # 系统故障检测
│   ├── fault_manager.sv           # 内部故障聚合管理
│   ├── timeout_monitor.sv         # 超时监控器
│   ├── tmr_voter.sv               # TMR 多数表决器
│   ├── ecc_encoder.sv             # SEC-DED ECC 编码器
│   ├── ecc_decoder.sv             # SEC-DED ECC 解码器
│   ├── crc_calculator.sv          # CRC-8 计算器
│   ├── fsm_checker.sv             # FSM 状态检查器
│   ├── parity_checker.sv          # 总线校验位检查 (AoU)
│   ├── self_test_controller.sv    # 潜伏故障自检 (附加分)
│   ├── reorder_buffer.sv          # 乱序重排缓冲 (附加分)
│   └── includes/
│       ├── axi4_types_pkg.sv      # AXI4 类型定义
│       └── safety_island_pkg.sv   # 安全岛参数与枚举
│
├── tb/                    # UVM 验证环境
│   ├── top_tb.sv                  # 顶层 Testbench
│   ├── si_test_pkg.sv             # UVM 包
│   ├── env/                       # UVM 环境组件
│   │   ├── si_env.sv
│   │   ├── si_env_config.sv
│   │   ├── si_scoreboard.sv
│   │   ├── si_coverage.sv
│   │   └── si_virtual_sequencer.sv
│   ├── agent/
│   │   ├── si_axi_slave_agent/    # S_AXI Agent (TB → DUT 配置)
│   │   └── si_axi_master_agent/   # M_AXI Agent (TB ← DUT 监控)
│   ├── sequence/                  # UVM 测试序列
│   ├── fault_injection/           # 故障注入框架
│   └── test/                      # UVM 测试用例
│
├── sim/                   # 仿真脚本
│   ├── Makefile                   # VCS 编译运行脚本
│   ├── rtl_filelist.f
│   ├── tb_filelist.f
│   └── waves.tcl
│
├── scripts/               # Python 自动化脚本
│   ├── run_regression.py          # 回归测试
│   ├── fault_campaign.py          # 故障注入
│   └── spfm_lfm_calculator.py     # SPFM/LFM 计算
│
├── doc/                   # 设计文档
│   ├── design_specification.md    # 详细设计规范
│   ├── safety_analysis.md         # 安全分析 (FMEDA)
│   └── verification_plan.md       # 验证计划
│
└── README.md
```

---

## 环境要求

| 工具 | 版本 | 用途 |
|------|------|------|
| Synopsys VCS | 2020.03+ | RTL 编译与仿真 |
| Verdi | 2020.03+ | 波形查看 (可选) |
| Python | 3.7+ | 自动化脚本 |
| GNU Make | ≥4.0 | 编译控制 |

---

## 快速开始

### 1. 编译设计

```bash
cd sim
make compile
```

### 2. 运行冒烟测试

```bash
make smoke
```

### 3. 运行全部回归

```bash
make regression
```

### 4. 查看波形

```bash
make waves
```

---

## 仿真复现步骤 (VCS)

### 完整仿真流程

```bash
# 1. 进入仿真目录
cd sim

# 2. 编译 RTL + UVM 验证环境
vcs -full64 -sverilog -debug_access+all \
    -ntb_opts uvm-1.2 \
    -f rtl_filelist.f \
    -f tb_filelist.f \
    -top top_tb \
    -o simv

# 3. 运行冒烟测试
./simv +UVM_TESTNAME=si_smoke_test +UVM_VERBOSITY=UVM_MEDIUM -l smoke.log

# 4. 运行配置测试
./simv +UVM_TESTNAME=si_config_test +UVM_VERBOSITY=UVM_MEDIUM -l config.log

# 5. 运行正常操作测试
./simv +UVM_TESTNAME=si_normal_op_test +UVM_VERBOSITY=UVM_MEDIUM -l normal.log

# 6. 运行超时测试
./simv +UVM_TESTNAME=si_timeout_test +UVM_VERBOSITY=UVM_MEDIUM -l timeout.log

# 7. 运行错误响应测试
./simv +UVM_TESTNAME=si_error_resp_test +UVM_VERBOSITY=UVM_MEDIUM -l error.log

# 8. 运行压力测试
./simv +UVM_TESTNAME=si_stress_test +UVM_VERBOSITY=UVM_MEDIUM -l stress.log

# 9. 运行故障注入
./simv +UVM_TESTNAME=si_stuckat_campaign +UVM_VERBOSITY=UVM_LOW -l stuckat.log
./simv +UVM_TESTNAME=si_transient_campaign +UVM_VERBOSITY=UVM_LOW -l transient.log
```

### 使用 Make 简化

```bash
# 单独测试
make smoke      # 冒烟测试
make config     # 配置测试
make normal     # 正常操作测试
make timeout    # 超时测试
make error      # 错误响应测试
make stress     # 压力测试
make stuckat    # Stuck-at 故障注入
make transient  # 瞬态故障注入

# 全部回归
make regression

# 覆盖率报告
make coverage

# 清理
make clean
```

---

## 故障注入与安全分析

```bash
# 自动化故障注入
cd scripts
python fault_campaign.py

# SPFM/LFM 计算
python spfm_lfm_calculator.py

# 结果查看
cat ../results/reports/fmeda_report.txt
```

---

## 设计亮点

### 1. 安全机制 (ASIL-D)
- **TMR (三模冗余)** — FSM 状态、数据累加器、关键控制信号
- **SEC-DED ECC** — 配置寄存器 (Hamming 72,64)
- **CRC-8 写保护** — 防止非法配置写入
- **超时监控** — 每事务独立超时计数器
- **总线校验位 (AoU)** — AR/R 通道偶校验
- **FSM 状态检查** — 非法状态立即报警
- **定期 PRBS 自检** — 探测组合逻辑潜伏故障
- **ECC 可纠错误计数** — 存储退化潜伏故障探测

### 2. 附加分功能
- ✅ **Out-of-Order** 读数据接收 (reorder_buffer)
- ✅ **Interleaving** R 通道数据交织
- ✅ **latent_fault_detect** 潜伏故障检测信号

### 3. 验证完备性
- 10 个定向测试用例
- 故障注入框架 (stuck-at, transient, AXI 总线故障)
- 自动分类 (已纠正/已探知/未探知)
- 覆盖率收集与分析

---

## 测试列表

| 测试 | 描述 | 预计时间 |
|------|------|---------|
| si_smoke_test | 基本连通性检查 | ~1ms |
| si_config_test | 配置读写 + CRC 保护 | ~2ms |
| si_normal_op_test | 5 Master 正常操作 | ~3ms |
| si_timeout_test | 超时注入每个 Master | ~5ms |
| si_error_resp_test | SLVERR/DECERR 注入 | ~5ms |
| si_stress_test | 极限压力测试 | ~5ms |
| si_stuckat_campaign | Stuck-at 故障遍历 | ~10ms |
| si_transient_campaign | 瞬态故障遍历 | ~10ms |
| si_regression_test | 全功能回归 | ~5ms |

---

## 作者

本设计为 2026 年汽车电子功能安全芯片设计竞赛参赛作品。

## 许可证

此代码仅供竞赛评审使用。
