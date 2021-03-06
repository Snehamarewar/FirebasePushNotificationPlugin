# Getting Started

## Setup
* NuGet: [Plugin.FirebasePushNotification](http://www.nuget.org/packages/Plugin.FirebasePushNotification) [![NuGet](https://img.shields.io/nuget/v/Plugin.FirebasePushNotification.svg?label=NuGet)](https://www.nuget.org/packages/Plugin.FirebasePushNotification/)
* `PM> Install-Package Plugin.FirebasePushNotification`
* Install into ALL of your projects, include client projects.

## Getting Started Video

Here a step by step video to get started by setting up everything in your project:

<a href="https://youtu.be/FrxPEMLfV-o" target="_blank">Getting started with Firebase Push Notifications Plugin</a>

[![Getting Started Video](https://img.youtube.com/vi/FrxPEMLfV-o/0.jpg)](https://youtu.be/FrxPEMLfV-o)]


## Starting with Android

### Android Configuration

Edit AndroidManifest.xml and insert the following receiver elements **inside** the **application** section:

```xml
<application android:label="MyAppName" android:icon="@drawable/icon">
    <receiver 
        android:name="com.google.firebase.iid.FirebaseInstanceIdInternalReceiver" 
        android:exported="false" />
    <receiver 
        android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver" 
        android:exported="true" 
        android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
            <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
            <category android:name="${applicationId}" />
        </intent-filter>
    </receiver>
</application>
```
Also add this permission:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Add google-services.json to Android project. Make sure build action is GoogleServicesJson

![ADD JSON](https://github.com/CrossGeeks/FirebasePushNotificationPlugin/blob/master/images/android-googleservices-json.png?raw=true)

Must compile against 24+ as plugin is using API 24 specific things. Here is a great breakdown: http://redth.codes/such-android-api-levels-much-confuse-wow/ (Android project must be compiled using 7.0+ target framework)

### Android Initialization

You should initialize the plugin on an Android Application class if you don't have one on your project, should create an application class. Then call **FirebasePushNotificationManager.Initialize** method on OnCreate.

There are 3 overrides to **FirebasePushNotificationManager.Initialize**:

- **FirebasePushNotificationManager.Initialize(Context context, bool resetToken)** : Default method to initialize plugin without supporting any user notification categories. Uses a DefaultPushHandler to provide the ui for the notification.

- **FirebasePushNotificationManager.Initialize(Context context, NotificationUserCategory[] categories, bool resetToken)**  : Initializes plugin using user notification categories. Uses a DefaultPushHandler to provide the ui for the notification supporting buttons based on the action_click send on the notification

- **FirebasePushNotificationManager.Initialize(Context context,IPushNotificationHandler pushHandler, bool resetToken)** : Initializes the plugin using a custom push notification handler to provide custom ui and behaviour notifications receipt and opening.

**Important: While debugging set resetToken parameter to true.**

Example of initialization:

```csharp

    [Application]
    public class MainApplication : Application
    {
        public MainApplication(IntPtr handle, JniHandleOwnership transer) :base(handle, transer)
        {
        }

        public override void OnCreate()
        {
            base.OnCreate();
            
            //If debug you should reset the token each time.
            #if DEBUG
              FirebasePushNotificationManager.Initialize(this,true);
            #else
              FirebasePushNotificationManager.Initialize(this,false);
            #endif

              //Handle notification when app is closed here
              CrossFirebasePushNotification.Current.OnNotificationReceived += (s,p) =>
              {


              };
         }
    }

```

On your main launcher activity **OnCreate** method (On the Activity you got MainLauncher= true set)

```csharp
 FirebasePushNotificationManager.ProcessIntent(Intent);
 ```

 **Note: When using Xamarin Forms do it just after LoadApplication call.**

## Starting with iOS 

### iOS Configuration

 Add GoogleService-Info.plist to iOS project. Make sure build action is BundleResource

![ADD Plist](https://github.com/CrossGeeks/FirebasePushNotificationPlugin/blob/master/images/iOS-googleservices-plist.png?raw=true)

On Info.plist enable remote notification background mode

![Remote notifications](https://github.com/CrossGeeks/FirebasePushNotificationPlugin/blob/master/images/iOS-enable-remote-notifications.png?raw=true)

Add FirebaseAppDelegateProxyEnabled in the app’s Info.plist file and set it to No 

![Disable Swizzling](https://github.com/CrossGeeks/FirebasePushNotificationPlugin/blob/master/images/iOS-disable-swizzling.png?raw=true)

### iOS Initialization

There are 3 overrides to **FirebasePushNotificationManager.Initialize**:

- **FirebasePushNotificationManager.Initialize(NSDictionary options,bool autoRegistration)** : Default method to initialize plugin without supporting any user notification categories. Auto registers for push notifications if second parameter is true.

- **FirebasePushNotificationManager.Initialize(NSDictionary options, NotificationUserCategory[] categories)**  : Initializes plugin using user notification categories to support iOS notification actions.

- **FirebasePushNotificationManager.Initialize(NSDictionary options,IPushNotificationHandler pushHandler)** : Initializes the plugin using a custom push notification handler to provide native feedback of notifications event on the native platform.


Call  **FirebasePushNotificationManager.Initialize** on AppDelegate FinishedLaunching
```csharp

FirebasePushNotificationManager.Initialize(options,true);

```
 **Note: When using Xamarin Forms do it just after LoadApplication call.**

Also should override these methods and make the following calls:
```csharp
        public override void RegisteredForRemoteNotifications(UIApplication application, NSData deviceToken)
        {
            #if DEBUG
                    FirebasePushNotificationManager.DidRegisterRemoteNotifications(deviceToken, FirebaseTokenType.Sandbox);
            #endif
            #if RELEASE
                    FirebasePushNotificationManager.DidRegisterRemoteNotifications(deviceToken,FirebaseTokenType.Production);
            #endif

        }

        public override void FailedToRegisterForRemoteNotifications(UIApplication application, NSError error)
        {
            FirebasePushNotificationManager.RemoteNotificationRegistrationFailed(error);

        }
        // To receive notifications in foregroung on iOS 9 and below.
        // To receive notifications in background in any iOS version
        public override void DidReceiveRemoteNotification(UIApplication application, NSDictionary userInfo, Action<UIBackgroundFetchResult> completionHandler)
        {
            // If you are receiving a notification message while your app is in the background,
            // this callback will not be fired 'till the user taps on the notification launching the application.

            // If you disable method swizzling, you'll need to call this method. 
            // This lets FCM track message delivery and analytics, which is performed
            // automatically with method swizzling enabled.
            FirebasePushNotificationManager.DidReceiveMessage(userInfo);
            // Do your magic to handle the notification data
            System.Console.WriteLine(userInfo);
        }

        public override void OnActivated(UIApplication uiApplication)
        {
            FirebasePushNotificationManager.Connect();
           
        }
        public override void DidEnterBackground(UIApplication application)
        {
            // Use this method to release shared resources, save user data, invalidate timers and store the application state.
            // If your application supports background exection this method is called instead of WillTerminate when the user quits.
            FirebasePushNotificationManager.Disconnect();
        }
```


## Using Firebase Push Notification APIs
It is drop dead simple to gain access to the FirebasePushNotification APIs in any project. All you need to do is get a reference to the current instance of IFirebasePushNotification via `CrossFirebasePushNotification.Current`:

### Events

Once token is registered/refreshed you will get it on **OnTokenRefresh** event.


```csharp
   /// <summary>
   /// Event triggered when token is refreshed
   /// </summary>
    event FirebasePushNotificationTokenEventHandler OnTokenRefresh;
```

```csharp        
  /// <summary>
  /// Event triggered when a notification is received
  /// </summary>
  event FirebasePushNotificationResponseEventHandler OnNotificationReceived;
```


```csharp        
  /// <summary>
  /// Event triggered when a notification is opened
  /// </summary>
  event FirebasePushNotificationResponseEventHandler OnNotificationOpened;
```

```csharp        
  /// <summary>
  /// Event triggered when there's an error
  /// </summary>
  event FirebasePushNotificationErrorEventHandler OnNotificationError;
```

Token event usage sample:
```csharp

  CrossFirebasePushNotification.Current.OnTokenRefresh += (s,p) =>
  {
        System.Diagnostics.Debug.WriteLine($"TOKEN : {p.Token}");
  };

```

Push message received event usage sample:
```csharp

  CrossFirebasePushNotification.Current.OnNotificationReceived += (s,p) =>
  {
 
        System.Diagnostics.Debug.WriteLine("Received");
    
  };

```

Push message opened event usage sample:
```csharp
  
  CrossFirebasePushNotification.Current.OnNotificationOpened += (s,p) =>
  {
                System.Diagnostics.Debug.WriteLine("Opened");
                foreach(var data in p.Data)
                {
                    System.Diagnostics.Debug.WriteLine($"{data.Key} : {data.Value}");
                }

                if(!string.IsNullOrEmpty(p.Identifier))
                {
                    System.Diagnostics.Debug.WriteLine($"ActionId: {p.Identifier}");
                }
             
 };
```




<= Back to [Table of Contents](../README.md)
