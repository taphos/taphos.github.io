---
layout: post
title:  "Four horrors of mobile graphics"
date:   2017-09-01
image: /assets/article_images/2017-01/portrait-hq.jpg
---

Recently lead mobile hardware vendors started to claim that mobile graphics reached the level of modern gaming consoles. Lets find out whether claims are true and if so, why don't we see much mobile games on the market which could prove the point.

Lets see what these claims are based on. In the table below you can find a GPU hardware specs of different devices both mobile and consoles.

|-----------------------|-----------------|
| PS4                   | 1843 gflops     |
| Xbox One              | 1310 gflops     |
| **iPhone 7**          | **729 gflops**  |
| **Samsung Galaxy S8** | **519 gflops**  |
| Xbox 360              | 240 gflops      |
| PS3                   | 192 gflops      |
| **iPhone 6S**         | **120 gflops**  |


Gflop is a number of billions of floating point operations that can be performed by a graphical processing unit in a single second. Based on these numbers it seems that vendor claims are relevant. Modern mobile devices exceed the capabilities of previous console generations and seems will be reaching current generation very soon taking into account the speed of growth. The question is whether speed of GPU computation is the only characteristic determining the quality of graphics?

I am presenting you the four horrors of graphics programmer which limit the capabilities of mobile apps. I will also give you some solutions and ways to deal with the problems.

## Horror №1: Programming Interface

Widely used APIs like OpenGL and DirectX were developed in the 1990s. Back then graphical hardware was quite different and unfortunately API wasn't made flexible enough to expose all the capabilities of modern hardware. Some hardware features are being squeezed into the outdated interface, some being exposed by ugly API extensions and some just stay unreachable for developers. 

### No multi-threading

When drawing a picture on the screen CPU communicates with GPU by issuing tasks. These tasks are called draw calls. One rendered image may consist of tens to thousands of draw calls. Each draw call requires some CPU resources to prepare textures, models and other required data. All modern CPUs are multicore and it would be efficient to prepare draw calls in multiple threads in parallel to save time. Unfortunately outdated API has a single global state which makes it impossible to send GPU tasks in parallel. This problem is not unique to mobile devices, if you ever checked out the CPU load on your PC while playing your favorite game you might notice that one CPU core is loaded to almost 100% while all other cores are resting. This is a bottle neck created purely by the interface.
  
### Too many shader compilers

Code which is executed by the GPU is compiled by the driver at the application runtime. This means that all shader code must be shipped as a source. Every GPU vendor must implement their own compiler with unnecessary syntax validation. The problem is that there are so many different shader compilers out there that it is almost impossible to write code which will be compiled without errors by all the versiosdfsdfns of drivers of all chipsets possible. Compiler bugs are common, especially with syntax parsing.

### Poor memory management

Previously CPU memory was always separated from GPU memory which is still true for PC and consoles as requirements are differds
 

     
Colons can be used to align columns.

| Tables        | Are           | Cool  |
|:--------------|--------------:|------:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |