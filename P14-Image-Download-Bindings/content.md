---
title: "Improving the Image Download with Bindings"
slug: parse-image-download-bindings
---

In this step you will learn about an awesome new concept called _bindings_ - it's an important one that we'll be using often throughout the rest of the tutorial.

We will use bindings to improve our photo downloading code. Currently it has three issues:

1. We are downloading all of the photos on the main thread - that blocks the UI of our app!
2. The image download is happening inside of the `TimelineViewController`, that's not necessarily where it belongs.
3. We are downloading the photos for all posts upfront - if we had 50 posts in the timeline we would download 50 photos right away; many which our users would likely not see (did you know that the average app usage time is just a little bit above one minute?)

We're going to fix all of these three issues throughout this step:

1. We will perform the download of photos in the background
2. We will move the photo download code into the `Post` class
3. We will only download photos of posts that are currently displayed

#Updating the Post Image Asynchronously

Before we dive into coding I want to discuss some of the concepts that we will use throughout this step. While implementing the changes outlined above, we will run into an interesting situation.

The way our timeline currently works, we wait until all data is available, then we create a Table View Cell that displays that data.

In future we want to download the photos lazily and on a background thread. That will lead to a situation as shown below:
![image](image_download.png)

First, we perform the timeline query. That query downloads all posts for a user's timeline. These posts contain metadata (e.g. which user created the post, when it was created, etc.). As soon as this metadata is available, we create the refresh our Table and create all of the Table View Cells. At that point however, none of the images are loaded yet.

As soon as a cell appears on screen, we start the download of the image. That's the lazy part in _lazy loading_ - we don't load the information until we need it. That download will take a little time. Once the download completes, we want to update the Table View Cell so that the downloaded photo gets displayed.

This is what we call an _asynchronous_ operation. Instead of having all the information we need available right now, we are getting some time in the future. As soon as that information is available, we want to perform a certain operation - in this case updating the image of the Table View Cell as soon as the photo is downloaded.

There are many different ways how we can deal with data that is available asynchronously. In this tutorial we will be using _bindings_. Using a Swift library called [Bond](https://github.com/SwiftBond/Bond) we can handle asynchronous data as following:

    // bind the image of the post to the 'postImage' view
    post.image ->> postImageView

The `->>` operator might look a little bit obscure, but what it does is pretty straightforward. Whenever the value on the left-hand side changes, the value on the right-hand side is updated. In this specific line we define that the `postImageView` shall be updated whenever the `image` property of a `post` changes.
