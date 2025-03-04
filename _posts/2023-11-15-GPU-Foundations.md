---
title: GPU Foundations
tags: Rendering GPU
---

## Basic Concepts
- Wavefront(Wrap): GPUs are massively parallel processors, processing an incredible amount of data at the same time. Crucial aspect is that, most of the time, we have a lot of threads (or pixels if you prefer to think about pixel shaders) running the same shader. To deal with this obvious situation, threads are batched in groups that we’ll call Wavefronts or waves (or warps in Nvidia lingo).[6]

![](post_img/gpu_foundations/wavefront.png)

All the threads that are in a wave run the same shader in lockstep. This is why if you have a branch that is going to be divergent within a wave, you end up paying the cost of both cases.

![](post_img/gpu_foundations/thread_execute.gif)

We can see in the gif I have something called exec mask. This is a bitmask, one bit per thread in the wave. If the bit for a thread is set to 1 then it means the thread (or lane) is active, if set to 0, it is considered an inactive lane and therefore whatever gets executed on that lane is to be disregarded. This is why each thread will have the correct result even though both branches are executed.
- Vector registers (VGPR): for everything that has values that are diverging between threads in a wave. Most of your local variables will probably end up in VGPRs.[6]
- Scalar registers (SGPR): everything that is guaranteed to have the same values for all threads in a wave will be put in these. An easy example are values coming from constant buffers.[6]

Here is an example shader code for explaining  VGPR and SGPR:
```c++
cbuffer MyValues
{
float aValue;
};
 
Texture2D aTexture;
StructuredBuffer aStructuredBuffer;
 
float4 main(uint2 pixelCoord) : SV_Target
{
    // This will be in a SGPR
    float s_value = aValue;
 
    // This will be put in VGPRs via a VMEM load as pixelCoord is in VGPRs
    float4 v_textureSample = aTexture.Load(pixelCoord);
 
    // This will be put in SGPRs via a SMEM load as 0 is constant.
    SomeData s_someData = aStructuredBuffer.Load(0);
 
    // This should be an SALU op (output in SGPR) since both operands are in SGPRs
    // (Note, check note [0])
    float s_someModifier = s_value + s_someData.someField;
 
    // This will be a VALU (output in VGPR) since one operand is VGPR.
    float4 v_finalResult = s_someModifier * v_textureSample;
 
    return v_finalResult;
}
```

- Clock cycle: A clock is simply a (typically periodic) signal used to trigger some regular operation in a system. For example, there's a clock for your CPU's memory bus controller, which typically is different from the clock for your CPU's execution units; this is a very general term, and there's typically many different clocks in a complex system such a modern CPU. A clock cycle is just the time between two triggering clock signal events (e.g. rising edges) [3]
- Instruction cycle: the time it takes to execute an execution. One or multiple machine cycles, as it's the time between an instruction being fetched and the result of the execution taking effect. [3]
- Machine cycle: Ambiguously defined, because the term is older than the fact that basically any CPU you'll find is pipelined. A common definition is that it's the time it takes for the pipeline to advance one step – e.g. for the result of the decode stage to be moved to the first execution stage. This can be one to very many clock cycles of some underlying clock that drives the internal state machines of these pipeline stages. [3]
- Occupancy: Occupancy is the ratio of active warps to maximum supported active warps, occupancy is 100% if the number of active warps equals the maximum. [5] There are 4 factors that will impact on the Occupancy below:
  - Warps per SM
  - Thread groups per SM
  - Registers per SM
  - Shared Memory per SM

>It is essential to define the thread nums and thread groups in your compute shader codes. Because it will impact the occupancy by the amount of wraps. And how you define your thread numbers and thread groups will result in different numbers of wraps in your program. For example, if GPU supports 64 wraps per SM(CU) as the maximum, you dispatch 8 active thread groups with 256 threads per group (8 warps per group) results in 64 active warps, and 100% theoretical occupancy. Similarly, 16 active groups with 128 threads per group (4 warps per group) would also result in 64 active warps, and 100% theoretical occupancy.

## GPU Architecture
There are two major architecture types being tile-based and immediate-mode rendering GPUs. 
### Immediate Mode
![](post_img/gpu_foundations/immediate_mode.png)

__front-end stages__: before rasterization.

__Back-end stages__: begin at rasterization.

### Tile Based Mode
![](post_img/gpu_foundations/immediate_mode.png)

