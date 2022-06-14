+++
title = "CPU and GPU profiling with Rust and NVIDIA"
date = 2022-06-12T00:00:00+00:00
description = "The NVIDIA Tools Exension SDK (NVTX) has a lot to offer for GPU and CPU profiling. This blog shows how to leverage these tools with Rust, and how we can make sense of the results."

[taxonomies]
categories = ["Rust"]
tags = ["rust", "graphics"]
[extra]
toc = true
+++

Not too long ago, I had to perform CPU and GPU profiling in support of rendering a thesis[^thesis]. After realizing the ecosystem of tools in Rust was somewhat immature, I had to create my own out of necessity. This post will show how Rustaceans can use my library and the NVIDIA Tools Extension SDK (NVTX) to perform trivial GPU and CPU profiling.

# Introduction to NVTX
NVTX is originally a C-based API which provides tracer annotations to CPU and GPU profiling. Through FFI and zero-cost abstractions, I bound the same functionality without additional overhead. Through NVTX and tracer annotations, we can identify key issues such as hardware starvation, insufficient parallelization, expensive algorithms, and more ([#show-me](#show-me)).

NVIDIA pedals some fantastic software for profiling, such as NSight Systems[^nsight_systems]. NSight Systems is a feature-ruch CLI and GUI profiler which will execute a given program and sample several measurements you subscribe to, waiting for termination. For example, I can tell NVIDIA Nsight Systems to run a binary which renders 50 frames and quits. Nsight systems will launch the binary on my behalf, inject metric sampling, wait for completion, and deliver a report file. In that order.

<div align="center">
{{ img(src="NSight_Systems_Example.png" alt="NSight Systems" w=838 h=600) }}

An NVIDIA NSight Systems Report, viewed with NSight Systems
</div>

These reports can be better understood with tracer annotations from my NVTX library crate on crates.io[^crate], which NSight Systems natively supports. My library currently offers macros to name your threads, add markers, and annotate timespans with range markers. The following image presents an example report, viewed through NSight Systems, with the NVTX layer in view. Particularly, the report has an annotated range "*Benchmark Test*" and 2 marker events named "*Step 1*" and "*Step 2*".

<div align="center">
{{ img(src="NVTX_Example.png" alt="NVTX Annoations" w=901 h=542) }}

The "Benchmark Test" range and two "Step" event markers annoted by NVTX[^crate].
</div>

# Code
NVTX makes it trivial to make sense of your reports temporally, and I will provide examples in the following subsections. Currently I have supported three functions of NVTX in the library: markers, ranges, and thread naming.

## Markers
Firstly, I will show markers. Markers are used to tag instantaneous events during the execution of an application. For example, dropping a marker can be used to annotate a step in an algorithm.

Markers are injected with the `mark!` macro, which is behaviorally equivalent to the `println!` macro.
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
Thread ranges are similar to markers except they annotate a span of time, rather than an instant of time. For example, a thread range can track the total time of an algorithm or process. These NVTX tracers are visible through the events view and the timeline view on NSight Systems.

Ranges are pushed with the `range_push!` macro, which is behaviorally equivalent to the `println!` macro. Ranges are popped with the `range_pop!` macro. Thread ranges are also safe to push and pop over thread boundaries, hence 'thread' in the name.

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
Lastly, threads can be named with the `name_thread!` macro, which is behaviorally equivalent to the `println!` macro. This macros aliases the calling OS thread with a friendly name. Currently this macro works cross-platform and has been tested on MacOS, Windows, Linux, and Android.

<div align="center">
{{ img(src="Thread_Example.png" alt="NVTX Thread Naming" w=1154 h=466) }}

Three named threads.
</div>

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

# Show me

To demonstrate the value of NSight Systems, NVTX, and Rust, I will present a typical case: measuring the benefits of GPU architecture. What can using a specific GPU feature provide? For an example, I'll show you a uniquely difficult problem and how we can measure the impact of GPU storage buffers with SVG rendering.

## Example Problem
SVG files are formatted in a unique way: implicltly. Varyings must be provided, such as scale, in order to calculate the image. You may imagine an image as a composition of dimensionless shapes and math equations. A renderer must provide the camera zoom and offset to calculate what the image would look like.

This requires a certain amount of work to be done, in order to render the image. For static SVG images, we could store that 'work' into a GPU storage buffer, and read it to inexpensively redraw future frames. So that is exactly what I am going to measure: How useful are GPU storage buffers in optimizing static SVG rendering?

## Sample images
The following SVG images will be used for benchmarking, chosen as an organic range with varying complexity.

<div align="center">
{{ img(src="all_images.svg" alt="Sample Images" w=1969 h=300) }}
</div>

## Results
As promised, [I wrote a naive GPU-based SVG renderer](https://github.com/kurbos/svg-tessellation-renderer) I am calling "*Render-Kit*". My renderer operates in three steps. Firstly, it tessellates the SVG primitives (curves, lines, paths) into discrete triangles with a library called [*Lyon*](https://github.com/nical/lyon). Ater, I store the the geometry into GPU storage buffers. **These first two steps are not graphed, we assume they take no time**. Once these pre-computation steps are complete, I benchmark the first 50 frames of the [sample images](#sample-images), using the pre-computed buffers to inexpensively redraw frames. The results are plotted below.

<div align="center">
{{ img(src="all_renderkit.svg" alt="My Results" w=576 h=432) }}
</div>

## Results
Did you notice how the first frame is much longer 

# Features in progress
The entire NVTX API can be used to measure CPU code blocks, track lifetime of CPU resources (e.g., malloc), log critical events[^nvtx_def], and more. While I have ported a fraction of the SDK, there are some features outstanding. Primarily, I'd love to port [filtering tracers by domains](https://nvidia.github.io/NVTX/doxygen/index.html#DOMAINS) and [resource naming](https://nvidia.github.io/NVTX/doxygen/index.html#RESOURCE_NAMING). Feel free to contribute or check out the [issue board](https://github.com/simbleau/nvtx/issues).

*Farvel! Until next time.*

---
<!-- Note: There must be a blank line between every two lines of the footnote difinition.  -->
[^thesis]: [Understanding Hardware-Accelerated 2D Vector Graphics](http://dx.doi.org/10.13140/RG.2.2.25593.54887)

[^nsight_systems]: [NVIDIA NSight Systems](https://developer.nvidia.com/nsight-systems)

[^crate]: [nvtx, on crates.io](https://crates.io/crates/nvtx)

[^examples]: [Examples using the nvtx crate](https://github.com/simbleau/nvtx/tree/main/examples)

[^nvtx_def]: [The NVIDIA Tools Extension Library (NVTX)](https://docs.nvidia.com/nsight-visual-studio-edition/2020.1/nvtx/index.html#nvidia-tools-extension-library-nvtx)