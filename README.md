# OneSignal | Native extension for Adobe AIR (iOS &amp; Android)

Development of this extension is supported by [Master Tigra, Inc.](https://github.com/mastertigra)

## Features

* Receiving push notifications sent from [OneSignal dashboard](https://onesignal.com/)
* Managing user subscription
* Segmenting users using tags
* Posting notifications from device

## Native SDK versions

* iOS `v1.13.03`
* Android `v2.06.00`

## Getting started

Create an app in the [OneSignal dashboard](https://onesignal.com/). Single OneSignal app can be configured for both iOS and Android.
* To support Android, follow the [tutorial on how to obtain necessary information from Google](https://documentation.onesignal.com/docs/android-generating-a-gcm-push-notification-key).
* To support iOS, follow the [tutorial on how to properly setup your iOS certificates and provisioning profiles](https://documentation.onesignal.com/docs/generating-an-ios-push-certificate).

### Additions to AIR descriptor

First, add the extension's ID to the `extensions` element.

```xml
<extensions>
    <extensionID>com.marpies.ane.onesignal</extensionID>
</extensions>
```

If you are targeting Android, add the following extensions as well (unless you know these libraries are included by some other extensions):

```xml
<extensions>
    <extensionID>com.marpies.ane.androidsupport</extensionID>
    <extensionID>com.marpies.ane.googleplayservices.iid</extensionID>
    <extensionID>com.marpies.ane.googleplayservices.gcm</extensionID>
    <extensionID>com.marpies.ane.googleplayservices.analytics</extensionID>
    <extensionID>com.marpies.ane.googleplayservices.location</extensionID>
    <extensionID>com.marpies.ane.googleplayservices.base</extensionID>
    <extensionID>com.marpies.ane.googleplayservices.basement</extensionID>
</extensions>
```

For iOS support, look for the `iPhone` element and make sure it contains the following `InfoAdditions` and `Entitlements`:

```xml
<iPhone>
    <InfoAdditions>
        <![CDATA[
        ...

        <key>UIBackgroundModes</key>
        <array>
            <string>remote-notification</string>
        </array>

        <key>MinimumOSVersion</key>
        <string>7.0</string>
        ]]>
    </InfoAdditions>

    <Entitlements>
        <![CDATA[
            <key>aps-environment</key>
            <!-- Value below must be changed to 'production' when releasing for AppStore or Test Flight -->
            <string>development</string>
        ]]>
    </Entitlements>

    <requestedDisplayResolution>high</requestedDisplayResolution>
</iPhone>
```

For Android support, modify `manifestAdditions` element so that it contains the following:

```xml
<android>
    <manifestAdditions>
        <![CDATA[
        <manifest android:installLocation="auto">
            <!-- OneSignal permissions -->
            <permission android:name="{APP-PACKAGE-NAME}.permission.C2D_MESSAGE"
                        android:protectionLevel="signature" />
            <uses-permission android:name="{APP-PACKAGE-NAME}.permission.C2D_MESSAGE" />
            <uses-permission android:name="android.permission.INTERNET" />
            <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
            <uses-permission android:name="android.permission.WAKE_LOCK" />
            <uses-permission android:name="android.permission.VIBRATE" />
            <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
            <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

            <!-- START: ShortcutBadger -->
            <!-- Samsung -->
            <uses-permission android:name="com.sec.android.provider.badge.permission.READ"/>
            <uses-permission android:name="com.sec.android.provider.badge.permission.WRITE"/>
            <!-- HTC -->
            <uses-permission android:name="com.htc.launcher.permission.READ_SETTINGS"/>
            <uses-permission android:name="com.htc.launcher.permission.UPDATE_SHORTCUT"/>
            <!-- Sony -->
            <uses-permission android:name="com.sonyericsson.home.permission.BROADCAST_BADGE"/>
            <!-- Apex -->
            <uses-permission android:name="com.anddoes.launcher.permission.UPDATE_COUNT"/>
            <!-- Solid -->
            <uses-permission android:name="com.majeur.launcher.permission.UPDATE_BADGE"/>
            <!-- End: ShortcutBadger -->

            <application>

                <!-- OneSignal BEGIN -->
                <meta-data android:name="com.google.android.gms.version"
                            android:value="@integer/google_play_services_version" />
                <meta-data android:name="onesignal_app_id"
                            android:value="{ONE-SIGNAL-APP-ID}" />
                <meta-data android:name="onesignal_google_project_number"
                            android:value="str:{GOOGLE-SENDER-ID}" />

                <receiver android:name="com.onesignal.GcmBroadcastReceiver"
                            android:permission="com.google.android.c2dm.permission.SEND" >
                    <intent-filter>
                        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                        <category android:name="{APP-PACKAGE-NAME}" />
                    </intent-filter>
                </receiver>
                <receiver android:name="com.onesignal.NotificationOpenedReceiver" />
                <service android:name="com.onesignal.GcmIntentService" />
                <service android:name="com.onesignal.SyncService" android:stopWithTask="false" />
                <activity android:name="com.onesignal.PermissionsActivity" android:theme="@android:style/Theme.Translucent.NoTitleBar" />

                <service android:name="com.onesignal.NotificationRestoreService" />
                <receiver android:name="com.onesignal.BootUpReceiver">
                    <intent-filter>
                        <action android:name="android.intent.action.BOOT_COMPLETED" />
                        <action android:name="android.intent.action.QUICKBOOT_POWERON" />
                    </intent-filter>
                </receiver>
                <receiver android:name="com.onesignal.UpgradeReceiver">
                    <intent-filter>
                        <action android:name="android.intent.action.MY_PACKAGE_REPLACED" />
                    </intent-filter>
                </receiver>
                <!-- OneSignal END -->

            </application>

        </manifest>
        ]]>
    </manifestAdditions>
</android>
```

In the snippet above, replace:
* `{APP-PACKAGE-NAME}` with your app package name (value of `id` element in your AIR app descriptor)
* `{ONE-SIGNAL-APP-ID}` with your OneSignal app id
* `{GOOGLE-SENDER-ID}` with your Google Sender ID (also known as Google Project Number) obtained from [the tutorial](https://documentation.onesignal.com/docs/android-generating-a-gcm-push-notification-key)

### Custom Android icons

The native OneSignal SDK for Android uses your app's icon in the notification area. Starting with Android 5, the OS forces the notification icon to be all white when your app targets Android API 21+. If you do not make a correct small icon, it will most likely be displayed as a solid white square or circle in the status bar. Therefore it is recommended you provide custom icons and repackage the extension.

You will need to create small icons in 4 sizes and replace the ones in the [android project res directory](android/com.marpies.ane.onesignal-res/):

* [mdpi](android/com.marpies.ane.onesignal-res/drawable-mdpi-v11/ic_stat_onesignal_default.png) 24x24 pixels
* [hdpi](android/com.marpies.ane.onesignal-res/drawable-hdpi-v11/ic_stat_onesignal_default.png) 36x36 pixels
* [xhdpi](android/com.marpies.ane.onesignal-res/drawable-xhdpi-v11/ic_stat_onesignal_default.png) 48x48 pixels
* [xxhdpi](android/com.marpies.ane.onesignal-res/drawable-xxhdpi-v11/ic_stat_onesignal_default.png) 72x72 pixels

The [xxhdpi directory](android/com.marpies.ane.onesignal-res/drawable-xxhdpi-v11/) also contains colorful large icon of size 192x192 pixels. This icon is displayed together with the small icon when the notification area is swiped down. You can delete the large icon, in which case only the small icon will show up.

After you replace the icons, run `ant` from the [build directory](build/) to create updated extension package.

Finally, add the [OneSignal ANE](bin/com.marpies.ane.onesignal.ane) or [SWC](bin/com.marpies.ane.onesignal.swc) package from the [bin directory](bin/) to your project so that your IDE can work with it. The additional Android library ANEs are only necessary during packaging.

### API overview

#### Callbacks

To be notified when a notification is received, specify a callback method that accepts single parameter of type `OneSignalNotification`:

```as3
OneSignal.addNotificationReceivedCallback( onNotificationReceived );
...
private function onNotificationReceived( notification:OneSignalNotification ):void {
    // callback can be removed using OneSignal.removeNotificationReceivedCallback( onNotificationReceived );
    // process the notification
}
```

It is recommended to add the callback before initializing the extension to receive any notifications which result in launching your app.

You can also add a callback to be notified when a push notification token is available:

```as3
OneSignal.addTokenReceivedCallback( onPushTokenReceived );
...
private function onPushTokenReceived( oneSignalUserId:String, pushToken:String ):void {
    if( pushToken != null ) {
        OneSignal.removeTokenReceivedCallback( onPushTokenReceived );
    }
    // 'pushToken' may be null if there's a server or connection error
}
```

#### Initialization

Now proceed with ANE initialization by providing your OneSignal app ID. The two `Boolean` values that follow specify whether you want to:
* `autoRegister` - register for push notifications immediately after initialization (i.e. prompt iOS user to confirm notifications).
* `showLogs` - show extension debug logs.

The `init` method should be called in your document class' constructor, or as early as possible after your app's launch.

```as3
if( OneSignal.init( "{ONE-SIGNAL-APP-ID}", false, true ) ) {
    // successfully initialized
}
```

If `autoRegister` is set to `false`, you will need to call `OneSignal.register()` later at some point to attempt registration with the notification servers. Generally, it is recommended to avoid auto registration to provide better user experience for users who launch your app for the very first time.

#### Managing user subscription

You can opt users out of receiving all notifications through OneSignal using:

```as3
OneSignal.setSubscription( false );
```

You can pass `true` later to opt users back into notifications.

#### Tagging

By using tags you can segment your user base and create personalized notifications. Use one of the following methods to assign new or update an existing tag:

```as3
// key - value
OneSignal.sendTag( "profession", "warrior" );

// Or multiple tags at a time
OneSignal.sendTags( {
    "profession": "warrior",
    "area": "desert"
} );
```

Use one of the following methods to delete previously set tags:

```as3
OneSignal.deleteTag( "profession" );

// Or multiple tags at a time
OneSignal.deleteTags( new <String>["profession", "area"] );
```

Use the following method to retrieve the values current user has been tagged with:

```as3
OneSignal.getTags( onTagsRetrieved );
...
private function onTagsRetrieved( tags:Object ):void {
    // tags may be null if there's a connection error or user has not been tagged
    if( tags != null ) {
        trace( tags["profession"] ); // warrior
        trace( tags["area"] ); // desert
    }
}
```

## Requirements

* iOS 7+
* Android 4+
* Adobe AIR 20+

## Documentation
Generated ActionScript documentation is available in the [docs](docs/) directory, or can be generated by running `ant asdoc` from the [build](build/) directory.

## Build ANE
ANT build scripts are available in the [build](build/) directory. Edit [build.properties](build/build.properties) to correspond with your local setup.

## Author
The ANE has been written by [Marcel Piestansky](https://twitter.com/marpies) and is distributed under [Apache License, version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

## Changelog

#### July 20, 2016 (v0.8.0)

* Public release
