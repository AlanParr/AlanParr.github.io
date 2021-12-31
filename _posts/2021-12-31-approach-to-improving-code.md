---
layout: post
title:  "My approach to improving a messy codebase."
date:   2021-12-31 00:00:00 +0000
tags:
  - Refactoring
  - Code Hygiene
category: Refactoring
---

In the just over a decade I have been a professional developer, I've had to work with alot of messy code, only some of which I wrote. As a result, I've spent alot of time refactoring messy code to try and make it in to something better and want to share some of my methods for achieving this.

How I approach this has changed over the years but the below are areas I try to focus on and why. The order I do them in and how hard I go on them depends on the code base.

## How did this happen?
First, we need to understand how messy and poorly structured code comes in to existence in the first place.

### Inexperienced developers 
One or more inexperienced devs with little or no experience were given a task and likely told to get on with it with no supervision and no experienced members to point them in the right direction. What results is a code-base that probably does the job, but is brittle and hard to change.

### Lack of standards and/or conventions
Give 10 developers a task and they will likely all implement it differently and all of them will probably be right in one way or another. Compound this over time in a team and you'll end up with an inconsistent code base with no defined architecture where individual pieces integrate poorly or not at all. One person's `Provider` is another person's `Manager`, which to another person is a `Factory`. Mix and match these terms and it becomes impossible to determine how any individual component of the system is going to behave.

### Lack of ownership (AKA just get it done)
This is similar to the previous item but is very much a function of developers being used as resources with no ownership over the codebase they work on and no power to define the priorities of what happens to it, all the while being under pressure to deliver more and more complex features ever faster than before.

When this happens over a prolonged period of time, the code base will become messier and no matter how much they shout about technical debt and the need to improve it, the business will push that back to `the next sprint` or `after this feature is done`, developer turnover will likely increase, and the business will be faced with slowing down development, reducing quality, or increasing the number of devs working on features to compensate for how difficult it is now to make changes. But this will not scale over time.


I'm sure there are other reasons I haven't listed, but these are the big ones I have encountered personally.

Whatever the reason, you've been left with an unholy mess that you not only need to understand but also be able to fix and change.


## Where to start?
So you've got this messy code-base in front of you and we'll assume you're new to it, so now you've got to figure out how to make it better in small ways that you can fit in between features. This is what I do.

The conventional wisdom will generally be "put tests in before touching anything". This advice is absolutely correct, but in reality, there are things you can do that are relatively low in risk without having test coverage.

I'm not going to go over how to restructure the code as there is already plenty of material available on how to do this. I want to talk more about the things that often don't get mentioned - tidying up the code as a precursor to performing massive life-altering refactorings.

## But why?
Old code-bases can be messy. They've seen many different `best practice` implementations which have changed over time. They've got dead-code, zombie-code, code where only the developer who wrote it and god knew how it worked and now only god knows.

Refactoring is the task of taking this code and transforming it in to something better, but that is not an overnight task nor a straight path.

I believe there is a great deal of value in performing simple tidying up of the code before you go to the extent of refactoring it, especially if you have to carry on changing it in the meantime.

It is akin to buying a house with a badly overgrown garden. You know you are probably going to rip it all out at some point, but for the near-term, you want to trim it all back, remove the rotten shed and the three mouldy seats taken from a 1992 Ford Mondeo that you found down behind the garage, so you can get an idea of what you've actually got to work with once all the cruft is removed.


### IDE Warnings
Every IDE will raise warnings, and they do this for a good reason. If you've got a messy code-base, chances are you've got a boatload of warnings. Some of these may be small problems like unused variables or larger issues like the IDE trying to warn you about code that will actually cause problems at runtime, such as mismatched assembly versions or suspicious looking code blocks.

In Visual Studio, and I'm sure in other IDEs, each warning has a unique code and will likely provide a link to a page explaining the reason for the warning and sometimes how to fix it.

You'll never see any of this, of course, as the warnings you are actually interested in become the proverbial needle in the haystack and the count in the warnings window is a daily reminder of how much the code-base sucks.

The first thing you can do is start reducing these. This is my strategy:

* Pick a warning code and filter by that code in the IDE.
* Start a branch, run through the list and fix each one. If you have a build server, commit and push it up for a build regularly.
* Once done, merge the branch and move on to the next one.

