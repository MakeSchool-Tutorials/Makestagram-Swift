---
title: "Uploading photos"
slug: uploading-photos
---     

It's time to implement our very first feature: uploading Photos to Parse! In our App we want to allow the users to upload or take a photo as soon as they tap the camera button in the middle of the Tab Bar.

Unforunately, using a Tab Bar View Controller, we cannot perform an arbitrary method when one of the Tab Bar Items is selected. All we can do is define which View Controller should be connected to which Tab Bar Item. 

However, as for many features you want to implement in an iOS App, there's a **workaround**.

