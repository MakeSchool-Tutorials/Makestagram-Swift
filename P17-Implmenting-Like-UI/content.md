---
title: "Keeping the UI up to Date"
slug: pare-ui-updates-likes
---

In the last step we completed the _like_ feature of _Makestagram_; however, users currently cannot see
which users have liked a post, and the like button won't activate (i.e. turn red) when a user likes a post.

In this step we will once again make use of the _Bond_ framework - this time we will use it to update the UI whenever a user likes a post.
We have already defined the `likes` stored within each `Post` as an `Observable` property. As you might remember from the implementation of the lazy image loading, having an `Observable` property allows us to listen for changes by using a _binding_.

#Adding a Binding for Likes

Before we start coding, I want to remind you what our first binding looked like. It was a binding between the `image` property of a `Post` and the`image` property of a `postImageView` in the `PostTableViewCell`:

    // bind the image of the post to the 'postImage' view
    post.image.bindTo(postImageView.bnd_image)

This is one of the two types of bindings that we will be using. It connects a property directly to a UI component. Whenever the property changes, the UI updates with it.

Now to update our UI based on the `likes` property of our `Post`, we will use the second type of binding. That second type simply allows us to perform an arbitrary block of code whenever a property changes.

We'll add the observer now, then we'll discuss the code in detail.

> [action]
Extend the `didSet` property observer of the `post` property as following:>
>
    var post: Post? {
      didSet {
        if let post = post {
          post.image.bindTo(postImageView.bnd_image)
          // 1
          post.likes.observe { (value: [PFUser]?) -> () in
            // 2
            if let value = value {
              // 3
              self.likesLabel.text = self.stringFromUserList(value)
              // 4
              self.likeButton.selected = value.contains(PFUser.currentUser()!)
              // 5
              self.likesIconImageView.hidden = (value.count == 0)
            } else {
              // 6
              self.likesLabel.text = ""
              self.likeButton.selected = false
              self.likesIconImageView.hidden = true
            }
          }
        }
      }
    }

1. The `observe` method is provided by the *Bond* framework and can be called on any object wrapped in the `Observable` type. The `observe` method takes one parameter, a closure (defined as a *trailing closure* in the code above), which in our case has type `[PFUser]? -> ()`. The code defined by the closure will be executed whenever `post.image` receives a new value. The constant named `value` in the closure definition will contain the actual contents of `post.likes`, that is, the `Observable` wrapper will have been removed.
2. Because `post.likes` contains an *optional* array of `PFUser`s, we use optional binding to ensure that `value` is not `nil`.
3. If we have received a value, we perform different updates. First of all we update the `likesLabel` to display a list of usernames of all users that have liked the post. We use a utility method `stringFromUserList` to generate that list. We'll add and discuss that method later on!
4. Next we set the state of the like button (the heart) based on whether or not the current user is in the list of users that like the currently displayed post. If the user has liked the post, we want the button to be in the `Selected` state so that the heart appears red. If not `selected` will be set to `false` and the heart will be displayed in gray.
5. Finally, if no one likes the current post, we want to hide the small heart icon displayed in front of the list of users that like a post.
6. If we haven't received a value yet, set all UI elements to default values.

There's a last step required before we can test our new binding: adding the `stringFromUserList` method!

> [action]
Add the following method to the `PostTableViewCell`:
>
    // Generates a comma separated list of usernames from an array (e.g. "User1, User2")
    func stringFromUserList(userList: [PFUser]) -> String {
      // 1
      let usernameList = userList.map { user in user.username! }
      // 2
      let commaSeparatedUserList = usernameList.joinWithSeparator(", ")
>
      return commaSeparatedUserList
    }

1. You have already seen and used `map` before. As we discussed it allows you to replace objects in a collection with other objects. Typically you use `map` to create a different representation of the same _thing_. In this case we are mapping from `PFUser` objects to the `username`s of these `PFObjects`.
2. We now use that array of strings to create one joint string. We can do that by using the `joinWithSeparator` method provided by Swift. The `joinWithSeparator` method can be called on any array of strings. After the method is called, we have created a string of the following form: _"Test User 1, Test User 2"_.

Time to see this in action!

You should be able to hit the like button on a post and see the UI updating correctly:

![image](like_working.png)

Awesome! Even after restarting the app you should see the like status being nicely restored in the UI:

![image](like_update_bug.png)

**Uh, what's that?** The heart isn't displayed in red, even though we have liked this post before; however, the _test_ username shows up in the list of users that liked the post. And if you hit the like button, you get to like the same post a second time.

**What's happening here? Spend a few minutes and try to debug this issue on your own.**

#When Equal Isn't Equal

If you had any luck with debugging this issue, you will have noticed that the issue occurs in the line where we set the `selected` property of the like button:

    self.likeButton.selected = value.contains(PFUser.currentUser()!)

The `contains` function is supposed to check whether or not an object is contained in a provided list; however, this line doesn't always seem to work correctly. Whenever we restart our app, this line returns `false`, even though the current user has definitely liked the post before.

The underlying problem is the different ways of how we can define equality and identity in computer programs. By default, when working with `PFObject`s, two variables are equal when they are referencing *exactly the same* object. However, in our app, we can have multiple _different_ `PFUser` objects that actually represent the same user on the Parse server!

Every time we retrieve a post or a user through a `PFQuery`, a new object is created. That new object is not the same as objects that we have received from previous queries - even though some of them reference exactly the same objects on the server.

For our Parse app we want to consider objects _equal_ whenever they reference the same object on the server. This means two objects are equal when they have the same `objectID`.

Luckily, Swift provides us with a way to change the definition of equality for different classes. We can do that by implementing the `Equatable` protocol on a class and adding an implementation of the `==` operator.

> [action]
Add the following extension to the end of the _ParseHelper.swift_ class (outside of the class definition of `ParseHelper`):
>
    extension PFObject {
>
      public override func isEqual(object: AnyObject?) -> Bool {
        if (object as? PFObject)?.objectId == self.objectId {
          return true
        } else {
          return super.isEqual(object)
        }
      }
>
    }

Now Swift knows to consider any two Parse objects equal if they have the same `objectId`.

You can run the app again and you should see that the issue shown earlier no longer exists! Our app now always correctly detects whether or not the current user has liked a post.

#Conclusion

This was an extremely important step: we have finished the entire _like_ feature of **Makestagram**. By now you have learned how to load information lazily and how to update server information based on user input.

In this chapter you have learned how to use a second type of _binding_. One where you *observe* changes to an object and provide a closure that contains code that runs whenever the object receives a new value. This allowed us to update our UI based on the likes a post has received. Bindings will be an extremely useful tool for your own app, so remember to come back to this chapter if you have forgotten how to use them.

<!-- As part of setting up the binding, we have also briefly spoken about _strong reference cycles_ - and how to avoid them. You have learned what the word _capture list_ means and how the `unowned` keyword can break potential retain cycles. Make sure to take a note of that section as well, it will also be very important for your own app. -->

Lastly, you have learned what object equality and identity mean. We have changed the default behavior of how Swift compares Parse objects to avoid bugs in **Makestagram**.

In the next step we will improve our timeline a little bit by allowing users to manually refresh their timeline! And, no need to worry, soon we will also take care of adding some more users to our app and making it less lonely.