One thing to note about this strategy is that while it will get rid of the warnings quickly, you end up touching a lot of files. To keep the chances of merge conflicts low, no individual branch should last longer than a day, two at most. This may mean you need multiple branches to tackle all the warnings for a single code, but branches are cheap and it will be better in the long run to not have a massive branch with hundreds of files in it to merge after 2 weeks of the rest of your team making changes, that way lies madness and bugs.

Another thing to note is that, while this is generally low-risk as fixing warnings will mostly not change logic, there is always a possibility of unexpected mutations, so keep your wits about you and exercise your own judgement on which warnings you fix and which you maybe leave until later, possibly until you've got time to improve unit test coverage around the code in question.

Reducing (or removing) warnings will have a number of benefits:
1. The warnings list becomes relevant again, so devs can use it for its intended purpose of the IDE warning them when they have potentially introduced an issue.
2. Working on a code-base that had 7k+ warnings and taking that down to 3k (still a way to go) had the very pleasing side-effect of improving performance of Visual Studio. It wasn't life changing, but it has been noticeable for me.
3. If you're new to the code-base, tackling the warnings will take you all over the place so it is a great way to familiarise yourself with the code. This increased knowledge will pay dividends in the future.

As an added bonus, when you're jumping around the code-base checking out these warnings, you may find code that appears to be obsolete or unused. Good examples of this are methods that are no-ops or have had their contents commented out (yes, this is a thing) or methods whose name suggests they have a side-effect (such as CreateFoo) yet they don't contain any code that would appear to achieve that objective. Where you see these, take some time to check out their usages and, if they appear to be dead code, mark then with the `System.Obsolete` attribute or at least a `//TODO:` so you can come back to them later to validate if they really are dead code or maybe just badly named.


### Obsolete or unused code code
Code that is marked with the `System.Obsolete` attribute will appear in warnings, but this is worth discussing on its own.

A code-base that has not been properly looked after over time will likely have an abundance of unused and obsolete code. If you're lucky it will have been marked with the obsolete attribute. Check these out see what you can safely remove.

Removing the obsolete code has an obvious benefit of reducing the size of the code base and therefore opportunities for things to break. Obsolete code is often an old version of something that was rewritten but never fully migrated to, so you'll probably find you're doing the same thing multiple different ways in multiple different places. Reducing this duplication will improve your stability and also ensure you're not using this obsolete code, which was probably obsolete for a reason. 

As you delete obsolete code, you'll often find more code that was only used by the now-deleted obsolete code that may also be able to be deleted. As a result, it may take a few passes to get it all. IDEs can be great aids in helping you find code that can be be safely deleted, but they are not infallible so look upon suggestions with some suspicion and try to validate code is actually obsolete/unused before you delete it.

This is much easier in a statically-typed, compiled language such as C# where the compiler will make you very aware if you've deleted something that is used. It is harder with interpreted languages but there are often tools available that will provide similar functionality.

One example that caught me out was on a large code-base with a lot of partial classes. Visual Studio would often flag up variables within these classes as not used, however it had managed to miss usage in another partial implementation of that class. Commit and build regularly and you can keep the chances of this happening low.

If you find obsolete code that cannot currently be removed, for example an old version of something that has been re-implemented elsewhere but is still in use in too many places to fix currently, use the `System.Obsolete` attribute to mark the code to come back to later and use the description attribute to explain why you think it is obsolete. It is extremely frustrating to find a piece of code with the Obsolete attribute on it but nothing to explain *why* it is obsolete.

## Nuget
This covers 2 issues that are often found in older code bases.

* Nuget packages that are massively out of date.
* Checked in assemblies from the days before Nuget existed. 

It is good practice to keep your dependencies up to date for obvious reasons such as security, but also to get performance improvements and new features. The cost of updating your dependencies will likely be lower if you're doing it regularly than if you try to jump several years worth of major versions in one go.

Before Nuget came along in 2010, we had to keep our dependencies somewhere, and that was often a folder, usually in the root of a repository, called "Third Party", "reference assemblies" or something similar.

If the code-base pre-dates Nuget and hasn't been well maintained, it will likely have a folder like this and you should seek to eliminate it.

