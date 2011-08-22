# Gotcha of `viewDidLoad` and `viewDidUnload` of `UIViewController` #

## Issue ##
We are often taught to balance out the calls in programming, especially coming from a manual memory management languages like Objective-C. That is, if you load something, you should unload something; if you create something, you should destroy something; etc.

So its understandable that many had assumed that `viewDidLoad` and `viewDidUnload` in `UIViewController` balance out, but the problem is, Apple didn't balance them out as many assume.

Many people have different ways of writing `UIViewController`, for folks who use `viewDidLoad` to allocate additional views programmatically instead of `loadView`, we often dealloc them in `viewDidUnload`. This is correct except that you only score 50 points out of the 100.

Why, you ask? Whats wrong with that? You balance out the calls. Everything should be fine... Except that Apple doesn't call `viewDidUnload` upon deallocation.

I can imagine your jaw dropping and yes that means your are leaking a whole lot of views.

## Solution ##

`viewDidUnload` is called when the app received memory warning for all hidden views. It does not get call when your `UIViewController` get deallocated.

You should deallocate/nullify your views but not your application states in `viewDidUnload`. So upon its reconstruction when it is needed (e.g. when you pop a `UIViewController` off the `UINavigationController`), it can return correctly without the user noticing anything.

The following is an excerpt from the `UIViewController` Class Reference.

> ###viewDidUnload
> Called when the controller’s view is released from memory.
> 
> \- (void)viewDidUnload
> 
> ####Discussion  
> This method is called as a counterpart to the `viewDidLoad` method. It is called during low-memory conditions when the view controller needs to release its view and any objects associated with that view to free up memory. Because view controllers often store references to views and other view-related objects, you should use this method to relinquish ownership in those objects so that the memory for them can be reclaimed. You should do this only for objects that you can easily recreate later, either in your `viewDidLoad` method or from other parts of your application. You should not use this method to release user data or any other information that cannot be easily recreated.
> 
> Typically, a view controller stores references to objects using an outlet, which is a variable or property that includes the `IBOutlet` keyword and is configured using Interface Builder. A view controller may also store pointers to objects that it creates programmatically, such as in the `viewDidLoad` method. The preferred way to relinquish ownership of any object (including those in outlets) is to use the corresponding accessor method to set the value of the object to `nil`. However, if you do not have an accessor method for a given object, you may have to release the object explicitly. For more information about memory management practices, see [Memory Management Programming Guide][].
> 
> By the time this method is called, the `view` property is `nil`.
> 
> ####Special Considerations  
> If your view controller stores references to views and other custom objects, it is also responsible for relinquishing ownership of those objects safely in its `dealloc` method. If you implement this method but are building your application for iOS 2.x, your `dealloc` method should release each object but should also set the reference to that object to `nil` before calling `super`.

[Memory Management Programming Guide]: <http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html> "Memory Management Programming Guide"

This is well documented by Apple but its a gotcha that will probably caught many iOS developers who didn't read the documentation as always advocated by Apple.