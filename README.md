# MinerAlt SDK for Android 

Note: you must have mineralt.io account to use this SDK.

## Contents
SDK distribution contains two files:
* `mlt-release.aar`: Android library (SDK) for mobile miner
* `mlt-demoapp-android.tgz`: Example Android application which uses MinerAlt SDK.

## Building and running example application

* Unpack example application archive: `tar xvfz mlt-demoapp-android.tgz`
* Open `mlt-demoapp-android` in Android studio. It will complain about missing `mlt-release` dependency. Now, do File->New Module,
choose `Import .JAR/.AAR file`, select SDK .AAR file and use `mlt-release` as imported module name (this is the default). 
* Choose 'release' build flavor, build application and run it on emulator or on your test device. That's it!

## Using MinerAlt SDK in your own application
Note: before using the MinerAlt SDK, you must register on https://mineralt.io and obtain your MinerId.

To add MinerAlt SDK to your Android application, do the following:
* File->New Module, choose 'Import .JAR/.AAR file`, select SDK .AAR file and use `mlt-release` as imported module name (this is the default). 
* Add the following to the `dependencies` section of `build.gradle` of your application:
```
  implementation 'com.neovisionaries:nv-websocket-client:2.4'
  implementation project(':mlt-release')
```
Now, you can use the API.

## API reference
API reference is provided below. The steps to use MinerAlt mining library are:
* import package: `import package blue.analytics.mlt.Mlt;`
* create Mlt instance: `Mlt mlt = Mlt.getInstance(context, "minerid");`. The first argument is Android activity or application Context.
For the second argument, use Miner ID provided by MinerAlt Web console.
* If you want to pass more parameters, you can use Bundle as a second parameter; bundle keys are defined as MPARAM_* below. 
* start miner, if necessary (if MPARAM_AUTOSTART is true, miner will start automatically): `mlt.setRunning(true)`.

CPU load can be set both from MinerAlt Web console, or by specifying MPARAM_LOAD (1..100). Note that Mineralt Web console setting 
overrides local value. If you want to use local value, don't specify explicit load in Mineralt Web console.

You typically will want to stop miner in `onPause()` of your activity/service, and start it again in `onResume()`. Otherwise,
users will probably complain about background task which eats their battery. As a compromise, you may want to post delayed Runnable in
`onPause()` which will stop miner after few minuntes in background.

If you need to know changes of miner state, use `RunState` callback; possible `runState` values are defined ad 'MLT_*'.

By default, miner will stop (and will not start) if battery level is below 20%. You can change this with `MPARAM_MINBATT`,
but it is not recommended (20% is reasonable default).

```java
public abstract class Mlt {
    static public final int MLT_RUNNING = 0;            // Miner is connected and running
    static public final int MLT_STOPPED = 1;            // Miner stopped by setRunning(false)
    static public final int MLT_NOT_CONNECTED = 2;      // Can't connect to server
    static public final int MLT_LOWBATT = 3;            // Miner stopped because of low battery

    static public final String MPARAM_MID       = "mid";        // MinerID (ascii alphanumeric string)
    static public final String MPARAM_LOAD      = "load";       // (int) Load factor, optional
    static public final String MPARAM_URL       = "url";        // Server URL, optional
    static public final String MPARAM_AUTOSTART = "autostart";  // (boolean) auto start on getInstance(), default=true
    static public final String MPARAM_MINBATT   = "minbatt";    // (int) Minimum battery level when miner will stop

    /* Miner run state callback, runState=MLT_* */
    public interface RunState {
        void    runStateChanged(int runState);
    }
    /* Set run state change callback */
    public abstract void       setRunStateCallback(RunState cb);

    /* Start/stop miner */
    public abstract void       setRunning(boolean run);

    /* Current miner run state */
    public abstract int        runState();

    /* Hashes per second for the current session */
    public abstract int        getHPS();

    /* Number of hashes for the current session */
    public abstract int        getHashes();

    /* Get ABI string (CPU, number of cores) */
    public abstract String     getAbiInfo();

    /* Miner version ID */
    public abstract int        getVersionInfo();

    /* Create instance with default parameters: load=30 (or taken from server),
     * autostart=true, minbatt=20
     */
    public static Mlt          getInstance(Context ctx, String minerid);

    /* Create new instance, with parameters passed in bundle. Bundle keys are
     * defined as MPARAM_*
     */
    public static Mlt          getInstance(Context ctx, Bundle params);

    /* Set/change parameters. Note that if any parameter except 'load' is changed,
     * miner will reconnect to server.
     */
    public abstract void       setParams(Bundle params);
}
```

### Any problems? Send e-mail to paul@atom.to.
