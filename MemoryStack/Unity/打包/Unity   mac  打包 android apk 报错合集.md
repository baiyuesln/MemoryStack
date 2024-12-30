#android打包

# 报错1：UnityPlayerActivity找不到了？（大概意思）
解决： 创建一个UnityPlayerActivity.java文件，放到指定位置即可1

UnityPlayerActivity.java示例：
```cs
package com.unity3d.player;

  

import android.app.Activity;

import android.content.Intent;

import android.content.res.Configuration;

import android.graphics.PixelFormat;

import android.os.Bundle;

import android.view.KeyEvent;

import android.view.MotionEvent;

import android.view.View;

import android.view.Window;

import android.view.WindowManager;

import android.os.Process;

  

public class UnityPlayerActivity extends Activity implements IUnityPlayerLifecycleEvents

{

protected UnityPlayer mUnityPlayer; // don’t change the name of this variable; referenced from native code

// Override this in your custom UnityPlayerActivity to tweak the command line arguments passed to the Unity Android Player

// The command line arguments are passed as a string, separated by spaces

// UnityPlayerActivity calls this from 'onCreate'

// Supported: -force-gles20, -force-gles30, -force-gles31, -force-gles31aep, -force-gles32, -force-gles, -force-vulkan

// See https://docs.unity3d.com/Manual/CommandLineArguments.html

// @param cmdLine the current command line arguments, may be null

// @return the modified command line string or null

protected String updateUnityCommandLineArguments(String cmdLine)

{

return cmdLine;

}

  

// Setup activity layout

@Override protected void onCreate(Bundle savedInstanceState)

{

requestWindowFeature(Window.FEATURE_NO_TITLE);

super.onCreate(savedInstanceState);

  

String cmdLine = updateUnityCommandLineArguments(getIntent().getStringExtra("unity"));

getIntent().putExtra("unity", cmdLine);

  

mUnityPlayer = new UnityPlayer(this, this);

setContentView(mUnityPlayer);

mUnityPlayer.requestFocus();

}

  

// When Unity player unloaded move task to background

@Override public void onUnityPlayerUnloaded() {

moveTaskToBack(true);

}

  

// Callback before Unity player process is killed

@Override public void onUnityPlayerQuitted() {

}

  

@Override protected void onNewIntent(Intent intent)

{

// To support deep linking, we need to make sure that the client can get access to

// the last sent intent. The clients access this through a JNI api that allows them

// to get the intent set on launch. To update that after launch we have to manually

// replace the intent with the one caught here.

setIntent(intent);

mUnityPlayer.newIntent(intent);

}

  

// Quit Unity

@Override protected void onDestroy ()

{

mUnityPlayer.destroy();

super.onDestroy();

}

  

// If the activity is in multi window mode or resizing the activity is allowed we will use

// onStart/onStop (the visibility callbacks) to determine when to pause/resume.

// Otherwise it will be done in onPause/onResume as Unity has done historically to preserve

// existing behavior.

@Override protected void onStop()

{

super.onStop();

  

if (!MultiWindowSupport.getAllowResizableWindow(this))

return;

  

mUnityPlayer.pause();

}

  

@Override protected void onStart()

{

super.onStart();

  

if (!MultiWindowSupport.getAllowResizableWindow(this))

return;

  

mUnityPlayer.resume();

}

  

// Pause Unity

@Override protected void onPause()

{

super.onPause();

  

if (MultiWindowSupport.getAllowResizableWindow(this))

return;

  

mUnityPlayer.pause();

}

  

// Resume Unity

@Override protected void onResume()

{

super.onResume();

  

if (MultiWindowSupport.getAllowResizableWindow(this))

return;

  

mUnityPlayer.resume();

}

  

// Low Memory Unity

@Override public void onLowMemory()

{

super.onLowMemory();

mUnityPlayer.lowMemory();

}

  

// Trim Memory Unity

@Override public void onTrimMemory(int level)

{

super.onTrimMemory(level);

if (level == TRIM_MEMORY_RUNNING_CRITICAL)

{

mUnityPlayer.lowMemory();

}

}

  

// This ensures the layout will be correct.

@Override public void onConfigurationChanged(Configuration newConfig)

{

super.onConfigurationChanged(newConfig);

mUnityPlayer.configurationChanged(newConfig);

}

  

// Notify Unity of the focus change.

@Override public void onWindowFocusChanged(boolean hasFocus)

{

super.onWindowFocusChanged(hasFocus);

mUnityPlayer.windowFocusChanged(hasFocus);

}

  

// For some reason the multiple keyevent type is not supported by the ndk.

// Force event injection by overriding dispatchKeyEvent().

@Override public boolean dispatchKeyEvent(KeyEvent event)

{

if (event.getAction() == KeyEvent.ACTION_MULTIPLE)

return mUnityPlayer.injectEvent(event);

return super.dispatchKeyEvent(event);

}

  

// Pass any events not handled by (unfocused) views straight to UnityPlayer

@Override public boolean onKeyUp(int keyCode, KeyEvent event) { return mUnityPlayer.injectEvent(event); }

@Override public boolean onKeyDown(int keyCode, KeyEvent event) { return mUnityPlayer.injectEvent(event); }

@Override public boolean onTouchEvent(MotionEvent event) { return mUnityPlayer.injectEvent(event); }

/*API12*/ public boolean onGenericMotionEvent(MotionEvent event) { return mUnityPlayer.injectEvent(event); }

}
```


