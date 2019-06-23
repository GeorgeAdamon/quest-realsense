# Oculus Quest - Intel RealSense Integration
Integration of Intel RealSense depth cameras with the Oculus Quest standalone VR headset using Unity.


## Foreword

<br/>

This project consolidates the knowledge gained while attempting to power & operate a RealSense D435i camera directly from an Oculus Quest VR headset. 

At the time of this attempt (5th-12th of June 2019), no prior work on this specific problem was available in the public realm, and the solution wouldn't be possible without the valuable contribution of members of the Intel development team.

The discussion which led to the solution, is available [here](https://github.com/IntelRealSense/librealsense/issues/4155), for future reference.

Aim of this project is to describe the steps necessary to achieve the Quest-RealSense integration inside Unity 2019.x, as well as to provide simple and generic source code.


_George Adamopoulos_

_23 of June 2019_

<br/>

## Introduction

The project was tested in the following environments.

**Development environment:**

| Label | Info  |
|----------------------------|-----------------|
| Operating System & Version | Windows 10 1803 |
| Language                   | C#              | 
| Unity Version              | 2019.1.4f1      |
| Graphics API               | OpenGLES3       |
| Scripting API Version      | .NET 4.x        |

<br/>

**RealSense Depth Camera Information:**

| Label  |  Info  |
|----------------------------|-----------------|
| Camera Model               | D435i           |
| Firmware Version           | 5.11.6.200 +    | 
| SDK Version                | 2.22.0 +        | 

<br/>

**Oculus Quest Information:**

| Label  |  Info  |
|----------------------------|-----------------|
| Headset Model              | Quest (May 2019)|
| Headset Version            | 256550.6170.5 + | 
| Unity Package Version      | 1.36 +          | 



## Step 1: Project Setup (Windows)
### Prerequisites #1: Oculus VR
This guide assumes that the necessary steps for setting-up the Unity development environment for the Oculus Quest have been completed, as described in the official Oculus Quest documentation: [[1]](https://developer.oculus.com/documentation/quest/latest/concepts/unity-build-android/) [[2]](https://developer.oculus.com/documentation/quest/latest/concepts/unity-mobileprep/)

The project should be able to produce a working Android .apk build, which runs on the Quest without issues, before proceeding to the next steps.

### Prerequisites #2: Intel RealSense wrappers
This guide assumes that the latest Intel RealSense Unity wrappers [**package**](https://github.com/IntelRealSense/librealsense/releases/download/v2.20.0/realsense.unitypackage) can be imported succesfully in Unity without errors, and that one of the provided example projects can be run succesfully on a Windows PC, provided a RealSense camera is connected to an appropriate USB port. More information in the official [realsense repository](https://github.com/IntelRealSense/librealsense/tree/master/wrappers/unity).

### Scripting Backend : Mono
Navigate to Unity's _**Project Settings > Player > Other Settings**_ and ensure that the Scripting Backend is set to **Mono**.
According to [this](https://github.com/IntelRealSense/librealsense/issues/4155#issuecomment-499363798) reply the RealSense library does not support Unity's IL2CPP library, at least at the time of writing this.

![](https://github.com/GeorgeAdamon/quest-realsense/blob/master/resources/img-scripting-backend.png)

## Step 2: Building the librealsense.aar Android library
### Build Process
In general, in order to allow a Unity project to access the RealSense cameras when targeting a platform other than Windows, the appropriate wrappers for this platform need to be built as Native Plugins first.

In this case, because we are targeting Android (the OS of Oculus Quest) we will have to build the **librealsense.aar** plugin from the provided Android Java source code, based on the [official guidelines](https://github.com/IntelRealSense/librealsense/tree/master/wrappers/android). 

In my experience, building from the Windows Command Prompt as an Administrator, using the ```gradlew assembleRelease``` [command](https://github.com/IntelRealSense/librealsense/tree/master/wrappers/android#build-with-gradle) proved to be the most straightforward, less error-prone, way:

![](https://github.com/GeorgeAdamon/quest-realsense/blob/master/resources/img-gradle-build.png)

A succesful build process should take around 10 minutes on a decent machine, and look like this:
![](https://github.com/GeorgeAdamon/quest-realsense/blob/master/resources/img-gradle-build-02.png)
![](https://github.com/GeorgeAdamon/quest-realsense/blob/master/resources/img-gradle-build-03.png)
![](https://github.com/GeorgeAdamon/quest-realsense/blob/master/resources/img-gradle-build-04.png)

If the build is succesful, the generated .aar file will be located in 
```<librealsense_root_dir>/wrappers/android/librealsense/build/outputs/aar```.

### Unity Side
The generated **librealsense.aar** file should be placed inside your Unity project, in the _**Assets / RealSenseSDK2.0 / Plugins**_ directory, alongside the Intel.RealSense.dll and librealsense2.dll. A succesful setup should look like this:

![](https://github.com/GeorgeAdamon/quest-realsense/blob/master/resources/img-unity-plugins.png)

## Step 3: Initializing the RsContext Java class from Unity
> Note: A big shout-out to [**ogoshen**](https://github.com/ogoshen) for generously providing the solution of this next step!

Now that all the libraries are in place, before actually being able to access the RealSense camera, we need a C# script that performs two crucial jobs: 
* Initializes a new instance of the Java class [**RsContext**](https://github.com/IntelRealSense/librealsense/blob/master/wrappers/android/librealsense/src/main/java/com/intel/realsense/librealsense/RsContext.java)
* Makes sure that Android Camera Permissions are explicitly requested from the user, if not provided already.

Attaching the following script to any GameObject in your Scene, would ensure that those two operations are executed in the beginning of your application:
```c#
using UnityEngine;

public class AndroidPermissions : MonoBehaviour
{
#if UNITY_ANDROID && !UNITY_EDITOR
    void Awake()
    {
        if (!UnityEngine.Android.Permission.HasUserAuthorizedPermission(UnityEngine.Android.Permission.Camera))
        {
            UnityEngine.Android.Permission.RequestUserPermission(UnityEngine.Android.Permission.Camera);

        }

        using (var javaUnityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer"))
        using (var currentActivity = javaUnityPlayer.GetStatic<AndroidJavaObject>("currentActivity"))
        using (var rsContext = new AndroidJavaClass("com.intel.realsense.librealsense.RsContext"))
        {
            Debug.Log(rsContext);
            rsContext.CallStatic("init", currentActivity);
        }
    }
#endif
}
```


## Step 4: Using Quest-friendly shaders
