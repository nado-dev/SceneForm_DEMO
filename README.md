# SceneForm Demo

此项目根据视频教程开发而成

[相关视频](https://www.youtube.com/watch?v=2xkZUPUbgoM&t=422s) 内含教学视频和3D模型下载链接

官方文档 [ARCore SceneForm](https://developers.google.cn/ar/develop/java/sceneform)

## 效果

通过点击选择模型（demo中只提供了两个模型），识别出完整平面后点击即可放置模型。点击模型上方的textView可以取消放置。


## 开发简介

* Android Studio配置

  本项目开发使用Android Studio 3.5.1

  * 安装插件`File->setting->plugins Google SceneForm Tools`用于导入3D模型(obj等)

* 项目配置

  * 添加依赖`bulid.gradle(app)`->`dependences`-> `implementation 'com.google.ar.sceneform.ux:sceneform-ux:1.15.0'`

  * 添加相关权限 相机权限 和 设置AR权限（必须、可选）

    AndroidManifest.xml :

    ```xml
    <uses-permission android:name="android.permission.CAMERA"/>
        <uses-feature android:name="android.hardware.camera.ar" android:required="true"/>
    ```

    在`application`标签内增加

    ```xml
    <meta-data android:name="com.google.ar.core" android:value="required"/>
    ```

    

* 导入3D模型

  **注意**：原3D模型和导入后的3D资源不要放在Assets内，防止加入不必要的资源

  1. 创建目录app/sampledata，将obj等文件放入其中。

  2. 要导入的资源 -> 右键  -> `import SceneForm Assets`

     一个配置

     ![setting_import_3D](.\setting_import_3D-1585656232990.jpg)

* 设计UI/UX

  在需要AR的Activity内实现一个xml布局文件

  使用`FrameLayout`

  增加一个布局 注意命名空间使用` android:name="com.google.ar.sceneform.ux.ArFragment"`

  ```xml
  <fragment
          android:id="@+id/sceneform_fragment"
          android:name="com.google.ar.sceneform.ux.ArFragment"
          android:layout_width="match_parent"
          android:layout_height="match_parent"/>
  ```

  通过

  ```java
  arFragment = (ArFragment)getSupportFragmentManager().findFragmentById(R.id.sceneform_fragment);
  ```

  找到这个`ArFragment`

  

* 基本SceneForm逻辑

  1. 加载模型

     ```java
      ModelRenderable.builder()
                     .setSource(this, R.raw.bear)
                     .build().thenAccept(renderable -> bearRenderable = renderable)
                     .exceptionally(
                             throwable -> {
                                 Toast.makeText(this, "模型加载失败", Toast.LENGTH_SHORT).show();
                                 return null;
                             });
     ```

  2. 设置plane点击事件 用于识别出点击位置

     ```java
      arFragment.setOnTapArPlaneListener(new BaseArFragment.OnTapArPlaneListener() {
                 @Override
                 public void onTapPlane(HitResult hitResult, Plane plane, MotionEvent motionEvent) {
                     // when  user tap on plane,add model
                         Anchor anchor = hitResult.createAnchor();
                         AnchorNode anchorNode = new AnchorNode(anchor);
                         anchorNode.setParent(arFragment.getArSceneView().getScene());
     
                         createModel(anchorNode, selected);
     
                 }
             });
     ```

     **注意：** 各个锚点`AnchorNode`可以形成类似`View`的树状结构，通过树状结构的增删可以配置各个`AnchorNode`和依赖于它的`ModelRenderable`

     ​	所以移除一个模型的方法`anchorNode.setParent(null);`

     

  3. 关联锚点与模型

     ```java
      TransformableNode bear = new TransformableNode(arFragment.getTransformationSystem());
                 bear.setParent(anchorNode);
     
                 bear.setRenderable(bearRenderable);
     
                 bear.select();
     ```

     