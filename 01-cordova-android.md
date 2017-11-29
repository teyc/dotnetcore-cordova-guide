Setting up Cordova and Android
-------------------------------

Contrary to most advice, you do not need to install Android Studio.

1. Manually install JDK 8 (There is no official support for JDK 9 for Android)
   from http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
   Select `jdk-8u152-windows-x64.exe`
   
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
       
2. PowerShell to download Android SDK

       wget https://dl.google.com/android/repository/sdk-tools-windows-3859397.zip -OutFile sdk-tools-windows.zip
    
       Expand-Archive sdk-tools-windows.zip
    
3. Use sdkmanager to download other dependencies

       # create an empty repositories.cfg file (or sdkmanager will barf)
       $repositoriesPath = "${env:HOMEPATH}\.android\repositories.cfg"
       If (-Not (Test-Path $repositoriesPath))
       {
         Set-Content $repositoriesPath ""
       }
     
       cd sdk-tools-windows
    
       # Updates list of files, EULA agreement
       .\tools\bin\sdkmanager.bat --update
     
       cd ..
       
4. Cordova requires `android-26`

        .\tools\bin\sdkmanager.bat "platforms;android-26"

5. Install NodeJS

       wget https://nodejs.org/dist/v8.9.1/node-v8.9.1-x86.msi -OutFile node-v8.9.1-x86.msi
       msiexec /i node-v8.9.1-x86.msi
    
6. Install Cordova

       npm i cordova -g
       

   
