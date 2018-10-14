---
layout: post
title: "iOS Development, Keyboard Dismiss"
date: 2016-03-25 12:04:33
comments: true
description: "iOS Development, Keyboard Dismiss"
keywords: "Keyboard"
categories: Development

tags:
- iOS
- Object-C
- Keyboard
---

In iOS development, there is one common problem. The keyboard didn't dismiss when user finished editing work in the text field or input field. Even when user pressed some button and went to the next step, the annoying keyboard was still there. Even worse is that sometimes user couldn't tap the button, which was fully covered by the keyboard.

Therefore, I found some solutions for this issue, from some on-line sources: [Easy way to dismiss keyboard?](http://stackoverflow.com/questions/741185/easy-way-to-dismiss-keyboard), [Managing the Keyboard](https://developer.apple.com/library/ios/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/KeyboardManagement/KeyboardManagement.html).

### 1. Keyboard dismisses touching view using `touchesBegan`
The easiest way to handle this issue. Just need to add following method to `ViewController.m`.

{% highlight objc %}
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    [[self view] endEditing:YES];
}
{% endhighlight %}

or using `resignFirstResponder`,

{% highlight objc %}
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    [self.someTextField resignFirstResponder];
}
{% endhighlight %}

`UIViewController` inherits `UIResponder`, which receives and handles events. There `UIViewController` can listen and respond to the `touchesBegan` touching gesture. In this way, the keyboard dismisses when user touches anywhere in the background view (UI objects won't trigger, like UIButton).

Differences between the two aforementioned solutions, [resignFirstResponder vs. endEditing for Keyboard Dismissal](http://stackoverflow.com/questions/29882775/resignfirstresponder-vs-endediting-for-keyboard-dismissal):

> `resignFirstResponder()` is good to use any time you know exactly which text field is the first responder and you want to resign its first responder status.

> `endEditing()` will look through the entire hierarchy of subviews and make sure whatever is the first responder resigns its status. This makes it less efficient then calling `resignFirstResponder()` if you already have a concrete reference to the first responder, but if not, it's easier than finding that view and having it resign.

### 2. Keyboard dismisses tapping button
Sometimes, you have a text field asking user to type some information, then press some button. By triggering some actions (e.g., button pressing), the keyboard should dismiss. One way is to use `resignFirstResponder`,

{% highlight objc %}
- (IBAction)testButtonTapped:(id)sender {
    [self.someTextField resignFirstResponder];
}
{% endhighlight %}

one more general solution is to send a `nil` targeted action to the application, which forces it to resign the first responder. This method is an alternative for replacing `[[self view] endEditing:YES];` in the first solution.

{% highlight objc %}
- (IBAction)testButtonTapped:(id)sender {
    [[UIApplication sharedApplication] sendAction:@selector(resignFirstResponder) to:nil from:nil forEvent:nil];
}
{% endhighlight %}

### 3. Keyboard dismisses pressing return key
This method belongs to `UITextFieldDelegate` protocol. The text field's delegate can dismiss the keyboard with `resignFirstResponder` method. Assigning its delegate to self via `someTextField setDelegate:self];` can handle the one page application.

{% highlight objc %}
-(BOOL)textFieldShouldReturn:(UITextField *)textField
{
    [textField resignFirstResponder];
    return YES;
}
{% endhighlight %}

The `textFieldShouldReturn` can be also used for resigning the responder to next text field, which is a classical case when processing user name and password inputs.


