---
title: "Adding a Signup and Login Screen"
slug: signup-login-parse
---

So far we have been using a placeholder login mechanism. In this step we will finally replace it with a real login and signup screen. Parse makes it fairly easy to create such a login screen by providing a default UI component that we can use in our app.

Signing up with an email address and a password is a standard feature that we'll provide; but it's a pretty old school way of joining a new app! We'll provide our users with a Facebook login option as well. That's a great way to take some friction out of the signup process.

Since this step will contain a few new concepts, we will discuss it in more detail than the last one. You'll learn how to change the initial View Controller of your app and also how to configure _Makestagram_ as a Facebook compatible application.

#Changing the Screen Flow

One of the first things we need to do, is changing the screen flow of our app. Currently the `TabBarViewController` is the main View Controller and gets presented as soon as the app starts.

Our new screen flow will depend on whether or not a user is currently signed in. If we have a signed in user we want to display the `TabBarViewController`; if no user is signed in we want to display the login / signup screen.

This will require some changes to our project settings. We can no longer simply display our Storyboard on launch, instead we need to write code that determines what the first View Controller in our app should be.

Let's start by changing a setting in the _Info.plist_ that currently defines that the app should always start by displaying the _Main.storyboard_.

> [action]
Remove the _Main storyboard file base name_ entry from the apps _Info.plist_, by selecting it as shown in the image below, and hitting the delete key:
![image](remove_main_storyboard.png)

This entry defines which storyboard should be loaded and displayed upon app start. By removing it, we have the opportunity to define the initial View Controller in code. You'll see how this works in just a second!

#Providing the Initial View Controller with an Identifier

When we want to write code to decide which View Controller should be displayed first, we need a way to reference that specific View Controller. For that purpose Storyboard provides each element with a _Storyboard ID_. By default that ID is empty. If we want to reference a Storyboard element in code we need to choose a _Stroyboard ID_. For _Makestagram_ we want to display the `TabBarViewController` as soon as a user is logged in.

Let's provide a Storyboard ID for that controller.

> [action]
> Set the Storyboard ID for the _TabBarController_ to: _TabBarController_:
>
![image](storyboard_id.png)

In many ways programming is a creative pursuit. When it comes to naming things however, you're mostly better of making the obvious yet boring choice.

We're done with preparing our configuration, we can now get down to coding!

#Adding the Code

You've heard this question before: where should we place code that runs when our app starts?
**Correct: in the `AppDelegate`**. The `AppDelegate` is the class that is mainly responsible for communicating with the iOS operating system; it is also the class that receives a message when your app has launched. That's where we need to decide whether or not we want to confront our users with a login screen.

Let's start by getting a dull task done - importing some modules.

> [action]
> Add the following import statements to _AppDelegate.swift_:
>
    import FBSDKCoreKit
    import ParseUI
