# Globelix

[![WebGPU](https://img.shields.io/badge/render-WebGPU%20native-blueviolet?style=flat-square)](https://www.w3.org/TR/webgpu/) [![TypeScript](https://img.shields.io/badge/language-TypeScript-3178c6?style=flat-square)](https://www.typescriptlang.org/) [![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

Globelix 是新一代**基于 WebGPU 原生构建的超现实三维引擎**，集**逼真实时绘制引擎**、**时空物理仿真引擎**与**三维 GIS 引擎**于一体。

引擎的核心命题是**全域多尺度连续视景渲染**：依托统一的场景与着色管线，实现从**微观物质细节**、**中观地表地物**，到**大尺度全球行星球面场景**的平滑连续 LOD 过渡与物理级逼真渲染仿真——在同一个连续视景中，从一颗螺丝的材质纹理推近到整颗行星，尺度切换不中断、不跳变、不降质。

Globelix 不是 WebGL 管线上的移植或兼容层，而是从资源模型、渲染图（Render Graph）、着色管线到 LOD 调度，围绕 WebGPU 的计算与渲染能力重新设计的原生引擎。

---

## ✨ 三大引擎，一套管线

### 1. 逼真实时绘制引擎

以**物理级光照与材质**为精度基线，构建可贯穿全尺度的实时绘制能力：

- 基于物理的材质与光照模型，统一的 BRDF / 能量守恒与色调映射链路
- 大气散射、体积介质（云、雾、气溶胶）与昼夜光照传输
- 屏幕空间与 GPU 端加速的全局光照近似、阴影与环境光遮蔽
- 模块化后处理链（抗锯齿、曝光、泛光、镜头效果），与场景光照在同一色彩空间内合成

### 2. 时空物理仿真引擎

把**时间**视为引擎的一等状态，而非外部动画变量：

- 统一时间轴驱动天体位置、日照角度、潮汐与气象场的连续演化
- 可回放、可推演、可对齐的时空状态快照（任意时刻场景全局一致）
- 物理场与渲染耦合：仿真结果直接进入绘制管线，而非离线烘焙
- GPU 计算管线承载大规模并行仿真，与渲染共享资源与调度

### 3. 三维 GIS 引擎

以真实地理坐标系为骨架的三维空间框架：

- 椭球 / 投影坐标框架、地理与笛卡尔空间的无损互转
- 地形（Quantized Mesh 等）、影像（XYZ / WMS / WMTS）与矢量（GeoJSON / MVT）多源承载
- 3D Tiles、glTF / GLB 三维模型与三维高斯泼溅场景接入
- 贴地、拾取、量算、通视与地形采样等空间分析能力

三者不是三个并列模块，而是同一套**统一场景与着色管线**上的三种能力切面——这正是跨尺度视觉一致性的前提。

---

## 🌐 全域多尺度连续视景渲染

这是 Globelix 区别于传统三维引擎的核心能力。传统方案通常是"微观一个引擎、宏观一个引擎"，跨尺度时要么切场景，要么牺牲精度与观感；Globelix 的目标是把整条尺度谱压进**一个连续视景**：

```text
微观物质细节  ←──────────────  连续 LOD 过渡  ──────────────→  全球行星球面
毫米 · 米          米 · 公里（地表地物 / 城市园区）          公里 · 万公里
    └──────────── 统一场景图 + 统一着色管线 + 统一光照模型 ────────────┘
```

支撑这一目标的四项关键设计：

- **连续 LOD**：层级切换以几何误差与屏幕空间投影误差为度量，过渡过程连续无跳变，跨尺度无裂缝、无接缝抖动
- **统一着色管线**：同一套材质、光照、大气与色彩管理贯穿全尺度，避免"近景一套着色、远景一套着色"的观感割裂
- **相机相对渲染（RTC）**：以相机为原点的相对坐标计算，配合深度精度优化，解决行星级尺度下的浮点精度塌陷
- **GPU 驱动调度**：剔除、LOD 选择与绘制提交尽量下沉到 GPU，把跨尺度海量节点的 CPU 开销压到最低

---

## 🧩 架构

```mermaid
%%{init: {"flowchart": {"subGraphTitleMargin": {"top": 8, "bottom": 16}}}}%%
flowchart TB
    subgraph app["应用层"]
        direction LR
        twin["数字孪生"]
        city["智慧城市"]
        sim["仿真推演"]
        planet["行星可视化"]
    end

    subgraph engine["Globelix 引擎"]
        direction TB

        subgraph engines["三大引擎切面"]
            direction LR
            render["逼真实时绘制引擎"]
            physics["时空物理仿真引擎"]
            gis["三维 GIS 引擎"]
        end

        subgraph core["统一场景与着色管线"]
            direction LR
            scene["统一场景图"]
            shading["统一着色管线"]
            lod["连续 LOD 调度"]
            time["统一时间轴"]
        end
    end

    subgraph gpu["WebGPU 原生层"]
        direction LR
        graph["Render Graph"]
        compute["Compute Pipeline"]
        res["显式资源与管线"]
    end

    browser["浏览器 / 系统 GPU"]

    app --> engines --> core --> gpu --> browser
```

---

## 🚀 快速开始

Globelix 处于早期研发阶段，首个预览版发布前，本节将提供安装方式、最小可运行示例与在线示例站。

目标形态是一个 ESM 包，初始化入口大致如下（**API 设计尚未定稿，仅示意**）：

```ts
import { Viewer } from "globelix"

const viewer = await Viewer.create({
  container: document.getElementById("app")!,
  // 从行星尺度到地物尺度，共用同一个 Viewer
})

viewer.clock.time = new Date("2026-10-15T12:00:00Z")
```

---

## 🗺️ 路线图

阶段性目标，随研发推进调整：

- [ ] WebGPU 设备与渲染图基础层
- [ ] 统一场景图与统一着色管线
- [ ] 地理坐标框架与地形 / 影像承载（三维 GIS 引擎）
- [ ] 连续 LOD 与相机相对渲染（全域多尺度）
- [ ] 物理级光照、大气与体积介质（逼真实时绘制引擎）
- [ ] 统一时间轴与 GPU 物理仿真（时空物理仿真引擎）
- [ ] 在线示例站与文档站

---

## 🌱 开源致谢

Globelix 站在 Web 图形与地理空间开源生态的肩膀上，特别致谢：

- [WebGPU](https://www.w3.org/TR/webgpu/) 规范与 Dawn / wgpu 等实现，让浏览器第一次拥有真正的现代图形与计算能力
- [Three.js](https://github.com/mrdoob/three.js) 等 Web 3D 项目十余年积累的工程范式与生态经验
- [3D Tiles](https://github.com/CesiumGS/3d-tiles)、[Quantized Mesh](https://github.com/CesiumGS/quantized-mesh) 等开放地理数据规范
- 所有推动 Web 实时图形与开源地理空间技术发展的贡献者们 ❤️

---

## ⚖️ License

[Apache-2.0](./LICENSE)。Globelix 可用于商业和非商业项目。
