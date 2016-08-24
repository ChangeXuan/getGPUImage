# getGPUImage
I will try to translate GPUImage

##Overview/*2016-8-16*/
这个GPUImage框架是一个BSD-licensed的库，这个库能够让你在使用GPU加速计算的条件下加上滤镜和其他的影响效果到图片，摄像机视频直播和电影。
相对于CoreImage，GPUImage允许你去写出你自定义的滤镜，支持部署在iOS4.0，以及它简单的接口。但是，对于CoreImage来说，它现在还缺少一些更
高级的功能，如人脸识别。<br/>
<br/>
对于大规模并行操作，如处理图像或实时的视频画面，GPU有着巨大的性能优势来超过CPU。在iPhone 4上，同样一个简单的图像滤镜，使用GPU完成能够
比基于CPU完成的滤镜快100倍。<br/>
<br/>
然而，在GPU上运行一个自定义滤镜，这些滤镜需要使用许多代码去设置和维护一个OpenGL ES 2.0的渲染对象。所有，我创建了一个简单的工程去做了这
件事：<br/>
<br/>
http://www.sunsetlakesoftware.com/2010/10/22/gpu-accelerated-video-processing-mac-and-ios<br/>
<br/>
我发现我不得不把这些模板代码写在其他的作品。因此，我把许多常见的任务一起封装到了这个框架中，当你处理图像和视频，你将会遇到并去使用它，所以你不需要去关心自己是否有OpenGL ES 2.0的基础。<br/>
<br/>
(以下是一些比较)
This framework compares favorably to Core Image when handling video, taking only 2.5 ms on an iPhone 4 to upload a frame from the camera, apply a gamma filter, and display, versus 106 ms for the same operation using Core Image. CPU-based processing takes 460 ms, making GPUImage 40X faster than Core Image for this operation on this hardware, and 184X faster than CPU-bound processing. On an iPhone 4S, GPUImage is only 4X faster than Core Image for this case, and 102X faster than CPU-bound processing. However, for more complex operations like Gaussian blurs at larger radii, Core Image currently outpaces GPUImage.
##License
BSD-style，和该框架相关的有效证书在Lincense.txt中
##Technical requirements(技术要求)
- OpenGL ES 2.0: 该应用不能在原始的iPhone，iPhone 3G和第一和第二代的iPod touches上运行。
- iOS 4.1 as a deployment target (4.0 didn't have some extensions needed for movie reading). iOS 4.3 is needed as a deployment target if you wish to show live video previews when taking a still photo.
- iOS 5.0去构建。
- 设备必须有摄像机去使用有关摄像机的功能(明显的)。
- 该框架使用自动引用计数(ARC)，但是项目应该支持自动引用计数和手动引用计数二者，如果添加在子项目下的注释。手动引用计数是对于iOS 4.x来说的，你将需要add -fobjc-arc去其他链接器标志对于你的应用工程来说。 

##General architecture(总体框架)
GPUImage使用OpenGL ES 2.0着色器去完成图像和视频的处理远远超过绑定在CPU上的程序。不过，它隐藏了与OpenGL ES API交互的复杂性在简单的Objective-C的接口中。这个接口让你在一系列中自定义输入图像和视频以及滤镜，还发送处理的图像或视频到你的屏幕上，一个UIImage上或磁盘中的一个影像上。<br/>
<br/>
图像和视频帧是从源中上传的，这里使用了GPUImageOutput的子类。这些包括了GPUImageVideoCamera(从摄像头中获取视频)，GPUImageStillCamera(从摄像头中获取相片)，GPUImagePicture(使用静止的图像)和GPUImageMovie(使用视频)。OpenGL ES使用从源中上传的图像作为纹理，然后把这些纹理去处理链中的下一个对象。<br/>
<br/>
链中的滤镜和其他的一些元素都遵守了GPUImageInput的协议，这让它们可以从上一个反应链中取到被提供或者被处理的纹理并且对它做出一些事情。下一个反应链作为目标，和能够处理多个分支来添加多个目标去单个输出或滤镜。<br/>
<br/>
例如，一个应用从摄像机中获取视频流，再把视频转化为棕色色调，然后展现视频到屏幕上，这个过程将会建立一个反应链，这个反应链看起来是这样的
>GPUImageVideoCamera -> GPUImageSepiaFilter -> GPUImageView

##Adding the static library to your iOS project(添加静态库到你的iOS工程)/*2016.8.17*/
备注：如果你像把这个库使用到一个采用Swift语言开发的工程中，你需要按照"Adding this as a framework(添加一个框架)"中的步骤。Swift需要第三方模块的代码。<br/>
<br/>
Once you have the latest source code for the framework, it's fairly straightforward to add it to your application. Start by dragging the GPUImage.xcodeproj file into your application's Xcode project to embed the framework in your project. Next, go to your application's target and add GPUImage as a Target Dependency. Finally, you'll want to drag the libGPUImage.a library from the GPUImage framework's Products folder to the Link Binary With Libraries build phase in your application's target.(你将很方便的使用最新的框架到你的工程中)。<br/>
<br/>
GPUImage需要在你的工程中使用一些其他的框架，所以你需要按照下表来添加库到你的目标应用中：
- CoreMedia
- CoreVideo
- OpenGLES
- AVFoundation
- QuartzCore

你还将需要找到这些框架的头文件，所以在项目的生成设置内将标题搜索路径设置为从应用程序到GPUImage源目录中的框架/目录的相对路径。让这头递归搜索路径。<br/>
<br/>使用GPUImage类在你的应用里边，只是在核心的代码类中使用以下的头文件：
>*#*import "GPUImage.h"

备注：如果你运行时遇到"Unknown class GPUImageView in Interface Builder"的错误，或者the like when trying to build an interface with Interface Builder(不能理解)，你可能需要在project's build settings中的Other Linker Flags中添加-ObjC。<br/>
<br/>
还有，如果你需要部署到iOS 4.x，it appears that the current version of Xcode (4.3) requires that you weak-link the Core Video framework in your final application or you see crashes with the message "Symbol not found: _CVOpenGLESTextureCacheCreate" when you create an archive for upload to the App Store or for ad hoc distribution. To do this, go to your project's Build Phases tab, expand the Link Binary With Libraries group, and find CoreVideo.framework in the list. Change the setting for it in the far right of the list from Required to Optional.(一些解决方案)<br/>
<br/>
Additionally, this is an ARC-enabled framework, so if you want to use this within a manual reference counted application targeting iOS 4.x, you'll need to add -fobjc-arc to your Other Linker Flags as well.(这个也是)<br/>
<br/>
###Building a static library at the command line(使用命令行来建立一个静态库)
If you don't want to include the project as a dependency in your application's Xcode project, you can build a universal static library for the iOS Simulator or device. To do this, run build.sh at the command line. The resulting library and header files will be located at build/Release-iphone. You may also change the version of the iOS SDK by changing the IOSSDK_VER variable in build.sh (all available versions can be found using xcodebuild -showsdks).(我觉得目前没必要)

##Adding this as a framework (module) to your Mac or iOS project(添加这个框架(模块)到你的Mac或者iOS工程)/*2016.8.18*/
Xcode 6和iOS 8支持去使用完整的框架，Mac也一样，这样将简化添加此框架到你的应用的过程。添加此框架到你的应用，我推荐把.xcodeproj后缀的工程文件拖拽到你的应用的项目中(就像你使用静态链接库一样)(自己生成GPUImage.Framework)<br>
<br>
在你的应用工程中，去到它的target build settings和选择Build Phases tab。在Target Dependencies grouping下，给iOS添加GPUImageFramework(不是GPUImage，构建那个静态链接库)或者给Mac添加GPUImage。在Link Binary With Libraries选项，添加GPUImage.framework。<br>
<br>
这样应该使得GPUImage 建立成了一个框架。在Xcode 6下，这将构建一个模块，并可以让你在Swift开发的项目下使用了。当你完成上面的设置，你应该只需要使用：
>import GPUImage

去调用它<br>
<br>
在Build Phases里，添加一个新的Copy Files，Destination选Frameworks，和添加GPUImage.framework进来。这样就可以在你的应用中使用这个框架了.


##Performing common tasks(执行常见的任务)/*2016.8.22*/
###过滤实时视频
想在iOS设备的摄像机中对实时的视频流进行处理，你能够使用下面的代码：
```objectivec
GPUImageVideoCamera *videoCamera = [[GPUImageVideoCamera alloc] initWithSessionPreset:AVCaptureSessionPreset640x480 cameraPosition:AVCaptureDevicePositionBack];<br>
videoCamera.outputImageOrientation = UIInterfaceOrientationPortrait;

GPUImageFilter *customFilter = [[GPUImageFilter alloc] initWithFragmentShaderFromFile:@"CustomShader"];<br>
GPUImageView *filteredVideoView = [[GPUImageView alloc] initWithFrame:CGRectMake(0.0, 0.0, viewWidth, viewHeight)];

// Add the view somewhere so it's visible

[videoCamera addTarget:customFilter];<br>
[customFilter addTarget:filteredVideoView];

[videoCamera startCameraCapture];
```
这个是设置视频的源从iOS设备的后摄像头中的到，并设置捕获的视频为640x480.设置视频捕获的接口为得到纵向视频，where the landscape-left-mounted camera needs to have its video frames rotated before display.(暂时不懂)。一个自定义滤镜，使用CustomShader.fsh文件中的代码，然后设置从摄像机中获取视频帧到目标。这些被使用滤镜的视频帧最后展现在屏幕，with the help of a UIView subclass that can present the filtered OpenGL ES texture that results from this pipeline.(。。。。。。)。<br>
<br>
GPUImage中的填充模式能够通过设置填充模式去改变，所以如果源视频的横纵比与View的横纵比不相同，这个视频将会被拉长，中间有黑条，或通过缩放来填充<br>
<br>
对于混合滤镜和其他多个图片，你可以创建多个输出和添加一个单独的滤镜，作为这两个输出的目标。The order with which the outputs are added as targets will affect the order in which the input images are blended or otherwise processed.<br>
<br>
还有，如果你希望能够使用麦克风去获取声音道电影中，你将需要去设置audioEncodingTarget，像下面那样做：
>videoCamera.audioEncodingTarget = movieWriter;

###Capturing and filtering a still photo(获取和过滤一张静止的照片)
去获取和过滤静态的照片，你可以使用在处理视频时的类似过程。使用GPUImageStillCamera去代替GPUImageVideoCamera：
```objectivec
stillCamera = [[GPUImageStillCamera alloc] init];
stillCamera.outputImageOrientation = UIInterfaceOrientationPortrait;

filter = [[GPUImageGammaFilter alloc] init];
[stillCamera addTarget:filter];
GPUImageView *filterView = (GPUImageView *)self.view;
[filter addTarget:filterView];

[stillCamera startCameraCapture];
```
This will give you a live, filtered feed of the still camera's preview video. Note that this preview video is only provided on iOS 4.3 and higher, so you may need to set that as your deployment target if you wish to have this functionality.<br>
<br>
如果你想获取一张照片，你可以使用下列的回调方法：
```objectivec
[stillCamera capturePhotoProcessedUpToFilter:filter withCompletionHandler:^(UIImage *processedImage, NSError *error){
    NSData *dataForJPEGFile = UIImageJPEGRepresentation(processedImage, 0.8);

    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];

    NSError *error2 = nil;
    if (![dataForJPEGFile writeToFile:[documentsDirectory stringByAppendingPathComponent:@"FilteredPhoto.jpg"] options:NSAtomicWrite error:&error2])
    {
        return;
    }
}];
```
上面的代码能获得一个满尺寸的且被处理过的照片，分别呈现在一个view上以及以JPEG格式保存在你应用的文件目录下。<br>
<br>
请注意，在一些老的设备中，该框架现在不能处理宽或高大于2048像素，因为纹理限制(Phone 4S, iPad 2, Retina iPad)。This means that the iPhone 4, whose camera outputs still photos larger than this, won't be able to capture photos like this. A tiling mechanism is being implemented to work around this. All other devices should be able to capture and filter photos using this method.<br>
<br>
###Processing a still image(处理一张静态图片)
这里有两种方法去处理一张静态图片和创建一个结果。第一种方法是你去创建一个静态图片源对象，和手动创建一个滤镜链：
```objectivec
UIImage *inputImage = [UIImage imageNamed:@"Lambeau.jpg"];

GPUImagePicture *stillImageSource = [[GPUImagePicture alloc] initWithImage:inputImage];
GPUImageSepiaFilter *stillImageFilter = [[GPUImageSepiaFilter alloc] init];

[stillImageSource addTarget:stillImageFilter];
[stillImageFilter useNextFrameForImageCapture];
[stillImageSource processImage];

UIImage *currentFilteredVideoFrame = [stillImageFilter imageFromCurrentFramebuffer];
```
请注意，对于从过滤器中手动获取图片，你需要去设置-useNextFrameForImageCapture，为了告诉过滤器你将在之后去获取一张图片。默认的，GPUImage
在过滤器中重复使用帧缓冲器用来节约内存，所以你需要时时让它知道你要去获取它<br>
<br>
对于在只单单在图片上使用过滤器，你可以这样简单的使用：
```objectivec
GPUImageSepiaFilter *stillImageFilter2 = [[GPUImageSepiaFilter alloc] init];
UIImage *quickFilteredImage = [stillImageFilter2 imageByFilteringImage:inputImage];
```
###Writing a custom filter(编写一个自定义的滤镜)/*2016-8-24*/
在iOS上，这个框架对于CoreImage来说有一个重要的优势，那就是你可以自己去编写一个属于你自己的滤镜去处理图片和视频。These filters are supplied as OpenGL ES 2.0 fragment shaders, written in the C-like OpenGL Shading Language.(......)<br>
<br>
一个自定义的滤镜的使用下面的代码进行初始化：<br>
```objectivec
GPUImageFilter *customFilter = [[GPUImageFilter alloc] initWithFragmentShaderFromFile:@"CustomShader"];
```
where the extension used for the fragment shader is .fsh.(......)。此外，你可以使用-initWithFragmentShaderFromString：把片段着色器作为一个字符串提供给初始化程序，如果你不想把片段着色器搬运到你的应用项目中。<br>
<br>
片段着色器在过滤的阶段执行，它们计算每一个像素用来进行渲染。它们使用OpenGL Shading语言(GLSL)，一个内置的用来添加特定的2D和3D图形。下面一个列子使用一个片段着色器去完成一个棕褐色的滤镜：<br>
<br>
```c
varying highp vec2 textureCoordinate;

uniform sampler2D inputImageTexture;

void main()
{
    lowp vec4 textureColor = texture2D(inputImageTexture, textureCoordinate);
    lowp vec4 outputColor;
    outputColor.r = (textureColor.r * 0.393) + (textureColor.g * 0.769) + (textureColor.b * 0.189);
    outputColor.g = (textureColor.r * 0.349) + (textureColor.g * 0.686) + (textureColor.b * 0.168);    
    outputColor.b = (textureColor.r * 0.272) + (textureColor.g * 0.534) + (textureColor.b * 0.131);
    outputColor.a = 1.0;

    gl_FragColor = outputColor;
}

```
For an image filter to be usable within the GPUImage framework, the first two lines that take in the textureCoordinate varying (for the current coordinate within the texture, normalized to 1.0) and the inputImageTexture uniform (for the actual input image frame texture) are required.<br>
<br>
The remainder of the shader grabs the color of the pixel at this location in the passed-in texture, manipulates it in such a way as to produce a sepia tone, and writes that pixel color out to be used in the next stage of the processing pipeline.<br>
<br>
One thing to note when adding fragment shaders to your Xcode project is that Xcode thinks they are source code files. To work around this, you'll need to manually move your shader from the Compile Sources build phase to the Copy Bundle Resources one in order to get the shader to be included in your application bundle.(......).<br>
<br>

###Filtering and re-encoding a movie(过滤和重编码一个电影)
电影通过该框架的GPUImageMovie类的通道来加载，被过滤，和使用GPUImageMovieWriter来写入。GPUImageMovieWriter也是一个足够快速的去记录实时的视频在iPhone 4's camera at 640x480下，所以一个直接过滤的视频源能够直接的输入。现在，在iPhone4上，GPUImageMovieWriter有足够快的速度去记录720p的直播视频，且帧数高达20 FPS。而在iPhone4s上可以达到30FPS。<br>
<br>
按照下面的例子，加载一个简单的电影，并使用像素化滤镜去处理该电影，并把处理后的电影(480x640)保存在硬盘中：
```objectivec
movieFile = [[GPUImageMovie alloc] initWithURL:sampleURL];
pixellateFilter = [[GPUImagePixellateFilter alloc] init];

[movieFile addTarget:pixellateFilter];

NSString *pathToMovie = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents/Movie.m4v"];
unlink([pathToMovie UTF8String]);
NSURL *movieURL = [NSURL fileURLWithPath:pathToMovie];

movieWriter = [[GPUImageMovieWriter alloc] initWithMovieURL:movieURL size:CGSizeMake(480.0, 640.0)];
[pixellateFilter addTarget:movieWriter];

movieWriter.shouldPassthroughAudio = YES;
movieFile.audioEncodingTarget = movieWriter;
[movieFile enableSynchronizedEncodingUsingMovieWriter:movieWriter];

[movieWriter startRecording];
[movieFile startProcessing];
```
当录像结束后，你应该记得去移除电影的记录者(movieWriter)，和关闭记录者，就像下面一样：
```objectivec
[pixellateFilter removeTarget:movieWriter];
[movieWriter finishRecording];
```
A movie won't be usable until it has been finished off, so if this is interrupted before this point, the recording will be lost.(......)

###Interacting with OpenGL ES(与OpenGL ES的相互关系)
GPUImage can both export and import textures from OpenGL ES through the use of its GPUImageTextureOutput and GPUImageTextureInput classes, respectively. This lets you record a movie from an OpenGL ES scene that is rendered to a framebuffer object with a bound texture, or filter video or images and then feed them into OpenGL ES as a texture to be displayed in the scene.<br>
<br>
The one caution with this approach is that the textures used in these processes must be shared between GPUImage's OpenGL ES context and any other context via a share group or something similar.

##Built-in filters(自带的滤镜)
该框架现在总共有125个自带的滤镜，分为一下几类：
###Color adjustments(颜色调节器)
- **GPUImageBrightnessFilter**： 调节图像的明亮度。
 + 明亮度: 调整 brightness (-1.0 - 1.0, with 0.0 as the default)
- **GPUImageExposureFilter**： 调节图像的曝光率。
 + 曝光率: 调整 exposure (-10.0 - 10.0, with 0.0 as the default)
- **GPUImageContrastFilter**： 调节图像的对比度。
 + 对比度: 调整 contrast (0.0 - 4.0, with 1.0 as the default)
- **GPUImageSaturationFilter**： 调节图像的饱和度。
- **GPUImageGammaFilter**： 调节图像的灰度系数。
- **GPUImageLevelsFilter**：就像Photoshop里边的等级调整。它里边的min，max，minOut，maxOut这四个值的取值范围是在[0，1].如果你在Photoshop的界面上看到的取值范围是[0，255]，你必须去转变它们到[0，1]。在gamma/mid参数是一个大于等于零的浮点数。从Photoshop中的值进行匹配。If you want to apply levels to RGB as well as individual channels you need to use this filter twice - first for the individual channels and then for all channels.(......)
- **GPUImageColorMatrixFilter**：通过一个矩阵来转变图像的颜色。
- **GPUImageRGBFilter**：调整图像中RGB三个值得其中一个通道。(Normalized values by which each color channel is multiplied. The range is from 0.0 up, with 1.0 as the default.)
- **GPUImageHueFilter**：调整图像的色调。
- **GPUImageVibranceFilter**：调节图像的自然饱和度。
- **GPUImageWhiteBalanceFilter**：调节图像的白平衡。
- **GPUImageToneCurveFilter**：调整基于样条曲线为每个颜色通道图像的颜色。
- **GPUImageHighlightShadowFilter**：调节图像的阴影和高光。
- **GPUImageHighlightShadowTintFilter**：允许你独立的去给图片的阴影和高光添加颜色。
- **GPUImageLookupFilter**：Uses an RGB color lookup image to remap the colors in an image. First, use your favourite photo editing application to apply a filter to lookup.png from GPUImage/framework/Resources. For this to work properly each pixel color must not depend on other pixels (e.g. blur will not work). If you need a more complex filter you can create as many lookup tables as required. Once ready, use your new lookup.png file as a second input for GPUImageLookupFilter.(......)
- **GPUImageAmatorkaFilter**：A photo filter based on a Photoshop action by Amatorka: http://amatorka.deviantart.com/art/Amatorka-Action-2-121069631 . If you want to use this effect you have to add lookup_amatorka.png from the GPUImage Resources folder to your application bundle.
- **GPUImageMissEtikateFilter**：A photo filter based on a Photoshop action by Miss Etikate: http://miss-etikate.deviantart.com/art/Photoshop-Action-15-120151961 . If you want to use this effect you have to add lookup_miss_etikate.png from the GPUImage Resources folder to your application bundle.
- **GPUImageSoftEleganceFilter**：Another lookup-based color remapping filter. If you want to use this effect you have to add lookup_soft_elegance_1.png and lookup_soft_elegance_2.png from the GPUImage Resources folder to your application bundle.
- **GPUImageSkinToneFilter**：这是一个肤色调整器，当出现独特肤色时，需要自己进行调整，默认值是白人的肤色。
- **GPUImageColorInvertFilter**：反转图像的颜色。
- **GPUImageGrayscaleFilter**：把图片转换为灰度图(这个转换比saturation滤镜要快一些，但是这个函数只能转换为灰色)。
- **GPUImageMonochromeFilter**：根据图像上每个像素的亮度，把图像转换为(a single-color version)(单色图？)(二值图？)。
- **GPUImageFalseColorFilter**：根据图像上每个像素的亮度，让图像的内容用用户自己定义的两种颜色混合表示。
- **GPUImageHazeFilter**：在图像上添加或移除雾霾(similar to a UV filter)
- **GPUImageSepiaFilter**：简单的棕色滤镜。
- **GPUImageOpacityFilter**：调整图像传入的Alpha通道。
- **GPUImageSolidColorGenerator**：这个会生成一张单色的图片。你需要使用 -forceProcessingAtSize: 去定义图片的大小。
- **GPUImageLuminanceThresholdFilter**：图像上的像素的亮度大于某值时，该像素呈现白色，反之，该像素呈现黑色。
- **GPUImageAdaptiveThresholdFilter**：在确定一个像素周围的亮度后，如果这个像素的亮度低于周围像素的亮度，这该像素呈现黑色，反之，该像素呈现白色。这个比较适合用在光照发生变化的条件下。
- **GPUImageAverageLuminanceThresholdFilter**：这个函数使用的阈值是根据现场的光照的平均值来不断调整的。
- **GPUImageHistogramFilter**：his analyzes the incoming image and creates an output histogram with the frequency at which each color value occurs. The output of this filter is a 3-pixel-high, 256-pixel-wide image with the center (vertical) pixels containing pixels that correspond to the frequency at which various color values occurred. Each color value occupies one of the 256 width positions, from 0 on the left to 255 on the right. This histogram can be generated for individual color channels (kGPUImageHistogramRed, kGPUImageHistogramGreen, kGPUImageHistogramBlue), the luminance of the image (kGPUImageHistogramLuminance), or for all three color channels at once (kGPUImageHistogramRGB).
- **GPUImageHistogramGenerator**：This is a special filter, in that it's primarily intended to work with the GPUImageHistogramFilter. It generates an output representation of the color histograms generated by GPUImageHistogramFilter, but it could be repurposed to display other kinds of values. It takes in an image and looks at the center (vertical) pixels. It then plots the numerical values of the RGB components in separate colored graphs in an output texture. You may need to force a size for this filter in order to make its output visible.
- **GPUImageAverageColor**：This processes an input image and determines the average color of the scene, by averaging the RGBA components for each pixel in the image. A reduction process is used to progressively downsample the source image on the GPU, followed by a short averaging calculation on the CPU. The output from this filter is meaningless, but you need to set the colorAverageProcessingFinishedBlock property to a block that takes in four color components and a frame time and does something with them.
- **GPUImageLuminosity**： Like the GPUImageAverageColor, this reduces an image to its average luminosity. You need to set the luminosityProcessingFinishedBlock to handle the output of this filter, which just returns a luminosity value and a frame time.
- **GPUImageChromaKeyFilter**：For a given color in the image, sets the alpha channel to 0. This is similar to the GPUImageChromaKeyBlendFilter, only instead of blending in a second image for a matching color this doesn't take in a second image and just turns a given color transparent.












