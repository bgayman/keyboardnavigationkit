# KeyboardNavigationKit
The kit provides navigators that manage keyboard focus for a list in an iOS application. Users can highlight rows using the arrow keys, then press Enter to select. As the focus highlight goes offscreen, the table view scrolls to continue the navigation. This behavior mirrors how iOS Spotlight works; KeyboardNavigationKit is a drop-in framework to achieve the same functionality in your own apps.

**[Watch KeyboardNavigationKit Demo on YouTube](https://www.youtube.com/watch?v=zTc5e7zXEag).**

## Getting Started

The `TableNavigator` exposes accessors for the responder chain and coordinates behavior through a delegate. In the parent of a table view, typically a `UIViewController` subclass, create a `TableNavigator`:

```swift
public class ListViewController : UIViewController {
  private let tableView = UITableView(style: .plain)
  var tableNavigator: TableNavigator!
   
  public override func viewDidLoad() {
    super.viewDidLoad()
    
    self.tableNavigator = TableNavigator(tableView: self.tableView, delegate: self)
  }
  
  ...
}
```

The `TableNavigatorDelegate` has two required methods and two optional methods for customizing behavior. Implement the required methods:

```swift
extension ListViewController : TableNavigatorDelegate {
  func tableNavigator(_ navigator: TableNavigator, didUpdateFocus focusUpdate: TableNavigator.FocusUpdate, completedNavigationWith context: TableNavigator.NavigationCompletionContext) {
    // update focus appearance for changed cells - indexPathsForChangedRows contains indexPaths corresponding to the newly-focused row and the previously-focused row
    
    self.tableView.reloadRows(at: focusUpdate.indexPathsForChangedRows, animated: false)
  }
      
  func tableNavigator(_ navigator: TableNavigator, commitFocusedRowAt indexPath: IndexPath) {
    // user committed the focus row, handle like UITableViewDelegate.tableView(_:, didSelectRowAt:)
    self.presentDetailViewController(forRowAt: indexPath)
  }
}
```

Whilst the `TableNavigator` manages the focus, the presentation of the focus is controlled by your application. Typically, this means customizing the cell appearance in the `tableView(_:, cellForRowAt:_)` data source. Check the `TableNavigator.indexPathForFocusedRow` to determine if a given cell is focused. 

The `tableNavigator(_, didUpdateFocus: completedNavigationWith:)` delegate includes a `FocusUpdate` which describes which cells have been affected by the change in focus. The `TableNavigator.removeFocus` method also returns a `FocusUpdate` so you can refactor focus appearance changes into a dedicated method. 

Finally, the view controller needs to expose key commands to the responder chain. The `TableNavigator` creates and responds to these events, but the view controller must delegate to it. To do this, implement the following two methods on the view controller.

```swift
public class ListViewController : UIViewController {
    ...
    
    public override var keyCommands: [UIKeyCommand]? {
        get {
            return self.listKeyboardNavigator.possibleKeyCommands
        }
        set {}
    }
    
    public override func target(forAction action: Selector, withSender sender: Any?) -> Any? {
        if let target = self.listKeyboardNavigator.target(forKeyCommandAction: action) {
            return target
        } else {
            return super.target(forAction: action, withSender: sender)
        }
    }
    
    ...
}
```

The `TableNavigator.possibleKeyCommands` creates an array of `UIKeyCommand` objects to pass through the responder chain. This collection updates frequently and should not be externally cached. Simply override the `keyCommands` responder chain method and return the `TableNavigator.possibleKeyCommands` array.

Overriding `target(forAction:, withSender:)` allows the `TableNavigator` to handle the key command actions. If `TableNavigator.target(forKeyCommandAction:)` is non-nil, return the target. If it is nil, then it means the action is not related to keyboard list navigation and should be handled by another object. In this example, we simply delegate to the super implementation.

By default, `TableNavigator` generates key commands that do not have a `discoverabilityTitle` set. Do not attempt to customize the `UIKeyCommand` created by the navigator directly. If you want to control which key commands are generated, implement the appropriate `TableNavigatorDelegate` method: `tableNavigator(_, keyCommandDescriptorsFor:, defaultDescriptors:) -> [KeyCommandDescriptor]`.

## To Be Completed (in no particular order)
* Better documentation.
* Add a `CollectionNavigator` for managing focus in a `UICollectionView`.
* The framework is named generically as it will eventually support other keyboard navigation beyond lists, such as dismissing modal views and popping controllers in a navigation stack.
* Probably a lot more I haven't thought about yet.

## Project Motivation

This framework was made out of a frustration that the iOS Spotlight system behavior was not exposed directly in UIKit. The initial design of the navigator's behavior mirrored the Spotlight behavior, and was extended where appropriate. The need for comprehensive keyboard navigation arose in the creation of my own app, [Visual Codes](http://visualcod.es). Get it on the App Store to try out a real-world use case of KeyboardNavigationKit.
