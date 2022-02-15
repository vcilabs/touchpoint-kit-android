# Touchpoint Mobile SDK
The purpose of the Touchpoint mobile SDK is to create an integration between a mobile app and Touchpoint. This integration enables the ability to publish and manage the Touchpoint activities that are displayed in a mobile app without the need to release a new version of the mobile app or make any code changes.

## Minimum Requirements
- minSdkVersion supported: 23

## Sample App
https://github.com/vcilabs/touchpointkit-sample-android

## Glossary
- Touchpoint activity: an activity created by a user via the Touchpoint dashboard.
- Touchpoint distribution: a distribution defines how an activity is propagated amongst respondents and manages the publishing lifecycle. A distribution can have different types, in this case it would be a mobile distribution. A distribution is linked to a Touchpoint activity.
- Screen name: a unique label given to a specific screen/view/page on a mobile app. Some examples are: Login, Product List, Product Detail, or Settings. A screen can be assigned to a particular Touchpoint distribution so that different activities can be displayed in different screens in your app.
- Visitor: the end user interacting with the mobile app. The app is able to set any arbitrary attributes for the current visitor. Some examples are: email, an internal unique identifier, shoe size, etc. These attributes are sent to Touchpoint and can be exported by Touchpoint admins.
- Banner: a UI component that the SDK can render at any time on any screen without a need for code change to the mobile app. Clicking on a banner opens a Touchpoint activity inside a webview.  A banner is linked to both a Touchpoint distribution and a screen. A banner has three main attributes:
    - Style: what does the banner look like on the mobile app? The style object includes text colour and background colour.
    - Caption: the text that is shown on the banner.
    - Screen name: which screen in the mobile app should show which particular banner.
- Custom components: UI components developed by integrators of the mobile SDK in their own mobile app codebase. Clicking on or otherwise invoking these components will open a Touchpoint activity inside of a webview.
- Tracking Visitors: a mechanism on the SDK used to track the visitors’ interactions with activities rendered by the SDK. The purpose of which is to ensure visitors don’t see the banners and custom components again if they have already completed the activity. The SDK uses device IDs on iOS and Android to track visitors’ activity.

## Methods for Triggering Touchpoint Activities
There are two ways to trigger Touchpoint activities using the mobile SDK: banners and custom components.

### Banners
Banners are designed with minimizing the integrator’s development effort in mind. Once integrated into the app there are no further code changes required; swapping out activities on the various mobile app screens can be controlled without any code change by invoking APIs on Touchpoint.

A banner has the following properties:
1. Screen name: the screen in the mobile app on which the banner should be shown.
1. Caption: the caption text to render inside the banner.
1. Font colour: the colour of the text in the banner.
1. Background colour: the colour used for the banner’s main container.

#### Banners FAQ
- Q: What happens after the user closes the activity?<br/>
A: The user will stay on the same screen of the app without any change in the state of the app.
- Q: What happens if the user closes the activity without completing the activity?<br/>
A: The SDK won’t show the same banner again to the user and it is marked as “Activity Collapsed”, which means the user closed the activity without completing it.
- Q: What happens if the user taps the X to close the banner without opening the activity?<br/>
A: The SDK tracks this event as “Activity Collapsed” and won’t show the same banner again.
- Q: Can the position of the banner be changed?<br/>
A: Not at this time. All banners render on the bottom of the screen.
- Q: How many banners will users see at any time?<br/>
A: As a rule only one banner will be displayed on a screen at any one time. If there are multiple banners published to the same screen (which the SDK distinguishes using the screen name), the user sees only one banner on each visit. The next time they visit the same screen they will see the next banner.

### Custom Components 
Custom components are designed to enable as many different use cases as possible by giving more control to the integrator. Whereas banners have specific look, positioning, and targeting behaviours, custom components allow the integrator to write their own behavioural logic and UI components to handle these functions. This allows the integrator to fully align the look and feel of their app.

Custom components can use any lifecycle event in an app to trigger a Touchpoint activity, such as a button tap, a page load, a timer going off, etc. Any custom logic can run before triggering the activity as well, such as only triggering an activity if a visitor’s attributes match certain criteria.

#### Custom Components FAQ
- Q: How many custom components can be displayed on the same screen at the same time?<br/>
A: Only one custom component per screen. The SDK will open the activity assigned to the screen and since the screen is unique there can’t be more than one custom component on the same screen.
- Q: Can a custom component be persistent in that it will trigger the activity even if the visitor has already seen it?<br/>
A: This scenario is useful for a use case like a “collect feedback” button where you always want the user to be able to respond to the activity. There is an “always_show” property in distributions which forces the SDK to open the distribution regardless of how many times the visitor has completed/closed that activity.

## Integration

### Touchpoint Setup
Some initial setup is required to create various resources in Touchpoint. Currently the Touchpoint API must be used to create these resources.

#### API Key and Secret
The API key and secret are used to uniquely identify your app with Touchpoint. Make note of the api_key and api_secret in the response as those will be used later.

