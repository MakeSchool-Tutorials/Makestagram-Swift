---
title: "Uploading photos to Parse"
slug: uploading-photos-to-parse
---

Up until now we have barely interacted with the Parse SDK, besides the initial setup in our `AppDelegate`. That will change moving forward! In this step you will learn how to insert data into the Parse database, later on you will also learn how to query and download data.

In this step we will write the code that uploads our photo to Parse!

#Writing information to Parse

Which steps are involved in writing Data to Parse? In most cases it is a three step process. Here's the simplest example from the Parse Quickstart guide:

    let testObject = PFObject(className: "TestObject")
    testObject["foo"] = "bar"
    testObject.saveInBackgroundWithBlock { (success: Bool, error: NSError?) -> Void in
      println("Object has been saved.")
    }

The three steps in this code snippet are:

1. Creating a `PFObject` with a class name that matches on of our Parse classes (in our app this could be "Post", "User", etc.)
2. Set a value for a certain property of that instance using a *subscript* (the square brackets after the variable name)
3. Call one of the available `save...` methods on the instance

After the last step completes, your data is stored in the Parse database. However, this is only the simplest of all use cases.

Uploading a photo in Makestagram is a little bit more complex, but it's still only a few lines of code.

#Adding the upload code

Why is our use case a little bit more complicated than the one shown above? Primarily, because we do not only want to upload an image, but we also want to create an instance of the Parse `Post` class.

Here's a short reminder of what the `Post` class looks like:

![image](post_model.png)

You can see that the actual image file is stored _as part of the Post_. This means that our upload code needs to create a Post object that can be stored in Parse. Additionally at needs to upload the image that gets stored within that Post.

Files are handled a little different than regular objects in Parse, so we don't use the `PFObject` class to create them. Instead we used the specialized `PFFile` class.

**Try to implement this step on your own! First create a PFFile with the image data, then a PFObject for the Post. Remember that the post needs a reference to the uploaded image! Place your solution in the `PhotoHelper` callback within `TimelineViewController`**

<div class="solution"></div>
Here's one possible implementation for the callback:

    photoTakingHelper = PhotoTakingHelper(viewController: self.tabBarController!, callback: { (image: UIImage?) in
      let imageData = UIImageJPEGRepresentation(image, 0.8)
      let imageFile = PFFile(data: imageData)
      imageFile.save()

      let post = PFObject(className: "Post")
      post["imageFile"] = imageFile
      post.save()
    })

There shouldn't be too many surprises in these lines. The most interesting one is the very first one. We turn the `UIImage` into an `NSData` instance because the `PFFile` class needs an `NSData` argument for its initializer.

Then we create and `save` the `PFFile`.

In the next step we create a `PFObject` of type Post. We assign the `"imageFile"` to this Post and then save it as well.
