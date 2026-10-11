# 四元数旋转变换演示

一个纯前端、零依赖的四元数（Quaternion）旋转可视化演示程序。在浏览器里拖拽旋转一个三维立方体，同时实时查看对应的**四元数**、**轴-角表示**、**欧拉角**和**旋转矩阵**，帮助直观理解四元数如何表示三维旋转。

## 运行方式

无需安装任何依赖，直接用浏览器打开即可：

```
双击  quaternion-rotation-demo.html
```

或在终端执行（任选其一）：

```bash
start quaternion-rotation-demo.html      # Windows
open  quaternion-rotation-demo.html      # macOS
xdg-open quaternion-rotation-demo.html   # Linux
```

程序完全离线运行，不需要网络。

## 功能特性

### 交互操作
- **拖拽旋转**：在画布上按住鼠标（或触摸）拖动，通过 Shoemake 轨迹球（Arcball）将拖拽映射到单位球面，再用四元数累乘 `q = Δq ⊗ q` 实现平滑旋转
- **主动旋转**：用三条滑杆设置旋转轴 `(x, y, z)` 与角度 `θ`，然后：
  - **应用旋转** `q = R·q`：在当前姿态上叠加一次旋转
  - **设为旋转** `q = R`：直接设为该旋转
- **快捷按钮**：绕 X / Y / Z 轴 90°、随机轴、重置为恒等
- **自动旋转**：勾选后绕当前滑杆轴连续旋转

### 实时读数
- 四元数分量 `w, x, y, z`（带彩色分条可视化）+ 模长 `|q|`
- 轴-角表示：旋转轴向量 + 旋转角 `θ`
- 欧拉角（ZYX 顺序：偏航 Yaw / 俯仰 Pitch / 滚转 Roll）
- 3×3 旋转矩阵 `R`

### 三维可视化
- 六面不同颜色的立方体（带方向光照、背面剔除与画家算法）
- 随物体旋转的 RGB 坐标轴（X 红 / Y 绿 / Z 蓝），显示当前姿态
- **琥珀色**实线：当前四元数对应的旋转轴
- **青色**虚线：待应用（滑杆设置）的旋转轴
- 灰色细线：未旋转的初始参考坐标轴

## 数学约定

| 项目 | 约定 |
| --- | --- |
| 四元数乘法 | Hamilton 乘积 `q = q₁ ⊗ q₂`（代码 `Quat.multiply`） |
| 姿态组合 | 世界系（左乘）：`q_new = R ⊗ q_old` |
| 轴-角构造 | `q = (cos(θ/2), sin(θ/2)·â)`，`â` 为归一化轴 |
| 向量旋转 | `v' = q · (0,v) · q*`（代码 `Quat.rotateVector`） |
| 轨迹球 | Shoemake Arcball，增量四元数轴取 `end × start` |
| 欧拉角 | ZYX 顺序，`R = Rz(ψ)·Ry(θ)·Rx(φ)`，角度制 |
| 旋转矩阵 | 标准四元数 → 旋转矩阵展开式（代码 `Quat.toMatrix`） |

> 关于旋转顺序的直观理解：`q = R ⊗ q_old` 表示“先做旧姿态，再绕世界坐标系的轴做一次 R 旋转”。滑杆轴在应用前会自动归一化；全零轴会回退为 `(0, 0, 1)`。

## 文件结构

```
.
├── quaternion-rotation-demo.html   # 单文件程序（HTML + CSS + JS 全部内联）
└── README.md                       # 本说明
```

代码组织（在 HTML 的 `<script>` 内）：

- **数学部分**：`vec*` 向量工具函数、`Quat` 四元数类（乘、共轭、归一化、向量旋转、轴-角、欧拉角、矩阵转换）
- **交互部分**：轨迹球（Pointer Events 拖拽映射）
- **渲染部分**：透视投影、立方体面着色、坐标轴与旋转轴绘制
- **读数部分**：面板实时更新与控件绑定

## 常见调整

- **拖拽方向感觉反了**：把 `quaternion-rotation-demo.html` 中 `vCross(cur, lastSphere)` 改为 `vCross(lastSphere, cur)`（约第 544 行）。
- **旋转速度**：修改自动旋转的角速度 `dt * 0.9`（约 0.9 rad/s）。
- **欧拉角顺序**：如需 XYZ 等其他约定，调整 `Quat.toEulerDeg` 中的提取公式。
- **立方体尺寸 / 相机距离**：调整 `cubeHalf`、`project()` 中的 `d` 与焦距系数。

## 数学正确性

核心数学逻辑（四元数旋转、轴-角往返、欧拉角提取、矩阵展开、姿态组合）已用 Node 脚本验证：

```
PASS  rotate +X by 90 deg about Z -> +Y
PASS  roundtrip angle=90
PASS  roundtrip axis=+Z
PASS  euler yaw=90 for Z-rot
PASS  matrix R00=0 / R01=-1 / R10=1
PASS  X90 then Y90 (world) on +Z -> (0,-1,0)
PASS  |q| stays 1
PASS  identity leaves vector
PASS  euler pitch=90 for Y-rot
```
