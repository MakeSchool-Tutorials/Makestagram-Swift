---
title: "Setting up the App Structure with a UITabBarController"
slug: setup-app-structure-uitabbarcontroller
---     

In this section we're going to set up the basic structure of our app. While building the *Make School Notes* app, you have learnt about *Container View Controllers*.

Just as a short reminder: Container View Controllers are View Controllers that can present other View Controllers. Most apps have at least one such a Container View Controller. In *Make School Notes* we used a `UINavigationController` as container. It allowed us to *push* and *pop* View Controllers.

iOS provides another very popular Container View Controller: the `UITabBarController`. Using a `UINavigationController` the user typically navigates between different View Controllers by selecting items from `UITableView`s, such as Apple's *Mail* app, or as in *Make School Notes*.

Using a `UITabBarController`, the user switches between different View Controllers by selecting one of the Tab Bar Items in a Tab Bar. Below is a screenshot from the finished *Makestagram* app that contains a Tab Bar:
![image](tab_bar_example.png)

The user can switch between the timeline, taking a photo, or finding friends, by selecting the respective button from the Tab Bar. 

Many apps use multiple types of Container View Controllers in one App. We will see how that approach works when we add the image filter feature to the *Makestagram* app later on.

#The App Structure

One of the first steps, once you have collected all the features you want to be part of your app, should be drawing an outline of the different screens that your app will contain. This outline should include connections between the different screens.

For *Makestagram* I've created this overview, that breaks down the entire app into View Controllers:

![image](app_structure.png)

I highly recommend you come up with a similar diagram before writing code for your original app - it will give you a very good idea of how much effort it will take to build your app. With this overview at hand, let's dive into the action of creating a bunch of these View Controllers.

#Setting the UITabBarController up

As mentioned earlier, we will skip the login screen for now. That means that the `UITabBarController` is the first View Controller we need to add to our app. We also want the Tab Bar Controller to be the initial View Controller of our app.

The Xcode project template has a simple `UIViewController` as its initial View Controller.
Let's first go ahead and delete it.

<div class="action"></div>
Open *Main.storyboard*. Select the existing View Controller; after you have selected it you should see a blue border surrounding it: 
![image](selected_vc.png)

<div class="action"></div>
Hit the *delete* key to remove this View Controller. Now you should see a blank Storyboard:
![image](blank_storyboard.png)

Now we can add the `UITabBarController`.

<div class="action"></div>
Drag the `UITabBarController` from the component library into the Storyboard:
![image](add_tab_bar.png)

Now we have set up the Container View Controller for our app - time to run it?

If you run the app now, you will see a blank, black screen, after the splash image disappears. You will also see the following error in the console:

> Makestagram[76593:9371786] Failed to instantiate the default view controller for UIMainStoryboardFile 'Main' - perhaps the designated entry point is not set?

**Why are you seeing this error?**

Every app needs to define an *initial View Controller*. Our app doesn't have one right now - we have deleted the default View Controller, and haven't set the Tab Bar Controller as the initial one. Let's fix that.

<div class="action"></div>
Configure the Tab Bar Controller to be our app's initial View Controller. First select the *Attributes Inspector* in the right bar (1). Then check the *is Initial View Controller* checkbox (2). As a confirmation you should see an arrow pointing to the Tab Bar View Controller (3): 
![image](initial_vc.png)



