Unfortunately, implementing a UI with textfields inside of a scrollview that allows for scrolling when the keyboard opens is not as trivial as one might imagine it to be. The solution I've developed is the [Apple recommended](https://developer.apple.com/library/content/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/KeyboardManagement/KeyboardManagement.html), and it involves a combination of Storyboard development with autolayout and programmatic resizing of views upon reception of certain keyboard events (hide/show). I've chosen to use Swift, but the amount of code is small and shouldn't be hard to adapt to Obj-C if needbe. 

To start, place a `UIScrollview` inside of the base view. We need to pin this scroll view to the base view using autolayout. I recommend using the `Add New Constraints` button insde of storyboard after you've clicked on the `UIScrollview`, using `(0, 0, 0, 0)` as the values for the `Space to Nearest Neighbor`. You could also ctrl-drag directly from to the `UIScrollview` to the base view and set the leading/trailing space to margins and the vertical space to top/bottom layout all to 0 (the `Add New Constraints` approach does just that.

Next, we place a `UIView` inside of the Scrollview and pin it to the `UIScrollView` again with the `(0,0,0,0)`. Change the name of this UIView to `ContentView` in the storyboard(just so it's easy to distinguish the constraints) and give it a unique background color. Now here comes the trick: we need the `ContentView` to be constrained to the base view, not the scrollview. The scrollview adjusts to the size of what's inside of it, so constraining what is inside of it to the scrollview itself creates ambiguity in the auto layout calculations. Give the `ContentView` Equal Width and Height to the base view by ctrl-dragging from between the two and set those constraints in the storyboard. 

Now, just for proving this works, we are going to vertically center a single `Text Field` inside of the `ContentView`. Do this by creating two constraints against the content view: Center Vertically and Horizontally in the container. Lastly, give the text field a fixed width of say `200` by clicking on it, `Add New Constraints->Width`. If you were to run the simulator now, you would see a vertically centered fixed with text field that adapts to screen rotations, but not yet to keyboard appearance. 

Now, we have to do view resizing according to Apple based on the showing/hiding of the keyboard. Inside the ViewController that is backing this View in the storyboard, we need to create an `IBOutlet` so we can reference it. Easiest way for me to do this is to open the split view, and ctrl drag from the `UIScrollView` to the View Ctrl, `Insert Outlet`, and name the variable `scrollView`. You should see the following added to the top of the ViewController file:

```
@IBOutlet weak var scrollView: UIScrollView!
```

Next, in the `viewDidLoad` function, after the super call, we need to handle two keyboard events:

```
NotificationCenter.default.addObserver(self, selector: #selector(self.keyboardDidShow(notification:)), name: NSNotification.Name.UIKeyboardDidShow, object: nil)
        
NotificationCenter.default.addObserver(self, selector: #selector(self.keyboardWillBeHidden(notification:)), name: NSNotification.Name.UIKeyboardWillHide, object: nil)
```

You'll probably get some errors now that the selector doesn't exist, so lets add the two selector fn's we've defined to handle the events:

```
func keyboardDidShow(notification: NSNotification) {
   if let keyboardSize = (notification.userInfo?[UIKeyboardFrameBeginUserInfoKey] as? NSValue)?.cgRectValue {
       let contentInsets = UIEdgeInsets(top: 0.0, left: 0.0, bottom: keyboardSize.height, right: 0.0)
       self.scrollView.contentInset = contentInsets
       self.scrollView.scrollIndicatorInsets = contentInsets
    }
}
    
func keyboardWillBeHidden(notification: NSNotification) {
    let contentInsets = UIEdgeInsets.zero
    self.scrollView.contentInset = contentInsets
    self.scrollView.scrollIndicatorInsets = contentInsets
}
```

What we are effectively doing is adjusting the contentInset property of the Scrollview to account for the fact that there is a keyboard that has appeared by adjusting this property based on the keyboard's size info that we receive in the Notifications.

Now, run the simulator, and you should be able to scroll the whole view once the keyboard is open. Hope this helps :)

