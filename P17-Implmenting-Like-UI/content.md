---
title: "Keeping the UI up to Date"
slug: pare-ui-updates-likes
---

In the last step we have completed the _like_ feature of _Makestagram_. However, users currently cannot see
which users have liked a post. And the like button won't activate, and become red when a user likes a post.

In this step we will make some more use of our _Bond_ framework, to update the UI whenever users like posts.
We have already defined the `likes` stored within each `Post` as a `Dynamic` property. As you might remember from the implementation of the lazy image loading, having a `Dynamic` property allows us to listen for changes by using a _binding_.

#Adding a Binding for Likes

Before we start coding, I want to remind what our first binding looked like. It was a binding between the `image` property of a `Post` and the `postImageView` in the `PostTableViewCell`:

    // bind the image of the post to the 'postImage' view
    post.image ->> postImageView

This is one of the two types of bindings that we will be using. It connects a property directly to a UI component. Whenever the property changes the UI updates with it.

Now, to update our UI based on the `likes` property of our `Post`, we will use the second type of binding. That second type simply allows us to perform an arbitrary block of code whenever a property changes. In the example above, the receiver of the binding was a `UIImageView`. Now, the receiver will be an instance of the class `Bond` that we will need to create ourselves.

We start by creating a property that will store our `Bond`.

> [action]
Add this property to the `PostTableViewCell` class:
>
    var likeBond: Bond<[PFUser]?>!

Defining a `Bond` property is fairly similar to defining a `Dynamic` property - in the angled brackets we provide the type of information that we will receive through this bond. In this case it's an optional array of `PFUser`s. This array represents the list of users that have liked a certain post.

