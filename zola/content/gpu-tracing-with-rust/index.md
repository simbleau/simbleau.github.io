+++
title = "NVIDIA GPU profiling with Rust"
date = 2022-06-12T00:00:00+00:00
description = "The NVIDIA Tools Exension SDK (NVTX) has a lot to offer for GPU and CPU profiling. This blog shows how to leverage these tools with Rust, and how we can make sense of the results."

[taxonomies]
categories = ["Rust"]
tags = ["rust", "graphics"]
[extra]
toc = true
+++

Not too long ago, I had to perform CPU and GPU profiling in support of rendering a thesis[^thesis]. When I realized the ecosystem of GPU profiling tools in Rust was somewhat immature, I had to scaffold my own tools out of necessity. This post will demonstrate how anyone can use the NVIDIA Tools Extension SDK (NVTX) from Rust to make GPU and CPU profiling trivial.

# Introduction to NVTX
NVTX is a C-based API for annotating events, code, and resources in your application. These annotations get captured by NVIDIA profiling software, such as NSight Systems[^nsight_systems]. Emitting tracer annotations during the runtime of your program can help identify key issues such as hardware starvation, insufficient parallelization, expensive algorithms, and more. Later in this post we will explore an example ([#show-me](#show-me)).

Thankfully, one of Rust's promises is its ability to interoperate with C APIs through foreign function interfacing (FFI) at identical performance[^zero_cost_abstractions]. This is called a zero-cost abstraction, and allows us to call NVTX functions with identical performance to C, in Rust.

# NSight Systems
NSight Systems is a feature-ruch CLI and GUI profiler which executes a command and samples certain opt-in measurements you subscribe to. For example, if I instruct NVIDIA Nsight Systems to run a binary, Nsight Systems will launch the binary on my behalf, collect measurements while waiting for termination, and deliver a report file. In that order.

<div align="center">
{{ img(src="NSight_Systems_Example.png" alt="NSight Systems" w=838 h=600) }}

An NVIDIA NSight Systems Report, viewed with NSight Systems
</div>

These reports can be better understood with annotations from my [nvtx crate](https://crates.io/crates/nvtx). With it, you may name your threads, add markers, and annotate timespans. Tracers emitted during program runtime appear on the NVTX layer of a report file when capturing NVTX annotations. The following image shows an example of some tracers, with nothing else captured.

<div align="center">
{{ img(src="NVTX_Example.png" alt="NVTX Annoations" w=901 h=542) }}

A thread range and two event markers annoted with [nvtx](https://crates.io/crates/nvtx).
</div>

# Code
The following sections provide some examples which demonstrate how trivial it is to augment runtime with annotations.

## Markers
Firstly, I will show markers. Markers are used to tag instantaneous events during the execution of an application. For example, dropping a marker can be used to annotate a step in an algorithm.

Markers are injected with the `mark!` macro, which accepts a name for the event. It behaves similarly to the `println!` macro with argument formatting.
```rs
use std::thread::sleep;
use std::time::Duration;

use nvtx::mark;

fn main() {
    mark!("Operation A - Begin");
    sleep(Duration::from_millis(150)); // Expensive operation here
    mark!("Operation B - Begin");
    sleep(Duration::from_millis(75)); // Expensive operation here
}
```

<div align="center">
{{ img(src="Marker_Example.png" alt="NVTX Marker" w=1410 h=129) }}

Two named markers visible through the events view on NSight Systems.
</div>

## Thread ranges
Thread ranges are similar to markers, except they annotate a span of time, rather than an instant of time. For example, a thread range can track the total time of an algorithm or process.

Ranges are pushed with the `range_push!` macro, and popped with the `range_pop!` macro. Thread ranges are also safe to push and pop over thread boundaries, hence *thread* in the name.

```rs
use std::thread::sleep;
use std::time::Duration;

use nvtx::{range_pop, range_push};

fn main() {
    range_push!("My Range");
    sleep(Duration::from_millis(100)); // Expensive operation here
    range_pop!();
}
```

<div align="center">
{{ img(src="Range_Example.png" alt="NVTX Thread Range" w=1148 h=600) }}

A range viewable from the timeline and events view in NSight Systems.
</div>

## Thread naming
Lastly, threads can be named with the `name_thread!` macro. This macros will alias the calling OS thread with a friendly name. Currently, this macro works cross-platform and has been tested on MacOS, Windows, Linux, and Android.

```rs
use std::thread::sleep;
use std::time::Duration;

use nvtx::name_thread;

fn main() {
    name_thread!("Main Thread");
    let handler2 = thread::spawn(|| {
        name_thread!("Thread 2");
        sleep(Duration::from_secs_f32(0.2));
    });
    let handler3 = thread::spawn(|| {
        name_thread!("Thread 3");
        sleep(Duration::from_secs_f32(0.3));
    });
    sleep(Duration::from_secs_f32(0.5));
    handler2.join().unwrap();
    handler3.join().unwrap();
}
```

<div align="center">
{{ img(src="Thread_Example.png" alt="NVTX Thread Naming" w=1154 h=466) }}

Three named threads.
</div>

# Show me

To demonstrate NSight Systems, NVTX, and Rust, I will present a finding I organically stumbled upon when writing a [naive SVG renderer](https://github.com/simbleau/svg-tessellation-renderer) using [Lyon](https://github.com/nical/lyon) and [wgpu](https://wgpu.rs/).

## Situation
My task was assessing if tessellation could have any usefulness in optimizing vector graphics. Vector graphic renderers solve a very complicated problem; they don't simply read rows of pixels like in raster graphics. You may imagine an SVG file as a composition of overlapping paths and lines, and a renderer must decide on varyings (scale, viewbox, etc.) to calculate the image. 

<div align="center">
{{ img(src="renderkit.svg" alt="Sample Images" w=1969 h=300) }}
</div>

Vector tessellation makes this easier. The idea is to transmute mathematical paths, curves, and lines into discrete triangles, a friendlier format for GPUs. My renderer would cache the triangle geometry in a storage buffer and re-use the work to inexpensively redraw the SVG.

I chose some sample images as an organic range with varying complexity, and proceeded to benchmark the frametimes.

<div align="center">
{{ img(src="all_images.svg" alt="Sample Images" w=1969 h=300) }}
</div>

## Results
The benchmark was simple: render 50 frames and record how long each frame took. The results are plotted below.

<div align="center">
{{ img(src="all_renderkit.svg" alt="My Results" w=576 h=432) }}
</div>

Tessellation-time was not plotted here, and no frame was rendered any differently. So to my surprise, why was the first frame taking longer? Using the power of NVTX annotations, I dropped some tracers to understand the instructions temporally. I annotated the first frame as "*Strange Behavior*".

<div align="center">
{{ img(src="NVTX_Trace.png" alt="NTVX Tracers" w=2243 h=965) }}
</div>

To my surprise, it seemed like the GPU wasn't working hard until the 2nd frame. The reasoning would be that wgpu (the graphics library I used) submitted the queue of geometry in the first frame. What we are witnessing is gpu latency, specifically, the time to transfer the geometry to the GPU storage buffers and return. This became an obvious que when able to inspect and see a timeline with ques: blocked state, PCIe bandwidth, and GPU occupancy.

This is one example of how annotations have helped me, and work will continue to help in more ways.

# Features in progress
I have only ported a fraction of the NVTX SDK, and there are some noteworthy features outstanding. NVTX can be used to measure CPU code blocks, track lifetime of CPU resources (e.g., malloc), log critical events[^nvtx_def], and more. I'd love to port [filtering tracers by domains](https://nvidia.github.io/NVTX/doxygen/index.html#DOMAINS) and [resource naming](https://nvidia.github.io/NVTX/doxygen/index.html#RESOURCE_NAMING). Feel free to contribute or check out the [issue board](https://github.com/simbleau/nvtx/issues).

*Farvel! Until next time.*

---
<!-- Note: There must be a blank line between every two lines of the footnote difinition.  -->
[^thesis]: [Understanding Hardware-Accelerated 2D Vector Graphics](http://dx.doi.org/10.13140/RG.2.2.25593.54887)

[^nsight_systems]: [NVIDIA NSight Systems](https://developer.nvidia.com/nsight-systems)

[^zero_cost_abstractions]: [Run Once, Run Everywhere](https://blog.rust-lang.org/2015/04/24/Rust-Once-Run-Everywhere.html)

[^nvtx_def]: [The NVIDIA Tools Extension Library (NVTX)](https://docs.nvidia.com/nsight-visual-studio-edition/2020.1/nvtx/index.html#nvidia-tools-extension-library-nvtx)

[^examples]: [Examples using the nvtx crate](https://github.com/simbleau/nvtx/tree/main/examples)