Android Choosing Image from Camera or Gallery with Crop Functionality
===================
Android sample project demonstrating choosing an image from gallery or camera with the cropping functionality.

![Demo](https://www.androidhive.info/wp-content/uploads/2019/02/android-image-from-gallery-or-camera-with-crop-min.jpg)

| [Complete Tutorial](https://www.androidhive.info/2019/02/android-choosing-image-camera-gallery-crop-functionality/)      |  [Apk](http://download.androidhive.info/apk/image-pick-crop.apk) | [Video Demo](https://www.youtube.com/watch?v=8CgZd1d9ujc)|
|----------|--------|------|

uCrop
---
Thanks to [Yalantis](https://github.com/Yalantis) for providing such a beautiful cropping ([uCrop](https://github.com/Yalantis/uCrop)) library. This example uses the uCrop library for cropping functionality.

How to Use
===================
Follow the below simple steps to add the library into your project.

1. Add [file_provider.xml](https://github.com/ravi8x/Android-Image-Picker-and-Cropping/blob/master/app/src/main/res/xml/file_paths.xml) to your **res -> xml** foler.
```
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <external-cache-path
        name="cache"
        path="camera" />
</paths>
```
2. In [AndroidManifext.xml](https://github.com/ravi8x/Android-Image-Picker-and-Cropping/blob/master/app/src/main/AndroidManifest.xml), add **CAMERA**, **DEXTER** and **STORAGE** permissions. Add **UCrop** activity and **FileProvider** paths.
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="info.androidhive.imagepicker">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        tools:ignore="GoogleAppIndexingWarning">
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:screenOrientation="portrait"
            android:theme="@style/AppTheme.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- Image Picker activity -->
        <activity android:name=".ImagePickerActivity" />

        <!-- uCrop cropping activity -->
        <activity
            android:name="com.yalantis.ucrop.UCropActivity"
            android:screenOrientation="portrait"
            android:theme="@style/AppTheme.NoActionBar" />

        <!-- cache directory file provider paths -->
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="${applicationId}.provider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
    </application>

</manifest>
```
5. Add **Dexter** and **uCrop** dependencies to your **app/build.gradle**
```gradle
dependencies {
    //...

    implementation "com.karumi:dexter:5.0.0"
    implementation 'com.github.yalantis:ucrop:2.2.2'
}
```
6. Copy [ImagePickerActivity.java](https://github.com/ravi8x/Android-Image-Picker-and-Cropping/blob/master/app/src/main/java/info/androidhive/imagepicker/ImagePickerActivity.java) to your project.

7. Launch **ImagePickerActivity** by passing required intent data. Once the image is cropped, you can received the path of the cropped image in **onActivityResult** method.
```java
public class MainActivity extends AppCompatActivity {
    private static final String TAG = MainActivity.class.getSimpleName();
    public static final int REQUEST_IMAGE = 100;

    @BindView(R.id.img_profile)
    ImageView imgProfile;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    // my button click function
    void onProfileImageClick() {
        Dexter.withActivity(this)
                .withPermissions(Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE)
                .withListener(new MultiplePermissionsListener() {
                    @Override
                    public void onPermissionsChecked(MultiplePermissionsReport report) {
                        if (report.areAllPermissionsGranted()) {
                            showImagePickerOptions();
                        } else {
                            // TODO - handle permission denied case
                        }
                    }

                    @Override
                    public void onPermissionRationaleShouldBeShown(List<PermissionRequest> permissions, PermissionToken token) {
                        token.continuePermissionRequest();
                    }
                }).check();
    }

    private void showImagePickerOptions() {
        ImagePickerActivity.showImagePickerOptions(this, new ImagePickerActivity.PickerOptionListener() {
            @Override
            public void onTakeCameraSelected() {
                launchCameraIntent();
            }

            @Override
            public void onChooseGallerySelected() {
                launchGalleryIntent();
            }
        });
    }

    private void launchCameraIntent() {
        Intent intent = new Intent(MainActivity.this, ImagePickerActivity.class);
        intent.putExtra(ImagePickerActivity.INTENT_IMAGE_PICKER_OPTION, ImagePickerActivity.REQUEST_IMAGE_CAPTURE);

        // setting aspect ratio
        intent.putExtra(ImagePickerActivity.INTENT_LOCK_ASPECT_RATIO, true);
        intent.putExtra(ImagePickerActivity.INTENT_ASPECT_RATIO_X, 1); // 16x9, 1x1, 3:4, 3:2
        intent.putExtra(ImagePickerActivity.INTENT_ASPECT_RATIO_Y, 1);

        // setting maximum bitmap width and height
        intent.putExtra(ImagePickerActivity.INTENT_SET_BITMAP_MAX_WIDTH_HEIGHT, true);
        intent.putExtra(ImagePickerActivity.INTENT_BITMAP_MAX_WIDTH, 1000);
        intent.putExtra(ImagePickerActivity.INTENT_BITMAP_MAX_HEIGHT, 1000);

        startActivityForResult(intent, REQUEST_IMAGE);
    }

    private void launchGalleryIntent() {
        Intent intent = new Intent(MainActivity.this, ImagePickerActivity.class);
        intent.putExtra(ImagePickerActivity.INTENT_IMAGE_PICKER_OPTION, ImagePickerActivity.REQUEST_GALLERY_IMAGE);

        // setting aspect ratio
        intent.putExtra(ImagePickerActivity.INTENT_LOCK_ASPECT_RATIO, true);
        intent.putExtra(ImagePickerActivity.INTENT_ASPECT_RATIO_X, 1); // 16x9, 1x1, 3:4, 3:2
        intent.putExtra(ImagePickerActivity.INTENT_ASPECT_RATIO_Y, 1);
        startActivityForResult(intent, REQUEST_IMAGE);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        if (requestCode == REQUEST_IMAGE) {
            if (resultCode == Activity.RESULT_OK) {
                Uri uri = data.getParcelableExtra("path");
                try {
                    // You can update this bitmap to your server
                    Bitmap bitmap = MediaStore.Images.Media.getBitmap(this.getContentResolver(), uri);

                    // loading profile image from local cache
                    loadProfile(uri.toString());
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

Once the **Uri** is received, you can create a bitmap and send to your server or preview on the screen.

Intent Parameters:
==========
| Param      | Description |
| ----------- | ----------- |
| INTENT_IMAGE_PICKER_OPTION      | To define source of the image Camera or Gallery. The values could be **REQUEST_IMAGE_CAPTURE** or **REQUEST_GALLERY_IMAGE**       |
| INTENT_LOCK_ASPECT_RATIO   | Pass `true` to lock the aspect ratio. (16x9, 1x1, 3:4, 3:2)       |
| INTENT_ASPECT_RATIO_X   | Width of aspect ratio (ex: 3)        |
| INTENT_ASPECT_RATIO_Y   | Height of the aspect ratio (ex: 4)        |
| INTENT_IMAGE_COMPRESSION_QUALITY   | The image compression quality 0 - 100. The default value is 80        |
| INTENT_SET_BITMAP_MAX_WIDTH_HEIGHT   | Pass `true` to limit the maximum width and height of the bitmap        |
| INTENT_BITMAP_MAX_WIDTH   | The maximum width of the cropped image        |
| INTENT_BITMAP_MAX_HEIGHT   | The maximum height of the cropped image        |

If you want additional options, you can customize the image picker activity.

Deleting Cached Images:
==========
While the image are taken with camera, they will stored in cached directory. You can clear the cached images once the bitmap is utilized.
```java
// Clearing older images from cache directory
// don't call this line if you want to choose multiple images in the same activity
// call this once the bitmap(s) usage is over
ImagePickerActivity.clearCache(this);
```
