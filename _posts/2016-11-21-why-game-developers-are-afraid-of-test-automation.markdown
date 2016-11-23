---
layout: post
title:  "Why Game Developers are afraid of Test Automation?"
date:   2016-11-21
image: /assets/article_images/2016-11/gameboy-test.jpg
---

Most video games don't enjoy long-term technical support. After game is released it receives a few bugfix patches at best. Developers don't deal with mistakes and bad decisions made, as new projects are often started from scratch. Recently the situation has begun to change, but are developers ready for it?

With increasing popularity of Free-to-Play games, the rules have changed. F2P games are actually living services similar to an online banking service. Both require constant updates to meet customer demands, both suffer from similar problems: legacy, over-engineering, unmanageable code and yes flood of bugs coming with every update. I have seen F2P projects collapse purely due to technical issues introduced in later game updates. Automated testing is one of the techniques widely used to solve these kind of problems. Unfortunately the topic of test automation is not popular among game developers. Though [some](http://yetanothergameprogrammingblog.blogspot.com.ee/2010/06/aaa-automated-testing.html) [top](http://gamesfromwithin.com/stepping-through-the-looking-glass-test-driven-game-development-part-1) game [developers](http://blog.agilegamedevelopment.com/) rise the issue, they mostly stay unheard.

Fortunately our development team had some previous test automation experience with web applications. Apparently web automation is similar, but still quite different, to games. We faced three key challenges:

* Which tools will we use?
* Which parts of the project testing process should be automated?
* Who will be writing tests?

Instead of making premature decisions we started experimenting with different variants, in this article I will explain what worked well and what didn't.

## Which tools to use?

Unlike web development gamedev lacks good ready-to-use testing tools. You are totally out of luck when building an inhouse engine. Though [Unity3D](https://bitbucket.org/Unity-Technologies/unitytesttools) and [UE4](https://docs.unrealengine.com/latest/INT/Programming/Automation) recently introduced their own testing systems, I haven't seen any projects successfully using those.

The provided [Unity tools](https://bitbucket.org/Unity-Technologies/unitytesttools) didn't fit our needs for several reasons. Mainly it didn't allow us to drive long chains of UI interactions in a way similar to how a user would, so we decided to develop test automation framework ourselves, but it wasn't clear at this point which approach should we take?

Probably the first mistake often made is to record test actions performed by manual QA engineer and replay them during the automated run. Idea seems simple and effective, implementation is straightforward and it allows to reuse knowledge of current QA specialists. I know teams who started with this approach and once I was a part of one. Experience has shown that recorded tests can't survive the system change. Logs, produced by recording software, are unmanageable and require to be re-recorded on every system update, thus making automated testing literally useless. Our decision was to write test code by hand. All we needed now is to choose the appropriate programming language suitable for the task. 
    
There are two approaches to take in language choice: whether we use platform native language (like Java for Android) or we go for engine native one (C# for Unity). While working on one of my previous projects built on JavaEE we had tests made with Ruby. At a time Ruby had a friendly WebDriver integration and we were excited about trying out something new and lightweight, comparing to enterprise horror. In the long run it didn't work well as we basically had to duplicate most fundamental system parts like database mappings, network communication protocols, object models etc. This time we decided to reuse tools and programming languages between automation framework and game itself.
  
So, our final decision was to make a DIY automation framework running in the same environment as our production code, which would allow to craft tests by hand using same tools and languages. For us those were:
 
* Unity3D
* C# (ancient .Net 2.0 API)
* Apache Thrift for network communication
* Custom UI solution
* Platforms: Android, iOS, Windows Desktop and Phone
* Inputs: touch screen, gamepad, accelerometer

Honestly, it was a toughest nut to crack which I ever worked with.

## Which parts should be automated?

We weighted popular approaches like [System](https://en.wikipedia.org/wiki/System_testing) level and [integration](https://en.wikipedia.org/wiki/Integration_testing) testing, they seemed good for manual QA and a useful addition for automation, though automated tests had to be different. We wanted our tests to be executed without any environment setup, this would allow us to execute more often, thus bringing more profit. We took the idea of Unit testing and extended it to UI and network protocols, we separated our application into logical layers to test them separatelly. Each test had to automatically setup a micro environment for the required layer before the action sequence execution. 

Below is a simplified diagram of our game architecture (basically any kind of game). Player Input/Output is made using game UI and scene objects (avatars, NPCs etc). Information is passed to controllers which process the data, probably give some feedback and use logical components to process game logic, calculate scores, give achievements and so on. Also some parts of the logic may be processed on the server side.

![Simplified game architecture](/assets/article_images/2016-11/diagram1.png){:width="300px"}

UI tests isolate user interface with controllers by replacing logical components by [mock objects](https://en.wikipedia.org/wiki/Mock_object).

![User layer testing](/assets/article_images/2016-11/diagram2.png){:width="500px"}

Unit tests are doing the same, but instead of using user interface they use program interface of logical components. This should be done both on front-end and server components.

![Component layer testing](/assets/article_images/2016-11/diagram3.png){:width="400px"}

This kind of layer isolation largely simplifies the test creation and execution. It would be much harder to setup and maintain system level test automation as it would require a separate deployment of client, server, databases etc. We would get alot of false fails due to deployment desynchronization and network connection failures.

To replace logical  components with Mock objects we implemented a small lightweight [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) framework. We could not use any of the frameworks already available on the market (like [NInject](http://www.ninject.org/)) because of outdated .Net API. Our own implementation took only about 150 lines of code and we didn't have to deal with complex configurations.

We thought about using [NSubstitute](http://nsubstitute.github.io/) for mocking, but we ended up just using plain OOP approach extending original classes and overriding methods. This worked well for us so we didnt introduce any complex mocking solution.

For driving UI interface we had no choice but to make a custom driver as we are using custom in-house UI (actually several generations of legacy UI solutions, all non-compatible but interacting with each other, true horror story). Implementation took no more then 400 lines of code.

Our DIY test automation framework was ready, but it still had to be integrated into development process.

## Who is writing tests?

Again, we tried to organize test implementation in all sorts of ways. We made the test code to be as simple as possible, it had to look like a readable human language (same approach as many web automation frameworks take). We thought that eventually code will be simple enough for QA engineers to write tests by themselves. 

QA team had no problems reading test code and writing some simple actions like pressing buttons and validating screen states, but when it came to mocking and test environment setup it became clear that they require some more solid programming skills.
 
Then we hired a dedicated automation engineer to help QA. It had to be a programmer with good understanding of testing team needs and knowledge of system internal structure. It came out that without taking part in building the system it was hard to get all the details and issues system had. Without this knowledge automation quality was mediocre. 

Finally we decided that it would be better for automation engineer to join the programmers team. The responsibility for test implementation fully moved to the developers hands. Every team member had to write and support automated tests. Fortunately we had everything ready for this, we had a single code base simplifying refactoring and in-house tools which are easy to maintain. 

## Test execution

We solved most important issues for test implementation, now tests had to be executed to make any kind of benefit.

### Jenkins

We are executing full set of tests before every build as a part of continuous integration process. This came out to be the most effective, as most bugs found by tests are revealed at this point.

### Special key execution

All builds of our game whether development or production configuration contain a set of tests ready to be executed. If application is executed with a special key, it starts the test sequence instead of the game. This allows us to validate any build even if it was downloaded from the appstore.

### Cthulhu

We setup a separate machine which would periodically download builds for different platforms and execute them on the connected devices. It got its name from its looks, as it really look like a squid with all the devices connected.

![Cthulhu testing stand](/assets/article_images/2016-11/cthulhu.png){:width="500px"}

Unfortunately this setup was very hard to maintain. As new build platforms were added it became clear that there is no simple way to install development application to Android, iOS and WP from the single system. Constant heavy load caused device batteries wear, and eventually some devices could not complete the test cycle on the single charge.
    
## Conclusion
      
Test automation becomes a must-have part of game development process. Developers should not be afraid of it and I am happy to see many teams adapting the technique. In this article I wanted to share our success story of adaptation. Story was not straight-forward, we made a few turns and met some dead ends on the way. So in the end our way is:

* Fully DIY implementation of the automation framework proved to be flexible, easier to configure and maintain comparing to ready-to-go solutions (about 600 lines of code)
* Unit-test inspired approach of logic layer separation and component mocking
* Test automation development as a part of the product code by the same team of developers.

We open-sourced code of our automation solution, so other projects could benefit from our findings easier:

[Github](https://github.com/taphos/unity-uitest), [Asset store](https://www.assetstore.unity3d.com/en/#!/content/72693) - Unity UI Test Automation Framework. Contains UI test driver, dependency injection, mocking solutions and examples.

[Github](https://github.com/creative-mobile/cthulhu) - Cthulhu, test execution server.

Checkout our official blog for the story from the QA team point of view: [Squashing Bugs](http://nitronation.com/devblog-squashing-bugs).
