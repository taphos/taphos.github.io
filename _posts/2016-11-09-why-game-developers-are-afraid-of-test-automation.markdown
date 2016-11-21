---
layout: post
title:  "Why Game Developers are afraid of Test Automation?"
date:   2016-11-21
image: /assets/article_images/2016-11/gameboy-test.jpg
---

Most video games don't enjoy long-term technical support. After game is released it receives a few bugfix patches at best. Developers don't deal with mistakes and bad decisions made, as new projects are often started from scratch. Recently the situation has begun to change, but are developers ready for it?

With increasing popularity of Free-to-Play games, the rules have changed. F2P games are actually living services similar to an online banking service. Both require constant updates to meet customer demands, both suffer from similar problems: legacy, over-engineering, unmanageable code and yes flood of bugs coming with every update. I have seen F2P projects collapse purely due to technical issues introduced in later game updates. Automated testing is one of the techniques widely used to solve these kind of problems. Unfortunately the topic of test automation is not popular among game developers. Though [some](http://yetanothergameprogrammingblog.blogspot.com.ee/2010/06/aaa-automated-testing.html) top game [developers](http://blog.agilegamedevelopment.com/) rise the issue, they mostly stay unheard.

Our team had previous experience with automating tests of web applications, but it seemed that game project was a bit different. We had 3 main organizational questions to solve before starting the work:

* Which tools will we use?
* Which parts of the project testing process should be automated?
* Who will be writing tests?

 We experimented with different variants of test implementations and organization, in this article I will explain what worked well and what didn't.

## Which tools to use?

Unlike, for example, web development gamedev lacks good ready-to-use testing tools. You are totally out of luck when building an inhouse engine. Though [Unity3D](https://bitbucket.org/Unity-Technologies/unitytesttools) and [UE4](https://docs.unrealengine.com/latest/INT/Programming/Automation) recently introduced their own testing systems, I haven't seen any projects successfully using those.

The provided [Unity Test Tools](https://bitbucket.org/Unity-Technologies/unitytesttools) didn't fit our needs for several reasons. Mainly it didn't allow us to drive long chains of UI interactions in a way similar to how a user would, so we decided to develop test automation framework ourselves. But which approach should we take?

Probably the first mistake is often made is to try recording the test actions performed by manual QA engineer and replay them during the automated run. Idea seems simple and effective, it is quite straight forward in implementation and it gives the opportunity to reuse the knowledge and experience of manual QA specialists you already have. I knew teams who started with this approach and once I was a part of one. My experience shows that recorded tests can't survive the system changes. Usually recording software produces an unmanageable log which needs to be re-recorded on every update, thus making automated testing literally useless. Our decision was to write test code by hand, all we needed now is to choose the best programming language suitable for the task. 
    
While working on one of my previous projects made on JavaEE we tried to implement tests using Ruby. This decision was made because at a time Ruby had a friendly WebDriver integration API and we were excited about trying out something new and lightweight. This didnt work well as we had to basically duplicate fundamental system parts like database mapping, network communication protocols, object models etc. This time we decided to reuse tools and programming languages between automation framework and game itself.
  
So, our final decision was to make a DIY automation framework running in the same environment as our production code, which would allow to craft tests by hand using same tools and languages. For us those were:
 
* Unity3D
* C# (old .Net 2.0 API)
* Custom UI solution
* Platforms: Android, iOS, Windows Desktop and Phone
* Inputs: touch screen, gamepad, accelerometer
* Heavy network communication

Honestly, it was a hardest nut to crack I ever worked with.

## Which parts should be automated?

We weighted popular approaches like [System](https://en.wikipedia.org/wiki/System_testing) level and [integration](https://en.wikipedia.org/wiki/Integration_testing) testing. They seemed good for manual QA and a useful addition for automation, though automated tests had to be different. Any kind of automation had to be executable without any environmanet setup, this would allow us to run tests more often, thus bringing more profit. We took the idea of Unit testing and extended it to UI and network protocols, we separated our application into logical layers to test them separatelly. Each test had to automatically setup a micro environment for the required layer before the action sequence execution. 

Below is a simplified diagram of the game architecture. Player Input and Output is done using game UI and scene objects (avatars, NPCs etc). Information is passed to controllers which process the data, probably give some feedback and use logical components to process game logic, calculate scores, give achievements and so on. Some parts of the logic may be processed on the server side.

![Simplified game architecture](/assets/article_images/2016-11/diagram1.png){:width="300px"}

UI tests isolate the layer of user interface and controller by replacing logical components with [mocks](https://en.wikipedia.org/wiki/Mock_object).

![User layer testing](/assets/article_images/2016-11/diagram2.png){:width="500px"}

Unit tests are doing the same, but instead of using user interface they use logical components program interface. This should be done both front-end and server components.

![Component layer testing](/assets/article_images/2016-11/diagram3.png){:width="400px"}

This layer isolation simplifies the test creation and execution. It would be much harder to setup and maintain system level test automation as it would require separate deployment of client, server, databases etc. We would get alot of false fails due to sudden network connection failures or deployment synchronization.

To replace logical  components with Mock objects we implemented a small lightweight [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) framework. We could not use any of the frameworks already available on the market, like [NInject](http://www.ninject.org/) because of outdated .Net API used by Unity3D. Our own implementation took about 150 lines of code and we didn't have to deal with complex configurations.

We thought about using [NSubstitute](http://nsubstitute.github.io/) for mocking, but we ended up just using plain OOP approach extending original classes and overriding methods. This works well for us so we didnt introduce any complex mocking solution.

For driving UI interface we had no choice but to make a custom driver as we are using custom in-house UI (actually several generations of legacy in-house UI solutions, all non-compatible but interacting with each other, true horror story). Implementation took about 400 lines of code.

## Who is writing tests?

Again we tried to organize test implementation in all sorts of ways. We tried to make test code to be as simple as possible, it had to look like a readable human language, same approach as many web automation frameworks take. 

We thought that eventually code will be simple enough for QA engineers to write tests by themselves. QA team had no problems reading test code and even writing some simple actions like press a button and validate a screen state, but when it came to mocking and test environment setup it became clear that they require some more solid programming skills.
 
Then we hired an automation engineer to help QA. It had to be a programmer with good understanding of QA team needs and knowledge about how system works. It came out that without taking part in building the system it was hard to get all the details and issues system had. Without this knowledge automation quality was mediocre. 

Finally we decided that it would be better to for automation engineer to join the programmers team. The responsibility for test implementation had to be fully in the developers hands, every one from the team had to write and support automated tests. Fortunately we had everything ready for this, we had a single code base simplifying refactoring and in-house tools which are easy to maintain. 

## Test execution

We solved most important issues for test implementation, now tests had to be executed sometimes to make any kind of benefit.

### Jenkins

Executing full set of tests before every build as a part of continuous integration process. This came out to be the most effective, as most bugs found by failing tests are revealed at this point.

### Special key execution

All builds of our game whether development or production configuration contain a set of tests ready to be executed. If application is executed with a special key, it starts the test sequence instead of the game. This allows us to validate any build even if it was downloaded from the appstore.

### Cthulhu

We setup a separate machine which would periodically download builds for different platforms and execute them on the connected devices. It got its name from its looks, as it really look like a squid with all the devices connected.

![Cthulhu testing stand](/assets/article_images/2016-11/cthulhu.png){:width="500px"}

Unfortunately this setup was very hard to maintain. As new build platforms were added it became clear that there is no simple way to install development application to Android, iOS and WP from the single system. Constant heavy load caused device batteries wear, and eventually some devices could not complete the test cycle on single charge.
    
## Conclusion
      
Test automation becomes a must-have part of game development process. Developers should not be afraid of it and I am happy to see many teams to adapt the technique. In this article I wanted to share our success story of adaptation. Story was not straight-forward, we made a few turns and met some dead ends on the way. So in the end our way is:

* Fully DIY implementation of the automation framework proved to be flexible, easier to configure and maintain comparing to ready-to-go solutions (about 600 lines of code)
* Unit-test inspired approach of logic layer separation and component mocking
* Test automation development as a part of the product code by the same team of developers.

We open-sourced code of our automation solution, so other projects could benefit from our findings easier:

[Github](https://github.com/taphos/unity-uitest), [Asset store](https://www.assetstore.unity3d.com/en/#!/content/72693) - Unity UI Test Automation Framework. Contains UI test driver, dependency injection, mocking solutions and examples.

[Github](https://github.com/creative-mobile/cthulhu) - Cthulhu, test execution server.