We won't dive into the details of the `!` after this property declaration for now - it has to do with some of Swift's initialization rules (if you're curious you can read a [related StackOverflow question](http://stackoverflow.com/questions/24218581/need-self-to-set-all-constants-of-a-swift-class-in-init)).

Next, we can initialize the `Bond` in an initializer of `PostTableViewCell`. As soon as we initialize the `Bond`, we need to provide a closure with all of the code that will run whenever our Bond receives a new value. We'll add the Bond now, then we'll discuss the code in detail.

> [action]
Add the following initializer to the `PostTableViewCell` class:
>
    required init(coder aDecoder: NSCoder) {
      super.init(coder: aDecoder)
>
      // 1  
      likeBond = Bond<[PFUser]?>() { [unowned self] likeList in
        // 2
        if let likeList = likeList {
          // 3
          self.likesLabel.text = self.stringFromUserList(likeList)
          // 4
          self.likeButton.selected = contains(likeList, PFUser.currentUser()!)
          // 5
          self.likesIconImageView.hidden = (likeList.count == 0)
        } else {
          // 6
          // if there is no list of users that like this post, reset everything
          self.likesLabel.text = ""
          self.likeButton.selected = false
          self.likesIconImageView.hidden = true
        }
      }
    }

1. We create a new `Bond`. That `Bond` has exactly the same type (`[PFUser]?`) as the `likeBond` property that we declared. The Bond takes a _trailing closure_ when it is initialized. That closure contains the code that will run whenever the `Bond` receives a new value. The Bond receives the list of users that have liked a post in the `likeList` parameter. There's something new hidden in this line:`[unowned self]`. The list in square brackets is called a _capture list_. We need the capture list to avoid _retain cycles_. Since `PostTableViewCell` is creating and storing this `Bond`, it has a _strong_ reference to it. Because we are accessing `self` from within the Bond's closure, the Bond would also have a _strong_ reference to the `PostTableViewCell`. This way we would have _retain cycle_ in which two objects reference each other strongly. You can [read in detail about this issue in Apple's Documentation](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID48) in the chapter _Strong Reference Cycles Between Class Instances_. The `unowned` keyword works similar to the `weak` keyword - we store a reference to `self`, but it isn't a strong reference that would keep the object in memory.
2. As a reminder: this code runs as soon as the value of `likes` on a `Post` changes. First, we check whether we have received a value for `likeList` or if we have received `nil`.
3. If we have received a value, we perform different updates. First of all, we update the `likesLabel` to display a list of usernames of all users that have liked the post. We use a utility method `stringFromUserList` to generate that list. We'll add and discuss that method later on!
4. Next, we set the state of the like button (the heart) based on whether or not the current user is in the list of users that like the currently displayed post. If the user has liked the post, we want the button to be in the `Selected` state so that the heart appears red. If not `selected` will be set to `false` and the heart will be displayed in gray.
5. Finally, if no one likes the current post, we want to hide the small heart icon displayed in front of the list of users that like a post.
6. If the value we have received in `likeList` is `nil`, we set the label text to be empty, set the like button not to be selected and hide the small heart icon.

Now we have the `likeBond` set up! Next, we need to actually bind it to the `likes` property of the `Post` class. Just as with the `image` of the `Post`, we want to establish a binding as soon as our `PostTableViewCell` receives a new `Post` instance. This means we need to extend the property observer of the `post` property:

> [action]
Extend the `didSet` property observer of the `post` property as following:
>
    var post:Post? {
      didSet {
        if let post = post {
          // bind the image of the post to the 'postImage' view
          post.image ->> postImageView
>
          // bind the likeBond that we defined earlier, to update like label and button when likes change
          post.likes ->> likeBond
        }
      }
    }

Not too much news in this change. We use the `->>` operator to bind the `likes` property of `post` to our `likeBond`.

There's a last step required before we can test our new binding: adding the `stringFromUserList` method!

> [action]
Add the following method to the `PostTableViewCell`:
>
    // Generates a comma separated list of usernames from an array (e.g. "User1, User2")
    func stringFromUserList(userList: [PFUser]) -> String {
      // 1
      let usernameList = userList.map { user in user.username! }
      // 2
      let commaSeparatedUserList = ", ".join(usernameList)
>
      return commaSeparatedUserList
    }

1. You have already seen and used `map` before. As we discussed it allows you to replace objects in a collection with other objects. Typically you use `map` to create a different representation of the same _thing_. In this case we are mapping from `PFUser` objects to the `username`s of these `PFObjects`.
2. We now use that array of strings to create one joint string. We can do that by using the `join` method provided by Swift. We first need to define the delimiter (_", "_ in our case) and can the call the `join` method on it. The `join` method takes an array of strings. After this method is called, we have created a string of the following form: _"Test User 1, Test User 2"_.

Time to see this in action!

You should be able to hit the like button on a post and see the UI updating correctly:

![image](like_working.png)

Awesome! You can even close the app, restart it again and you should see the like status being nicely restored in the UI:

![image](like_update_bug.png)

**Uh, what's that?** The heart isn't displayed in red, even though we have liked this post before! However, the _test_ username shows up in the list of users that liked the post. And if you hit the like button, you get to like the same post a second time.

**What's happening here? Spend a few minutes and try to debug this issue on your own.**

#When Equal Isn't Equal

If you had luck with debugging this issue, you will have notice that the issue occurs in the line where we set the `selected` property of the like button. This is the evil line, it's inside of the `likeBond`:

    self.likeButton.selected = contains(likeList, PFUser.currentUser()!)

The `contains` function is supposed to check whether or not an object is contained in a provided list. However, this line doesn't always seem to work correctly. Whenever we restart our app, this line returns `false`, even though the current user has definitely liked the post before.

The underlying problem is the different ways of how we can define equality and identity in computer programs. By default, when working with `PFObject`s, two variables are equal when they are referencing *exactly the same* object. However, in our app, we can have multiple _different_ `PFUser` objects that actually represent the same user on the Parse server!

Every time we retrieve a post or a user through a `PFQuery`, a new object is created. That new object is not the same as objects that we have received from previous queries - even though some of them reference exactly the same objects on the server.

For our Parse app we want to consider objects _equal_ whenever they reference the same object on the server. This means two objects are equal when they have the same `objectID`.

Luckily, Swift provides us with a way to change the definition of equality for different classes. We can do that by implementing the `Equatable` protocol on a class and adding an implementation of the `==` operator.

> [action]
Add the following extension to the end of the _ParseHelper.swift_ class (outside of the class definition of `ParseHelper`):
>
    extension PFObject : Equatable {
>
    }
>
    public func ==(lhs: PFObject, rhs: PFObject) -> Bool {
      return lhs.objectId == rhs.objectId
    }

Now Swift knows to consider any two Parse objects equal if they have the same `objectId`.

You can run the app again and you should see that the issue shown earlier no longer exists! Our app now always correctly detects whether or not the current user has liked a post.

#Conclusion

This was an extremely important step: we have finished the entire _like_ feature of _Makestagram_. By now you have learned how to load information lazily and how to update server information based on user input.

In this chapter you have learned how to use a second type of _binding_. One where you create a _Bond_ object and provide a closure that contains code that runs whenever the Bond receives a new value. We have used that Bond to update our UI based on the likes a post has received. Bindings will be an extremely useful tool for your own app, so remember to come back to this chapter if you have forgotten how to use them.

As part of setting up the binding, we have also briefly spoken about _strong reference cycles_ - and how to avoid them. You have learned what the word _capture list_ means and how the `unowned` keyword can break potential retain cycles. Make sure to take a note of that section as well, it will also be very important for your own app.

Lastly, you have learned what object equality and identity mean. We have changed the default behavior of how Swift compares Parse objects to avoid bugs in _Makestagram_.

In the next step, we will improve our timeline a little bit. We'll allow users to manually refresh their timeline! And, no need to worry, soon we will also take care of adding some more users to our app and making it less lonely.
