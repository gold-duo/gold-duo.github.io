---
title: 17Particles
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

粒子是动态生成的小物体（用于制造烟雾），这节除了制造烟雾还给开枪时在枪口制造几个粒子

## GPUParticles2D/CPUParticles2D 节点生成粒子效果

### Process Material：

处理材质(有两种ShaderProcessMaterial、ParticleProcessMaerial)

- Texture:渲染的纹理，不设置是一个小的四方体

- ParticleProcessMaterial
    - Spawan->Position -> Emission Shape:发射形状(Sphere[球体]、Point...)
        -Sphere: Sphere radius决定球体大小
    - Spawan->Position->Volocity
        - Direction方向:xyz=(1,0,0)则粒子只在x轴上移动
        - Initial Velocity Min/Max：初始速度
        - Spread：初始方向范围
    - Display
        - Scale:godot在此设置的最大最小范围**随机缩放粒子**
        - Animation
            - Offset Min:最小帧.0是第一帧，1是最后一帧
            - Offset Max:最大帧.
    - Accelerations
        - Gravity(重力决定粒子下落速度)
            - x、y：xy坐标的重力。(都为0则不会运动，y为负数则向上发射)
    - Color curves:发射粒子的颜色
        - color Ramp(色带):GradientTexure1D(渐变。粒子从开始发射渐变到消失时的颜色。基本处理两个颜色：开始、结束)


### Time
- Lifetime:存活时长
- One Shot：只发射一次

### Emitting:是否发射（代码调用`node.emitting=true`）
- Amount:粒子数量

## 这粒子效果的制造，属性这么多完全就是拼体力

## 苦口婆心：粒子消耗性能