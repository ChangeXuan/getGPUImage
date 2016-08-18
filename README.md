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

















