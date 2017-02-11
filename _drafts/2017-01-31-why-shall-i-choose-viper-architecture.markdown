---
layout: post
title:  "Why shall I choose VIPER architecture?"
date:   2017-01-31 14:12:20 +0100
categories: moviper
---

## **TL;DR**
**VIPER architecture allows you to easily TDD your app, keep the code clean, share modules, and distribute the work. It speeds up a general development process by increasing frequency of sending the new functionalities to QA and Product Owner. It speeds up a multi-platform development as Android and iOS can share the general logic. It also makes you more agile allowing changing only specific parts of an app at any moment. For each screen it introduces five or six modules, but there are tools to generate these for you. It has some entry threshold, but it can be overcame in a few days. There are plenty iOS articles and examples to learn from, and a lil bit of Android ones, but I will try to change this deficiency in this blog, using [Moviper][moviper] VIPER architecture library for Android.**

# Introduction

Every developer in his path encounters pretty similar obstacles while trying to advance from basics to advanced solutions. I think that today's tutorials and blog posts aren't very helpful in such cases, because they form two groups in general - very basic ones and very advanced ones, so obviously there is a huge gap between these clusters. In my blog I will try to fill the mentioned gap on the example of a one specific architecture, that, I think, makes the journey from primitive to the masters knowledge somehow easy and pleasant. In this post I will try to convince you that using this architecture - a VIPER one - is worth trying. Let's begin with reminding ourselves what caused the most of the problems on our developers path.

# Junior Dev Retrospection

Fellow mobile developer, at the very beginning, most probably you implemented your first hello world app, then started to explore the depths of your platform. Very quickly you have realized that your Activities / View Controllers have grown to the enormous size, making you flea to the Google to find some solution for that. Then you found it, the glorious MVP/MVC/MVVM/MV-something architecture that claimed it will resolve your problem for more than sure. You grabbed your coffee, rushed the keyboard and rewrote your app in the brand new, proper way. After a few pretty long hours you laid back in the moment of enjoyment of the great piece of software you have just crafted but then you have frozen in horror. Your Activities / View Controllers are pretty slim right now, but wait, we have another god-class, the Controller/Presenter/ViewModel! Oh noes, they are responsible of handling the routing through the app, data management, interacting with system framework and many other things! That nasty classes for sure do not fulfill the idea of the [Single Responsibility Principle][srp]. What shall I do now? Some senior developer looked at you with indulgence and advised warmly: *"Delegate your responsibilities from presenter"*. Do what? Delegate what? After a quick research you instantly cached up, but then you got stuck again. *"I still don't know what I shall delegate! How do I know if I should delegate every method, bunch of them, or maybe shall I just split the presenter to the half? I'm just a junior developer, I don't have a such intuition!"*"

Let's skip the later part for now. Now think about that other situation, the situation in which you have found a bug in your app. You found it, so you fixed it. The problem is that when you have fixed it, the two new nasty regressions came up in your project. After fixing one of them, another two showed up to accompany the one that have left from the first wave. It was like fighting a hydra. Very quickly you found yourself banging your head against the wall trying to get a grip on the project again. That sad scenario lead you to discover testing and [Test Driven Development][tdd]. You quickly went through all of the tutorials of TDD-ing a simple calculator app, instantly grasped the basics and rolled up your sleeves to reimplement your app in the proper way. It rapidly turned out that it's kinda like tutorial of drawing an owl:

