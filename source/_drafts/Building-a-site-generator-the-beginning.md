title: Building a site generator - the beginning
categories:
  - Technical
  - filepress
tags:
  - JavaScript
  - filepress
  - Node
  - webdev
---

Side projects, we all have them. We start them because we want to build that one thing ourselves or because somethings is missing out there or just for fun. Or maybe because we want to try something out to see how it works.

![How side projects go.](/images/tech/side-projects.jpg)

<!-- more -->

My most recent side project is named filepress for now. After building a markdown editor in [BlankUp](https://github.com/HoverBaum/BlankUp-Electron) and a theme for hexo in [meilidu](https://meilidu.github.io/) filepress is my next step towards a site generation ecosystem. It is a static site generator taking in markdown files and turning them into a static HTML website. Basically what tools like [hexo](https://hexo.io/) or [metalsmith](http://www.metalsmith.io/) do. Which would be my inspirations. But before we get into that lets stay on side projects for a while.

## Learning

What makes me write this post is the realization that more than anything else side projects like this are a learning experience. And especially side projects which build something similar to existing tools. Because by building something from the ground up we learn how and why things work this way. During my internship last year my senior had me build a framework for one page applications from scratch to get a feeling for the pitfalls that all frameworks had to solve and thus develop an understand of their solutions.

And it is no different with a site generator. They all basically solve the same problems only with different approaches thus being better for different scenarios.

## Filepress

Now besides always wanting to build thing myself I wanted to build filepress because I really like how easy it is to get into hexo and use its CLI. On the other hand I like how easy it is to use and integrate with metalsmith programmatically. Logical conclusion: build "best of both worlds" tool.

So my goal is a tool that has a programmatic interface which is easy to reason about and at the same time provides a standard CLI that enables users to quickly have it up and running.

And so I started learning....

## The inspiration

**hexo** comes with a nice CLI that provides standard commands to create new source files, handle drafts, preview the generated site and publish it. I really love that about hexo. While I was building a theme for it however I discovered that its API was a pain to understand. I basically had to hope to find an example of someone doing what I wanted to in order to do it myself.

**metalsmith** is doing that part way better. It has a system of plugins that can be chained to tell metalsmith what to do. It only provides a scaffold and an API for these plugins. Now this is really nice to work with and easier to think about.

I really love the idea of metalsmith to have plugins that tell it how to work. Basically generate objects from all source files and than run transforms on those until they are written to disc again. When I started filepress I was surfing the hype of streams and thus decided to internally model this as a stream. Taking the functional approach to have objects and transforms on those. Thus filepress exposes a `use()` method that takes in a transform and provides an API those transforms have to follow. Unlike metalsmith however filepress simply passes the objects for source files along the stream. While metalsmith passes all the source files to its plugins filepresses transforms only get one object at a time. Again following a bit of a function approach I think that makes it easier to reason about what a transform does.

Now what I like more about hexo is how its CLI is set up for the task of running websites. It also uses a YAML config file which I venture to say is easier to use for the average user than metalsmiths config file in JSON. You also need to know less about hexo to use it than you need to know about metalsmith or lets say use it "savely". For in metalsmith you could blindly copy a config file from somewhere and that could work. So filepress provides a CLI that can handle things like themes for your website or which transforms to use.

## Why not ...?

So why not just build an elaborate CLI around metalsmith or at least use that as the core?

Well because I think metalsmith has some things build in I would think are already transforms, I think only handling one file at a time is easier for transforms and quite frankly I want to build something from the ground up.

## First lessons

The first lessons came quick. After setting up a system that would generate objects for the source files and pass them along a stream I realized that some transforms would want to access all objects.

Comic from [commitstrip.com](http://www.commitstrip.com/en/2014/11/25/west-side-project-story/)
