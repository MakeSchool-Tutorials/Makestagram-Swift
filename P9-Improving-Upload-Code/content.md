---
title: "Improving the upload code and adding a Post class"
slug: improving-photo-upload-parse
---

Now it's time to move from a working solution to a good one. In the last step we have identified two issues with our current uploading code:

1. We need to resolve the warnings in the console
2. We need to store more information along with the `Post` that we're creating. Right now we are only storing the image file, but we also need to store the `user` to which the post belongs

Actually, there's a third issue. All of our uploading code is currently contained in the `TimelineViewController` class. However, that code isn't related to the timeline feature of the app. If we place all of our code directly in the `TimelineViewController` we will end up with a huge mess.

Instead of having the code in the `TimelineViewController`, the photo upload should be handled by a separate `Post` class.

Let's start by creating this separate class - then we will tackle the two issues listed above!

#Creating a custom Parse class

As you have seen, we can store information in Parse without using custom classes:

    let imageData = UIImageJPEGRepresentation(image, 0.8)
    let imageFile = PFFile(data: imageData)
    imageFile.save()

    let post = PFObject(className: "Post")
    post["imageFile"] = imageFile
    post.save()

We can simply use the `PFObject` class and specify a `className`. This way however, we don't have a good place to store the image uploading code. And if we need to modify many more of the post's properties, the code will become difficult to read.

It would be better, if you could upload a post like this:

    let post = Post()
    post.image = image
    post.uploadImage()

This way we could hide a lot of code inside of `uploadImage`.

I would recommend this approach for your own apps as well. If any of your Parse classes needs to be stored or loaded with complex code upload / download code - consider using a custom class.

So how can we create a custom `Post` class?

##Creating the skeleton for the `Post` class

The Parse framework has some specific requirements for creating custom classes. You can find the [documentation here](https://www.parse.com/docs/ios/guide#objects-subclasses); however, we will discuss all of the required steps.

The first step will be creating a source code file for our new class. We'll also create a new folder and a new Xcode group for this class.

<div class="action"></div>
Create a new _Models_ folder as child of the _Makestagram_ folder:
![image](add_models_folder.png)
Then add the folder to Xcode and create a new _Swift_ file callsed _Post.swift_. The result in Xcode should look like this:
![image](models_group.png)

We're going to fill the `Post` class with some [_boilerplate_](http://en.wikipedia.org/wiki/Boilerplate_code#In_object-oriented_programming) code.

Replace the content of _Post.swift_ with the following source code:

    import Foundation
    import Parse

    // 1
    class Post : PFObject, PFSubclassing {

      // 2
      @NSManaged var imageFile: PFFile?
      @NSManaged var user: PFUser?


      //MARK: PFSubclassing Protocol

      // 3
      static func parseClassName() -> String {
        return "Post"
      }

      // 4
      override init () {
        super.init()
      }

      override class func initialize() {
        var onceToken : dispatch_once_t = 0;
        dispatch_once(&onceToken) {
          // inform Parse about this subclass
          self.registerSubclass()
        }
      }

    }

1. To create a custom Parse class you need to inherit from `PFObject` and implement the `PFSubclassing` protocol
2. Next, define each property that you want to access on this Parse class. For our `Post` class that's the `user` and the `imageFile` of a post. That will allow you to change the code that accesses properties through strings:
       post["imageFile"] = imageFile
    Into code that uses Swift properties:
       post.imageFile = imageFile
3. By implementing the `parseClassName` you create a connection between the Parse class and your Swift class
4. `init` and `initialize` are pure boilerplate code - copy these two into any custom Parse class that you're creating

Now, we have set up the skeleton for our Parse class. Follow these steps whenever you want to create a custom Parse class in your own apps.
