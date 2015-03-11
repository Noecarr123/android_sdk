## Summary

This is the Android SDK of adjust™. You can read more about adjust™ at
adjust.com.

## Basic Installation

These are the minimal steps required to integrate the adjust SDK into your
Android project. We are going to assume that you use Android Studio for your
Android development.

### 1. Get the SDK

Download the latest version from our [releases page][releases]. Extract the
archive in a folder of your choice.

### 2. Create the Adjust project

In the Android Studio menu select `File → Import Module...`.

![][import_module]

In the `Source directory` field, locate the folder you extracted in step 1.
Select and choose the folder `./android_sdk/Adjust/adjust`.  Make sure the
module name `:adjust` appears before finishing.

![][select_module]

The `adjust` module should be imported into your Android Studio project
afterwards.

### 3. Add the adjust library to your project

Open the `build.gradle` of your app and find the `dependencies` block. Add the
following line into it:

```
compile project(":adjust")
```

![][build_graddle]

### 4. Add permissions

#### Google Play Store

Since the 1st of August of 2014, apps in the Google Play Store must use the
[Google Advertising ID][google_ad_id] to uniquely identify devices. To allow
the adjust SDK to use the Google Advertising ID, you must integrate the [Google
Play Services][google_play_services]. If you haven't done it yet, follow this steps:

1. Open the `build.gradle` of your app and find the `dependencies` block. Add the
following line into it:

    ```
    compile 'com.google.android.gms:play-services:6.5.87'
    ```

    You can use instead the ads subset of the Google Play Services, to minimize the [dex 65k limit][multidex] issue.

    ```
    compile 'com.google.android.gms:play-services-ads:6.5.87'
    ```

2. In the Package Explorer open the `AndroidManifest.xml` of your Android project.
Add the following `meta-data` tag inside the `<application>` element.


    ```xml
    <meta-data android:name="com.google.android.gms.version"
           android:value="@integer/google_play_services_version" />
    ```

In the Package Explorer open the `AndroidManifest.xml` of your Android project.
Add the `uses-permission` tag for `INTERNET` if it's not present already.

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

If you are using Proguard, add these lines to your Proguard file:

```
-keep class com.adjust.sdk.** { *; }
-keep class com.google.android.gms.common.** { *; }
-keep class com.google.android.gms.ads.identifier.** { *; }
```

#### Other Stores

In the Package Explorer open the `AndroidManifest.xml` of your Android project.
Add the `uses-permission` tags for `INTERNET` and `ACCESS_WIFI_STATE` if they
aren't present already.

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
```

![][permissions]

If you are using Proguard, add these lines to your Proguard file:

```
-keep class com.adjust.sdk.** { *; }
```

### 5. Integrate Adjust into your app

To start with, we'll set up basic session tracking.

#### Basic Setup

We recommend using a global android [Application][android_application] class to
start the SDK. If don't already have one in your app, follow this steps:

1. Create one a class that extends `Application`.
2. Open the `AndroidManifest.xml` file and locate the `<application>` element.
3. Add the attribute `android:name` with the value equal to the created class name prefixed with a full stop, `.`.

Our example app has an `Application` class named `GlobalApplication`, so the manifest file is configured as:
```xml
 <application
   ...
   android:name=".GlobalApplication">
     ...
</application>
```
With your `Application` class, use the `onCreate` method to add the code that initializes the adjust SDK:

```java
import com.adjust.sdk.Adjust;
import com.adjust.sdk.AdjustConfig;

public class YourApplicationClass extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        // configure Adjust
        String appToken = "{YourAppToken}";
        String environment = AdjustConfig.ENVIRONMENT_SANDBOX;
        AdjustConfig config = new AdjustConfig(this, appToken, environment);
        Adjust.onCreate(config);
    }
}
```

![][settings]

Replace `{YourAppToken}` with your app token. You can find this in your
[dashboard].

Depending on whether you build your app for testing or for production, you must
set `environment` with one of these values:

```java
String environment = AdjustConfig.ENVIRONMENT_SANDBOX;
String environment = AdjustConfig.ENVIRONMENT_PRODUCTION;
```

**Important:** This value should be set to `AdjustConfig.ENVIRONMENT_SANDBOX`
if and only if you or someone else is testing your app. Make sure to set the
environment to `AdjustConfig.ENVIRONMENT_PRODUCTION` just before you publish
the app. Set it back to `AdjustConfig.ENVIRONMENT_SANDBOX` when you start
developing and testing it again.

We use this environment to distinguish between real traffic and test traffic
from test devices. It is very important that you keep this value meaningful at
all times! This is especially important if you are tracking revenue.

#### Adjust Logging

You can increase or decrease the amount of logs you see in tests by calling
`setLogLevel` on your `AdjustConfig` instance with one of the following
parameters:

```java
config.setLogLevel(LogLevel.VERBOSE);   // enable all logging
config.setLogLevel(LogLevel.DEBUG);     // enable more logging
config.setLogLevel(LogLevel.INFO);      // the default
config.setLogLevel(LogLevel.WARN);      // disable info logging
config.setLogLevel(LogLevel.ERROR];     // disable warnings as well
config.setLogLevel(LogLevel.ASSERT);    // disable errors as well
```

#### Add broadcast receiver

In your `AndroidManifest.xml`, add the following `receiver` tag inside the
`application` tag.

```xml
<receiver
    android:name="com.adjust.sdk.AdjustReferrerReceiver"
    android:exported="true" >
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
    </intent-filter>
