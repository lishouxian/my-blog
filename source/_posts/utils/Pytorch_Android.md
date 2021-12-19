---
title: 'PyTorch图像分割模型安卓部署'
date: 2020-08-19 23:00:00
tags: [深度学习]
categories: utils
published: true
hideInList: true
feature: 
isTop: false
---

## 软件及版本信息

在PyTorch官网中有如下建议

> We recommend you to open this project in [Android Studio 3.5.1+](https://developer.android.com/studio). At the moment PyTorch Android and demo applications use [android gradle plugin of version 3.5.0](https://developer.android.com/studio/releases/gradle-plugin#3-5-0), which is supported only by Android Studio version 3.5.1 and higher. Using Android Studio you will be able to install Android NDK and Android SDK with Android Studio UI.

建议的[Android Studio](https://developer.android.com/studio)版本应该在3.5.1及以上，这里推荐使用最新版。

建议的自动化构建工具[android gradle plugin of version](https://developer.android.com/studio/releases/gradle-plugin#3-5-0)版本应该为3.5.0，**请注意**，不要将`gradle`的版本升级至`4.0.1`，在开发过程中发现该版本不太稳定。

<!-- more -->

## 模型修改及依赖导入

我们通过`model.save`保存的模型不能直接在Java程序中使用，需要使用`torch.jit.trace(model, example)`命令将模型从Java程序不可读取转为可读取。**请注意**这一步如果未处理会导致读取文件失败。

```Python
import torch
import torchvision

model = torchvision.models.resnet18(pretrained=True)
model.eval()
example = torch.rand(1, 3, 224, 224)
traced_script_module = torch.jit.trace(model, example)
traced_script_module.save("app/src/main/assets/model.pt")
```

在安卓程序`build.gradle`配置中加入如下依赖，**请注意**这里的`org.pytorch:pytorch_android`和`org.pytorch:pytorch_android_torchvision`需要更新到最新版。即下图的设置是无法读取PyTorch版本大于`1.4`的模型。

```json
repositories {
    jcenter()
}

dependencies {
    implementation 'org.pytorch:pytorch_android:1.4.0'
    implementation 'org.pytorch:pytorch_android_torchvision:1.4.0'
}
```

## 预处理图像获取

在安卓程序中我们一般通过两种方式来获取图像，分别是通过调用相机和通过调用相册。

### 调用相机

通过`MediaStore.ACTION_IMAGE_CAPTURE`来调用系统相机

```java
int cameraRequestCode = 001;
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    Button capture = findViewById(R.id.capture);
    capture.setOnClickListener(new View.OnClickListener(){
        @Override
        public void onClick(View view){
            Intent cameraIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
            startActivityForResult(cameraIntent,cameraRequestCode);
        }
    });
}
```

在匹配结果后通过`data.getExtras()`获取相机返回的`android.graphics.Bitmap`对象

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == cameraRequestCode && resultCode == RESULT_OK) {
        Intent resultView = new Intent(this, Result.class);
        resultView.putExtra("imagedata", data.getExtras());
        startActivity(resultView);
    }
}
```

### 调用系统相册

根据安卓版本不同，权限管理之间的差异，这一步比较复杂，需要获取照片真实地址，略。

## 模型读取和图像处理

### 预处理

第一步、通过上述两种方式获取图像。

第二步、将获取到的图像转为`android.graphics.Bitmap`对象

> As a first step we read `image.jpg` to `android.graphics.Bitmap` using the standard Android API.

第三部、模型读取：

1、在`Utils`工具类中增加如下静态方法`assetFilePath`，这个方法的作用是通过文件名使用文件流的方式读取模型文件。

```Java
    public static String assetFilePath(Context context, String assetName) {
        File file = new File(context.getFilesDir(), assetName);

        try (InputStream is = context.getAssets().open(assetName)) {
            try (OutputStream os = new FileOutputStream(file)) {
                byte[] buffer = new byte[4 * 1024];
                int read;
                while ((read = is.read(buffer)) != -1) {
                    os.write(buffer, 0, read);
                }
                os.flush();
            }
            return file.getAbsolutePath();
        } catch (IOException e) {
            Log.e("pytorchandroid", "Error process asset " + assetName + " to file path");
        }
        return null;
    }
```

2、读取模型，**请注意**，这里的`assetName`为`assert`文件夹下的模型文件名称。

```Java
Module module = null;
final String moduleFileAbsoluteFilePath = new File(
        Objects.requireNonNull(Utils.assetFilePath(this, "example.pt"))).getAbsolutePath();
module = Module.load(moduleFileAbsoluteFilePath);
```

### 模型计算

第四步、预处理为输入`Tensor`对象

```Java
Tensor inputTensor = TensorImageUtils.bitmapToFloat32Tensor(bitmap,
    TensorImageUtils.TORCHVISION_NORM_MEAN_RGB, TensorImageUtils.TORCHVISION_NORM_STD_RGB);
```

第五步、调用模型得到输出`Tensor`对象

```Java
Tensor outputTensor = module.forward(IValue.from(inputTensor)).toTensor();
float[] scores = outputTensor.getDataAsFloatArray();
```

第六步、将输出`Tensor`对象重新转为`android.graphics.Bitmap`对象

```Java
final float[] scores = outputTensor.getDataAsFloatArray();


Bitmap test = Bitmap.createBitmap(256, 256, Bitmap.Config.ARGB_8888);

int[] pixels = new int[256 * 256];
for (int i = 0; i < 256 * 256; ++i) {
    //关键代码，生产灰度图
    pixels[i] = (int) (scores[i] * 50);

}
test.setPixels(pixels, 0, 256, 0, 0, 256, 256);
```

## 输出UI设计

主要需要注意的部分是`android:adjustViewBounds="true"`将两个`ImageView`控件都设置为相同位置，相同大小的正方形，这样结果部分的图像才能重合。

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/black">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:adjustViewBounds="true"
        android:src="@drawable/ic_launcher_background"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        android:id="@+id/image"
        />

    <ImageView
        android:id="@+id/image2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:adjustViewBounds="true"
        android:src="@drawable/ic_launcher_background"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```