---
layout: post
title:  "Horrors of mobile graphics"
date:   2017-09-07
image: /assets/article_images/2017-09/fightclub.jpg
---

Recently [lead mobile](https://venturebeat.com/2016/02/16/mobile-devices-will-be-more-powerful-than-playstation-4-xbox-one-in-2017-arm-forecasts) [hardware vendors](http://www.eurogamer.net/articles/digitalfoundry-2014-nvidia-and-unreal-engine-4-reveal-the-future-of-mobile-graphics) started to claim that mobile graphics reached the level of modern gaming consoles. Lets find out whether claims are true and if so, why don't we see much mobile games on the market which could prove the point.

Vendors base their claims on a single spec parameter: the number of floating point operations that can be performed by a graphical processing unit in a single second. Lets compare the specs of different devices both mobile and consoles.

|-----------------------|-----------------|
| PS4                   | 1843 gflops     |
| Xbox One              | 1310 gflops     |
| **Samsung Galaxy S8** | **567 gflops**  |
| **iPhone X**          | **350 gflops**  |
| Xbox 360              | 240 gflops      |
| PS3                   | 192 gflops      |
| **iPhone 6S**         | **120 gflops**  |


From the first glance it seems that claims are relevant. Modern mobile devices exceed the capabilities of previous console generations and seems will be reaching current generation very soon taking into account the speed of growth. The question is whether speed of GPU computation relevant for graphics quality measurement? Is there something marketers not telling us about?

Behold the everyday horrors of graphics programmer which might be the reason for graphic level limitation. I will also give you some solutions and ways to deal with these problems along the way.

## Horror №1: Programming Interface

Widely used APIs like OpenGL and DirectX were developed in the 1990s. Back then graphical hardware was quite different and unfortunately API wasn't made flexible enough to expose all the capabilities of modern hardware. Some current hardware features are being squeezed into the outdated interface, some being exposed by ugly API extensions and some just stay unreachable for developers.
 
[![xkcd standards](/assets/article_images/2017-09/standards.png)](https://xkcd.com/927)

### Bye-bye multi-threading

When drawing a picture on the screen CPU communicates with GPU by issuing tasks. These tasks are named draw calls. One rendered image may consist of tens to thousands of draw calls. Each call requires some CPU resources to prepare textures, models and other required data. All modern CPUs are multicore and it would be efficient to prepare draw calls in multiple threads in parallel to save time. Unfortunately outdated API has a single global state which makes it impossible to send GPU tasks in parallel. This problem is not unique to mobile devices, if you ever checked the CPU load on your PC while playing your favorite video game you might notice that one CPU core is loaded to almost 100% while all other cores are resting. This is a bottle neck created purely by the software interface.
  
### Flood of shader compilers

Code which is executed by GPU is compiled by the driver at the application runtime. This means that all shader code must be shipped as a source. Every GPU vendor must implement their own compiler with unnecessary syntax validation. The problem is that there are so many different shader compilers out there that it is almost impossible to write code which will be compiled without errors by all the versions of drivers of all chipsets possible. Compiler bugs are common, especially with syntax parsing.

### Poor memory management

Historically CPU memory was separated from GPU memory as they have different performance requirements. This is still true for PC and consoles. When preparing a draw call CPU must copy all the required data (textures, meshes etc.) to the graphic memory. On the other hand separate memory chips would be an overkill for mobile devices as it would require more physical space, so both processors usually operate the same memory on mobiles. Despite this, interfaces force CPU to make unnecessary data copies anyway, which creates an additional processing time overhead.

### No programmable Blending

Old GPUs had framebuffer color blending implemented on the hardware side, so blending variety was limited by a set of fixed functions. Currently many graphic chips use programmable blending which theoretically allows to use any kind of functions, but as you might have guessed, it is inaccessible. Some vendors, like PowerVR, expose programmable blending via [special extension](https://www.khronos.org/registry/OpenGL/extensions/EXT/EXT_shader_framebuffer_fetch.txt) but as it is not widely spread and remains mostly unused.

### Will API be fixed?

Recently a new generation of GPU programming interfaces arrived. [Vulkan](https://www.khronos.org/vulkan) is developed by Khronos group as a replacement for outdated OpenGL. Unfortunately war of standards continues as Apple and Microsoft developed their own proprietary APIs called [Metal](https://developer.apple.com/metal) and DirectX12. New generation of interfaces provide lower level access to GPU hardware and allow to solve most of the problems mentioned above. It will probably take some time for new standards to be widely adopted. I think it will be no less then 10 years for mobile graphics programmers to be able to exclusively rely on new APIs.
    
## Horror №2: Heat

Comparing with PC and consoles mobile GPU and CPU are usually packed into a single chip. Additionally comparing to large efficient air (sometimes liquid :O) coolers mobiles rely on natural cooling. The reason for this are consumers who want their devices as thin and lightweight as possible.

![Nvidia GTX 1080 and Apple A7](/assets/article_images/2017-09/gpu-reality.png)

While device physical parts become tinier and packed together along with consuming more power and radiating much heat, physical limit for natural cooling is getting reached. In fact it takes about 10 minutes working full power in room temperature environment for modern mobile device to overheat. Fortunatelly devices are equipped with burning protection mechanism. When certain temperature is reached device starts throttling processing unit frequency to cool down. While throttling saves device from burning, it is devastating for graphical applications and user experience.

I made a couple of performance tests with devices of Samsung Galaxy series. Both device CPU and GPU were loaded for nearly 100% for duration of 10 minutes. After that I measured rendering performance of GPU vertex and pixel programs.   

| Model      | Vertices (20ms) | Pixels (20ms) |
|:-----------|----------------:|--------------:|
| Galaxy S4  | 500k            | 7 screens     |
| Galaxy S5  | 937k            | 20 screens    |
| Galaxy S6  | 260k            | 10 screens    |
| Galaxy S7  | 272k            | 15 screens    |

S4 and S5 didn't overheat at all, keeping the performance on the same level during whole test. S5 could render twice as much vertices and pixels as its chip has double the power. S6 and S7 could not reach the rendering speed of older devices as they had anti-burning mechanisms activated quickly. This means that Galaxy S5 was the last balanced device and all succeeding ones will not get better performance, but just overheat faster. Same is happening with all high-end devices produced in 2016 and later.

The question is, whether vendors aware of this problem? I think yes, for sure. Why do they keep installing more powerfull processors? I think the problem is marketing, consumers tend to trust phone specifications when choosing devices. They don't realize that specs are just numbers and such processing power can not be achieved in reality.
   
The irony is that in non-capitalistic market vendors would use common sense keeping device heat in balance helping app developers to use full potential of the hardware. Are there any Cuban mobile phones yet? :)
    
### How to deal with overheating?

With last project I have been working on we started by measuring the length of the game sessions. It came out that average game session was about 15 minutes long. We decided that keeping 30 frames per second for 15 minutes on all devices would be good enough for most users. After some trial and error we stopped on using about 60% of high-end device resources with best graphics quality settings. This solved our problem, but limited the image quality for the consumer. If your application requires longer sessions, you have no simple solution but to make image quality even lower.

TODO: iPhone 7,8,X same performance. X - less power
    
## Horror №3: Tools

Tools for profiling and analysing mobile GPUs are not standardized. Basically every vendor makes its own tools from scratch. Hardware developers are known to be worst software developers, so most tools are almost unusable. GPU profilers tend to crash, fail to show required information or connect to devices. On the other hand device producers tend to close all possible holes for hardware analysis, probably for security reasons. For example all iOS devices contain PowerVR graphic chips but no official PowerVR tools can be used with those. Fortunately Apple provides its own proprietary tools, many Android devices just have no way for low-level hardware analysis.
  
Will tools become better in the future? Probably not. As mobile device market is expanding, number of hardware vendors is also growing with no hope for standardisation. Large app development companies make their own tools, small developers usually dont have enough resources for that. Though some shortcuts can be used when making analysis tools, check out [this article](http://aras-p.info/blog/2017/01/23/Chrome-Tracing-as-Profiler-Frontend) about using Google Chrome Tracing as a profiler frontend.   

## Horror №4: Diversity

Currently there are hundreds of different mobile GPU models on the market. Each model has several versions of drivers. Probably thousands of different CPU+GPU combinations. It is impossible to test all of them event using automated QA services (P.S. Check out my [article on test automation with Unity](/2016/11/21/why-game-developers-are-afraid-of-test-automation.html)). Additionally there are some inadequate combinations on the marked with powerful GPUs and CPU too weak to support the power or vice-verse, usually produced by no-name chinese companies.

In the PC world developers usually allow users to choose exactly the level of graphics they want to see exposing very detailed graphics settings. This solution does not work good for mobile game market, as mobile players are not hardcore enough to understand what all these FXAA, SSAO and HBDO mean.

Good solution for mobile app developers is to make 3-4 predefined levels of graphics, using LODs and several versions of post processing effects. Graphic quality level should be chosen automatically. We use device model for iOS devices and GPU model for Android. Check out [this GDC talk](http://www.gdcvault.com/play/1023299/Delivering-Console-Car-Visuals-on) about mobile visuals optimisation.
      
## Conclusion

Well lets answer the question, whether claims about mobile graphics quality reaching the level of consoles are true? If you read the whole article you might have guessed that answer is NO. Those claims are obscuring the most important part the truth, so I would consider them a lie. GPU vendors are lying as a part of their marketing campaigns. Unfortunately they are forced to do so by the capitalistic competition.
 
Modern mobile devices are not made for gaming or any other graphically intense applications. First priority for the consumer are everyday tasks like reading emails and browsing facebook which don't require much processing power. Device producers will keep installing more and more powerful hardware just to keep spec numbers big. Probably in the future we will see a truly successful gaming mobile device which will change the market, we will see.  