</receiver>
```

![][receiver]

We use this broadcast receiver to retrieve the install referrer, in order to
improve conversion tracking.

If you are already using a different broadcast receiver for the
`INSTALL_REFERRER` intent, follow [these instructions][referrer] to add the
Adjust receiver.

### 6. Integrate adjust into your app

To provide proper session tracking it is required to call certain Adjust
methods every time any Activity resumes or pauses. Otherwise the SDK might miss
a session start or session end. In order to do so you should follow these steps
for **each** Activity of your app:

1. Open the source file of your Activity.
2. Add the `import` statement at the top of the file.
3. In your Activity's `onResume` method call `Adjust.onResume`. Create the
  method if needed.
4. In your Activity's `onPause` method call `Adjust.onPause`. Create the method
  if needed.

After these steps your activity should look like this:

```java
import com.adjust.sdk.Adjust;
// ...
public class YourActivity extends Activity {
    protected void onResume() {
        super.onResume();
        Adjust.onResume(this);
    }
    protected void onPause() {
        super.onPause();
        Adjust.onPause();
    }
    // ...
}
```

![][activity]

Repeat these steps for **every** Activity of your app. Don't forget these steps
when you create new Activities in the future. Depending on your coding style
you might want to implement this in a common superclass of all your Activities.

### 7. Build your app

Build and run your Android app. In your LogCat viewer you can set the filter
`tag:Adjust` to hide all other logs. After your app has launched you should see
the following Adjust log: `Install tracked`

![][log_message]

## Additional Features

Once you have integrated the adjust SDK into your project, you can take
advantage of the following features.

### 8. Add tracking of custom events

You can use adjust to track any event in your app. Suppose you want to track
every tap on a button. You would have to create a new event token in your
[dashboard]. Let's say that event token is `abc123`. In your button's `onClick`
method you could then add the following lines to track the click:

```java
AdjustEvent event = new AdjustEvent("abc123");
Adjust.trackEvent(event);
```

The event instance can be used to configure the event even more before tracking
it.

### 9. Add callback parameters

You can register a callback URL for your events in your [dashboard]. We will
send a GET request to that URL whenever the event gets tracked. You can add
callback parameters to that event by calling `addCallbackParameter` on the
event instance before tracking it. We will then append these parameters to your
callback URL.

For example, suppose you have registered the URL
`http://www.adjust.com/callback` then track an event like this:

```java
AdjustEvent event = new AdjustEvent("abc123");

event.addCallbackParameter("key", "value");
event.addCallbackParameter("foo", "bar");

Adjust.trackEvent(event);
```

In that case we would track the event and send a request to:

```
http://www.adjust.com/callback?key=value&foo=bar
```

It should be mentioned that we support a variety of placeholders like
`{android_id}` that can be used as parameter values. In the resulting callback
this placeholder would be replaced with the AndroidID of the current device.
Also note that we don't store any of your custom parameters, but only append
them to your callbacks. If you haven't registered a callback for an event,
these parameters won't even be read.

You can read more about using URL callbacks, including a full list of available
values, in our [callbacks guide][callbacks-guide].


### 10. Partner parameters

You can also add parameters to be transmitted to network partners, for the
integrations that have been activated in your adjust dashboard.

This works similarly to the callback parameters mentioned above, but can be
added by calling the `addPartnerParameter` method on your `AdjustEvent` instance.

```java
AdjustEvent event = new AdjustEvent("abc123");

event.addPartnerParameter("key", "value");
event.addPartnerParameter("foo", "bar");

Adjust.trackEvent(event);
```

You can read more about special partners and these integrations in our [guide
to special partners.][special-partners]

### 11. Add tracking of revenue

