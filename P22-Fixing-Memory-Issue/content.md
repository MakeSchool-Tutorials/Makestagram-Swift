---
title: "Fixing a Memory Issue"
slug: fixing-a-memory-issue
---

Throughout this tutorial we won't dive too deep into detecting and fixing memory issues. However, this step will discuss a specific issue of our _Makestagram_ app - that should make you aware of potential problems in your own app.

#What are Memory Warnings?

If you spent some time using the current version of _Makestagram_ and upload a large amount of photos you will likely see a _memory warning_ show up in the Xcode console. Potentially your app will even crash. Despite the fact that todays iPhones are powerful devices with lots of memory and processing power, we still have limited resources to work with.

Users of today's mobile devices expect apps to be extremely responsive and the mobile operating systems help ensure that by dividing the available resources efficiently between the different apps running on a phone. **What does this mean for you?**
If your app is too greedy in its memory consumption, iOS will terminate it to avoid that your app causes the entire phone to slow down.

If you see a memory warning in the console or if your app even crashes due to a memory warning, it's time to act.

#What Are Typical Causes of Memory Warnings?

Most apps run into memory problems when objects are _retained_ too long. This means you have objects or resources in your program, that stay around even though you no longer need access to them. Further, most memory problems are caused by large resources such as videos or images.

#What's the Problem in Makestagram?

Since we won't discuss the debugging of memory issues as part of this tutorial, I'll provide the answer for you. However, first I'd like to give you a chance to guess which part of our code could be responsible for our memory issues.

> [solution]
> It is the code that is handling the images files! Currently, images are stored together with `Posts`. These images are stored as long as the post images stick around. If we load 100 posts for our timeline and a user scrolls through all posts, we have 100 images stored in memory - that's a pretty big amount! Ideally we only want to keep the images in memory that are actually displayed.

#How Can We Fix the Problem?

We can discard the image data that is stored in the `image` property of a `Post`, when the `Post` is no longer displayed. **Where can we find out about posts that are no longer displayed?** Inside of the `PostTableViewCell`!

Whenever the `PostTableViewCell` receives a new post that should be displayed, we can check if the image of the old post needs to be retained or if we can free that memory up.

Let's add the code and then discuss it in detail.

> [action]
> Update the `didSet` observer of the `post` property in the `PostTableViewCell` class to look as following:
>
    var post:Post? {
      didSet {
        // free memory of image stored with post that is no longer displayed
        // 1
        if let oldValue = oldValue where oldValue != post {
          // 2
          likeBond.unbindAll()
          postImageView.designatedBond.unbindAll()
          // 3
          if (oldValue.image.bonds.count == 0) {
            oldValue.image.value = nil
          }
        }
>
        if let post = post {
          // bind the image of the post to the 'postImage' view
          post.image ->> postImageView
>
          // bind the likeBond that we defined earlier, to update like label and button when likes change
          post.likes ->> likeBond
        }
      }
    }

1. The `oldValue` variable is available automatically in the `didSet` property observer. It provides us with a way to access the previous value of a property. We check if an `oldValue` exists and if that `oldValue` is different from the new `post`. If that's the case, we know that we need to do some cleanup.
2. By adding the calls to `unbindAll()` we are secretly fixing an issue that we haven't even discussed yet. Without this code in place, we are adding a new binding whenever a new post gets assigned to our `PostTableViewCell`. I most cases where you create a binding, you should also have code that destroys that binding when it is no longer needed. In case of the `PostTableViewCell` we don't need the binding anymore if the cell is displaying a new post. By calling `unbindAll` on the `likeBond`, we unsubscribe from future updates of the old post. We do the same for `postImageView.designatedBond` - that's the bond that updates the `postImageView` when the image of the current post is loaded successfully. It is called `designatedBond` because it exists by default, without that we have to explicitly create it (like we had to with the `likeBond`). The `Bond` framework adds `designatedBond`s to most UI components such as `UIImageView` and `UITextField`.
3. After we have unbound from the old post, we check if the image of the old post has any bindings left. If `oldValue.image.bonds.count` is _0_, we know that no one is binding to the image of the old post anymore. This means we can free up the memory by setting the `image.value` to `nil`.

Great! Whenever an image is no longer displayed, we are freeing up the memory. That means we'll only have about 3 images in memory at any point in time. Our memory issue is fixed!

#Won't This Make Our App Slower?
