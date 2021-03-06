## Android real-time effect filter rtmp streaming library <br/> 安卓实时滤镜RTMP推流库

This project uses Android lastest [MediaCodec API](https://developer.android.com/reference/android/media/MediaCodec.html) for video/audio encoding and popular C ibrary librtmp (source code included) for rtmp streaming, in addionion, provides ability to implement real-time effect filters after camera capturing phase and before encoding phase. Some features are:

- Support Android 4.3 and higher (Android 6.0/7.0-preview tested)
- Filters support soft mode (CPU processing) and hard mode (GPU/OpenGLES rendering)
- Soft mode filter can be implemented by processing NV21 image array captured from camera  
- Hard mode filter can be implemented by rendering image texture captured from camera
- Support pixel rotation and flip
- Support encoding paramaters like resolution(depend on device), bitrate, etc
- H264 profile is auto-adjusted on different device
- Fast front/rear camera swaping without interrupt rtmp streaming

### Soft Filter Mode

- Color format is NV21
- Create your own filter by extending BaseSoftVideoFilter 
- Filter effect can be manipulated via java, ndk or renderscript
- Adjust your filter properties between acquireSoftVideoFilter and releaseSoftVideoFilter, call releaseSoftVideoFilter as soon as operation finished (after acquireSoftVideoFilter), interval better be less than 3ms


### Hard Filter Mode

- Create your own filter by extending either BaseHardVideoFilter or OriginalHardVideoFilter
- OriginalHardVideoFilter is more simple to use, just need to proivde custmom shader to get  various effects
- BaseHardVideoFilter is more powerful and flexible and need write openGLES code by yourself. You can theoretically achieve any effects you want via BaseHardVideoFilter.
- Adjust your filter properties between acquireHardVideoFilter and releaseHardVideoFilter, call releaseHardVideoFilter as soon as operation finished (after acquireHardVideoFilter), interval better be less than 3ms

### Start/stop recording and streaming:
```java
    @Override
    public void onCreate(final Bundle savedInstanceState) {
        ... ...
        RESConfig resConfig = RESConfig.obtain();
        resConfig.setFilterMode(RESConfig.FilterMode.SOFT);
        resConfig.setTargetVideoSize(new Size(720, 480));
        resConfig.setBitRate(1000 * 1024);
        resConfig.setRenderingMode(RESConfig.RenderingMode.OpenGLES);//setrender mode in softmode
        resConfig.setDefaultCamera(Camera.CameraInfo.CAMERA_FACING_FRONT);
        if (!resClient.prepare(resConfig)) {
            Log.e("Main", "prepare,failed!!");
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        resClient.start();//start streaming & preview
    }

    @Override
    protected void onPause() {
        super.onPause();
        resClient.stop();//stop streaming & preview
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        resClient.destroy();
    }
```

### Set recording direction:
```java
    if (this.getResources().getConfiguration().orientation == Configuration.ORIENTATION_PORTRAIT) {
        resConfig.setFrontCameraDirectionMode(RESConfig.DirectionMode.FLAG_DIRECTION_ROATATION_90 | RESConfig.DirectionMode.FLAG_DIRECTION_FLIP_HORIZONTAL);
        resConfig.setBackCameraDirectionMode(RESConfig.DirectionMode.FLAG_DIRECTION_ROATATION_90);
    } else {
        resConfig.setFrontCameraDirectionMode(RESConfig.DirectionMode.FLAG_DIRECTION_FLIP_HORIZONTAL);
    }
```

### Add filter:
```java
    BlackWhiteSoftFilter bwsFilter = new BlackWhiteSoftFilter();
    resClient.setSoftVideoFilter(bwsFilter);
    BaseSoftVideoFilter filter = resClient.acquireSoftVideoFilter();
    if (filter != null) {
        if (filter instanceof BlackWhiteFilterSoft) {
            BlackWhiteFilterSoft bwsFilter = (BlackWhiteFilterSoft) filter;
            blackWhiteFilter.setGap((byte) 128); // 0 ~ 255
        }
    ... ...
    resClient.releaseSoftVideoFilter();
```

#### For more information, please checkout sample code in [me.lake.librestreaming.sample.softfilter](https://github.com/lakeinchina/librestreaming/tree/master/sample/src/main/java/me/lake/librestreaming/sample/softfilter) and [me.lake.librestreaming.sample.hardfilter](https://github.com/lakeinchina/librestreaming/tree/master/sample/src/main/java/me/lake/librestreaming/sample/hardfilter).

### 简介：
- 支持cpu滤镜和gpu滤镜。
- 硬编码基于Mediacodec。
- 推流基于librtmp。

### 特性：
- 支持cpu滤镜，并可以通过处理图像数组来自定义滤镜。
- 支持gpu滤镜，并可以通过opengles绘制图像纹理来自定义滤镜。
- 前后摄像头快速切换，不会打断推流。
- 可以选择图像大小，码流比特率，具体取决于设备支持。


### 关于滤镜：

支持两种滤镜模式：使用cpu的软模式和使用gpu(opengles)的硬模式，硬模式效率通常高于软模式。

软模式下：

- 你可以通过继承BaseSoftVideoFilter创建自定义滤镜。
- sample.softfilter中有一些使用Java，NDK和RenderScript来处理图像的滤镜示例。
- 在acquireSoftVideoFilter和releaseSoftVideoFilter之间可以安全的修改滤镜的属性，不要持有滤镜超过3毫秒。
- 滤镜中的颜色格式为NV21。

硬模式下：

- 你可以通过继承BaseHardVideoFilter或者OriginalHardVideoFilter来创建自定义滤镜。
- sample.hardfilter中有一些示例。
- 继承OriginalHardVideoFilter只需要通过提供Shader就可以容易的实现大部分效果。
- 继承BaseHardVideoFilter需要自己处理opengl细节，但是可以实现任何效果。
- 在acquireHardVideoFilter和releaseHardVideoFilter之间可以安全的修改滤镜的属性。不要持有滤镜超过3毫秒。


### 关于硬编码(mediacodec)：

- 软模式下使用buffer to buffer。
- 硬模式下使用surface to surface。