These old dependencies will be out of date and therefore potentially a security risk. They are also a ticking time bomb, waiting for the day when you want to change framework versions or make some other changes that will break them and leave you in a pickle.

If they exist in Nuget, it should be a simple case of referencing that instead and getting rid of your copy. If you're lucky, the same version may even be available as a quick win if you don't want to risk updating right now.

If you can't find a dependency on Nuget then it's time to either write something that does the same job or search Nuget for a current library that will fill the same role.

You'll have to do this eventually so you may as well do it at a time of your choosing rather than when a new feature or bug fix forces your hand.

## Modernise the style
You've probably got a lot of code that is on the older end of the spectrum and doesn't conform to modern standards or make use of modern language features.

This can result in the IDE seemingly assaulting you with so many code improvement suggestions that you can't see the wood for the trees.

While modernising the style of the code may seem like a trivial matter that we really shouldn't worry about, these new language features were introduced for a reason, often to improve readability, and in our messy code-base, we could use every bit of readability we can get.

To see how much of a difference this makes in your case, pick any source file, preferrably one that is small but tough to understand. Then start following the IDE recommendations.

* Convert explicit typing to implicit typing with var.
* Convert uses of `String` or `Int32` to their keyword equivalents of `string` and `int`.
* Rename variables to match an agreed upon convention, either using the language default or agree one within your team and use the IDE to automate implementing it.
* Remove unnecessary blank lines between methods, variables, and classes.
* Remove unused usings.
* Remove `this`, renaming variables where necessary to remove the need for it.

Try every recommendation, even if you're not sure about it. Don't be afraid to reverse it if you think it makes the code look worse.

Once you've done all this, you may well find that the readability of this file has just increased massively, or you may find it has made no difference at all other than to reduce how much it upsets the IDE, your mileage may vary but give it a go and see if it makes a material difference in your code.


## Testing
At what point you start to focus on this really depends on your appetite for risk.

You might make this your first stop before touching anything or you may have the confidence that you can leave it until you have tidied things up a bit, that one is down to you.

Most of the tasks I have listed thus far can be done without test coverage with relatively minimal risk.

At some point though, once you've tidied up, you're going to have to start making higher risk changes that you just don't want to attempt without any test coverage.

Writing tests on a new code-base is usually very simple, but on an older one, it can be disproportionately difficult to get in to a position where you can test even the smallest piece of functionality and there is a whole spectrum between those two options, so now you need to find out where you sit on that spectrum.

Pick a simple case as a canary to set how easy it is to create tests for existing code and just starting writing. First try to get to a point where you can call the method without it failing or instantiate the class without it throwing an exception.

This will often expose things like hard dependencies on external services or subsystems, such as databases and file systems. It will also begin to show you the hidden assumptions that exist within the application.

In the first instance, try to find ways to provide these dependencies without making changes to the code. This should be relatively easy if you have Dependency Injection, but may require you to be [creative](/dependency-substitution-for-legacy-code) if you haven't.

If you need to make changes, try to keep them to a minimum and introduce as little extra logic in to the calling code as possible, we don't want to make the mess any worse.

Factories are often a good choice here as they can generally slip in with relatively little ceremony. They likely won't be the end-game but can be a useful aide in your journey.

You may sometimes have to introduce abstractions that you don't really like in order to unblock a piece of code and allow you to move on to the next stage of improvement. Don't be afraid to do this as a step on the path from where you are to where you want to be.

Once you've got to the point where you can call your method or instantiate your class without exception and get some asserts in there, you've just created a foundation that you can build upon to improve the coverage on your code-base and start testing more and more complicated parts of the system.

I don't want to go too deeply in to the finer points of refactoring as there is a great deal of information out there written by people who know far more and are significantly better at communicating that knowledge than I am, however one book I can recommend is [Working Effectively with Legacy Code](https://smile.amazon.co.uk/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052/ref=sr_1_1?keywords=working+effectively+with+legacy+code&qid=1640980440&sprefix=working+effe%2Caps%2C73&sr=8-1) by Michael Feathers, I found it very useful to fill in some gaps in my knowledge and inform my approach as i've embarked on refactorings in the past.


I hope the above was helpful. If you've got a tip or trick that works for you, feel free to post it in a comment.