# Add UIButton to UIKit

* Proposal: [00000003](00000003-uibutton-in-uikit.md)
* Implementation: [fang-ling/javascript-core-kit#7](https://github.com/fang-ling/javascript-core-kit/pull/7), [fang-ling/core-animation-kit#6](https://github.com/fang-ling/core-animation-kit/pull/6), [fang-ling/ui-kit#8](https://github.com/fang-ling/ui-kit/pull/8)

## Summary of changes

Add `UIControl`, `UIButton`, `UIButtonConfiguration`, and `UIAction` to UIKit,
enabling interactive controls with modern action-based event handling. This
initial implementation supports text-based buttons.

## Motivation

UIKit currently provides foundational view types such as `UIView` and `UILabel`,
but it does not provide any interactive controls. As a result, applications can
display content but cannot expose reusable mechanisms for users to trigger
actions.

Buttons are among the most fundamental user-interface components. They provide
the primary way for users to interact with an application, initiate workflows,
and navigate between screens. Without a button control, developers must either
implement custom interaction handling themselves or cannot build meaningful
interactive interfaces.

Introducing `UIButton` also establishes the control infrastructure required by
future UIKit components. Controls such as switches, sliders, text fields,
segmented controls, and menus all rely on a common mechanism for dispatching
user actions. By introducing `UIControl` alongside `UIButton`, this proposal
lays the foundation for future interactive controls.

This proposal adopts a modern action-based API using `UIAction`. Closure-based
actions provide a concise and expressive programming model while avoiding the
need to introduce selector-based target-action infrastructure before it is
required by other controls. The resulting API is straightforward for developers
and maps naturally to event-driven environments.

## Proposed solution

This proposal introduces four new UIKit types:

- `UIControl`, a base class for interactive controls.
- `UIButton`, a control that executes custom code in response to user
  interaction.
- `UIButtonConfiguration`, a configuration type used to describe button content
  and appearance.
- `UIAction`, an object representing an action that can be triggered by a
  control.

The initial implementation intentionally focuses on a minimal but complete
feature set:

- Text-based buttons through `UIButtonConfiguration`.
- Primary-action event handling through `UIAction`.
- A minimal `UIControl` event model consisting of
  `kUIControlEventPrimaryActionTriggered`.

## Detailed design

### UIAction

`UIAction` represents an action that may be triggered by a control.

```objective-c
/**
 * A type that defines the closure for an action handler.
 */ 
typedef void (^)(UIAction*) UIActionHandler;

/**
 * A type that defines the closure for an action handler.
 */
@interface UIAction: ObjectiveCObject

/**
 * The unique identifier for the action.
 */
@property (nonatomic, readonly) FoundationString* identifier;

/**
 * Creates an action with the specified identifier and handler.
 *
 * - Parameters 
 *   - identifier: The unique identifier for the action. Specify `nil` to let
 *     this method create a unique identifier.
 *   - handler: The handler to invoke after a person selects the action. This
 *     handler has the following parameter:
 *       - action: The action that a person selects.
 */
+ (UIAction*)makeActionWithIdentifier:(nullable FoundationString*)identifier
                              handler:(UIActionHandler)handler;

@end
```

Actions are uniquely identified by their identifier. Multiple controls may
reference the same action object.

### UIControl

`UIControl` is the base class for interactive controls.

```objective-c
/**
 * Constants describing the types of events possible for controls.
 *
 * You set up a control so that it sends an action message to a target object by
 * associating both target and action with one or more control events.
 */
typedef enum UIControlEvents: CUnsignedInteger32 {
  /**
   * A semantic action triggered by buttons.
   */
  kUIControlEventPrimaryActionTriggered = 1 << 0
} UIControlEvents;

/**
 * The base class for controls, which are visual elements that convey a specific
 * action or intention in response to user interactions.
 */
@interface UIControl: UIView

/**
 * Adds the UIAction to a given event.
 *
 * `UIAction`s are uniqued based on their identifier, and subsequent actions
 * with the same identifier replace previously added actions. You may add
 * multiple `UIAction`s for corresponding `controlEvents`, and you may add the
 * same action to multiple `controlEvents`.
 *
 * - Parameters:
 *   - action: The action to associate with the control event.
 *   - controlEvents: The control events that trigger the action.
 */
- (void)addAction:(UIAction*)action
 forControlEvents:(UIControlEvents)controlEvents;

/**
 * Calls the action methods associated with the specified events.
 *
 * You call this method when you want the control to perform the actions
 * associated with the specified events. This method iterates over the control's
 * registered targets and action methods and calls the ``sendAction:`` method
 * for each one that is associated with an event in the `controlEvents`
 * parameter.
 *
 * - Parameter controlEvents: A bit-mask with flags that specify the control
 *   events for which the control sends action messages. See ``UIControlEvents``
 *   for bit-mask constants.
 */
- (void)sendActionsForControlEvents:(UIControlEvents)controlEvents;

/**
 * This method is called by ``sendActionsForControlEvents:``. You may override
 * this method to observe or modify behavior. If you override this method, you
 * should call super precisely once to dispatch the action, or not call super to
 * suppress sending that action.
 *
 * - Parameter action: The action to dispatch.
 */
- (void)sendAction:(UIAction*)action;

@end
```

This proposal introduces a minimal subset of UIControl.

Only `kUIControlEventPrimaryActionTriggered` is supported. Additional control
events may be introduced by future proposals.

### UIButtonConfiguration

`UIButtonConfiguration` describes the appearance and content of a button.

```objective-c
/**
 * A configuration that specifies the appearance and behavior of a button and
 * its contents.
 */
@interface UIButtonConfiguration: ObjectiveCObject

/**
 * The text of the title label the button displays.
 */
@property (nullable, nonatomic, copy) FoundationString* title;

/**
 * Creates a configuration for a button with a transparent background.
 *
 * - Returns: A new configuration object.
 */
+ (instancetype)makePlainButtonConfiguration;

@end
```

This proposal introduces only the plain button style. Additional configuration
styles may be added in the future.

### UIButton

`UIButton` is a control that executes custom code in response to user
interactions.

```objective-c
/**
 * A control that executes your custom code in response to user interactions.
 *
 * When you tap a button, or select a button that has focus, the button performs
 * any actions attached to it. You communicate the purpose of a button using a
 * text label, an image, or both. The appearance of buttons is configurable, so
 * you can tint buttons or format titles to match the design of your app.
 */
@interface UIButton: UIControl

/**
 * The configuration for the button's appearance.
 *
 * Setting a configuration opts the button into a configuration system based on
 * ``UIButtonConfiguration``.
 */
@property (nonatomic, copy) UIButtonConfiguration* configuration;

/**
 * Creates a new button with the specified configuration and registers the
 * primary action event.
 *
 * - Parameters:
 *   - configuration: The button configuration.
 *   - primaryAction: The action to perform for the
 *     ``kUIControlEventPrimaryActionTriggered`` control event.
 *
 * - Returns: A new button.
 */
+ (instanceType)
  makeButtonWithConfiguration:(UIButtonConfiguration*)configuration
                primaryAction:(UIAction*)primaryAction;

@end
```

### Rendering

The rendering mechanism is an implementation detail.

Browser-based implementations choose to render buttons using DOM `<button>`
elements.

### Event dispatch

When a user activates a button, the browser implementation receives the
corresponding DOM activation event and dispatches the
`kUIControlEventPrimaryActionTriggered` event on the `UIButton`.

The button then invokes the action registered for 
`kUIControlEventPrimaryActionTriggered` by calling
`sendActionsForControlEvents:`. The action is subsequently dispatched through
`sendAction:` and its handler closure is executed.

## Source compatibility

This proposal is purely additive.

Existing source code continues to compile and behave unchanged.

Because no existing APIs are modified, removed, or overloaded, no source
compatibility concerns are expected.

## Implications on adoption

This feature can be freely adopted and un-adopted in source code without
deployment constraints and without affecting source compatibility.

Applications may incrementally adopt `UIButton` alongside existing UIKit APIs.

Future controls built on top of `UIControl` will be able to integrate naturally
with the event model introduced by this proposal.

## Future directions

### Additional configuration types

Future proposals may introduce additional configuration styles, including:

- Filled buttons
- Bordered buttons
- Tinted buttons

Additional appearance customization APIs may also be considered.

### Image support

Future proposals may allow buttons to display images alongside or instead of
text content.

### Additional control events

Future proposals may expand `UIControlEvents` with additional event types such
as:

- Touch down
- Touch drag enter
- Touch drag exit
- Value changed

These events would support more advanced controls and interaction patterns.

### Button states

Future proposals may introduce support for button states such as:

- Normal
- Highlighted
- Disabled
- Selected

State-dependent appearance customization could then be layered on top of the
configuration system.

### Accessibility

Future work may integrate controls with accessibility infrastructure to support
assistive technologies and alternative input methods.

## Alternatives considered

### Traditional target-action pattern

One alternative is to adopt the traditional Objective-C target-action model from
the beginning.

This approach is familiar to UIKit developers and provides compatibility with
existing patterns.

However, target-action introduces additional selector dispatch infrastructure
and requires APIs that are not otherwise necessary for the initial control
implementation. A closure-based `UIAction` model provides a smaller initial
surface area while remaining expressive and extensible.

Future proposals may introduce target-action APIs without affecting the design
introduced by this proposal.
