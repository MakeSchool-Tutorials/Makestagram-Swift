---
title: "Keeping the UI up to Date"
slug: pare-ui-updates-likes
---

In the last step we have completed the _like_ feature of _Makestagram_. However, users currently cannot see
which users have liked a post. And the like button won't activate, and become red when a user likes a post.

In this step we will make some more use of our _Bond_ framework, to update the UI whenever users like posts.
We have already defined the `likes` stored within each `Post` as a `Dynamic` property. As you might remember from the implementation of the lazy image loading, having a `Dynamic` property allows us to listen for changes by using a _binding_.

#Adding a binding for likes

Before we start coding, I want to remind what our first binding looked like. It was a binding between the `image` property of a `Post` and the `postImageView` in the `PostTableViewCell`:

    // bind the image of the post to the 'postImage' view
    post.image ->> postImageView

This is one of the two types of bindings that we will be using. It connects a property directly to a UI component. Whenever the property changes the UI updates with it.

Now, to update our UI based on the `likes` property of our `Post`, we will use the second type of binding. That second type simply allows us to perform an arbitrary block of code whenever a property changes. In the example above, the receiver of the binding was a `UIImageView`. Now, the receiver will be an instance of the class `Bond` that we will need to create ourselves.

We start by creating a property that will store our `Bond`.
<div class="action"></div>
Add this property to the `PostTableViewCell` class:

    var likeBond: Bond<[PFUser]?>!

Defining a `Bond` property is fairly similar to defining a `Dynamic` property - in the angled brackets we provide the type of information that we will receive through this bond. In this case it's an optional array of `PFUser`s. This array represents the list of users that have liked a certain post.

We won't dive into the details of the `!` after this property declaration for now - it has to do with some of Swift's initialization rules (if you're curious you can read a [related StackOverflow question](http://stackoverflow.com/questions/24218581/need-self-to-set-all-constants-of-a-swift-class-in-init)).

Next, we can initialize the `Bond` in an initializer of `PostTableViewCell`. As soon as we initialize the `Bond`, we need to provide a closure with all of the code that will run whenever our Bond receives a new value. We'll add the Bond now, then we'll discuss the code in detail.

<div class="action"></div>
Add the following initializer to the `PostTableViewCell` class:

    required init(coder aDecoder: NSCoder) {
      super.init(coder: aDecoder)

      // 1  
      likeBond = Bond<[PFUser]?>() { [unowned self] likeList in
        // 2
        if let likeList = likeList {
          // 3
          self.likesLabel.text = self.stringFromUserlist(likeList)
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
3. If we have received a value, we perform different updates. First of all, we update the `likesLabel` to display a list of usernames of all users that have liked the post. We use a utility method `stringFromUserlist` to generate that list. We'll add and discuss that method later on!
4. Next, we set the state of the like button (the heart) based on whether or not the current user is in the list of users that like the currently displayed post. If the user has liked the post, we want the button to be in the `Selected` state so that the heart appears red. If not `selected` will be set to `false` and the heart will be displayed in gray.
5. Finally, if no one likes the current post, we want to hide the small heart icon displayed in front of the list of users that like a post.
6. If the value we have received in `likeList` is `nil`, we set the label text to be empty, set the like button not to be selected and hide the small heart icon.

Now we have the `likeBond` set up! Next, we need to actually bind it to the `likes` property of the `Post` class. Just as with the `image` of the `Post`, we want to establish a binding as soon as our `PostTableViewCell` receives a new `Post` instance. This means we need to extend the property observer of the `post` property:

<div class="action"></div>
Extend the `didSet` property observer of the `post` property as following:

    var post:Post? {
      didSet {
        if let post = post {
          // bind the image of the post to the 'postImage' view
          post.image ->> postImageView

          // bind the likeBond that we defined earlier, to update like label and button when likes change
          post.likes ->> likeBond
        }
      }
    }

Not too much news in this change. We use the `->>` operator to bind the `likes` property of `post` to our `likeBond`.

There's a last step required before we can test our new binding: adding the `stringFromUserlist` method!

<div class="action"></div>
Add the following method to the `PostTableViewCell`:

    // Generates a comma seperated list of usernames from an array (e.g. "User1, User2")
    func stringFromUserlist(userList: [PFUser]) -> String {
      let usernameList = userList.map { user in user.username! }
      var commaSeperatedUserList = ", ".join(usernameList)

      return commaSeperatedUserList
    }


#Conclusion

`unowened`
