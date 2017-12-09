Setting up Cordova and Android
-------------------------------

This guide covers the following versions:

- cordova@7.1.0
- jdk8.152
- node@8.9.1
- android sdk 3859397
- gradle@4.3.1

You should start with the specific version numbers in order to ensure success. Android SDK, in particular,
has a habit of regularly moving their tools around from version to version.

Once you have successfully set up the environment, you can retrace the steps but with the latest versions.

Contrary to most advice, you do not need to install Android Studio.

Setting up Cordova
-------------------

1. Install NodeJS (32 bit)

   I suggest you use 32bit node because some teams use Node that comes with Visual Studio, which is 32 bits.
   If your team is in the habit of checking in `node_modules`, 32bit and 64bit binary packages like Sass 
   are different, and you'll less likely run into difficulties.
   
       wget https://nodejs.org/dist/v8.9.1/node-v8.9.1-x86.msi -OutFile node-v8.9.1-x86.msi
       msiexec /i node-v8.9.1-x86.msi
    
2. Install Cordova

       npm i cordova@7.1.0 -g

Setting up Android
-------------------

1. Manually download JDK 8 (There is no official support for JDK 9 for Android)
   from [Oracle](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
   Select `jdk-8u152-windows-x64.exe`. Once downloaded, you can install it from the command line.
   
       jdk.exe /s ADDLOCAL="ToolsFeature"
   
2. Set JAVA_HOME and path

       $jdk = gci 'C:\Program Files\Java\jdk*'  | sort LastWriteTime | Select-Object -Last 1
       $jdkHome = $jdk.FullName
       If ($env:JAVA_HOME -ne $jdkHome)
       {
           $env:JAVA_HOME = $jdkHome
           [Environment]::SetEnvironmentVariable("JAVA_HOME", $env:JAVA_HOME, [System.EnvironmentVariableTarget]::User)
       }
       
       If ($env:Path.Split(';') -notcontains "$jdkHome\Bin")
       {
           $env:Path = "${env:PATH};$jdkHome\Bin"
           [Environment]::SetEnvironmentVariable("PATH", $env:PATH, [System.EnvironmentVariableTarget]::User)
       }
       
3. PowerShell to download Android SDK

       wget https://dl.google.com/android/repository/sdk-tools-windows-3859397.zip -OutFile sdk-tools-windows.zip
    
       Expand-Archive sdk-tools-windows.zip
    
4. Use sdkmanager to download other dependencies

       # create an empty repositories.cfg file (or sdkmanager will barf)
       $repositoriesPath = "${env:HOMEPATH}\.android\repositories.cfg"
       If (-Not (Test-Path $repositoriesPath))
       {
         Set-Content $repositoriesPath ""
       }
     
       
5. Cordova requires `android-26`

       cd sdk-tools-windows
    
       .\tools\bin\sdkmanager.bat "platforms;android-26"
       .\tools\bin\sdkmanager.bat "platform-tools"

       # Sign EULA agreement
       .\tools\bin\sdkmanager.bat --licenses 

6. Install Gradle and add it to the PATH

   Android Studio ships with Gradle, and Cordova will look for the Android Studio's version, falling back
   to your locally installed one. However, Android Studio is 1 GB download. We suggest you install Gradle manually.
   
       wget https://services.gradle.org/distributions/gradle-4.3.1-bin.zip -Outfile gradle.zip
       
       Expand-Archive gradle.zip -DestinationPath C:\
       $gradleBinPath = "C:\Gradle-4.3.1\bin"
       If ($env:Path.Split(';') -notcontains $gradleBinPath)
       {
           $env:Path = "${env:PATH};$gradleBinPath"
           [Environment]::SetEnvironmentVariable("PATH", $env:PATH, [System.EnvironmentVariableTarget]::User)
       }
       
Setting up Emulator (optional)
---------------------------------

 1. Download system image to `$ANDROID_HOME\system-images\...`, sign the EULA

        tools\bin\sdkmanager.bat "system-images;android-26;google_apis;x86"
        tools\bin\sdkmanager.bat --licenses
 
 2. Download Intel HAXM and install it manually
 
        tools\bin\sdkmanager.bat "extras;intel;Hardware_Accelerated_Execution_Manager"
        pushd extras\intel\Hardware_Accelerated_Execution_Manager
        .\silent-install.bat
        popd
        
 3. Create virtual device
 
        tools\bin\avdmanager create avd -n "GoogleApisx86" --package "system-images;android-26;google_apis;x86" --abi "google_apis/x86"
 
 4. Disable Hyper-V with PowerShell
 
        Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
        