![That's how the first approach to the testing looks like!]({{ site.url }}/assets/HowToDrawAnOwl.jpg){:.center-image}
*That's how the first approach to the testing looks like!*{: style="text-align: center; display: block;"}
*Source: http://forimpact.org/for-impact-ideas/how-to-draw-an-owl/*{: style="text-align: center; display: block; font-size: 60%;"}

You quickly realized that you have no idea how to test your system. *"What am I supposed to test? I click this button in app and load my data in Activity / View Controller, there is nothing to test really!"*

# Reveal

Here comes the VIPER architecture — a reveal for these problems. It proposes a pretty decoupled architecture of each scene (let's say *"scene"* like that for the simplicity right now) that's a good start for making all of your responsibilities delegated. Moreover, decoupling the architecture makes it contains a reasonable amount of [seams][seams] that allows you TDD your app. Actually, it's not a completely different approach than a well-known MVP, it's just a concept of adding some modules to the presenter to delegate them responsibilities that repeatedly appears in all of the mobile application scenes. Let's dive in!

Each VIPER module consists of following sub-modules:
1. **View**: user interface
2. **Presenter**: delegating appropriate tasks to Router and Interactor
3. **Router / Routing**: system framework interactions: mostly app routing (switching views)
4. **Interactor**: data-related business logic
5. **Entity** (-ies): objects representing your data
6. (sometimes) **Builder**: starts a whole VIPER module

![That's the general look of the Viper architecture.]({{ site.url }}/assets/ViperDiagram.png){:.center-image}
*That's the general look of the Viper architecture.*{: style="text-align: center; display: block;"}

I won't cover each of them in this post as it's rather a material for an another publication. I will discuss it in the next article, but if you are really curious right now, you can read [the original objc VIPER article][objc-viper] (I guess). For now let's focus on the general idea of the VIPER. It makes your every screen a independent module you can move around your project or even your whole projects portfolio. There's even more: it makes every sub-module independent so you can ie use VIPER from one project in another one switching a implementation of only one of sub-modules to meet the requirements of the target application. Having all of the modules and sub-modules decoupled and independent makes them extremely easy to test. Every sub-module has it's own responsibility, so it's much more easier to keep them small, neat and clean throughout whole development process. Moreover, it introduces a huge acceleration to the enterprise mobile development process as a whole team can work on the one screen at once and that leads to increase frequency of a dev - QA iterations during the sprint. That powerfully eases agile approach - customer can review screens earlier, and it also soothes a dev - QA edge by keeping QA busy on the reasonable level, it's not like in an „old” approach where bored QA takes care about other projects on the beginning of a sprint, and when a whole bunch of views is ready at the end of a sprint you have to wait for QA to finish another projects, and then you DDoS them using all the views you have developed. Moreover, it introduces a unified responsibilities split that allows developers understand the new modules at a glance. Furthermore, it makes multi-platform development much easier as Android and iOS can share interfaces / protocols, Rx streams and general logic of internals, if you manage to maintain uniform architecture across platforms. In addition, if you write your Android code in [Kotlin][kotlin], you can port the presenter, tests and some utils between platforms with very little effort, as Kotlin and Swift have very similar grammar. Android and iOS parts of the team can do different modules at once, and after finishing that they can switch and base their code on already done components of the another platform. What is more, making your app decoupled allows your team to be more agile, especially when circumstances of your project make the specification to change frequently, because you can change any of the modules or the sub-modules without even touching the rest of the codebase. That's a pretty bunch of advantages!

# Doubt

Yes, yes, I can hear you yelling *"But my view X is so simple, I don't care about creating a six components for that! It's over-engineering! There's even nothing to test!"*. Now think about that app you did last time. You have pretty similar, very simple view X in there, but now it has grown to the enormous size. Have you managed to refactor it? No? Well. Is there something to test right now? And is it tested out? Well... Now you see. Moreover, I don't think that having to create some components is an argument if we can delegate this dirty work to some scripts or plugins like [Moviper Templates Generator][moviper-generator] or [Generamba][generamba]. Of course there is also some entry threshold but it usually takes only a few days to feel really confident in this architecture, and a gain of using VIPER grows with every line of code that adds a pinch of a complexity to your codebase. Of course there is also some boilerplate added to define all of the essential sub-modules, but I don't think that it's a remarkable downside in the face of the upsides of this approach.

# Sum up

Let's sum up the upsides of VIPER

- clean code,
- easy TDD,
- reusable modules and sub-modules,
- easier work distribution,
- instant understanding of modules implemented by other devs,
- sharing logic across platforms,
- enhanced agility.

And check out the downsides as well:

- needs additional work to repeatedly create all of the modules **[BUSTED]**,
- some entry threshold [to beat in few days],
- some boilerplate added [but it weighs nothing in comparison to all of the upsides].

For me it looks like upsides of this architecture weigh much more than downsides, so I think that it's definitely worth trying to adopt it to your workshop.

# But wait...

All of the iOS developers are happy now and they've just started reading various articles about iOS VIPER implementation, as there are plenty of them. You, iOS fellas, can stop reading right now as you have lots articles to read about that.  On the other hand, Android developers got somehow confused, as searching for the ["Android VIPER architecture"][googleViper] gives some appropriate results, but there isn't even a full page of Google results. Let's review what do we have here:

* Lyubomir Ganev [series of posts](http://luboganev.github.io/blog/clean-architecture-pt1/) and a [related sample](https://github.com/luboganev/Carbrands)
* Richa Khandelwal [post on a Realm blog](https://realm.io/news/360andev-richa-khandelwal-effective-android-architecture-patterns-java/) and a [related sample](https://github.com/richk/CourseraDemoApp)
*  Jiri Helmich [presentation](https://speakerdeck.com/helmisek/android-viper-architecture-implementation) and a [related sample](https://github.com/Helmisek/android-viper)
* Dmytro Zaitsev [presentation](https://speakerdeck.com/dmitriyzaitsev/viper-sexy-architecting-or-mvp-on-steroids) and a [library he created](https://github.com/RxViper/RxViper)
* and others that I probably couldn't find.

So there are already some sources to learn from, and it's great if you liked and wanted to adopt one of these! Looking in the samples you can see that everyone has it's own interpretation of VIPER. In first two samples I somehow feel the lack of an architecture-enforced Router / Routing (correct me if I'm wrong of course). The following two ones, unfortunately, have only presentations with not much text that describes their usage what makes it not so easy to start using them. I discovered them after I have been developing my VIPER library, [Moviper][moviper], and I perceive the roles of the VIPER components in a slightly different perspective. You can check out [Moviper][moviper] to investigate the differences for now, in later posts I will describe my feeling of this architecture in details. As you can see, there isn't much about VIPER in the Android society, and this concept still isn't monolithic throughout developers so I feel like there is yet a lil bit of a room for me. That's why I decided to publish some articles about general VIPER and [Moviper][moviper] usage and features. Stay tuned!

[objc-viper]: https://www.objc.io/issues/13-architecture/viper/
[moviper-generator]: https://github.com/mkoslacz/MoviperTemplateGenerator
[generamba]: https://github.com/rambler-digital-solutions/Generamba
[moviper]: https://github.com/mkoslacz/Moviper
[srp]: http://www.oodesign.com/single-responsibility-principle.html)
[tdd]: http://agiledata.org/essays/tdd.html
[seams]: https://www.philosophicalhacker.com/post/what-makes-android-apps-testable/
[kotlin]: https://kotlinlang.org/
[googleViper]: https://www.google.pl/#safe=off&q=android+viper+architecture
