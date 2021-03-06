# Bug info

To reproduce this, note you have to have a reader (f.e. Wisepad 3). Click on the Discover button.

I've used kotlin to test. Update your stripe backend URL.

No readers are discovered when minifyEnabled and shrinkResources are true in Example/kotlinapp/build.gradle

# Stripe Terminal Android <img src="https://img.shields.io/badge/Beta-yellow.svg">

For information on migrating from earlier beta versions of the Android SDK, see the [Stripe Terminal Beta Migration Guide](https://stripe.com/docs/terminal/beta-migration-guide).

# Requirements

The Stripe Terminal Android SDK is compatible with apps supporting Android API level 24 and above. Apps can be written using Kotlin or [Java 8](https://developer.android.com/studio/write/java8-support).

# Try the example app

The Stripe Terminal Android SDK includes two open-source example apps (one in Java and the other in Kotlin), which you can use to familiarize yourself with the SDK before starting your own integration. To get started with the example app, clone the repo from \[Github\](https://github.com/stripe/stripe-terminal-android).

To build the example app:

1. Import the `Example` project into Android Studio
2. Navigate to our [example backend](https://github.com/stripe/example-terminal-backend) and click the button to deploy it on Heroku.
2. In `ApiClient.kt` (or `ApiClient.java` if you're using the Java example), set the URL of the Heroku app you just deployed
3. Build and run the app. The app includes a reader simulator, so you have no need for a physical reader to start your integration. Note that while the example app will work in an Android emulator, you will only be able to connect to a simulated reader due to lack of bluetooth capabilities. 

## Installation
In order to use the Android version of the Terminal SDK, you first have to add the SDK to the `dependencies` block of your `build.gradle` file:


    dependencies {
      implementation "com.stripe:stripeterminal:1.0.17"
    }
    
Next, since the SDK relies on Java 8, you’ll need to specify that as your target Java version (also in `build.gradle`:


    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

### Configure your app

Location access must be enabled in order to use the SDK. You’ll need to make sure that the `ACCESS_COARSE_LOCATION` permission is enabled in your app. To do this, add the following check before you initialize the `Terminal` object:

```java
if (ContextCompat.checkSelfPermission(getActivity(), 
  Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
    String[] permissions = {Manifest.permission.ACCESS_COARSE_LOCATION};
        
    // REQUEST_CODE should be defined on your app level
    ActivityCompat.requestPermissions(getActivity(), permissions, REQUEST_CODE);
}
```

 You should also verify that the user allowed the location permission, since the SDK won’t function without it. To do this, override the `onRequestPermissionsResult` method in your app and check the permission result.

```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    if (requestCode == REQUEST_CODE_LOCATION && grantResults.length > 0
            && grantResults[0] != PackageManager.PERMISSION_GRANTED) {
        throw new RuntimeException("Location services are required in order to " +
                "connect to a reader.");
    }
}
```


> Note: Stripe needs to know where payments occur to reduce risks associated with those charges and to minimize disputes. If the SDK can’t determine the Android device’s location, payments are disabled until location access is restored.

### Have an Application Class

As of RC1, we've worked hard to make our SDK lifecycle aware. In order to prevent memory leaks and enable proper cleaning up of long running Terminal processes, your application needs to have a subclass of `Application`, where the `TerminalLifeCycleObserver` is configured. Notably, it needs to both register the activity lifecycle callbacks as well as implement the `onTrimMemory` method, so that if your application is ever running low on memory we can suitably prune our memory usage and keep your app responsive for users! Check out the example app for how to do this:

```kotlin
// Substitue with your application name
class StripeTerminalApplication : Application() {
    private val observer: TerminalLifecycleObserver = TerminalLifecycleObserver.getInstance()

    override fun onCreate() {
        super.onCreate()

        // Register our observer for all lifecycle hooks!
        registerActivityLifecycleCallbacks(observer)
    }

    // Don't forget to let the observer know if your application is running low on memory!
    override fun onTrimMemory(level: Int) {
        super.onTrimMemory(level)
        observer.onTrimMemory(level, this)
    }
}
```

Lastly, don't forget to set your Application class in your `AndroidManifest.xml` accordingly. See the following taken from the example app:

```xml
<application
    android:name=".StripeTerminalApplication" // Or whatever your application class name is
    android:allowBackup="false"
    android:icon="@mipmap/launcher"
    android:label="@string/app_name"
    android:supportsRtl="true"
    android:theme="@style/Theme.Example"
    tools:ignore="GoogleAppIndexingWarning">
    <activity android:name="com.stripe.example.MainActivity"
        android:screenOrientation="fullSensor">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />

            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
</application>
```


## Documentation
 - [Getting Started](https://stripe.com/docs/terminal/sdk/android)
 - [API Reference](https://stripe.dev/stripe-terminal-android)


