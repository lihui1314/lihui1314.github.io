---
layout: post
title: "SDWebImage进行增量图片下载"
excerpt: "增量图片下载"
---
SDWebImage进行增量图片下载

代码如下

```
- (UIImage *)incrementallyDecodedImageWithData:(NSData *)data finished:(BOOL)finished {
    if (!_imageSource) {
        _imageSource = CGImageSourceCreateIncremental(NULL);
    }
    UIImage *image;
    
    // The following code is from http://www.cocoaintheshell.com/2011/05/progressive-images-download-imageio/
    // Thanks to the author @Nyx0uf
    
    // Update the data source, we must pass ALL the data, not just the new bytes
    CGImageSourceUpdateData(_imageSource, (__bridg
    e CFDataRef)data, finished);
    
    if (_width + _height == 0) {
        CFDictionaryRef properties = CGImageSourceCopyPropertiesAtIndex(_imageSource, 0, NULL);
        if (properties) {
            NSInteger orientationValue = 1;
            CFTypeRef val = CFDictionaryGetValue(properties, kCGImagePropertyPixelHeight);
            if (val) CFNumberGetValue(val, kCFNumberLongType, &_height);
            val = CFDictionaryGetValue(properties, kCGImagePropertyPixelWidth);
            if (val) CFNumberGetValue(val, kCFNumberLongType, &_width);
            val = CFDictionaryGetValue(properties, kCGImagePropertyOrientation);
            if (val) CFNumberGetValue(val, kCFNumberNSIntegerType, &orientationValue);
            CFRelease(properties);
            
            // When we draw to Core Graphics, we lose orientation information,
            // which means the image below born of initWithCGIImage will be
            // oriented incorrectly sometimes. (Unlike the image born of initWithData
            // in didCompleteWithError.) So save it here and pass it on later.
#if SD_UIKIT || SD_WATCH
            _orientation = [SDWebImageCoderHelper imageOrientationFromEXIFOrientation:orientationValue];
#endif
        }
    }
    
    if (_width + _height > 0) {
        // Create the image
        CGImageRef partialImageRef = CGImageSourceCreateImageAtIndex(_imageSource, 0, NULL);
        
        if (partialImageRef) {
#if SD_UIKIT || SD_WATCH
            image = [[UIImage alloc] initWithCGImage:partialImageRef scale:1 orientation:_orientation];
#elif SD_MAC
            image = [[UIImage alloc] initWithCGImage:partialImageRef size:NSZeroSize];
#endif
            CGImageRelease(partialImageRef);
            image.sd_imageFormat = [NSData sd_imageFormatForImageData:data];
        }
    }
    
    if (finished) {
        if (_imageSource) {
            CFRelease(_imageSource);
            _imageSource = NULL;
        }
    }
    
    return image;
}
```

**首先调用```CGImageSourceCreateIncremental```用与创建增量图片源。API描述如下:**

```
The function CGImageSourceCreateIncremental creates an empty image source container to which you can add data later by calling the functions CGImageSourceUpdateDataProvider or CGImageSourceUpdateData. You don’t provide data when you call this function.
An incremental image is an image that is created in chunks, similar to the way large images viewed over the web are loaded piece by piece.

函数CGImageSourceCreateIncremental创建一个空的图片数据源容器，稍后可以通过调用函数CGImageSourceUpdateDataProvider或CGImageSourceUpdateData向该容器添加数据。调用此函数时不提供数据。
增量图像是一种以块的形式创建的图像，类似于通过web查看的大型图像的加载方式。
```

**通过调用```CGImageSourceUpdateData```更新增量图像源。**

**调用```CGImageSourceCopyPropertiesAtIndex```**

在第一次增加图片数据源后调用函数```CGImageSourceCopyPropertiesAtIndex```返回图像数据源指定位置的属性。API描述：

```
Returns the properties of the image at a specified location in an image source.
返回图像源中指定位置的图像的属性。
```

**调用```CGImageSourceCreateImageAtIndex``` **

为与图像源中的指定索引关联的图像数据创建CGImage对象，索引是从0开始的。

**调用```- (instancetype)initWithCGImage:(CGImageRef)cgImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation NS_AVAILABLE_IOS(4_0);```方法创建具有指定比例和方向因子的图像对象。**













