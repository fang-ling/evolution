# Image Display in UIKit

* Proposal: [00000004](00000004-image-in-ui-kit.md)
* Implementation: [fang-ling/core-animation-kit#7](https://github.com/fang-ling/core-animation-kit/pull/7), [fang-ling/javascript-core-kit#8](https://github.com/fang-ling/javascript-core-kit/pull/8), [fang-ling/ui-kit#9](https://github.com/fang-ling/ui-kit/pull/9)

## Summary of changes

Add `UIImage`, `UIImageConfiguration`, `UIImageSymbolConfiguration` and
`UIImageView` to UIKit, enabling image data to be represented and displayed.
This initial revision focuses on symbol image support rendered via SF Pro font
in browser environments.

## Motivation

Images are fundamental building blocks of any app. Currently, UIKit provides
no mechanism for managing or displaying them. Developers who need images must
work entirely outside the UIKit abstraction.

Among image types, symbol images are the most immediately needed. They are
used pervasively throughout system interfaces: buttons, navigation bars,
list rows, and toolbars all routinely display SF Symbols icons.

This proposal introduces the minimal set of types needed to support symbol
images in UIKit, providing a foundation that future revisions can extend to
cover asset catalog images and asynchronous images.

## Proposed solution

We propose adding `UIImage`, `UIImageConfiguration`,
`UIImageSymbolConfiguration` and `UIImageView` to UIKit. Together, these
classes allow symbol images to be created, configured, and embedded in a
view hierarchy.

```objective-c
let configuration = [UIButtonConfiguration makePlainButtonConfiguration];
configuration.title = @"Checkout";
configuration.image = [UIImage makeImageWithSystemName:@"cart.fill"];

let button = [UIButton makeButtonWithConfiguration:configuration
                                     primaryAction:/* ... */];
```

`UIImage` acts as a lightweight descriptor--it records what kind of image is
needed and where its data comes from--while `UIImageView` (and its internal
subclass `_UISymbolImageView`) handles actual rendering into the DOM.

## Detailed design

### `UIImage`

`UIImage` stores an image type alongside the content of each image type
(a symbol character for symbol images, a resource URL for assets, and a
URL for asynchronous images).

```objective-c
/**
 * An object that manages image data in your app.
 *
 * You use image objects to represent image data of all kinds, and the
 * ``UIImage`` class is capable of managing data for all image formats
 * supported by the underlying platform. Image objects are immutable, so you
 * always create them from existing image data, such as an image file over the
 * the network or programmatically created image data. An image object may
 * contain a single image or a sequence of images for use in an animation.
 *
 * You can use image objects in several different ways:
 *
 *   - Assign an image to a ``UIImageView`` object to display the image in
 *     your interface.
 *   - Use an image to customize system controls such as buttons, sliders, and
 *     segmented controls.
 *   - Draw an image directly into a view or other graphics context.
 *   - Pass an image to other APIs that might require image data.
 *
 * Although image objects support all platform-native image formats, it's
 * recommended that you use PNG or JPEG files for most images in your app.
 * Image objects are optimized for reading and displaying both formats, and
 * those formats offer better performance than most other image formats.
 * Because the PNG format is lossless, it's especially recommended for the
 * images you use in your app's interface.
 *
 * ## Create image objects
 *
 * When creating image objects using the methods of this class, you must have
 * existing image data located in the server or data structure. You can't
 * create an empty image and draw content into it. There are many options for
 * creating image objects, each of which is best for specific situations:
 *
 *   - Use the ``makeImageWithName:`` method to create an image from an image
 *     asset in your app's main bundle (or some other known bundle). Because
 *     this method caches the image data automatically, it's especially
 *     recommended for images that you use frequently.
 *   - Use the ``makeImageWithURL:`` method to create an image object where
 *     the initial data isn't in a bundle. This method loads the image data
 *     over the network each time, so don't use them to load the same image
 *     repeatedly.
 *   - Use the ``makeImageWithSystemName:`` method to creates an image object
 *     that contains a system symbol image.
 *
 * > Note: Because image objects are immutable, you can't change their
 *   properties after creation. Most image properties are set automatically
 *   using metadata in the accompanying image file or image data. The
 *   immutable nature of image objects also means they’re safe to create and
 *   use from any thread.
 *
 * Image assets are the easiest way to manage the images that ship with your
 * app. Each Xcode project can contain an assets library, to which you can add
 * multiple image sets. An image set contains the variations of a single image
 * that your app uses. A single image set can provide different versions of an
 * image for different platforms, for different trait environments (compact or
 * regular), and for different scale factors.
 *
 * ## Topics
 *
 * ### Loading and caching images
 *
 * - ``makeImageWithName:``
 * - ``makeImageWithURL:``
 * - ``makeImageWithSystemName:``
 * - ``makeImageWithSystemName:configuration``
 */
@interface UIImage: ObjectiveCObject

/**
 * The configuration details for the image.
 *
 * Use this property to access the traits associated with the image. The system
 * uses the specified traits to determine which variant of the image to load and
 * draw, falling back on the current environment for any unspecified traits. The
 * default value of this property is a configuration object with unspecified
 * traits.
 *
 * You can't modify this property directly, but you can use the
 * ``copyImageWithConfiguration:`` method to create a new image object with a
 * specific set of traits. You might do so when you want to render the image
 * yourself using a specific set of traits.
 *
 * If the image is a symbol image, this property always contains a
 * ``UIImageSymbolConfiguration`` object.
 */
@property (nonatomic, copy, readonly) UIImageConfiguration* configuration;

/**
 * Creates an image object that contains a system symbol image.
 *
 * Use this method to retrieve system-defined symbol images. To retrieve a
 * custom symbol image you store in an asset catalog, use the
 * ``makeImageWithName:`` method instead.
 *
 * This method searches for an image with the name you specify and returns the
 * variant of that image that's best suited for the main screen.
 *
 * To look up the names of system symbol images, download the SF Symbols app
 * from [Apple Design Resources](https://developer.apple.com/design/resources/).
 *
 * - Parameter systemName: The name of the system symbol image.
 *
 * - Returns: The object containing the specified symbol image, or `nil` if no
 *   suitable image was found.
 */
+ (instancetype)makeImageWithSystemName:(FoundationString*)systemName;

/**
 * Creates an image object that contains a system symbol image with the
 * specified configuration.
 *
 * Use this method to retrieve system-defined symbol images. To retrieve a
 * custom symbol image you store in an asset catalog, use the
 *  ``makeImageWithName:configuration:`` method instead.
 *
 * This method searches for an image with the specified name and returns the
 * variant of that image that's best suited for the configuration you specify.
 *
 * To look up the names of system symbol images, download the SF Symbols app
 * from [Apple Design Resources](https://developer.apple.com/design/resources/).
 *
 * - Parameters:
 *   - systemName: The name of the system symbol image.
 *   - configuration: The image configuration the system applies to the image.
 */
+ (instancetype)makeImageWithSystemName:(FoundationString*)systemName
                          configuration:(UIImageConfiguration*)configuration;

@end
```

This proposal focuses on the implementation of symbol images.

Because `UIImage` targets WebAssembly and does not itself perform rendering,
it stores only a type flag and a content value. Internally:

```objective-c
/* UIImage+Private.h */

/**
 * Flags used by ``UIImage`` to indicate the type of an image.
 */
typedef enum UIImageType {
  /**
   * An image created from a symbol image.
   */
  kUIImageTypeSymbol,

  /**
   * An image loaded from an asset catalog image set.
   */
  kUIImageTypeImageSet,

  /**
   * An image whose content is loaded asynchronously.
   */
  kUIImageTypeAsynchronousImage
} UIImageType;

@property (nonatomic) UIImageType type;

@property (nonatomic) FoundationString content;
```

### `UIImageConfiguration` and `UIImageSymbolConfiguration``

This revision focuses on changing the size of a symbol image.

```objective-c
/**
 * A configuration object that contains the traits that the system uses when
 * selecting the current image variant.
 *
 * Images may contain multiple variants to account for environmental factors,
 * such as whether the interface is light or dark. The image configuration
 * object lets you override the current environment and render an image with
 * specific attributes. For example, you might want to render a specific version
 * of your image to your interface.
 *
 * ``UIImageConfiguration`` objects are immutable and you don't create them
 * directly.
 */
@interface UIImageConfiguration: ObjectiveCObject

@end

/**
 * An object that contains the specific font, size, style, and weight attributes
 * to apply to a symbol image.
 *
 * Symbol image configuration objects include details such as the point size,
 * scale, text style, weight, and font to apply to your symbol image. The system
 * uses these details to determine which variant of the image to use and how to
 * scale or style the image.
 *
 * ``UIImageSymbolConfiguration`` objects are immutable after you create them.
 *
 * ## Topics
 *
 * ### Creating a symbol configuration
 *
 * - ``configurationWithPointSize:``
 */
@interface UIImageSymbolConfiguration: UIImageConfiguration

/**
 * Creates a configuration object with the specified point-size information.
 *
 * - Parameter pointSize: The system font point size to use for the
 *   configuration.
 *
 * - Returns: A new symbol configuration object with the specified information.
 */
+ (instancetype)configurationWithPointSize:(CFloatingPoint)pointSize;

@end
```

### `UIImageView`

`UIImageView` is the actual view that displays the image.

```objective-c
/**
 * A view that displays a single image or a sequence of animated images in your
 * interface.
 *
 * Image views let you efficiently draw any image that can be specified using a
 * ``UIImage`` object. For example, you can use the ``UIImageView`` class to
 * display the contents of many standard image files, such as JPEG and PNG
 * files. You can configure image views programmatically and change the images
 * they display at runtime. For animated images, you can also use the methods of
 * this class to start and stop the animation and specify other animation
 * parameters.
 *
 * ## Topics
 *
 * ### Creating an image view
 *
 * - ``makeImageViewWithImage:``
 */
@interface UIImageView: UIView

/**
 * The image displayed in the image view.
 *
 * This property contains the main image displayed by the image view. This image
 * is displayed when the image view is in its natural state. When highlighted,
 * the image view displays the image in its ``highlightedImage`` property
 * instead. If that property is set to `nil`, the image view applies a default
 * highlight to this image. If the ``animationImages`` property contains a valid
 * set of images, those images are used instead.
 *
 * Changing the image in this property does not automatically change the size of
 * the image view. After setting the image, call the ``sizeToFit`` method to
 * recompute the image view's size based on the new image and the active
 * constraints.
 *
 * This property is set to the image you specified at initialization time. If
 * you did not use the ``makeImageViewWithImage:`` or
 * ``makeImageViewWithImage:highlightedImage:`` method to initialize your image
 * view, the initial value of this property is `nil`.
 */
@property (nonatomic) UIImage* image;

/**
 * Creates an image view initialized with the specified image.
 *
 * The image you specified is used to configure the initial size of the image
 * view itself. Use constraints and the image view's content mode to adjust the
 * image view's final size onscreen.
 *
 * If you specify an animated image whose duration is greater than 0, the image
 * view automatically starts playing the animation.
 *
 * - Parameter image: The initial image to display in the image view. You may
 *   specify an image object that contains an animated sequence of images.
 *
 * - Returns: An initialized image view object.
 */
+ (instancetype)makeImageViewWithImage:(UIImage*)image;

@end
```

When the image is a symbol image, the `_UISymbolImageView` subclass is used
instead, because symbol images use a `<span>` tag rather than a standard
`<img>` tag during rendering.

```objective-c
/**
 * A view that displays a symbol image.
 */
@interface _UISymbolImageView: UIImageView

/**
 * Creates an image view initialized with the specified symbol image.
 *
 * The image you specified is used to configure the initial size of the image
 * view itself. Use constraints and the image view's content mode to adjust the
 * image view's final size onscreen.
 *
 * - Parameter image: The initial image to display in the image view.
 *
 * - Returns: An initialized image view object.
 */
- (instancetype)initWithImage:(UIImage*)image;

@end
```

### Image rendering

#### `kUIImageTypeSymbol`

To render symbol images in a browser environment, an SF Pro font resource
file is required (the process for obtaining the font file is outside the
scope of this proposal). The symbol is then rendered as follows:

```html
<style>
  @font-face {
    font-family: "UIKit SF Pro";
    src: url("/Assets/Fonts/SF-Pro.ttf") format("truetype");
  }

  .sfSymbol {
    font-family: "UIKit SF Pro";
  }
</style>

<span class="sfSymbol" style="font-size: 16pt">
  􀀻
</span>
```

A mapping between system symbol names and their corresponding Unicode
characters must be maintained as part of this implementation.

## Source compatibility

All changes are additive. No existing APIs are modified or removed.
Downstream code that builds against earlier releases will continue to build
and behave identically with this change present.

## Implications on adoption

Adopters will need a version of UIKit that includes this change. Because all
additions are new API surface, adopting this feature in a library does not
break existing users of that library, and it can be un-adopted without
affecting source compatibility.

Symbol image support carries one deployment constraint: the SF Pro font file
must be present at `/Assets/Fonts/SF-Pro.ttf`.

## Future directions

### Support for additional image types

This proposal implements only symbol image rendering. Support for asset
catalog images (`kUIImageTypeImageSet`) and asynchronous remote images
(`kUIImageTypeAsynchronousImage`) would complete the `UIImage` type system
introduced here. These are strong candidates for a follow-on revision.

## Alternatives considered

### Using `<img>` tags for all image types

One approach is to render all image types—including symbols—as `<img>`
elements by converting symbols to rasterized PNG resources at build time.
This avoids the runtime font dependency and requires no special DOM handling.

However, font-based rendering preserves the scalable, tintable nature of SF
Symbols without requiring a build step or a large set of pre-rasterized
assets at multiple resolutions. It also more closely mirrors the behavior of
UIKit on native Apple platforms, keeping the API semantics consistent across
environments. The build-time rasterization approach would also require
tooling infrastructure not currently present in this project.
