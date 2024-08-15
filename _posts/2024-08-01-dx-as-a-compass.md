---
title: Developer Experience (DX) as a Compass
description: Developer Experience (DX) is a crucial aspect of software engineering that impacts productivity and code quality. It encompasses choosing the right tools, optimizing work environments, implementing practices like Test Driven Development, making timely decisions, and maintaining good software design. 
categories: Miscellaneous
featured_image: /assets/img/dx-as-a-compass/compass.webp
image: /assets/img/dx-as-a-compass/compass.webp
featured_image_alt: An illustration a developer experience compass 
comments: true
---

The end of July 2024 marks 7 years since I started working professionally as a software engineer. during these 7 long and amazing years, I worked in different environments, in 5 different countries, and each its own mindset and approach to business.

One of the environments were very high paced, to stressful. An environment where requirements changed each week, where we had to pivot very frequently. An environment where we started with a simple form, changed it to a reporting app, then adapt it to a different industry.

Other environment had a very complex domain, the energy and the banking market, where we had to deal with complicated business processes and complicated calculations and formulas.

As a developer in these diverse environments and with some time, experience, trial, and error, I had come to the conclusion that my experience is very important and it should be a target for optimization. And this is far from being out of selfishness, but I found out that doing so helps also with the quality of the software I create and that’s for many reasons which I’ll try to go through in this article.

But to take off (’m writing this on a plane), let’s define things before we conquer them.  

### What is Developer Experience?

Developer experience (DX) refers to many things. First of all, it is about how I feel when doing my job, am I relaxed? am I frustrated? am I bored? am I stressed? am I beating my head over the keyboard? It is about the journey from analysis, to problem solving, to shipping. It is about how good is my physical and virtual environment, the tooling, and the technics.

A good DX let me do my job efficiently, spending time on doing useful things, let me focus on shipping the feature to the user with an optimized feedback loop.

And to reach this Nirvana, there a couple of technics and concept that I use. Let’s start from the very simple one.

## The right Tools for the job

As obvious as it may sound, but choosing the right tool for the job helps a lot with getting the job done **efficiently** (efficiently is the keyword). But what makes the tool “right”? Let’s break it down. 

### The target runtime

For instance, tools used for mobile development and tools used for web development are not the same, so choosing the wrong tools for the runtime might get the job done but not always efficiently. 

Let me tell you about my experience, I started my professional career as an Android developer then with the raise of Flutter, I decided to switch to it and I ***instantly*** became a cross platform (web included) developer, right?. 

As you might know Google conceived Flutter to be one framework to rule all runtimes and listening to that I was developing web apps using it, mainly out of laziness to learn other frameworks. Then, I spent some time learning a real web framework and to tell you that I was in complete denial to what your DX would be like using the right tool for the job, maybe we will see Flutter getting better in the web but for now I’m not a picking it for webapps (unless there are reasons to do so, e.g. a mobile app is the priority and the web might be a good addition, as a low focus part, to the business)

### Completeness of the Tool

Let’s be honest to each other, there are some languages and frameworks that are more “complete” than others, meaning, they have all that you need to get your job done without any external dependencies or you rarely need to install any external package. 

