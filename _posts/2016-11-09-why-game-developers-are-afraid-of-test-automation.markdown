---
layout: post
title:  "Why Game Developers are afraid of Test Automation?"
date:   2016-11-09
image: /assets/article_images/2016-11/gameboy-test.jpg
---

Most video games don't enjoy long-term technical support. After game is released it receives only a couple of bugfix patches at best. Developers don't deal with mistakes and bad decisions made, as new projects are often started from scratch. Recently situation began to change, but are developers ready for it?

With increasing popularity of Free-to-Play games rules have changed. F2P games are actually a living services similar to an online banking service. Both require constant updates to meet customer demands, both suffer from similar problems: legacy code, over-engineering, unmanageable code and yes flood of bugs coming with every update. I have seen F2P projects collapse purely due to technical issues introduced in later game updates. Automated testing is one of the techniques widely used to solve these kind of problems.

Unfortunately test automation topic is not popular among game developers. Though [some](http://yetanothergameprogrammingblog.blogspot.com.ee/2010/06/aaa-automated-testing.html) top game [developers](http://blog.agilegamedevelopment.com/) rise the issue, they mostly stay unheard.

Unlike, for example, web development gamedev lacks good testing tools. You are totally out of luck when building an inhouse engine. Though [Unity3D](https://bitbucket.org/Unity-Technologies/unitytesttools) and [UE4](https://docs.unrealengine.com/latest/INT/Programming/Automation) recently introduced their own testing systems, but I havent seen any projects successfully using those.

Our game is built using Unity. The provided [Unity Test Tools](https://bitbucket.org/Unity-Technologies/unitytesttools) didn't fit our needs for several reasons. Mainly it didn't allow us to drive long chains of UI interactions in a way similar to how a user would, so we decided to develop test automation framework ourselves. We experimented with different variants of test implementations and organization, in this article I will explain what worked well and what didn't.

## Recording vs Coding

While recording a test sequence might sound easy and effective, my experience shows that recorded tests don't live longer then one update release. They don't survive refactoring and should be re-recorded again and again, which destroys the point of automation.

##  Which parts should be automated?

[System](https://en.wikipedia.org/wiki/System_testing) and [integration](https://en.wikipedia.org/wiki/Integration_testing) testing are good approaches for manual testing and a useful addition for automation, though automated tests should use a different approach, they should isolate layers of application and test them separately.

Below is a simplified diagram of the game architecture. Player Input and Output is done using game UI and scene objects (avatars, NPCs etc). Information is passed to controllers which process the data, probably give some feedback and use logical components to process game logic, calculate scores, give achievements and so on. Some parts of the logic may be processed on the server side.


![Simplified game architecture](/assets/article_images/2016-11/diagram1.png){:width="300px"}

UI tests isolate the layer of user interface and controller by replacing logical components with [mocks](https://en.wikipedia.org/wiki/Mock_object).

![User layer testing](/assets/article_images/2016-11/diagram2.png){:width="500px"}

Unit tests are doing the same, but instead of using user interface they use logical components program interface. This should be done both frond end and server components.

![Component layer testing](/assets/article_images/2016-11/diagram3.png){:width="400px"}

This layer isolation simplifies the test creation and execution. It would be much harder to setup and maintain system level test automation as it would require separate deployment of client, server, databases etc. We would get alot of false fails due to sudden network connection failures or deployment synchronization.

## Mocks

## Running (jenkins, ktulhu)

## Who should automate?

## assetstore github

## conclusion