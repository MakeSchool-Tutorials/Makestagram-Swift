---
title: "What You Have Learned So Far"
slug: learned-so-far
---

You have come a very long way since starting of with the _Makestagram_ project!

At this point you have covered many of the most important fundamentals of iOS development. This chapter is designed to be decision point for you: you can stop the tutorial here and start working on your individual app. You can also keep on going and dive more into the design details and the fine tuning of our app. You can also pause here and come back later.

#What You Have Learned so Far

Let's take a look at what you have learned so far.

##Creating a Parse App

At the beginning of this tutorial you learned how to set up a new Parse App.

##Designing a Data Model

After setting up the basic app, we discussed the data model of _Makestagram_. We also covered the process of how you can identify the data model based on different requirements for your app.

##Creating the Data Model in Parse

You learned how to take the data model you designed and create the according classes, columns and relationships in Parse's data browsers. Remember what a _PFPointer_ is?

##Connecting to Parse

In this step you connected your iOS app to the Parse backend. You also set up a simple login mechanism.

##Setting up the App Structure

You learned how to create an App that uses a UITabBarController. You built the screen flow for the _Makestagram_ app in Storyboard.

##Implementing Photo Taking

You implemented a fairly complex feature that involved communication between different classes. You got to use callbacks and delegates - two popular tools in iOS development. This chapter should help you as a reference of understanding more complex information flow in your apps. You also learnt about the basic strategy of trying to avoid placing all of your code in View Controllers.

##Implementing Photo Upload

In this chapter you wrote your first changes to the Parse backend! You also learned how to verify that your code works by using the Parse data browser.

##Improving the Upload Code

A very important chapter of this tutorial! You learned how and why to create custom classes for Parse objects, while creating the custom `Post` class. You learned what it means to perform something in the _background_ and why that is necessary to create responsive apps.

##A Word on Security

You learned what ACLs are, why they are important and how to set up a good default configuration.

##Fetching Data from Parse and Building the Timeline

You spent some time in Storyboard, setting up the basic Table View for the _Makestagram_ timeline. Then you learned about `PFQuery`. You built a fairly complex query that fetches all the relevant posts for a users timeline.

##Creating a Custom Post Table View Cell

You learned how to create a custom Table View Cell. You also learned how to download `PFFile`s. By the end of this step you were displaying photos on a users' timeline.

##Restructuring the Query Code

You learned how you should structure your Parse query code in larger apps.

##Improving the Image Download with Bindings

An extremely important chapter! You learned about _lazy loading_ and _asynchrony_ - two core concepts for most of the apps that you will build. You also learned how to use the `Dynamic` class together with _bindings_ to deal with information that is available asynchronously. You used that knowledge to update the images displayed on a user's timeline, as soon as their download completed.

Understanding the concepts in this step is essential!

##Decorating the Post Table View Cell

In this step you spent some more time with Auto Layout and further customized the `PostTableViewCell`.

##Implementing the Like Feature

Another extremely important step - you implemented the _like_ feature. That's one of the core features of the app. In this chapter you re-iterated many known concepts, such as building parse queries and using bindings and covered them in more detail. You also learned about the useful Swift tools `filter` and `map`.

##Keeping the UI up to Date

This step essentially taught you how to build interactive features that trigger changes in your app and on the Parse server. Another essential step that you should bookmark and come back to when you start building your own app. You also learned about issues with _retain cycles_ when using closures and how object _equality_ is defined and can be redefined in Swift.

##Pull-To-Refresh and Endless Scrolling

You learned how to be more thoughtful of your user's resources, by querying the relevant subset of information to show in your app. You learned how to use the `TimelineComponent` to add pull-to-refresh and endless scrolling to your timeline.

#Where to go from here?

You have two options. You have learned enough to get started on your own app and come back if you are interested in any of the tweaks and features that we will be implementing throughout the next steps.

However, you can also continue with this tutorial and learn what goes into finishing the entire _Makestagram_ app.

The remainder of this tutorial can also be used as a reference and you can back to it after you have started working on your own app.

**It's up to you and your schedule!**
