---
layout: post
title: "Fun with NUnit and ReSharper Templates"
date: 2012-01-07
post_id: 2
---

About six months ago I took a fairly deep dive into test-drive development (TDD); part of that process involved gobbling up as many training resources as I could download, buying a book or two on the subject, and making a focused exploration of the tools I’m already familiar with. Always a fan of efficiency tools in general and ReSharper specifically, I looked to see how this favorite tool could help me eek out a bit more productivity with TDD and my unit test framework of choice: NUnit. Enter [ReSharper code templates](http://www.jetbrains.com/resharper/features/code_templates.html).

ReSharper has three flavors of code templates and in general they allow you to new-up code files or insert code from templates, automatically filling placeholders with variables either automatically or from user input. With NUnit the two I use most are file templates for creating test fixture class files and live templates for inserting new test methods.

## Test Fixture File Template

Creating test fixture classes is a common occurrence and an ideal candidate for templates, especially when I noticed I was performing the same half dozen simple steps just to get started writing new tests:

1.  add new class file to test project, rename
2.  decorate with TestFixture and add NUnit reference
3.  wrap class with ReSharper disable comments
4.  add new test method and decorate with Test

![](https://andy.azureedge.net/blog/2012-01-05-9-18-18-am-636217952061762594.png)

Aside: I find I favor the test naming convention introduced to me in [Roy Osherove’s TDD Master Class video from TekPub](http://tekpub.com/view/tdd/1): Method_Scenario_Expected(). ReSharper enjoys reminding me that it disapproves of underscored method names and an easy way to silence it is with its own disable comments: <font color="#008000">// ReSharper disable InconsistentNaming</font>. Alternatively you can [change ReSharper’s naming style for test methods](http://atombrenner.blogspot.com/2010/07/how-to-change-resharper-naming-style.html) altogether but this only applies to your machine and your team members would need to do the same.

## Creating a File Template

You can access ReSharper’s file templates via ReSharper > Templates Explorer… > File Templates [tab] and you can import the templates I’ve linked to below or add you own. In this case I have a template simply called Test Fixture:

![](https://andy.azureedge.net/blog/2012-01-07-11-26-29-am-636217952069529322.png)

When creating templates you’ll notice how $PLACEHOLDERS$ can be used to dynamically insert environmental, solution, or project variables into your code or allow you to tab, tab, tab your way through completion:

![](https://andy.azureedge.net/blog/2012-01-07-11-32-21-am-636217952077895851.png)

Finally, creating a new class file from your template is easy: you can create it from within a code file via the ReSharper [keyboard shortcut](http://www.jetbrains.com/resharper/webhelp/Reference__Keyboard_Shortcuts.html) _Ctrl+Alt+Insert_ or from the VS project context menu: Add > New from Template > Test Fixture. Bam! Too easy.

## Test Method Live Template

Live templates are born from VS’s code snippets feature but raised like Kobe beef and are equally delicious. I’ve long since replaced all my homegrown code snippets with live templates, it’s an easy exercise well-worth the effort. My live template for a Test method is quite simply:

![](https://andy.azureedge.net/blog/2012-01-07-11-42-45-am-636217952084607013.png)

![](https://andy.azureedge.net/blog/2012-01-07-11-44-32-am-636217952091141113.png)

At this point you can simply start typing “test” in any class to expose the ReSharper live templates context menu and tab your way through to generate your test method stub.

## Import Templates

ReSharper allows you to import and export templates to share with others, these are the two I use most that you’re welcome to enjoy:

*   [NUnit Test Fixture File Template](https://andy.azureedge.net/blog/nunit-file-template-636217953372931148.dotsettings)
*   [NUnit Test Method Live Template](https://andy.azureedge.net/blog/nunit-live-template-636217953395742482.dotsettings)