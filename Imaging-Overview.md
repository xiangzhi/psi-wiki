This is a brief overview of imaging components and image-related APIs available in Platform for Situated Intelligence.

## Stream Operators

Images and operations on them are defined in the `Microsoft.Psi.Imaging` namespace. Images come in several flavors, all deriving from `ImageBase` and generally wrapped in a `Shared<_>` to mitigate the costs of frequent large memory allocations and garbage collection as described in the [Shared Objects topic](./Shared-Objects.md).

Images coming from the `MediaCapture` component, for example, are `Shared<Image>` while those coming from the depth stream on the `AzureKinect` component are `Shared<DepthImage>`. Furthermore, either may be encoded to `Shared<EncodedImage>` or `Shared<EncodedDepthImage>`.

There are operators to convert between `Shared<Image>` and `Shared<DepthImage>`:

* `ToDepthImage(...)` - Converts a stream of images into a stream of depth images.
* `ToImage(...)` - Converts a stream of depth images into a stream of `Gray_16bpp` format images.

Operations that apply only to depth image streams include:

* `PseudoColorize(...)` - Convert a producer of depth images into pseudo-colorized images, where more distant pixels are blue, and closer pixels are reddish.

Some operations have overloads for both types:

* `Crop()` - Crops a shared image or depth image using the specified rectangle.
* `Decode(...)` - Decodes an encoded (depth)image using a specified decoder component or stream decoder.
* `Encode(...)` - Encodes a shared (depth)image using a specified encoder component or stream encoder.

Many operations apply only to regular image streams:

* `AbsDiff(...)` - Computes the absolute difference between two images.
* `Clear(...)` - Clears a shared image to the specified color.
* `DrawCircle(...)` - Draws a circle over a shared image.
* `DrawLine(...)` - Draws a line over a shared image.
* `DrawRectangle(...)` - Draws a rectangle over a shared image.
* `DrawText(...)` - Draws a piece of text over a shared image.
* `ExtractChannel(...)` - Extracts a color channel from a shared image. Returned image is of type `Gray_8bpp`.
* `Flip(...)` - Flips a shared image about the horizontal or vertical axis.
* `Invert(...)` - Inverts each color channel in a shared image.
* `Resize(...)` - Resizes a shared image.
* `Rotate(...)` - Rotates a shared image by the specified angle.
* `Scale(...)` - Scales a shared by the specified scale factors.
* `Threshold(...)` - Thresholds the image. See Threshold for what modes of thresholding are available.
* `ToGray(...)` - Converts a shared image to grayscale.
* `ToPixelFormat(...)` - Converts the source image to a different pixel format.
* `Transform(TransformDelegate, PixelFormat, ...)` - Converts a shared image to a different pixel format using the specified transformer.

## Basic Components

The following are currently the components provided by \\psi for handling of images:

* `ImageDecoder` - Decodes a compressed image
* `ImageEncoder` - Compresses an image
* `TransformImageComponent` - Performs some transformation on the image

## Common Patterns of Usage

The following are some examples of how to use the Image components in \\psi.

### Encode an image into .png

The following example shows how to take an image and compress it into a .png

```csharp
using Microsoft.Psi.Imaging;

public EncodedImage EncodeImage(Image testImage)
{
    EncodedImage encImg = new EncodedImage();
    encImg.EncodeFrom(testImage, new PngBitmapEncoder());
    return encImg;
}
```

### Decode a .png into a <see cref="Microsoft.Psi.Imaging.Image">Microsoft.Psi.Imaging.Image</see>

The following example shows how to take an image and compress it into a .png

```csharp
using Microsoft.Psi.Imaging;

public Image DecodeImage(EncodedImage encodedImage)
{
    Image decImg = new Image(encodedImage.Width, encodedImage.Height, encodedImage.Width * 3,  PixelFormat.Format24bppRgb);
    encodedImage.DecodeTo(decImg);
    return decImg;
}
```

### Crop a stream of images

This final sample shows how to create a stream of images from a single image, and crop each image in the stream by a fixed set of coordinates; finally writing each cropped image out to a file.

```csharp
public void CropImages(Image testImage)
{
    using (var pipeline = Pipeline.Create())
    {
        var sharedImage = Microsoft.Psi.Imaging.ImagePool.GetOrCreate(testImage.Width, testImage.Height, testImage.PixelFormat);
        testImage.CopyTo(sharedImage.Resource);
        var images = Generators.Sequence(pipeline, sharedImage, x => sharedImage, 100);
        var rects = Generators.Sequence<System.Drawing.Rectangle>(pipeline, new System.Drawing.Rectangle(), x => {
	        System.Drawing.Rectangle rect = new System.Drawing.Rectangle();
	        rect.X = 0;
	        rect.Y = 0;
	        rect.Width = 100;
	        rect.Height = 100;
	        return rect;
        }, 100);
        var croppedImages = images.Pair(rects).Crop();
        int count = 0;
        images.Do((img, e) => {
            img.Resource.ToManagedImage().Save(@"c:\temp\image-" + count.ToString() + ".bmp");
            count++;
        });
        pipeline.Run();
    }
}
```