# 报错2:> com.android.ide.common.signing.KeytoolException: Failed to read key jiate from store "/Users/user.keystore": Invalid keystore format

解决：
	把路径显示无效的user.keystore删了，重新打包即可

# 报错3>
Win32Exception: ApplicationName='/Applications/Unity/Hub/Editor/2021.3.16f1c1/PlaybackEngines/AndroidPlayer/SDK/tools/bin/sdkmanager', CommandLine='--list', CurrentDirectory='/Applications/BuruXIuxian/chuanQi', Native error= Access denied

解决:

	使用sudo修改权限
sudo chmod -R 755 "/Applications/Unity/Hub/Editor/2021.3.16f1c1/PlaybackEngines/AndroidPlayer/SDK"

	修改所有者
sudo chown -R $USER "/Applications/Unity/Hub/Editor/2021.3.16f1c1/PlaybackEngines/AndroidPlayer/SDK"

# 报错4:
Internal build system error. BuildProgram exited with code 2.
System.IO.FileNotFoundException: ==Could not load file or assembly 'Unity.IL2CPP.Bee.BuildLogic.Android,== Version=1.0.0.0, Culture=neutral, PublicKeyToken=null'. The system cannot find the file specified.

File name: 'Unity.IL2CPP.Bee.BuildLogic.Android, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null'

解决:
1
	仔细查看报错信息发现,在这个报错上面还有一个找不到Temp/xx/xx的黄色报错信息,与热更有关,所以点击发现
	HybirdCLR  生成 all  也是报同样的错
	 ==重新安装HybirdCLR 从新生成==
2
	下载 mac build support(IL2CPP)  下载  ,然后下载器报错:“==安装器遇到了一个错误，导致安装失败。请联系软件制造商以获得帮助==”
	 查看报错log   cd /var/log/installer.log  cat installer.log    
	 发现没有找到路径下的Untiy.app    移动到它这里指定的路径,下载成功
3
	api compatibility level  -- > .net Standard2.1

# 报错5:
==Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8==  
  
FAILURE: Build failed with an exception.  
  