### comparison
Based on the description of [GPU architecture types explained](https://www.rastergrid.com/blog/gpu-tech/2021/07/gpu-architecture-types-explained/). We first need to look at the way how the two architectures access/exchange data throughout the pipeline.

![](post_img/gpu_foundations/imr_data_path.png)
*Key data paths of immediate-mode GPUs.*

![](post_img/gpu_foundations/tbr_data_path.png)
*Key data paths of tile-based GPUs.*

I quote the saying from [GPU architecture types explained](https://www.rastergrid.com/blog/gpu-tech/2021/07/gpu-architecture-types-explained/):

>In contrast, TBR GPUs write the primitive data off-chip into per-tile primitive bins which then are consumed by the subsequent per-tile operations issued in the second phase of the pipeline.\
>However, as the back-end stages of the TBR GPU operate on a per-tile basis, all framebuffer data, including color, depth, and stencil data, is loaded and remains resident in the on-chip tile memory until all primitives overlapping the tile are completely processed, thus all fragment processing operations, including the fragment shader and the fixed-function per-fragment operations, can read and write these data without ever going off-chip.


-|IMR|TBR
---|---|--- 
Advantages|<ul><li>In primitive assembly, GPU uses little external memory bandwidth storing and retrieving intermediate geometry</li></ul>|<ul><li>On-chip memory access reduces bandwidth for mobile platform in fragment processing
Disadvantages|<ul><li>Require large memory and bandwidth in the fragment stage</li></ul>|<ul><li>Overlap primitive with tile needs additional storage</li><li>Large numbers of primitives cause latency since primitive binning requirement.</li></ul>|

## GPU Pipeline Basics
### Graphics Pipeline
![](post_img/gpu_foundations/graphics_pipeline.png)
This is the basic graphic pipeline in the GPU.

When the shader program is executed, the number of vertex shaders always not equal to the number of pixel shaders. What if there is a fixed number of vertex shader and pixel shader processor designed by the GPU Architecture? This will cause bad performance of the GPU balance. So The GPU Graphics hardware improves a new method to fixed this problem. The Unified Shader, means that there is a sheduler to assign which unified shader should be used as vertex shader or pixel shader.

![](post_img/gpu_foundations/shedule_shader.png)

![](post_img/gpu_foundations/rtx_gpu_architechure.png)
*GeForce RTX 40 Series GPUs*

### Compute Pipeline
There are only a few steps in the compute pipeline, with data flowing from input to output through the programmable compute shader stage.

So the Compute Pipeline just has one stage: compute shader stage.

## GPU Memory
### Memory Types in GPU
- Global Memory: The main memory accessible by all threads. It’s large but has high latency, making access optimization essential.
- Registers: Each thread has its own set of registers, which offer extremely low-latency access. However, registers are limited, and excessive use can lead to “register spilling” where data overflows into slower global memory.
- Local Memory: Private to each thread and has lower latency, used for storing local variables. However, excessive usage can reduce efficiency.
- Shared Memory: Shared among threads within the same block (in CUDA), allowing rapid access for data sharing and intermediate calculations.
- Constant Memory: Read-only memory that is accessible by all threads, optimized for situations where all threads read the same data.
- Texture Memory: Special memory optimized for spatial locality and used primarily in graphics applications for data that requires interpolation or 2D spatial access.

Here is another answer about gpu memory info from Quora:
Short answer: Local memory is not real memory but an abstraction of global memory in case of NVIDIA GPUs and is used typically for register spills or when the SMs runs out of resources. They are cached and addressing is usually resolved by compiler. (read Page 133 in https://docs.nvidia.com/cuda/pdf/CUDA_C_Programming_Guide.pdf v10.2)

Long Answer:
GPUs offer different types of memories:
1. Register file - very fast memory but limited (typically 1-4 cycles). Private to threads. Can be shared with different threads in a warp using shuffle instruction
2. Shared Memory - fast scratchpad memory (few KBs and is in 1–4 cycles). Private to thread block. In modern GPUs (like Volta) this memory is shared with L1 cache.
3. Constant Memory - non-mutable memory (no writes allowed and fixed-size). Private to the kernel or program
4. Texture Memory - non-mutable memory (no writes allowed and fixed-size). Private to the kernel or program
5. L2 Cache - it is a cache shared by all threads across all SMs (200s cycles).
6. Global Memory (and local memory resides here) - entire GPU memory or also called video memory (in GBs). Private to program or kernel. The difference between local memory and global memory is that local memory is private for the thread while the global memory is private for the kernel.

What Memory to choose between shared/register/local?
1. Use Shared memory whenever possible. If there is reuse in the same thread or across threads in a block, shared memory is the right place to the data.
2. Of course, registers are fastest and should be used always but registers memory is limited and can easily get exhausted. When this happens, register spills occur. The spilled registers go to local memory.
3. Local memory is used to hold automatic variables or arrays that require large structures or arrays whose index is not constant quantities. The compiler makes use of the local memory when it finds our there is not enough register space to hold variables. Now the local memory region may be cached in the memory hierarchy (no guarantee) but definitely exists in the global memory region. Thus the performance of local memory may not great. Specifically, for this reason, it is advised not to have register spillovers in your GPU code.

## References
https://www.bilibili.com/video/BV1u3411M72A/?spm_id_from=333.788&vd_source=0dd2cff1ee7ab1541a302289fb2e75b6 [1]

https://ec.europa.eu/programmes/erasmus-plus/project-result-content/52dfac24-28e9-4379-8f28-f8ed05e225e0/lec03_gpu_architectures.pdf [2]

https://electronics.stackexchange.com/questions/529562/difference-between-clock-cycle-machine-cycle-and-instruction-cycle-of-the-cpu [3]

https://www.amd.com/system/files/documents/rdna-whitepaper.pdf [4]

https://docs.nvidia.com/gameworks/content/developertools/desktop/analysis/report/cudaexperiments/kernellevel/achievedoccupancy.htm [5]

https://flashypixels.wordpress.com/2018/11/10/intro-to-gpu-scalarization-part-1/ [6]

https://www.techplayon.com/memory-management-in-gpu/ [7]

