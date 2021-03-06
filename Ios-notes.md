# Deprecation Note: the current Xamarin.iOS based backend is about to be deprecated with the release of libgdx 0.9.9. Instead, we will focus on the (free) RoboVM backend. For more information see http://www.badlogicgames.com/wordpress/?p=3156 #

Here we document the setup, limitations and gotcha's of the current iOS back-end. Please read everything carefully.

## General Information ##
The iOS backend currently uses MonoTouch/IKVM to run Java code on iOS. IKVM is used to 1) compile Java byte-code to .Net byte-code and 2) provide a run-time environment including much of the Java run-time library on top of the .Net run-time library. MonoTouch is used to compile the .Net byte-code to native code (ARM). This is a requirement by Apple in order for an application to be accepted on the app store. JIT compilers are currently not feasible on iOS. The code of an application has to be ahead of time compiled.

Much of the progress on the iOS back-end is documented on the [blog](http://www.badlogicgames.com/wordpress/?cat=36). Of special interest is the blog post on [setting up things for testing](http://www.badlogicgames.com/wordpress/?p=2602). Read the sections below for additional information on gotchas, debugging and profiling.

## Quick Start Guide ##
You'll need a Mac, XCode and a Xamarin.iOS license. If you are a student, you can get a [license for 79$](http://www.badlogicgames.com/wordpress/?p=2629), otherwise you can buy an "Indie" license for 299$. The free "Starter" license does not work with libgdx due to feature restrictions.

You only need a license if you want to deploy to the App Store. If you just want to test if your game runs at all, you can install the "30-day business trial" version and test your game on both the simulator or a real device. 

You will also need to pay 99$ to Apple to be allowed to deploy to your own devices and to publish to the App Store. Go figure...

You need to install the following things:

  * Eclipse, with the usual plugins (Android, Google Web Toolkit if you want to deploy to HTML5).
  * JDK, make sure javac is in your $PATH and executable from the terminal
  * [Xamarin.iOS](http://xamarin.com/monotouch), 30-day business trial evaluation, student or "Indie" license (app store deployment).
  * [Ant](http://tweedo.com/mirror/apache//ant/binaries/apache-ant-1.8.4-bin.zip). Download and extract the zip to say /Users/you/ant, then create a sym-link via `ln -s /Users/you/ant/bin/ant /usr/bin/ant`

Make sure both `javac` and `ant` work from the terminal. Compiling your project in Xamarin Studio will fail if one of those two is not accessible system wide!

Now run the [gdx-setup-ui](http://code.google.com/p/libgdx/wiki/ProjectSetupNew), create a new project. You'll get a myproject-ios folder, containing a .sln file. Open that file in Xamarin Studio. Now you can compile/run/debug the app. The first time you do so, Xamarin Studio will complain that mygame.dll is not available. Simply close and reopen the solution.

Every Time you compile/run your iOS project, the Java classes of your core project are compiled via an ant script called convert.xml, located in your iOS project. The resulting .class files are then compiled to a .NET assembly (.dll file) via IKVM, via the same ant file. You can modify the convert.properties file to include other source locations, see below.

You can debug Java code from within Xamarin Studio. Simply open the Java file from your core project in Xamarin Studio, set a breakpoint and start debugging.

If you add assets, you need to make sure to add them to your project in Xamarin Studio as well. After linking to a new file, you also need to set its Build Action to "Content". All of this can be done by right clicking and using the context menus in the solution view.

Make sure to always clean and rebuild your iOS project when you made changes to your Java code. Let me repeat: *clean* and rebuild your iOS project if you change your Java code! Otherwise the Ant script will not be executed.

You can also profile your application via [Instruments](http://developer.apple.com/library/mac/#documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/Introduction/Introduction.html). 

Make yourself comfortable with Xamarin Studio by reading the [docs on Xamarin's site](http://docs.xamarin.com/ios/Guides/Getting_started/Introduction_to_Mobile_Development). This is also essential reading if you want to integrate native UI stuff. You are likely to want to do that in C#, using Xamarin.iOS' wrappers around Apple's ObjectiveC APIs. 


## PNG optimization ##
*Update #2:* As of Xamarin Studio 6.2 (April 2013), you do no longer have to perform the below steps. The iOS project generated by gdx-setup-ui will automatically disable pngcrush. Alternatively you can toggle that setting in the Build Options of your Xamarin.iOS project. All libgdx nightlies starting from May 20th 2013 support this out of the box, the next release 0.9.9 will also support it.

*Update:* Xamarin released a new IDE called Xamarin Studio. If you use this latest version, follow [this workaround](http://www.badlogicgames.com/wordpress/?p=2859). If you use the old MonoDevelop/MonoTouch environment continue below.

The iOS SDK includes a script called `iphoneos-optimize` which is applied to all PNG files of an iOS application prior to bundling. Sadly, this "optimization" converts the PNGs into a format that is not PNG spec compatible. Libgdx uses stb_image to load various image formats. stb_image can't cope with these messed up PNGs. 

To disable this optimization when using MonoDevelop, one has to modify the script. Locate the file via 

```
$ locate iphoneos-optimize
```

Open the file in your favorite text editor and uncomment the implementation of the function optimizePNGs:

```
sub optimizePNGs {
        #my $name = $File::Find::name;
        #if ( -f $name && $name =~ /^(.*)\.png$/i) {
        #       my $crushedname = "$1-pngcrush.png";
        #       my @args = ( $PNGCRUSH, "-iphone", "-f", "0", $name, $crushedname );
        #       if (system(@args) != 0) {
        #               print STDERR "$SCRIPT_NAME: Unable to convert $name to an optimized png!\n";
        #               return;
        #       }
        #       unlink $name or die "Unable to delete original file: $name";
        #       rename($crushedname, $name) or die "Unable to rename $crushedname to $name";
        #       print "$SCRIPT_NAME: Optimized PNG: $name\n";
        #}
}
```

This will make sure the PNGs aren't scrambled. Note that once you comment the code as above, normal XCode iOS and MonoTouch iOS projects won't be able to apply this optimization either. Just revert the script if you need the optimization.

note: in case it doesn't work, locate could be returning an invalid location to an old script or something, not sure, what you can do is find the real location when compiling to iPhone, the logs says where the script is located when invoking it.

## Debugging ##
To debug Java classes directly in Xamarin Studio, you have to make sure to


  * Compile your Java source code with debugging information. Javac uses the flag `-g` for this
  * Tell IKVM to generate an `.mdb` file containing the debugging information in Mono format
  * Tell IKVM where the original Java source code files are located

All of this is best achieved in the convert.xml Ant script that all the libgdx iOS demos have. Here's an example from the [Pax Britannica demo](https://github.com/libgdx/libgdx/tree/master/demos/pax-britannica):

```xml
<project name="gdx" default="convert" basedir=".">
	<property environment="env"/>
	<property name="IKVM_HOME" value="${env.IKVM_HOME}"/>
	<property name="MONO_HOME" value="/Developer/MonoTouch/usr/lib/mono/2.1"/>
	<property name="IN" value="-recurse:target/core/*.class"/>
	<property name="OUT" value="gdx.dll"/>
	<property name="SRC" value="src/"/>
	<property name="CLASSPATH" value=""/>
	<property name="EXCLUDE" value=""/>

	<target name="compile">
		<delete dir="target"/>
		<mkdir dir="target"/>
		<javac srcdir="${SRC}" debug="on" destdir="target" classpath="${CLASSPATH}">
			<include name="**/*.java"/>
			<exclude name="${EXCLUDE}"/>
		</javac>
	</target>

	<target name="convert">
		<exec executable="mono">
			<arg value="${IKVM_HOME}/bin/ikvmc.exe"/>
			<arg value="-nostdlib"/>
			<arg value="-target:library"/>
			<arg value="-debug"/>
			<arg value="-out:${OUT}"/>
			<arg value="-r:${MONO_HOME}/mscorlib.dll"/>
			<arg value="-r:${MONO_HOME}/System.dll"/>
			<arg value="-r:${MONO_HOME}/System.Core.dll"/>
			<arg value="-r:${MONO_HOME}/System.Data.dll"/>
			<arg value="-r:${MONO_HOME}/OpenTK.dll"/>
			<arg value="-r:${MONO_HOME}/monotouch.dll"/>
			<arg value="-r:${MONO_HOME}/Mono.Data.Sqlite.dll"/>
			<arg value="-srcpath:${SRC}"/>
			<arg line="${IN}"/>
		</exec>
	</target>
</project>
```

The "compile" target uses `javac` to compile the Java source code to a set of .class files. Note the `debug="on"` attribute. It tells javac to compile the Java source files with debugging information.

The "convert" target has two arguments, one called "-debug", making sure IKVM creates an .mdb file containing the converted debugging information found in the Java class files. The other is called "-srcpath" and tells IKVM where it can find the original .java source files. These are needed so we can set breakpoints and do other debugging tasks in MonoTouch.

Note: It seems debugger can't find the source if more than one path is used with -srcpath parameter, [reference](http://badlogicgames.com/forum/viewtopic.php?f=11&t=6387&p=31179#p31148).

To check whether debugging information was properly generated, check if there is a file called your-apps-dll.dll.mdb, where your-apps-dll would be the name your give to the Ant script via the "OUT" parameter. In Pax Britannica's case this would be "pax-britannica.dll" containing the actual code, and "pax-britannica.dll.mdb" containing the debugging information. If the .mdb file is bigger than 112 bytes, debugging information was created successfully.

Debugging information generation is turned on by default for the gdx.dll and gdx-backend-iosmonotouch.dll. If you compile these as described in the blog post linked to above, you'll get the corresponding .mdb files automatically.

In Xamarin Studio you can now open any of your Java source files (File -> Open), and set breakpoints. When you run your application in debug mode (configuration Debug|IPhoneSimulator or Debug|IPhone), the debugger will stop at the breakpoints in your Java source code, just as it does for C# code. You can also observe variable and field contents.

*Note: converting via IKVM with debugging information can result in slightly slower code. Turn this feature of for release builds!*

### File conversion in Xamarin Studio ###

The custom commands in the Xamarin Studio options settings have a pre-build command for doing file conversion (as mentioned above.)  Rather than maintaining all the conversion in there in a hard to maintain line, you can use an ant properties file.  The custom command would then read:

```
ant -f convert.xml compile convert
```

Example convert.xml (notice the convert.properties reference):

```xml
<project name="gdx" default="convert" basedir=".">
    <property file="convert.properties" />
	<property environment="env"/>
	<property name="IKVM_HOME" value="${env.IKVM_HOME}"/>
	<property name="MONO_HOME" value="/Developer/MonoTouch/usr/lib/mono/2.1"/>
	<property name="IN" value="-recurse:target/core/*.class"/>
	<property name="OUT" value="gdx.dll"/>
	<property name="SRC" value="src/"/>
	<property name="CLASSPATH" value=""/>
	<property name="EXCLUDE" value=""/>

	<target name="compile">
		<delete dir="target"/>
		<mkdir dir="target"/>
		<javac srcdir="${SRC}" debug="on" destdir="target" classpath="${CLASSPATH}">
			<include name="**/*.java"/>
			<exclude name="${EXCLUDE}"/>
		</javac>
	</target>

	<target name="convert">
		<exec executable="mono">
			<arg value="${IKVM_HOME}/bin/ikvmc.exe"/>
			<arg value="-nostdlib"/>
			<arg value="-target:library"/>
			<arg value="-debug"/>
			<arg value="-srcpath:../../toyslots/src"/>
			<arg value="-out:${OUT}"/>
			<arg value="-r:${MONO_HOME}/mscorlib.dll"/>
			<arg value="-r:${MONO_HOME}/System.dll"/>
			<arg value="-r:${MONO_HOME}/System.Core.dll"/>
			<arg value="-r:${MONO_HOME}/System.Data.dll"/>
			<arg value="-r:${MONO_HOME}/OpenTK.dll"/>
			<arg value="-r:${MONO_HOME}/monotouch.dll"/>
			<arg value="-r:${MONO_HOME}/Mono.Data.Sqlite.dll"/>
			<arg line="${IN}"/>
		</exec>
	</target>
</project>
```

Example convert.properties.  Place it in the same directory as convert.xml:

```
SRC=\
../../toyslots/src/;\
../../../artemis-framework/src/;\
../../gushiku-common/src/;\
../../../tween-engine/tween-engine-api/src/
CLASSPATH=\
../../toyslots-android/libs/Swarm.jar;\
../../../libgdx/gdx/bin/;\
../../../libgdx/backends/gdx-backend-iosmonotouch/bin/;\
../../../libgdx/backends/gdx-backend-iosmonotouch/libs/monotouch-jars/monotouch-5.4.0.jar;\
../../../libgdx/backends/gdx-backend-iosmonotouch/libs/monotouch-jars/mscorlib-4.0.jar;\
../../../libgdx/backends/gdx-backend-iosmonotouch/libs/monotouch-jars/opentk-5.4.0.jar;\
../../../libgdx/backends/gdx-backend-iosmonotouch/libs/monotouch-jars/system-2.1.jar;\
../../../libgdx/backends/gdx-backend-iosmonotouch/libs/monotouch-jars/system-core-2.1.jar;\
../../../libgdx/backends/gdx-backend-iosmonotouch/libs/monotouch-jars/system-data-2.1.jar;\
../../../libgdx/backends/gdx-backend-lwjgl/bin/
IN=-r:../../../libgdx/backends/gdx-backend-iosmonotouch/libs/gdx.dll -recurse:target/*.class
OUT=toyslots.dll
```


## Profiling ##
Libgdx iOS apps can be profiled like any other MonoTouch application. Follow the guides at the Xamarin Headquaters:

  * [Memory Profiling](http://docs.xamarin.com/ios/tutorials/MONOTOUCH_PROFILER)
  * [Profiling via Instruments](http://docs.xamarin.com/ios/tutorials/Profiling_MonoTouch_applications_with_Instruments Method)
  * [More information on Instruments, e.g. OpenGL ES profiling etc.](http://developer.apple.com/library/mac/#documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/Introduction/Introduction.html)

## Limitations: Java API ##
The iOS back-end does only support a limited set of functionality from the standard Java API. Whatever is supported by IKVM MonoTouch (used to convert Java to C# and from there to iOS) should be available in the LibGDX iOS back-end as well. 

Most notably not included are:
  * java.net (use Gdx.net instead)
  * ResourceBundle (x)
  * DateFormatter  
  * GregorianCalendar (x)
  * lots more... (obviously java.awt, javax.swing, javax.sql etc.)

(x) I (Chris/noblemaster) might have some classes that might help. They don't match the Java API, nevertheless they would be able to help you get up to speed. Contact me and I'll email you a copy (for free of course). We could probably integrate some stuff into LibGDX as well if it is desired but they would need some cleanup first.

For more information also visit the [IKVM MonoTouch Repository](https://github.com/samskivert/ikvm-monotouch).

Using System.out doesn't work fine after closing and reopening the application, at least when using with UIApplicationExitsOnSuspend, so for now rely always on Gdx.app.log() which calls .net code and seems to work fine.

## Info.plist Settings ##
Below are a few settings you can add to [Info.plist](http://developer.apple.com/library/ios/#documentation/general/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html) to circumvent problems when porting to iOS:
  * If your app doesn't resume correctly after pause; i.e. you have a blank screen or crash, use the following option to force your app to restart from scratch after pause:
```xml
	<key>UIApplicationExitsOnSuspend</key>
	<true/>
```
  (note: this is supposed to be fixed now that the resume() method is called)
  * iOS has a property to specify the supported orientations:
```xml
	<key>UISupportedInterfaceOrientations</key>
	<array>
		<string>UIInterfaceOrientationLandscapeRight</string>
		<string>UIInterfaceOrientationLandscapeLeft</string>
	</array>
```
 Be aware that iOS uses the first declared orientation as the default device orientation when the game starts, that means you have to take that in mind when using a Splash screen to avoid the game starts and auto-rotates to the inverse orientation. 
  (arielsan: There is a property UIInterfaceOrientation which is supposed to declare the initial orientation but didn't work for us while changing the UISupportedInterfaceOrientations order did)

## Classpath assets ##

Loading assets from classpath is not supported since the assets are not exported by IKVM monotouch when converting the project to DLL. Also, from the IKVM monotouch code, we believe the AssemblyClassLoader doesn't support searching for resources in the classpath so that would be another limitation to all this, even if the assets were exported in the DLL. So, for now, classpath files are not supported, in particular, we can't create a BitmapFont using default constructor since it tries to load the font files from the classpath.

As a workaround, in the monotouch project we have to reference to the assets directly and configure them as Content. In case of BitmapFont, use other constructor specifying the files to use.

## MonoDevelop Settings ##

### iPhone Build ###

In case you make heavy use of interfaces the game fails in runtime because "Ran out of trampolines of type 2". To fix that, increase the number of trampolines by adding -aot "nimt-trampolines=512" (by default is 128, use whatever adapts to your game) to the iPhone Build as part of the project configuration in mono touch. More info at [XamarinDocs](http://docs.xamarin.com/ios/troubleshooting)

The game appears to crash at startup for certain "Region Format" (Japan, Russia, China) settings if you don't include all the internationalization files. In "Project Options : Build : iPhone Build : Advanced : Internationalization" make sure to check all the boxes for "west", "cjk", "rare", "other" and "mideast" (especially important for the "AppStore" configuration if you decide to distribute globally). 

### Accessing Info.plist Version ###

You may wish to reference the 'Version' field of the Info.plist file from your app.  One approach is shown below:

```objectivec
	var str = NSBundle.MainBundle.InfoDictionary.ValueForKey(new NSString("CFBundleVersion"));
	Settings.mVersionName = str.ToString(); // Settings has been defined as a static Java class in the project
```

### View Troubleshooting ###

The repo below is a fork that implements a handy hierarchy viewer for iOS. It may be helpful if you ever need to debug your views:

[Hierarchy viewer repo on Github](https://github.com/tescott/hierarchy-viewer-monotouch)

In MonoTouch, all you need to do is add:

```objectivec
HierarchyViewer.iOSHierarchyViewer.Start();
```

...to your FinishedLaunching() method. Next, kick off your app on the simulator. The Application Output pane should indicate what URL to visit with Chrome or Safari. You can then navigate around, hit refresh, and see everything that your View hierarchy contains.

### Admob Support ###

If you want to have AdMOb ads correctly positioned at the bottom of your app, Tim Scott (aka tescott) has got a real simple [MonoTouch binding project repo](https://github.com/tescott/admob-monotouch) that supports Google AdMob. Specifically, it supports Smart Banners. See [Keeping A Smart Banner Docked To The Bottom Of The Screen on iOS](http://googleadsdeveloper.blogspot.com/2012/06/keeping-smart-banner-docked-to-bottom.html) for info on how to dock a Smart Banner to the bottom of an iOS screen.

Another option for ads is to use [Github Repo](https://github.com/dalexsoto/AlexTouch.GoogleAdMobAds AlexTouch.GoogleAdMobAds).  Note, however, that project currently does not expose the `CGSizeFromGADAdSize()` method from the Google AdMob library.  This is required to correctly position a Smart Banner based on current screen size.  But, it supports other types of things such as interstitials.

## Sound ##

You are probably going to want to use .caf files for your iOS project, since .ogg is not supported.  A quick and easy method for generating .caf files is to use [ffmpeg](http://ffmpeg.com).  The script below takes .wav source files and converts them to both .ogg (for Android deployment) and .caf for iOS deployment.

Note (by noblemaster): For best platform compatibility I recommend using WAV (sound FX less than 5 seconds) and MP3 (music). They both work on iOS as well as all the other backends (Android, Mac, PC, Linux). You might get slightly better compression using CAF (?) and OGG but it will make it harder to maintain as you have to generate different music files for each backend. For easy maintenance I don't recommend converting WAV to CFA. Simple leave the WAV files as is. Otherwise including Android I recommend using MP3 (not OGG) as you can just leave them as is as well (they work everywhere).

```bash
#!/usr/bin/bash

#convert wav to .caf files for iOS
for i in *.wav; do
  /cygdrive/f/ffmpeg/ffmpeg -i $i -acodec pcm_s16le caf/${i%.wav}.caf
done

#convert wav to .ogg files for Android
for i in *.wav; do
  /cygdrive/f/ffmpeg/ffmpeg -i $i -acodec libvorbis -aq 5 ogg/${i%.wav}.ogg
done
```

## Reported Screensize for iPhone / iPod Touch 5th Gen ##

If your app appears in letterbox format on an 5th gen device, you can correct this by adding a Default-568h@2x.png to your project.

Look here for further details on this problem:

http://stackoverflow.com/questions/12398819/iphone-5-letterboxing-screen-resize

## iOS Lifecycle ##

http://developer.apple.com/library/ios/#documentation/iphone/conceptual/iphoneosprogrammingguide/ManagingYourApplicationsFlow/ManagingYourApplicationsFlow.html
http://stackoverflow.com/questions/6519847/what-is-the-life-cycle-of-an-iphone-application
http://www.cocoanetics.com/2010/07/understanding-ios-4-backgrounding-and-delegate-messaging/

## Others ##

arielsan/noblemaster(confirmed): We had a problem with String toUpperCase() method when using some special locales for countries/languages, in particular with Turkey/Turkish. In order to fix that, we are forcing another locale like English to avoid the java.util/lang to fail. The following code fixes the problem (call as early as possible): 
```
 Locale.setDefault(Locale.ENGLISH);
```