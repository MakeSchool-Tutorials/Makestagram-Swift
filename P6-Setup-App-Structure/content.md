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