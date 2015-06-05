---
title: "Adding Basic Error Handling"
slug: error-handling
---

Error Handling is not the most exciting topic, but definitely an important one! It also is anything but trivial. Imagine, a user liked a photo, you try to send this information to Parse and as a response you receive an error. What should you do? How do you inform the user? Should you mark the post as liked or unliked? Each possible error will pose such questions - answering them is well beyond the scope of this tutorial, especially since most errors will be fairly specific to your own application.

However, this chapter shall provide us with some guidelines for error handing, that should help you when building your own app. There are two different ways to respond to an error:

1. Generic. We perform some default action, independent of the type of error
2. Specific. We react to a specific subset of errors and handle them with specific code

In any case, you should at least do the _1._ step. Then you should identify parts of your app that are especially critical and apply step _2._ to these.

#How Do We Find out If an Error Occurred?

The most common source of errors will be anything that involves network communication, e.g. whenever you fetch information from or write information to Parse. Most APIs, including Parse's, indicate that an error could occur by passing an `NSError?` argument to all callback functions.

If the optional `NSError` contains a value, an error occurred. If it contains `nil`, all is well.

#Generic Error Handling

As part of the template project we have already provided you with a generic error handing method: `ErrorHanding.defaultErrorHandler(_:)`. Let's take a look at what that method looks like:

    static let ErrorTitle           = "Error"
    static let ErrorOKButtonTitle   = "Ok"
    static let ErrorDefaultMessage  = "Something unexpected happened, sorry for that!"

    /**
      This default error handler presents an Alert View on the topmost View Controller
    */
    static func defaultErrorHandler(error: NSError) {
      var alert = UIAlertController(title: ErrorTitle, message: ErrorDefaultMessage, preferredStyle: UIAlertControllerStyle.Alert)
      alert.addAction(UIAlertAction(title: ErrorOKButtonTitle, style: UIAlertActionStyle.Default, handler: nil))

      let window = UIApplication.sharedApplication().windows[0] as! UIWindow
      window.rootViewController?.presentViewControllerFromTopViewController(alert, animated: true, completion: nil)
    }

All this method does, is create `UIAlertController` to display a popup message. The popup doesn't provide the user with specific information about the issue, instead it states: _"Something unexpected happened, sorry for that!"_.

This isn't example of great error handling, but it is definitely better than nothing! In your own app you want to add such a general error handling method to which you can fall back, if you don't have time to provide code to handle the error specifically.

When you add such a generic error handler to your app, you should also log any errors that occur, using some _analytics_ frameworks, such as [Mixpanel](https://mixpanel.com/). Logging the error using an analytics framework will make all the error messages that users encounter available to you. That will make it easier to fix issues in subsequent releases.

So how can we use this error handling method in _Makestagram_? For _Makestagram_ we won't do any specific error handling, that means we should at least handle all potential errors by calling the `defaultErrorHandler`.

This means you should find all requests in the _Makestagram_ app that have an `error` parameter and call the `defaultErrorHandler` in case that parameter is not `nil`. Here's an example of how to extend the `unlikePost` method of `ParseHelper` with generic error handling:

    static func unlikePost(user: PFUser, post: Post) {
      let query = PFQuery(className: ParseLikeClass)
      query.whereKey(ParseLikeFromUser, equalTo: user)
      query.whereKey(ParseLikeToPost, equalTo: post)

      query.findObjectsInBackgroundWithBlock {
        (results: [AnyObject]?, error: NSError?) -> Void in
          if let error = error {
            ErrorHandling.defaultErrorHandler(error)
          }

          if let results = results as? [PFObject] {
            for likes in results {
              likes.deleteInBackgroundWithBlock(nil)
            }
          }
      }
    }

> [action]
> Go ahead and improve the _Makestagram_ app by adding this generic error handler to all operations that receive `error` arguments.

When you're done, we're going to tackle a second category of errors that we aren't catching currently. We are making a few Parse requests where we aren't passing a completion block. Here's an example from the `unlikePost` method in the `ParseHelper` class:

    if let results = results as? [PFObject] {
      for likes in results {
        likes.deleteInBackgroundWithBlock(nil)
      }
    }

We are passing `nil` to the callback block, because we don't want to perform any code when the deletion of the object is completed. However, this also means that we won't be informed about potential errors.

For this case, the template project provides an `errorHandlingCallback` as part of the _ErrorHandling.swift_ file. If we aren't interested in the result of a Parse operation, we should pass on that `errorHandlingCallback` instead of `nil`, so that we will at least be informed about an error.

Here's what the `errorHandlingCallback` looks like:

    /**
      A PFBooleanResult callback block that only handles error cases. You can pass this to completion blocks of Parse Requests
    */
    static func errorHandlingCallback(success: Bool, error: NSError?) -> Void {
      if let error = error {
        ErrorHandling.defaultErrorHandler(error)
      }
    }

Using that callback we can improve the code from the `unlikePost` method as following:

    if let results = results as? [PFObject] {
      for likes in results {
        likes.deleteInBackgroundWithBlock(ErrorHandling.errorHandlingCallback)
      }
    }

Now we are passing the `errorHandlingCallback` instead of `nil` as a completion block, and will be informed about potential errors.

> [action]
> Improve the error handling in _Makestagram_ further, by replacing all `nil` callbacks with the `errorHandlingCallback`.

Now _Makestagram_ fulfills the minimum requirements for error handling. If we add analytics code to our error handler we will be notified about almost any error that can occur in our app!

In an ideal world you want to handle all errors specifically, realistically however this can be very time consuming. Therefore you should spend some time finding the critical areas of your app and add specific error handling code there.

#Specific Error Handling

What do we mean by _specific_ error handling? Each type of error has a different meaning for the state of your application.

If, for example, a like in the _Makestagram_ app fails to be synchronized with the server, you might want to inform the user about this and deactivate the like button as a response to the error. This is a specific reaction to a special type of error.

When building your app, think about especially critical features and equip them with special error handling code.

#Conclusion

In this step you've learned how to capture errors in your app. You should respond to all errors - at minimum with a generic response that logs the error to some analytics platforms and informs the user that an error occurred.

For individual core features you should think about common error cases and write custom error handling code to recover from these errors as good as possible.

The core features of the app are complete! The remainder of this tutorial will focus on polishing the UI of _Makestagram_.
