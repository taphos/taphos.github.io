---
layout: post
title:  "Why Game Developers are afraid of Test Automation?"
date:   2016-11-09
image: /assets/article_images/2016-11/night-track.JPG
image2: /assets/article_images/2016-11/night-track-mobile.JPG
---

Unfortunately test automation topic is not popular among game developers. Maybe it is due to the nature of project based development cycle. Most game developers are not used to support their own code in the long run. After game is released it receives maybe a couple of bugfix patches at most, so developer don't have to deal with mistakes and bad decisions made, new project will be started mostly from scratch. Though [some](http://yetanothergameprogrammingblog.blogspot.com.ee/2010/06/aaa-automated-testing.html) top game [developers](http://blog.agilegamedevelopment.com/) rise the problem, they mostly stay unheard. Gamedev is not unique in this sense, same situation can be seen in other industries which don't provide a persistently evolving service or product.

With increasing popularity of free to play games rules have changed. Free to play game is actually a living service similar to, lets say, an online banking service. Same problems emerge: legacy code, over-engineering, bad architecture and yes flood of bugs coming with every update. The saddest part is that game developers are not used to it, most still have a "we will do it better next time" mindset.

Game development projects are often managed by non-technical people who don't recognize the difference between project-based and service-based development. I have seen projects collapse due to this problem. Fortunately the solution is invented long time ago. [Robert Martin](https://en.wikipedia.org/wiki/Robert_Cecil_Martin) speaks about test-driven development in his [book](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882), he compares it with [double accounting](https://en.wikipedia.org/wiki/Double-entry_bookkeeping_system) where same calculations are done twice and results are compared.

Unlike web development gamedev lacks testing tools. Especially if you are using inhouse engine. General purpose engines like Unity and Unreal are starting to be taken by professionals seriously, but they lack good testing tools too. Though recently Unity introduced [NUnit test runner](https://docs.unity3d.com/Manual/testing-editortestsrunner.html) and [Integration Test Tools](https://bitbucket.org/Unity-Technologies/unitytesttools).

Unit testing is a good start but certainly not enough. Unity integration tests did not fit our project as they allow to test interactions one at a time. What we needed is to drive the game same way as human does with long chains of interactions. This is true especially for mobile games as mobile gameplay tend to have complex UI interactions comparing to console and desktop games. I decided to write my own test automation framework and make it as simple and lightweight as possible. It came out to be easier and more manageable then integrating bloated test frameworks.

## Recording vs Coding

While recording a test sequence might sound easy and effective, my experience shows that recorded tests don't live longer then one update release. They don't survive refactoring and should be re-recorded again and again, which destroys the point of automation.

##  Which parts should be automated?

[System](https://en.wikipedia.org/wiki/System_testing) and [integration](https://en.wikipedia.org/wiki/Integration_testing) testing are good approaches for manual testing and a useful addition for automation, though automated tests should use a different approach, they should isolate layers of application and test them separately.

Below is a simplified diagram of the game architecture. Player Input and Output is done using game UI and scene objects (avatars, NPCs etc). Information is passed to controllers which process the data, probably give some feedback and use logical components to process game logic, calculate scores, give achievements and so on. Some parts of the logic may be processed on the server side.


![diagram1](/assets/article_images/2016-11/diagram1.png){:width="300px"}

UI tests isolate the layer of user interface and controller by replacing logical components with [mocks](https://en.wikipedia.org/wiki/Mock_object).

![diagram2](/assets/article_images/2016-11/diagram2.png){:width="500px"}

Unit tests are doing the same, but instead of using user interface they use logical components program interface. This should be done both frond end and server components.

![diagram3](/assets/article_images/2016-11/diagram3.png){:width="400px"}

This layer isolation simplifies the test creation and execution. It would be much harder to setup and maintain system level test automation as it would require separate deployment of client, server, databases etc. We would get alot of false fails due to sudden network connection failures or deployment synchronization.

## Mocks

## Running (jenkins, ktulhu)

## Who should automate?

## assetstore github

## conclusion