---
layout: post
title:  "Why shall I choose VIPER architecture?"
date:   2017-01-31 14:12:20 +0100
categories: moviper
---

# Junior Dev Retrospection

Fellow mobile developer, at the very beginning let's remind ourselves your very first steps at the mobile world.

Most probably you implemented your first hello world app, then started to explore the depths of your platform. Very quickly you have realized that your Activities / View Controllers have grown to the enormous size, making you flea to the Google to find some solution. Then you found it, the glorious MVP/MVC/MVVM/MV-something architecture that claimed it will resolve your problem for more than sure. You grab your coffee, rushed the keyboard and rewritten your app in the brand new, proper way. After few pretty long moments you laid back in the moment of enjoyment of the great piece of software you have just crafted but then you have frozen in horror. Your Activities / View Controllers are pretty slim right now, but wait, we have another god-class, the Controller/Presenter/ViewModel! Oh noes, they are responsible of handling the routing through the app, data management, interacting with system framework and many other things! That nasty classes for sure do not fulfill the idea of SRP
!!!LINK!!!
principle. What shall I do now? Some senior developer looked at you with indulgence and advised warmly: "delegate your responsibilities from presenter". Do what? Delegate what? After a quick research you instantly cached up, but then you stuck again. What shall I delegate? How do I know if I should delegate every method, bunch of them, or maybe shall I just split the presenter to the half? I'm just a junior developer, I don't have a such intuition!

Let's skip the later part for now. Now think about that other situation, the situation in which you have found a bug in your app. You found it, so you fixed it. The problem is that when you have fixed it, the two new nasty regressions came up in your project. After fixing one of them, another two showed up to accompany the one that have left from the first wave. It was like fighting a hydra. Very quickly you found yourself banging your head against the wall trying to get a grip on the project again. That sad scenario lead you to discover testing and TDD
!!!LINK!!!
 . You quickly went through all of the tutorials of TDD-ing a simple calculator app, instantly grasped the basics and rolled up your sleeves to reimplement your app in the proper way. It rapidly turned out that it's kinda like tutorial of drawing an owl:

![That's how the first approach to the testing looks like!]({{ site.url }}/assets/HowToDrawAnOwl.jpg){:.center-image}
*That's how the first approach to the testing looks like!*{: style="text-align: center; display: block;"}
!!!SOURCE!!

You quickly realized that you have no idea how to test your system. "What am I supposed to test? I click this button in app and load my data in Activity / View Controller, there is nothing to test really!"

# Reveal

Here comes the VIPER architecture — a reveal for these problems. It proposes a pretty decoupled architecture of each view (let's say it like that for the simplicity right now) that's a good start for making all of your responsibilities delegated. Moreover, decoupling the architecture makes it contains a reasonable amount of seams
!!!LINK!!!
that allows you TDD your app. Let's dive in!

Each VIPER module consists of following sub-modules:
1. View
2. Presenter
3. Router
4. Interactor
5. Entity (-ies)
6. (sometimes) Builder

I won't cover each of them in this post as it's rather a material for an another publication. For now let's focus on the general idea of the VIPER. It makes your every screen a independent module you can move around your project or even your whole projects portfolio. There's even more: it makes every sub-module independent so you can ie use VIPER from one project in another one switching a implementation of only one of sub-modules to meet the requirements of the target application. Having all of the modules and sub-modules decoupled and independent makes them extremally easy to test. Every sub-module has it's own responsibility, so it's much more easier to keep them small, neat and clean throughout whole development process. Moreover, it introduces a huge acceleration to the enterprise mobile development process as a whole team can work on the one screen at once (each dev taking one sub-module instead of every dev works on his own screen, where there’s nothing for long and then lots of screens and busy QA), and that leads to increase frequency of develompent team - QA iterations during the sprint what prevents a QA DDoSing on the end of the sprint. That powerfully eases agile approach - customer can review screens earlier, and soothes qa-dev edge keeping qa busy on reasonable level, not like in „old” approach - bored qa takes care about other projects, then bunch of views is ready, you have to wait for qa to finish another projects, and then qa is very busy because of having lots of views to check at a time. Moreover it introduces a unified responsibilities split that allows developers understand the new modules at a glance.

# Doubt

Yes, yes, I can hear you yelling "But my view X is so simple, I don't give a ship about creating a six components for that! It's over-engineering! There's even nothing to test!". Now think about that app you did last time. You have pretty similar, very simple view X in there, but now it has grown to the enormous size. Have you managed to refactor it? No? Well. Is there something to test right now? And is it tested out? Well... Now you see. Moreover, I don't think that having to create some components is an argument if we can delegate this dirty work to some scripts or plugins like [Moviper Templates Generator][moviper-generator] or [Generamba][generamba]. Of course there is also some entry threshold but it usually takes only a few days to feel really confident in this architecture, and a gain of using VIPER grows with every line of code that adds a pinch of a complexity to your codebase. Of course there is also some boilerplate added to define all of the essential sub-modules, but I don't think that it's a remarkable downside in the face of the upsides of this approach.

# Sum up

Let's sum up the upsides of VIPER

- clean code
- easy TDD
- reusable vipers
- reusable modules
- easier work distribution
- instant understanding of other devs modules

And check out the downsides as well:

- needs additional work to repeatedly create all of the modules **[BUSTED]**
- some entry threshold [to beat in few days]
- some boilerplate added [but it weighs nothing in comparision to all of the upsides]

## Android Doubt

All of the iOS developers are happy now and they 


[moviper-generator]: https://github.com/mkoslacz/MoviperTemplateGenerator
[generamba]: https://github.com/rambler-digital-solutions/Generamba