```bash
curl --location --request POST 'https://api-touchpoint.<pod>.visioncritical.com/dashboard/dashboard.Builder/MobileSDKCreate' \
--header 'x-application-id: <application_id>' \
--header 'Authorization: Bearer <jwt>' \
--header 'Content-Type: application/json' \
--data-raw '{
   "name":"<name>",
   "device_type":"<device_type>",
   "is_active":true
}'
```

Where:
- `pod`: the “pod” that the Touchpoint instance belongs to. This can be determined from the URL bar when you are logged into Touchpoint, such as https://app.eu2.visioncritical.com/touchpoint. Can be one of: `na1`, `na2`, `eu1`, `eu2`, `ap2`.
- `application_id`: this can be obtained by looking at the network traffic when logging in to Alida. Look for an API request like https://eu2.visioncritical.com/c/uishellcontext/appcontext?applicationId=<application_id>, the application ID is in the query string.
- `jwt`: this can be obtained by looking at the network traffic when logging in to Alida. Look for an API request like https://cc.eu2.visioncritical.com/session/validate and inspect the response. The token property in the response is the JWT.
- `name`: an arbitrary name to give this resource, just used for convenience so they can be easily differentiated.
- `device_type`: either `DEVICE_ANDROID` or `DEVICE_IOS`.

#### Distribution
This will create a mobile distribution that can be used by the mobile SDK. Make a note of the ID that is returned in the response, this will be used in subsequent requests.

```bash
curl --location --request POST 'https://api-touchpoint.<pod>.visioncritical.com/dashboard/dashboard.Builder/DistCreate' \
--header 'x-application-id: <application_id>' \
--header 'Authorization: Bearer <jwt>' \
--header 'Content-Type: application/json' \
--data-raw '{
   "activity_id":<activity_id>,
   "dist_name":"<name>",
   "dist_type":"DIST_MOBILE"
}'
```

Where:
- `pod`: the “pod” that the Touchpoint instance belongs to. This can be determined from the URL bar when you are logged into Touchpoint, such as https://app.eu2.visioncritical.com/touchpoint. Can be one of: `na1`, `na2`, `eu1`, `eu2`, `ap2`.
- `application_id`: this can be obtained by looking at the network traffic when logging in to Alida. Look for an API request like https://eu2.visioncritical.com/c/uishellcontext/appcontext?applicationId=<application_id>, the application ID is in the query string.
- `jwt`: this can be obtained by looking at the network traffic when logging in to Alida. Look for an API request like https://cc.eu2.visioncritical.com/session/validate and inspect the response. The token property in the response is the JWT.
- `name`: an arbitrary name to give this resource, just used for convenience so they can be easily differentiated.
- `activity_id`: the ID of the activity to be displayed. This can be obtained by navigating on the web to the report page for that activity. The location bar in the browser will have a number which is the activity ID. For example: https://app.eu2.visioncritical.com/touchpoint/activities/1902/report, in this case 1902.

#### Banners
A banner ties a distribution to a screen name. A screen name is required for both banner and custom component methods.

```bash
curl --location --request POST 'https://api-touchpoint.<pod>.visioncritical.com/dashboard/dashboard.Builder/MobileSDKBannerCreate' \
--header 'x-application-id: <application_id>' \
--header 'Authorization: Bearer <jwt>' \
--header 'Content-Type: application/json' \
--data-raw '{
   "api_key": "<api_key>",
   "caption_text": "<caption_text>",
   "dist_id": "<distribution_id>",
   "screen_name": "<screen_name>",
   "style": {
       "backgroundColor": "<bg_color>",
       "textColor": "<text_color>"
   },
   "use_banner_styling": <styling>
}'
```

Where:
- `pod`: the “pod” that the Touchpoint instance belongs to. This can be determined from the URL bar when you are logged into Touchpoint, such as https://app.eu2.visioncritical.com/touchpoint. Can be one of: `na1`, `na2`, `eu1`, `eu2`, `ap2`.
- `application_id`: this can be obtained by looking at the network traffic when logging in to Alida. Look for an API request like https://eu2.visioncritical.com/c/uishellcontext/appcontext?applicationId=<application_id>, the application ID is in the query string.
- `jwt`: this can be obtained by looking at the network traffic when logging in to Alida. Look for an API request like https://cc.eu2.visioncritical.com/session/validate and inspect the response. The token property in the response is the JWT.
- `api_key`: this is found in the response from the request made in the API Key and Secret section above.
- `caption_text`: the text that will appear in the banner, used only in the banner method.
- `distribution_id`: the ID found in the response from the request made in the Distribution section above.
- `screen_name`: the name of the screen this banner should appear on.
- `bg_color`: the hex value of the background colour, such as #00ff00. Used only in the banner method.
- `text_color`: the value of the text colour, such as #00ff00. Used only in the banner method.
- `styling`: a boolean value that specifies whether or not to use the specified style. Either true or false.

## Installation
Add the jitpack.io repository to the build.gradle or settings.gradle file. For example:

