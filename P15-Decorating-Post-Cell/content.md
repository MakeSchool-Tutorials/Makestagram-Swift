---
title: "Decorating the Post Table View Cell"
slug: decorating-post-table-view-cell
---

Now we'll spend some time in Interface Builder and with Auto Layout to polish up our extremely simple UI. We'll also add some more assets to our app. That will allow us to improve the look & feel throughout the next chapters.

#Adding additional resources

This step is pretty simple; we're going to add some more resources to our Asset Catalog. You've already downloaded the Art Pack for this tutorial earlier on, in case you no longer know where you've stored it, you can get it again [here](https://www.dropbox.com/s/kt9w4m94z4y3m4f/Art.zip?dl=0) (I don't blame you; my file system is pretty messy!).

<div class="action"></div>
Add new image sets for each of these resources (if you no longer know how this works, go back to the step where we set up the Tab Bar). The result should look like this (note that you don't need to set up _camera_ and _AppIcon_):

![image](add_all_resources.png)

Make sure you have added all of the resources with the correct names! Now we have some hearts and buttons that can we use to turn our Table View Cell into a nice looking interface.

When we are entirely done, we want the Table View Cell to look like this:

![image](finished_cell.png)

We won't finish the entire set up in this step - but we'll get pretty far!

#Designing the PostTableViewCell

First, we'll make the cell a little bit larger to make space for our additional UI elements.

TODO: Decide height later...

> Make it easy for people to interact with content and controls by giving each interactive element ample spacing. Give tappable controls a hit target of about 44 x 44 points.

##Adding the like button

Now let's start designing one of the core experiences of our app - the _like_ button!

<div class="action"></div>
1. Drag a Button from the object library and add it to the Table View Cell (technically the _Content View_ of the Table View Cell.)

Now we're going to configure the like button to show the heart image.

<div class="action"></div>
1. Select the Button that you just created.
2. Open the _Attributes Inspector_.
3. Set the Button Type to _Custom_. Whenever you want to create a button that just shows an image, you should choose the _Custom_ type. The _System_ type that is selected by default, tints all images in a bright blue.
4. Remove the _Title_ from the button
5. Set the image to the _Heart_ button
![image](like_button_setup2.png)

Ok, our basic button is set up. Now we need to take care of some Auto Layout!

While integrating buttons into your apps, you

Source: [Apple's Human Interface Guideline](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/LayoutandAppearance.html)
