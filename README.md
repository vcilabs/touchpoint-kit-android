# touchpoint-kit-android

## About
- version: LATEST_TAG
- minSdkVersion supported: 23

## Installation
###### Import `touchpointkit-release.aar` file as a module in your project and add the `implementation project(path: ':touchpointkit-release')` in `build.gradle` file. Add following dependencies in build.gradle file.
```
// For Kotlin project
2implementation project(path: ':touchpointkit-release')
3implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
4implementation 'androidx.core:core-ktx:1.3.2'
5implementation 'androidx.appcompat:appcompat:1.2.0'
6implementation 'com.google.android.material:material:1.2.1'
7implementation 'com.android.volley:volley:1.1.1'
8implementation "androidx.startup:startup-runtime:1.0.0-rc01"
```

```
// For Java project
2implementation project(path: ':touchpointkit-release')
3implementation 'org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.2.31'
4implementation 'androidx.core:core-ktx:1.3.2'
5implementation 'androidx.appcompat:appcompat:1.2.0'
6implementation 'com.google.android.material:material:1.2.1'
7implementation 'com.android.volley:volley:1.1.1'
8implementation "androidx.startup:startup-runtime:1.0.0-rc01"
```

## Integration
###### Add following strings to `string.xml` file:
```
<string name="api_key">API_KEY</string>
<string name="api_secret">API_SECRET</string>
<string name="pod_name">DEV2</string> <!--POD_NAME value must be any of: NA1, NA2, EU1, EU2, AP2, DEV1, DEV2, SIT1, PUB -->
<bool name="disable_api_filter">true</bool> <!--This is optional. Default is false-->
<bool name="enable_debug_logs">true</bool> <!--This is optional. Default is false-->
<bool name="disable_all_logs">false</bool> <!--This is optional. Default is false-->
<bool name="disable_caching">false</bool> <!--This is optional. Default is false-->
```

###### Import TouchPoint SDK  `import com.visioncritical.touchpointkit.utils.TouchPointActivity` & add following code snippet on Activity's `onCreate` function:
```
// Kotlin
val screenNames:List<String> = listOf("Demo 1", "Demo 2")
val visitor: HashMap<String, String> = HashMap<String, String>()
visitor["id"] = "100"
visitor["email"] = "android@gmail.com"
visitor["role"] = "dev"
TouchPointActivity.shared.configure(screenNames, visitor)
```

```
// Java
List<String> screenNames =new ArrayList<String>();
screenNames.add("Demo 1");
screenNames.add("Demo 2");

HashMap<String, String> visitor = new HashMap<>();
visitor.put("id", "200");
visitor.put("email", "java@test.com");

TouchPointActivity.Companion.getShared().configure(screenNames, visitor)
```

###### In Activity, Import TouchPoint SDK  `import com.visioncritical.touchpointkit.utils.TouchPointActivity` & set the screen name using following code snippet:
```
// Kotlin
TouchPointActivity.shared.setCurrentScreen(ACTIVITY_CONTEXT, SCREEN_NAME)
```

```
// Java
TouchPointActivity.Companion.getShared().setCurrentScreen(ACTIVITY_CONTEXT, SCREEN_NAME)
```

###### This will lookup for any banner for current screen ( SCREEN_NAME) and display the banner automatically. If you want to open TouchPoint activity directly, you can directly call the following method:
```
// Kotlin
TouchPointActivity.shared.openActivityForScreen(ACTIVITY_CONTEXT, SCREEN_NAME, this)
```

```
// Java
TouchPointActivity.Companion.getShared().openActivityForScreen(ACTIVITY_CONTEXT, SCREEN_NAME, this)
```

###### `TouchPointActivityInterface` is required if you want the callback when the `TouchPointActivity` completed. Otherwise you can pass null in interface. Following is the `TouchPointActivityInterface` delegate method.
```
// Kotlin
override fun onTouchPointActivityFinished() {
    // Calls when TouchPoint activity is completed.
}
```

```
// Java
@Override void onTouchPointActivityFinished() {
    // Calls when TouchPoint activity is completed.
}
```

###### Before calling `openActivityForScreen` function, you can check if `TouchPoint activity` need to be shown or not using following method:
```
// Kotlin
if (TouchPointActivity.shared.shouldShowActivity(SCREEN_NAME)) {
    // Call openActivity function of TouchPointActivity
}
```

```
// Java
TouchPointActivity.Companion.getShared().shouldShowActivity(SCREEN_NAME)) {
    // Call openActivity function of TouchPointActivity
}
```