And that’s how I feel when I compare my experience with .NET and other frameworks. .NET comes with a Base Class Library (BCL), that is very rich and well designed for the developer. And I can go very far without thinking “Wait, do I need an external package for that?”. And that’s the same for [ASP.NET](http://ASP.NET) which is the de facto .NET web framework.  

In certain ecosystems, external dependencies are a necessity while in some others they are a cherry-on-top, and I tend to prefer the later than the former when I have a choice. 

We might introduce a metric “Mean time to add a necessary external dependency” (notice, necessary and external) and the higher this value is the better. 

And if you are asking yourself, “why does this guy hate external dependencies” please read along until LRM. 

### Maturity of the Tool

Being an early adopter of a tool might be a good way to excel with it, or participate in its development but it can be a high risk decision. Some tools come to optimize a certain aspect and if you use the tool where this aspect represents a small part of your need, then you are officially pushing yourself over the edge. You will find yourself doing weird stuff just to do something that could be done with an existing, less shiny and less recent, tool. 

And some tools that existed for decades, have a certain maturity also in best practices and how to use the tool efficiently. That’s why I usually recommend for my fellow developers to take a look to what exists even if they are not going to use it. There is always things to learn and things that can benefit newer or modern ecosystems. Sometimes you can’t see the “why” behind new things if you didn’t taste the old way of doing things. A clear example of that is jumping from Server rendered web pages, to full single page applications to then a balance between the two with Server side component, hydration, and all the hype words. 

## The right environment

In addition to the right build technology, it’s also important to have a good environment. By that I mean, your note taking app, your diagraming tool, your code editor and its plugins, your IDE, and your terminal. 

Everyone can have this but (10x Developers. No. I’m joking) to have a better DX, it’s important to know your environment. If we are talking about your IDE, it’s important to learn the shortcuts, or customizing it to make repetitive stuff easy. This is one of the reasons I use JetBrains IDEs and Ideavim, the experience across all the editors is similar and with Ideavim I have a set of key bindings that are editor agonistic but also customized to make repetitive stuff easy, like surround the highlight with parenthesis or brackets, run the test, rerun the last test, build the project, run the project, open the refactoring options, etc. If you are interested you can find the link to my .[ideavimrc](https://gist.github.com/karimkod/ac39d065688f03b37e56030ef6551b70). 

Having a set of aliases or ready to run commands in your terminal as well as knowing a scripting language can help you increase your productivity. 

All this will help you get started easily, and even help with procrastination, and believe a lot of bad code and design come from procrastination and doing things “later” that is spelled “n-e-v-e-r”. 

Now that we have talked about the technical toolbox, let’s talk a bit about some technics and concepts that helped me with improving me DX. these have helped me personally, see if it applies to you as well and your feedback is always welcome.

## Test Driven Development (TDD)

Test driven development is an approach to driving the implementation of the system’s behavior using executable specifications defined as tests. 

That’s a mouthful, let’s break this down with an example. Let’s suppose you talk with your users, and they describe to you a feature they would like to have. You take the feature, you write down it’s different use cases in a piece of paper, or your todo list or as a comment in your test file.

You start by the first use case, you write a failing test for it. You then write the minimum required code to run that use case, then refactor the code to make it better. Then you translate the next use case to a failing test, you write the code to make it pass, then you refactor and you continue until you implement the whole feature. 

This simple approach helps with so many aspects. 

### Learning the Domain

Breaking down things and try to think about the next test to write has helped me ask the right questions and dig deeper than what’s written in the user story or the Jira Ticket. It pushes me to get out of my bubble and ask the domain experts, the users, the designer, or any other stakeholder.

Anecdotally, we even had our client in the energy market say that our calculation are more correct than the legacy system because we thought of all cases.

### Tackling Complexity

Some domains are very complex, and breaking down things and learning them bit by bit helps managing the complexity. 

And some solutions can be complex and the ease and confidence to refactor helps with exploring different solutions and potentially find a simpler one. 

### Peace of Mind

Having tests as a safety net, let me refactor peacefully, add new features without worrying about breaking things. Pushing code knowing that I have implemented it as the user expects it or at least as the team understood the requirements. When something breaks while changing the code, I’m quite certain that it is the last change that caused it and I can just revert to the last green state.

### Feedback Loop

One of the traumas I have with my early days of coding comes from manually iterating while coding a feature, the scenario is as follows : 

1. Fill out the fields of a form
2. Submit the form
3. Run a GET request to check if the data is restored properly
4. Clean the database and start over again

and I had to do this for all possible scenarios, if there is one word that can describe this way of working is HELL. 

It was a very very long feedback loop and there is a lot of time wasting (imagine all the clicks). With TDD, I write the test of the use case then run it as much as I can until I'm done, if I have different input I can parameterize the test. If I have a combinatorial explosion I can isolate the piece of code and run all different use cases on it.

And the perk is that all this is repeatable for the future.

### Interrupt-ability

With the list of your tests, you know where you are and what you have done and with a failing test you know what you are planning to implement, this will help keeping the progress and if your teammates calls you and you get distracted, a simple look at your screen and you will know where to resume. And you can leave to your lunch break without any overthinking and think about what to order instead (It’s always a pizza).

Ok, enough of the TDD fanboyism and let’s move to the next concept.

## The Last Responsible Moment

This concept is a pretty powerful and it refers to the moment where taking a decision is necessary and optimal, if taken earlier than it will be a burden for the project and if taken later then a cost that could have been prevented would occur.

This can be applied to every decision, let me enumerate some that I can think of: 

- Choosing the framework: usually comes at the start of the project
- Choosing a database: usually comes when we know our data model and the interactions with the data.
- Adding an external dependency or a package: the moment when there is a real necessity for it.
- The deployment model: at the moment of deployment.
- Any change: when there is a need for it.

You may ask, how is this related to DX. Well, when decisions are taken before the LRM, there is a burden that come with them, for example if you include a dependency in the project before really needing it, then if it breaks your for whatever reason than you will waste your time fixing it rather than focusing on important stuff.

And in contrast to that, if you don’t include it at the LRM than you will also sacrifice some of your DX and time and maybe money doing things the hard way while there is an easier way. 

So, LRM is about stay focused on the important stuff and avoid decisions until their optimal moment.

I hope you see it now.

## Good Design is Good DX

Last but not least the better the quality of your code the easier it is to work with and the happier you are. 

Learning how to architect the software and how to write better code will only benefit you and your work. 

### Domain Driven Design

Use Domain Driven Design to better capture the concepts of the business domain in your code, it will help reduce the communication overhead, and it will easily allow you to map the business to your code. And it will drastically reduce the ramp up time for new colleagues

### Stay up to date

Know your abstractions, because you will need to know their internals when your code breaks, paraphrasing a tweet that I encountered:

![Tweet of knowing the abstractions](/assets/img/dx-as-a-compass/tweet.png)

Read documentation of your tools, stay updated, the only constant is change and there is always better ways to do things. All this will help you to do things properly and will make it easier for you on the long run. 

### Production is down?

Mind your monitoring tools, as they are your go to when production breaks. 

Mind your CI/CD pipelines as they will allow you to iterate quickly and push fixes to production easily in an automated manner.

This will help you sleep peacefully and if you woke up because of a night call, you will go back to sleep pretty quickly. 

# Conclusion

To conclude, I hope this article helped you see why I decided to improve my developer experience in order to improve the quality of my craft and I hope you now know the technics that helped me achieve that.

If you have any feedback you can reach out to me via my socials and all C&C are welcome 😁