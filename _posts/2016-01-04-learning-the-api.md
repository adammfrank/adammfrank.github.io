---
layout: post
title:  "Learning the Plugin API"
date:   2016-01-04 13:39:36 -0500
categories: blog updates
---

In this series of posts, I'm documenting my project to build a "find-file" IntelliJ plugin.  The goal is to create a plugin which provides the same functionality as the built in "find-file" function in Emacs ("c-x c-f").
After setting up the development environment, I went through the tutorials on plugins.  They don't say anything about how to add UI components, so I decided to go into the source code for the built in c-x c-f action, to see how it adds a UI component, hoping that the same logic will apply for a plugin.
I have no idea where to start in the code, so I go to Grep.  I search recursively for the action name "file...", which leads me to an xml file that doesn't seem  useful.  At this point, I realize that I don't have enough background knowledge to start figuring out where to look, so I go back to Google.  A search for "built in actions intellij source code", leads me to this page, http://keithlea.com/idea-actions/.  It shows me that the ID of that action is called "GotoFile".  Back to grep!  Grep for "GotoFile" finds me "GotoFileAction.java".  I Use the c-x c-f, I'm trying to replace to go there directly.

Inside FileAction.java

FileAction extends GoToActionBase, which itself extends AnAction.  I start by reading the GoToAction class file to see what goToActionPerformed does.  It looks like that calls actionPerformed of the subclass assuming everything in the class is valid.  Next I find out what GoToFileModel does.  AcceptItem looks important, it's super method looks like it just checks to make sure the item being selected exists in the filtered list of options.  GotoFileModel checks if the file is a PsiFile.  I don't know what that is so, I do more Googling.

According to [this page] PsiFile stands for Program Structure Interface File. A PSI (Program Structure Interface) file is the root of a structure representing the contents of a file as a hierarchy of elements in a particular programming language.  It sounds like I might need to use those, so I will probably need to use GotoFileModel's conversion to Virtual File in my own AnAction.
Back in GotoFileAction, GotoFileItemProvider, is where the file corresponding to the user supplied path is actually found from the file system.  It parses the string and looks for the file or directory.
"showNavigationPopup" is a method of GotoActionBase, which instantiates the actual Swing component of the modal and text boxes.
I have some vague idea of what is going on now, so how about I try to create something.  I'll start simple: Create an action in the navigate group that creates a popup.

[Here], I see that there are several ways to create a popup in the API.  My popup is going to need multiple components (a text input, and a list), so it sounds like I should use the createComponentPopupBuilder.  That takes a JComponent as a parameter, which I think is just a Swing object of some sort, which will be the content of the popup.  According to java docs, a JComponent is "The base class for all Swing components except top-level containers".  I use a JPanel to which I've attached a JTextInput.
[javacodegeeks.com] has some really helpful examples of how to create a popup as well as how to write plugins in general.
Popups use the Factory design pattern, which I have heard of, but have never used before.  To fix this, I do some more googling.
From [tutorialspoint.com], I learn that a Factory is an interface that contains methods for creating concrete classes.  Just like the example has "getShape", JBPopupFactory has "getInstance".  Instead of taking the desired class as a parameter, JBPopupFactory has different methods for each type of popup.  

Here is my first, original code for this project:

{% highlight java %}
JTextField myTextField = new JTextField("Here I am", 20);
JComponent myPanel = new JPanel();
myPanel.add(myTextField);
JBPopupFactory.getInstance().createComponentPopupBuilder(myPanel, myTextField).createPopup().show(myPanel);
{% endhighlight %}

[this page]: http://www.jetbrains.org/intellij/sdk/docs/basics/architectural_overview/psi_files.html
[Here]: http://www.jetbrains.org/intellij/sdk/docs/user_interface_components/popups.html
[javacodegeeks.com]: http://www.javacodegeeks.com/2012/07/developing-plugin-for-intellij-idea.html
[tutorialspoint.com]: http://www.tutorialspoint.com/design_pattern/factory_pattern.htm