* What went wrong:  
Could not determine the dependencies of task ':launcher:processReleaseResources'.  
> Could not resolve all task dependencies for configuration ':launcher:releaseRuntimeClasspath'.  
> Could not resolve com.pangle.cn:mediation-admob-adapter:17.2.0.49.  
Required by:  
project :launcher > project :unityLibrary  
> Could not resolve com.pangle.cn:mediation-admob-adapter:17.2.0.49.  
> Could not get resource 'https://maven.google.com/com/pangle/cn/mediation-admob-adapter/17.2.0.49/mediation-admob-adapter-17.2.0.49.pom'.  
> Could not GET 'https://maven.google.com/com/pangle/cn/mediation-admob-adapter/17.2.0.49/mediation-admob-adapter-17.2.0.49.pom'.  
> Connect to maven.google.com:443 [maven.google.com/142.250.204.46] failed: connect timed out  
> Could not resolve com.pangle.cn:mediation-baidu-adapter:9.34.1.  
Required by:  
project :launcher > project :unityLibrary  
> Could not resolve com.pangle.cn:mediation-baidu-adapter:9.34.1.  
> Could not get resource 'https://maven.google.com/com/pangle/cn/mediation-baidu-adapter/9.34.1/mediation-baidu-adapter-9.34.1.pom'.  
> Could not GET 'https://maven.google.com/com/pangle/cn/mediation-baidu-adapter/9.34.1/mediation-baidu-adapter-9.34.1.pom'.  
> Connect to maven.google.com:443 [maven.google.com/142.250.204.46] failed: connect timed out  
> Could not resolve com.pangle.cn:mediation-gdt-adapter:4.570.1440.0.  
Required by:  
project :launcher > project :unityLibrary  
> Could not resolve com.pangle.cn:mediation-gdt-adapter:4.570.1440.0.  
> Could not get resource 'https://maven.google.com/com/pangle/cn/mediation-gdt-adapter/4.570.1440.0/mediation-gdt-adapter-4.570.1440.0.pom'.  
> Could not GET 'https://maven.google.com/com/pangle/cn/mediation-gdt-adapter/4.570.1440.0/mediation-gdt-adapter-4.570.1440.0.pom'.  
> Connect to maven.google.com:443 [maven.google.com/142.250.204.46] failed: connect timed out  
> Could not resolve com.pangle.cn:mediation-klevin-adapter:2.11.0.3.11.  
Required by:  
project :launcher > project :unityLibrary  
> Could not resolve com.pangle.cn:mediation-klevin-adapter:2.11.0.3.11.  
> Could not get resource 'https://maven.google.com/com/pangle/cn/mediation-klevin-adapter/2.11.0.3.11/mediation-klevin-adapter-2.11.0.3.11.pom'.  
> Could not GET 'https://maven.google.com/com/pangle/cn/mediation-klevin-adapter/2.11.0.3.11/mediation-klevin-adapter-2.11.0.3.11.pom'.  
> Connect to maven.google.com:443 [maven.google.com/142.250.204.46] failed: connect timed out  
> Could not resolve com.pangle.cn:mediation-ks-adapter:3.3.63.0.  
Required by:  
project :launcher > project :unityLibrary  
> Could not resolve com.pangle.cn:mediation-ks-adapter:3.3.63.0.  
> Could not get resource 'https://maven.google.com/com/pangle/cn/mediation-ks-adapter/3.3.63.0/mediation-ks-adapter-3.3.63.0.pom'.  
> Could not GET 'https://maven.google.com/com/pangle/cn/mediation-ks-adapter/3.3.63.0/mediation-ks-adapter-3.3.63.0.pom'.  
> Connect to maven.google.com:443 [maven.google.com/142.250.204.46] failed: connect timed out  
> Could not resolve com.pangle.cn:mediation-mintegral-adapter:16.5.47.3.  
Required by:  
project :launcher > project :unityLibrary  
> Could not resolve com.pangle.cn:mediation-mintegral-adapter:16.5.47.3.  
> Could not get resource 'https://maven.google.com/com/pangle/cn/mediation-mintegral-adapter/16.5.47.3/mediation-mintegral-adapter-16.5.47.3.pom'.  
> Could not GET 'https://maven.google.com/com/pangle/cn/mediation-mintegral-adapter/16.5.47.3/mediation-mintegral-adapter-16.5.47.3.pom'.  
> Connect to maven.google.com:443 [maven.google.com/142.250.204.46] failed: connect timed out  
> Could not resolve com.pangle.cn:mediation-sigmob-adapter:4.15.1.2.  
Required by:  
project :launcher > project :unityLibrary  
> Could not resolve com.pangle.cn:mediation-sigmob-adapter:4.15.1.2.  
> Could not get resource 'https://maven.google.com/com/pangle/cn/mediation-sigmob-adapter/4.15.1.2/mediation-sigmob-adapter-4.15.1.2.pom'.  
> Could not GET 'https://maven.google.com/com/pangle/cn/mediation-sigmob-adapter/4.15.1.2/mediation-sigmob-adapter-4.15.1.2.pom'.  
> Connect to maven.google.com:443 [maven.google.com/142.250.204.46] failed: connect timed out  
> Could not resolve com.pangle.cn:mediation-unity-adapter:4.3.0.17.  
Required by:  
project :launcher > project :unityLibrary  
> Could not resolve com.pangle.cn:mediation-unity-adapter:4.3.0.17.  
> Could not get resource 'https://maven.google.com/com/pangle/cn/mediation-unity-adapter/4.3.0.17/mediation-unity-adapter-4.3.0.17.pom'.  
> Could not GET 'https://maven.google.com/com/pangle/cn/mediation-unity-adapter/4.3.0.17/mediation-unity-adapter-4.3.0.17.pom'.  
> Connect to maven.google.com:443 [maven.google.com/142.250.204.46] failed: connect timed out  
  
* Try:  
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.  
  
* Get more help at https://help.gradle.org  
  
BUILD FAILED in 2m 7s  
  
UnityEngine.GUIUtility:ProcessEvent (int,intptr,bool&) (at /Users/bokken/buildslave/unity/build/Modules/IMGUI/GUIUtility.cs:189)

解决:
	无法连接到maven.google.com的问题,连个外网ok了