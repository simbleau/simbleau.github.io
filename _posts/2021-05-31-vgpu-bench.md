---
layout: post
title:  "vgpu-bench: A new 2D performance analysis"
categories: vgpu
---
# Introduction
I am starting research in the field of hardware-accelerated vector graphics rendering. Focusing particularly on GPU analysis, I will be documenting findings, alongside potential conferences and publications. Much of the research I conduct will be worked on publicly and as open source under my repo [vgpu-bench](https://github.com/simbleau/vgpu-bench).

For those unaware, vector graphics compete directly with traditional raster graphics that are ubiquitous today. Vector graphics promise many benefits such as smaller file size, malleable paths, lossless graphic fidelity, and abstract scaling. However, rendering vector graphics are considerably more GPU-hostile compared to raster types and still considered unfeasible for real-time interaction.

Rendering vector graphic types at interactive rates is a very hard problem to solve. Given that vector graphics are stored as point and curve data (rather than accessible rows of color data), there is additional overhead to path-finding from this implicit form. It is difficult to achieve an interactive refresh rate such as 60 Hz, especially with visibly complex and dynamic scenes.

# Current Approaches
Many varying approaches have been surfacing in recent years to solve this problem. Examples include [discrete triangle tessellation](https://github.com/nical/lyon), [random access vector graphics](http://hhoppe.com/proj/ravg/), [a massively parallel pipeline](http://w3.impa.br/~diego/projects/GanEtAl14/), [novel scan-line algorithms](http://kunzhou.net/zjugaps/pathrendering/), and [GPU architecture leveraging](https://raphlinus.github.io/rust/graphics/gpu/2020/06/12/sort-middle.html).

A great brief on current vector graphic research has been curated previously [here](https://2d.graphics/book/gpu_rendering.html) by [Dr. Raph Levien](https://levien.com) on his website [2d.graphics](https://2d.graphics).

# Response - Moving forward
Given the surrounding literature and recent techniques to solving this problem, it is an appropriate step to respond with a performance GPU analysis. There is a felt lack of comparison between techniques and libraries which survey the current rendering capability, and for good reason. This is also *very* hard to analyze, but it is necessary to push the literature forward. The benefit of a contextual analysis would hopefully lead to further optimization in the field of vector graphics. 

The difficulty exists because 2D rendering has more cultural application than 3D. Computer graphics in 3D typically have graphically-intense use-cases like games. On the contrary, in 2D, we have many more motivations for optimization. We may optimize for latency, power consumption (mobile!), contention of scarce resources(CPU/GPU bandwidth), or consider balancing several of these factors. Some applications may want vector graphics to leverage an aptly-tuned pipeline capable of scaling for a rich scene graph. Examples may include scientific visualization or computer aided-design. However, some others may not require intricate plumbing, such as a basic user-interface.

This makes a 2D hardware-accelerated performance analysis a stark contrast to traditional single-threaded benching, typically done using elapsed time or discrete measurements (e.g. frames-per-second). In such cases, those metrics are usually enough, and Big-O is a decent proxy for performance. Hence, GPU analysis is more contextual, requiring more trials, instrumentation, and hardware to yield residual data for further interpolation.

# Scope

Because of an eclectic variance of optimization goals, I need to narrow the scope and nail-down some concrete parameters for benching.

**This research will particularly focus on interactivity and scalability.** To be clear, this research will be insightful for individuals benefitting from interactivity, visual complexity, and dynamism. Examples include: interactive simulations, scientific visualization, and games. Other use-cases may not find this research to be as relevent.

I present, in no order of priority, goals which accentuate our scope:
 * [Minimizing CPU load](#minimizing-cpu-load)
 * [Keeping GPU latency consistently under 16 ms](#keeping-gpu-latency-consistently-under-16-ms)
 * [Optimization for scene complexity and dynamism](#optimization-for-scene-complexity-and-dynamism)

The motivation for each of these scopes is explained below.

## Minimizing CPU Load

Interactive applications will attempt to avoid contention between CPU and GPU resources to encourage stability. By forcing a heavier GPU load, we mitigate the risk of throttling CPU-bound application logic. For example, when a game is delayed on updates, this may lead to packet loss and/or latency.

## Keeping GPU latency consistently under 16 ms

The GPU needs to rasterize frames at interactive time standards. 16 milliseconds is the number this research will primarily use for comparison, but it is a strange number in need of justification.

Television and livestreaming are regulated examples when mentioning minimum frame-rate standards. Established for over a decade, [PAL, NTSC, and SECAM](https://www.sony.com/electronics/support/articles/00006681) enforce frame rates between 24-30 frames per second. Almost every country currently adopts these standards. 

![Television frame rates by country](/assets/video_framerate.png)

However, these are antiquated standards being replaced as technology and bandwidth speed progress. Recent technology has gravitated towards higher frame-rates for a smoother experience. For example, a refresh rate of 60 Hz is supported for HDR in [Apple 4k Televisions](https://www.apple.com/apple-tv-4k/specs/). Modern monitors typically clok higher, capable of refresh rates above 144Hz.

Reducing frame-time is an effort to minimize the perceived notion of input latency, wherein user input appears delayed due to the amount of frames in a second being rendered. Lower frame-rates attribute longer fractions-of-time when input feels invalidated due to lack of visual feedback.

![Frames per second](/assets/fps_illustration.png)

60 Hz is a common, but subjective, industry-standard based on user feedback. 16 ms is the amount of time a frame should be on screen before being replaced in order to render motion media at roughly 60 frames-per-second (1÷60 ≈ 0.167). Hence, a goal is to analyze the precarious line where graphic fidelity can be maximized before this frame-time is compromised.

## Optimization for scene complexity and dynamism

Interactivity requires input handling. Computer graphics applications which handle input may have a rich, complex scene graph. In terms of computer-aided-design software, audio mixing software, or scientific simulations, one may have hundreds of visual elements at any time being rendered.

Therefore, my research will analyze the scene-scaling capability and dynamism of rendering pipelines. In this context, dynamism can refer to animation as well as scene movement.

# Testing

Given the scope of optimization, my research will explore hypotheses to test and validate or falsify.

Namely,
 * Where do current vector graphics libraries maximize graphic richness without sacrificing frame-time across a range of hardware, dynamism, and scene complexity?
 * To what effect is the GPU architecture leveraged in each library?
 * What are the benefits of tessellation?
 * What are the consequences of a pre-computation model?
 * Can compute-centric approaches improve performance?
 * Can low-level GPU features (subgroups, bindings) improve the imaging model?

This research hopes to answer most of these questions. The next step would be to collect a set of desired libraries for which to test these hypotheses, and determine the methodology for testing.

Thank you for reading,\
~SCI