# LSS UAV Reconnaissance Effectiveness — Simulation Code

低慢小无人机对地面移动目标侦察效能的仿真与统计分析代码
Simulation and statistical analysis code for the reconnaissance effectiveness
of low-altitude slow small (LSS) UAVs against ground moving targets.

配套论文 / Companion paper:

> 低慢小无人机对地面移动目标侦察效果的试验设计与仿真分析
> Experimental Design and Simulation Analysis of Reconnaissance Performance
> for Low-Altitude Slow Small UAV Against Ground Moving Targets

---

## 1. 这是什么 / What this is

本仓库包含论文第 2 节模型的完整实现、第 3 节的全因子仿真、以及第 4 节的全部统计分析，
用于复现文中所有数值结果（144 个设计点、720 次重复仿真、方差分析、回归与无量纲化简）。

This repository contains the full implementation of the model in Section 2, the
full-factorial design of Section 3, and all statistical analyses of Section 4.
Running it reproduces every number reported in the paper.

---

## 2. 模型 / Model

```
GSD    = H · d_s / (f · N_p)                  [m/px]
N      = L / GSD                              [cycles]
P(N)   = (N/N50)^k / (1 + (N/N50)^k),  k = 2.7,  N50 = 1 / 4 / 8
alpha  = exp(−3.912 · H_km / R_vis)           Koschmieder, nadir path
B      = (v_t + eps · v_u) · T_int / GSD      [px], worst case
rho    = exp(−1.2 · min(1, B/N))
gamma  = min(1, lg(1 + A/GSD²) / 3)
RE     = 100 · (w1·P_D + w2·P_R + w3·P_I) · alpha · rho · gamma
```

参数 / Parameters: `f = 35 mm`, `d_s = 17.3 mm`, `N_p = 4000`, `fps = 30`,
`v_u = 15 m/s`, `eps = 0.15`, `w = (0.20, 0.40, 0.40)`.

设计 / Design: 4 × 3 × 3 × 4 = 144 cells, 5 replicates with ±2 % multiplicative
jitter on the inputs, uniform, seed = 42 → 720 runs.

| 因子 / Factor | 水平 / Levels |
|---|---|
| A 能见度 visibility (km) | 15, 7, 3, 0.8 |
| B 车辆长度 length (m) | 2, 3.5, 6 |
| C 车速 speed (km/h) | 20, 60, 100 |
| D 飞行高度 altitude (m) | 50, 100, 150, 200 |

---

## 3. 运行 / How to run

```bash
pip install -r requirements.txt
python sim_recon.py
```

Requires Python ≥ 3.9 and NumPy only. No other dependencies.

---

## 4. 输出 / Outputs

Written to `results/`:

| 文件 | 内容 |
|---|---|
| `results.txt` | 全部统计量（描述统计、ANOVA、η²、回归、无量纲化简） |
| `sim_cells.csv` | 144 个格子的 RE 均值（A,B,C,D,RE） |
| `sim_reps.csv` | 720 次重复的原始结果（A,B,C,D,rep,RE） |

---

## 5. 关键结果 / Key results

| 项目 | 数值 |
|---|---|
| 720 次 RE 均值 ± 标准差 | 66.01 ± 17.62 |
| 最小 / 最大 | 20.01 / 93.76 |
| 中位数 (Q1, Q3) | 68.54 (53.35, 79.95) |
| 极差 A / C / B / D | 32.32 / 17.84 / 15.50 / 13.72 |
| η² A / C / B / D | 0.520 / 0.171 / 0.131 / 0.084 |
| 交互项 η² (A×D) | 0.059 |
| 二次响应面 R²（144 设计点） | 0.9804 |
| 无量纲闭式解 R²（144 设计点） | 1.000000，最大误差 0.046 分 |

无量纲闭式解 / Closed form:

```
ln RE = 4.6054 − 3.913e−3·(H/R) − 0.0400·(v/L) − 0.0911·(1/L)
```

拟合系数与理论值（−3.912e−3、−1.2T、−1.2T·eps·v_u）一致。

---

## 6. 说明 / Notes

- 重复仿真的扰动施加在**输入参数**上，经模型传播后形成误差项，不是在输出端叠加随机噪声。
- 结果对随机种子不敏感（`SEED` 可改），但 F 值会随种子变化，η² 与极差稳定。
- 本代码仅用于学术复现；模型中的经验系数（1.2、0.15 等）未经过实测标定。

---

## 7. 许可 / License

MIT License, see `LICENSE`.

---

## 8. 引用 / Citation

若使用本代码，请引用上述论文（DOI 待补 / DOI to be added upon publication）。
