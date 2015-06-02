---
title: "Implementing a Friend Search in Parse"
slug: friend-search-follow
---

In this chapter we will add a feature that will allow users to find & follow friends. We will start by setting up the UI in Storyboard, then we'll fill in the implementation in code.

The explanations and instructions in this step will not be as detailed as in the previous ones - the View Controller we are about to build behaves very similar to the main View Controller in the notes app. Therefore it won't be necessary to discuss all of the involved concepts.

If you are incorporating any kind of search into your app, the code in this step should serve as a very good template!

#Setting up the Friend Search UI

This is what the final UI will look like, once the entire app is complete:

![image](friend_search_complete.png)

At the root level we only have two components: a Search Bar and a Table View.

##Adding a Search Bar

> [action]
> Add a Search Bar to the `FriendSearchViewController` as show in the video below:
> <video width="100%" height="400pt" controls>
  <source src="https://s3.amazonaws.com/mgwu-misc/SA2015/SearchBarConstraints_small.mov" type="video/mp4">
>
> Remember that the last step is to use the shortkey _⌘⌥=_ to update the frame according to its new constraints.

##Adding a Table View

> [action]
> Add a Table View to the `FriendSearchViewController` as shown in the video below:
> <video width="100%" height="400pt" controls>
  <source src="https://s3.amazonaws.com/mgwu-misc/SA2015/FriendSearchTableViewConstraints_small.mov" type="video/mp4">

##Adding a Custom Table View Cell

> [action]
> Add a Table View Cell to the `FriendSearchViewController` as shown in the video below:
> <video width="100%" height="400pt" controls>
  <source src="https://s3.amazonaws.com/mgwu-misc/SA2015/CustomCellFriendSearch_small.mov" type="video/mp4">

#Creating Code Connections
