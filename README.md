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

## Step 1: Project Setup
### Prerequisites
This guide assumes that the necessary steps for setting-up the Unity development environment for the Oculus Quest have been completed, as described in the official Oculus Quest documentation: [[1]](https://developer.oculus.com/documentation/quest/latest/concepts/unity-build-android/) [[2]](https://developer.oculus.com/documentation/quest/latest/concepts/unity-mobileprep/)

The project should be able to produce a working Android .apk build, which runs on the Quest without issues, before proceeding to the next steps.

### Scripting Backend : Mono
Navigate to Unity's _Project Settings > Player > Other Settings_ and ensure that the Scripting Backend is set to **Mono**.
According to [this](https://github.com/IntelRealSense/librealsense/issues/4155#issuecomment-499363798) reply the RealSense library does not support Unity's IL2CPP library, at least at the time of writing this.

## Step 2: Building the librealsense.aar Android library

## Step 3: Initializing the RsContext Java class from Unity

## Step 4: Using Quest-friendly shaders