```gradle
dependencyResolutionManagement {
   repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
   repositories {
       google()
       mavenCentral()
       maven { url 'https://jitpack.io' }
   }
}
```

Then add the following dependencies to the build.gradle file:

```gradle
implementation 'com.github.vcilabs:touchpoint-kit-android:0.1.4'
implementation 'com.android.volley:volley:1.2.1'
implementation 'com.google.android.material:material:1.0.0'
implementation 'androidx.startup:startup-runtime:1.0.0'
```

## Implementation
Add the following to the strings.xml file:

```xml
<string name="api_key">API_KEY</string>
<string name="api_secret">API_SECRET</string>
<string name="pod_name">EU2</string> <!--POD_NAME value must be any of: NA1, NA2, EU1, EU2, AP2 -->
<bool name="disable_api_filter">true</bool> <!--This is optional. Default is false-->
<bool name="enable_debug_logs">true</bool> <!--This is optional. Default is false-->
<bool name="disable_all_logs">false</bool> <!--This is optional. Default is false-->
<bool name="disable_caching">false</bool> <!--This is optional. Default is false-->

```

Where:
- The values for `API_KEY`, and `API_SECRET` will come from the API calls made above.
- `pod_name`: the “pod” that the Touchpoint instance belongs to. This can be determined from the URL bar when you are logged into Touchpoint, such as https://app.eu2.visioncritical.com/touchpoint. Can be one of: `na1`, `na2`, `eu1`, `eu2`, `ap2`.
- `disable_api_filter`: used during testing to allow an activity to be shown to a user multiple times.
- `enable_debug_logs`: forces the SDK to produce additional logging output.
- `disable_all_logs`: prevents the SDK from producing any logs.
- `disable_caching`: prevents the SDK from caching resources.

Import the Touchpoint SDK using `import com.visioncritical.touchpointkit.utils.TouchPointActivity` and add the following initialization code. For example in the `onCreate` function of `MainActivity` or similar entry point.

```kotlin
// Kotlin
val screenNames:List<String> = listOf("Demo 1", "Demo 2")
val visitor: HashMap<String, String> = HashMap<String, String>()
visitor["id"] = "12345"
visitor["email"] = "android_sample@example.com"
visitor["favorite_food"] = "apples"

TouchPointActivity.shared.configure(screenNames, visitor)
```

```java
// Java
List<String> screenNames = new ArrayList<String>();
screenNames.add("Demo 1");
screenNames.add("Demo 2");

HashMap<String, String> visitor = new HashMap<>();
visitor.put("id", "12345");
visitor.put("email", "android_sample@example.com");
visitor.put("favorite_food", "apples");

TouchPointActivity.Companion.getShared().configure(screenNames, visitor)
```

Where
- `screenNames`: should match the screens defined when creating the banners in the API call above.
- `visitor`: defines the attributes of the current user of the app that are important to the integration with Touchpoint. These attributes are sent to Touchpoint when an activity is viewed. `id` should always be provided as that will uniquely identify the user.

Then in any Android Activity where a Touchpoint activity will be displayed using the banner method, import the Touchpoint SDK using `import com.visioncritical.touchpointkit.utils.TouchPointActivity` and set the screen name.

```kotlin
// Kotlin
TouchPointActivity.shared.setCurrentScreen(ACTIVITY_CONTEXT, SCREEN_NAME)
```

```java
// Java
TouchPointActivity.Companion.getShared().setCurrentScreen(ACTIVITY_CONTEXT, SCREEN_NAME)
```

This will perform a lookup for any banner for the specified screen (`SCREEN_NAME`) and display the banner automatically.

For the custom component method, directly call the following method:

```kotlin
// Kotlin
TouchPointActivity.shared.openActivityForScreen(ACTIVITY_CONTEXT, SCREEN_NAME, this)
```

```java
// Java
TouchPointActivity.Companion.getShared().openActivityForScreen(ACTIVITY_CONTEXT, SCREEN_NAME, this)
```

`TouchPointActivityInterface` is the third argument passed in to `openActivityForScreen` and is required if a callback is desired for when the `TouchPointActivity` has completed. Otherwise this can be `null`. The following is the `TouchPointActivityInterface` delegate method:

```kotlin
// Kotlin
override fun onTouchPointActivityFinished() {
    // Called when Touchpoint activity has completed.
}
```

```java
// Java
@Override void onTouchPointActivityFinished() {
    // Called when Touchpoint activity has completed.
}
```

Before calling the `openActivityForScreen` function, it is possible to check if a Touchpoint activity need to be shown or not using following method. This is useful if you have a custom UI component that should be rendered only if a Touchpoint activity is available.

```kotlin
// Kotlin
if (TouchPointActivity.shared.shouldShowActivity(SCREEN_NAME)) {
    // Call openActivity function of TouchPointActivity
}
```

```java
// Java
TouchPointActivity.Companion.getShared().shouldShowActivity(SCREEN_NAME)) {
    // Call openActivity function of TouchPointActivity
}
```
