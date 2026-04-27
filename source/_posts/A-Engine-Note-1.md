---
layout: post
title: 小引擎随笔(1) - 渲染RHI架构设计
date: 2026/4/23 12:44:00
tags: [A-Engine, Rendering]
author: Henry Han
cover: https://sirenri2001.github.io/assets/cornell.png
---

# 引言

现在想来，小引擎这个项目也已经断断续续写了一年多了。这个项目起初是本人学习Vulkan时，重构vulkan-tutorial上面的教程代码的，后来有了一个想法，加D3D12后端和仿照UE写一套渲染相关的实现。同时也顺便补一下我拿了B-的Advanced Graphics。其实此时AI已经发展的相当厉害了，去年的这个时候我还以为AI不会看文档，不会解决复杂问题，但是现在看来，小引擎内部的一些实现（包括PBR和Importance Sampling）我还必须得请教AI。翻阅PBRT的痛苦谁看谁知道。但后续本人可能会结合自己读过的部分写一些笔记，但这都是后话了。这个博客就让我聊聊小引擎里我最引以为傲的部分：RHI层。

# RHI层简介

全程是Render Hardware Interface，即渲染硬件接口。一般定义是RHI封装了不同Graphics API之间的细节，提供了一个统一的架构。这个硬件接口的定义并非关系Nvidia、AMD等硬件厂商，其实在一般实现中我们主要围绕不同Graphics API的行为去做RHI的封装。当然，一些Graphics API有对应硬件厂商的特殊API，但目前我们不在这里进行讨论。

Graphics API主要包括：
- **OpenGL**: 最为经典（或者古老）的API，使用相对简单，但是也逐渐被淘汰。目前主要用于教学。一般设置固定管线或者使用可编程管线编写glsl shader进行渲染。
- **Vulkan**：现代API，OpenGL的后继。可以使用跨平台spir-v，原生支持还是glsl。同时命名也保持了一致（如uniform，location binding等）。所有人都在说Vulkan的初始化很复杂，不过我个人认为，vulkan的API定义十分细致，以至于任何一个错误都会出现在vulkan文档中，而不是fall through到driver，至少目前我开发引擎过程中没有遇到这种情况。相比较D3D12有时只给一个invalid parameter这种错误，我认为vulkan api还是相对容易使用。同时vulkan可以运行在android端设备，虽然我还没有尝试过。
- **DX系列**：其实我应该将DX11和DX12分开讲，但是我只用过DX12。DX12可以看到微软做了很多兼容策略，比如使用DXGI这种可以跨各个DX版本的封装。其实这样设计也能理解，因为DX得考虑Windows不同版本，以及XBOX硬件的适配。不过从OpenGL转到现代API，还是使用vulkan最为舒适。
- **Web系列**：首先WebGL看起来是发展相对成熟。当然WebGPU也算一类。但是WebGPU终归还是限制太多了，而且开发的shader语言是slang，同时WebGPU对于一些浏览器、特定渲染功能的支持还不是很好。最重要的是，WebGPU用不了renderdoc。不过，从API封装的角度来说，由于web端平台统一，所以WebGPU的封装做的相当好。相对于Vulkan来说更加顺手和符合逻辑。
- **Metal**：没用过所以不太清楚。

# RHI层架构

A-Engine主要是将Vulkan和D3D12（按照UE的命名规范）两种API的主要功能层面做了一个封装。两种API的语义很接近，所以这个工作并不是特别困难。下面对RHI的主要流程做一个拆解：

1. Context管理：其实对应的就是VkDevice，功能上负责创建API的环境，根据硬件选择设备，窗口创建，以及设备队列的等待操作。设备同时也需要创建各类对象，但我将其放在了下列每个对象的Initialize函数中。
2. Image Resource和Buffer Resource管理：对应2D/3D贴图，mipmap创建，sampler, resource state transition以及host - device，device - device之间的拷贝操作。当然这些资源的所有拷贝和转换操作都需要传入command buffer进行。
3. Pipeline管理：由Pipeline factory对应其中的一个shader file set(VS/PS或CS)，然后factory创建出一个pipeline object，绑定所有实际的资源，以及给command buffer进行draw call或dispatch操作。
4. Command buffer管理：由Context分配command pool。目前暂时由Swapchain执行command buffer的submit操作，后续这里需要重构修改。这里的设计逻辑与UE不同，command buffer只是作为一个对象，而没有transition、copy函数，所有相关的函数均放在resource类中。
5. RenderPass和FrameBuffer：RenderPass你只需要知道target格式就可以创建了，但是framebuffer需要传入具体的resource。所以目前的逻辑是，Pipeline factory创建时不需要任何一个，但是创建pipeline object时递入render pass确定target格式，执行Begin renderpass时传入frame buffer。这样所有资源在begin renderpass之前都能创建好，同时也可以解决一个frame buffer对多个renderpass，或者一个begin renderpass对多个pipeline的情况。
6. Swapchain：这个是最头疼的地方。因为要确保host device之间同步。目前暂时的解决办法就是停等，后续可能加入其他的semaphore处理同步操作。

今天先写到这里。下次更新D3D12的Descriptor Heap分配策略和Resource state transition实现策略。
