---
layout: post
permalink: /js-leave-the-classes-to-those-other-languages
title: 'JS: Leave the classes to those other languages'
path: 2016-04-22-js-leave-the-classes-to-those-other-languages.md
time: 3
---

> _Who have never used pseudoclasses in JavaScript?_

It's too hard for a "traditional OO developer" to not use classes. They are his way to describe objects.<br>

The first time I bumped into JS I was the "traditional developer" and the first question I asked to myself was how can I work with classes in JS?<br>
__JS does not have classes__ but I realised pretty soon there are plenty of solutions to fake classes.<br>

Instead of understanding and embracing the language I gave up to the temptation of using my traditional languages (Ruby, Java, etc...) knowledge with JS and I started to work using pseudoclasses. I immediately realised that doing the same things I used to do in Ruby so smoothly were much harder and more verbose to do in JS.<br>
If I didn't want to end up hating JS I needed to learn how the language works in more depth.

This series of articles aims to answer essentially 2 questions:

* Why is so complicated to work with pseudoclasses?
* Do we really need to fake classes in JS?

To do so we will cover the following 3 points in distinct posts:

* [JS Prototype chain mechanism](/js-prototype-chain-mechanism)
* [Pseudoclasses and Prototypal Inheritance in JS](/pseudoclasses-and-prototypal-inheritance-in-JS)
* [Alternative patterns in JS: OLOO style](/alternative-patterns-in-JS-OLOO-style)

This topic was also the subject of the speech I presented as part of my [ThoughtWorks](https://www.thoughtworks.com) application process. Slides and video are available at the following links.

* [Slides](https://speakerdeck.com/giamir/js-leave-the-classes-to-those-other-languages)
* [Video _15 mins_](https://www.youtube.com/watch?v=HnQ0FPwcaZ4&feature=youtu.be)

Let's start our path understanding how the [JS Prototype chain](/js-prototype-chain-mechanism) works.