If your users can generate revenue by tapping on advertisements or making
in-app purchases you can track those revenues with events. Lets say a tap is
worth one Euro cent. You could then track the revenue event like this:

```java
AdjustEvent event = new AdjustEvent("abc123");
event.setRevenue(0.01, "EUR");
Adjust.trackEvent(event);
```

This can be combined with callback parameters of course.

Please note that while the currency token must be set, adjust will not perform
a currency conversion. **You should always transmit revenues in the currency
that you have set in your adjust dashboard.**

You can read more about revenue and event tracking in the [event tracking
guide.][event-tracking]

### 12. Set up deep link reattributions

You can set up the adjust SDK to handle deep links that are used to open your
app. We will only read certain adjust specific parameters. This is essential if
you are planning to run retargeting or re-engagement campaigns with deep links.

For each activity that accepts deep links, find the `onCreate` method and add
the folowing call to adjust:

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    Intent intent = getIntent();
    Uri data = intent.getData();
    Adjust.appWillOpenUrl(data);
    //...
}
```

### 13. Enable event buffering

If your app makes heavy use of event tracking, you might want to delay some
HTTP requests in order to send them in one batch every minute. You can enable
event buffering with your `AdjustConfig` instance:

```java
config.setEventBufferingEnabled(true);
```

### 14. Set listener for attribution changed

You can register a listener callback to be notified of tracker attribution
changes. Due to the different sources considered for attribution, this
information can not by provided synchronously. The simplest way is to create a
single anonymous listener:

Please make sure to consider our [applicable attribution data
policies.][attribution-data]

- With the `AdjustConfig` instance, before starting the SDK, add the anonymous listener:

    ```java
    config.setOnAttributionChangedListener(new OnAttributionChangedListener() {
        @Override
        public void onAttributionChanged(AdjustAttribution attribution) {
        }
    });
    ```

- Alternatively, you could implement the `OnAttributionChangedListener`
  interface in your `Application` class and pass `this`:

    ```java
    config.setOnAttributionChangedListener(this);
    ```

The listener function will be called when the SDK receives final attribution
data.  Within the listener function you have access to the `attribution`
parameter.  Here is a quick summary of its properties:

- `String trackerToken` the tracker token of the current install.
- `String trackerName` the tracker name of the current install.
- `String network` the network grouping level of the current install.
- `String campaign` the campaign grouping level of the current install.
- `String adgroup` the ad group grouping level of the current install.
- `String creative` the creative grouping level of the current install.

### 15. Disable tracking

You can disable the adjust SDK from tracking any activities of the current
device by calling `setEnabled` with parameter `false`. This setting is
remembered between sessions.

```java
Adjust.setEnabled(false);
```

You can check if the adjust SDK is currently enabled by calling the function
`isEnabled`. It is always possible to activate the adjust SDK by invoking
`setEnabled` with the enabled parameter as `true`.

[dashboard]:     http://adjust.com
[releases]:      https://github.com/adjust/adjust_android_sdk/releases
[import_module]: https://raw.github.com/adjust/adjust_sdk/master/Resources/android/v4/01_import_module.png
[select_module]: https://raw.github.com/adjust/adjust_sdk/master/Resources/android/v4/02_select_module.png
[build_graddle]: https://raw.github.com/adjust/adjust_sdk/master/Resources/android/v4/03_build_gradle.png
[permissions]:   https://raw.github.com/adjust/adjust_sdk/master/Resources/android/v4/04_permissions.png
[settings]:      https://raw.github.com/adjust/adjust_sdk/master/Resources/android/v4/05_settings.png
[receiver]:      https://raw.github.com/adjust/adjust_sdk/master/Resources/android/v4/06_receiver.png
[activity]:      https://raw.github.com/adjust/adjust_sdk/master/Resources/android/v4/07_activity.png
[log_message]:   https://raw.github.com/adjust/adjust_sdk/master/Resources/android/v4/08_log_message.png
[referrer]:      doc/referrer.md
[attribution-data]:     https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[google_play_services]: http://developer.android.com/google/play-services/setup.html
[android_application]:  http://developer.android.com/reference/android/app/Application.html
[application_name]:     http://developer.android.com/guide/topics/manifest/application-element.html#nm
[google_ad_id]:         https://developer.android.com/google/play-services/id.html
[callbacks-guide]:      https://docs.adjust.com/en/callbacks
[event-tracking]:       https://docs.adjust.com/en/event-tracking
[special-partners]:     https://docs.adjust.com/en/special-partners
[multidex]:             https://developer.android.com/tools/building/multidex.html

## License

The adjust SDK is licensed under the MIT License.

Copyright (c) 2012-2015 adjust GmbH,
http://www.adjust.com

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
