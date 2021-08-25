# The Mobile Hacking CheatSheet

The Mobile Hacking CheatSheet is an attempt to summarise a few interesting basics info regarding tools and commands needed to assess the security of Android and iOS mobile applications.

You can get the PDF versions:

* [Mobile Hacking iOS CheatSheet](https://github.com/randorisec/MobileHackingCheatSheet/blob/master/pdf/Mobile_Hacking_iOS_cheatsheet_v1.0.pdf)
* [Mobile Hacking Android CheatSheet](https://github.com/randorisec/MobileHackingCheatSheet/blob/master/pdf/Mobile_Hacking_Android_cheatsheet_v1.0.pdf)

And the PNG versions:

* [Mobile Hacking iOS CheatSheet](https://github.com/randorisec/MobileHackingCheatSheet/blob/master/LEGACY.md#ios-cheatsheet)
* [Mobile Hacking Android CheatSheet](https://github.com/randorisec/MobileHackingCheatSheet/blob/master/LEGACY.md#android-cheatsheet)

## Main Steps

1. Review the codebase
2. Run the app
3. Dynamic instrumentation
4. Analyze network communications

## OWASP Mobile Security Testing Project

### Mobile Security Testing Guide

[https://github.com/OWASP/owasp-mstg](https://github.com/OWASP/owasp-mstg)

### Mobile Application Security Verification Standard 

[https://github.com/OWASP/owasp-masvs](https://github.com/OWASP/owasp-masvs)

### Mobile Security Checklist

[https://github.com/OWASP/owasp-mstg/tree/master/Checklists](https://github.com/OWASP/owasp-mstg/tree/master/Checklists])

## Android CheatSheet

### APK Structure

* __META-INF__: Files related to the signature scheme (v1 scheme only)
* __lib__: Folder containing native libraries (ARM, MIPS, x86, x64)
* __assets__: Folder containing application specific files
* __res__: Folder containing all the resources files (layouts, strings, etc.) of the application
* __classes.dex [classes2.dex] ...__: Dalvik bytecode of the application
* __AndroidManifest.xml__: Manifest file describing essential information about the app (permissions, components, etc.)

### Package Name

The package name represents the app’s unique identifier (e.g. for YouTube):

```
com.google.android.youtube
```

### Data Storage

User applications

```bash
/data/data/<package-name>/
```

Shared Preferences Files

```bash
/data/data/<package-name>/shared_prefs/
```

SQLite Databases

```bash
/data/data/<package-name>/databases/
```

Internal Storage

```bash
/data/data/<package-name>/files/
```

### adb

Connect throug USB

```bash
adb -d shell
```

Connect through TCP/IP

```bash
adb -e shell
```

Get a shell or execute the specified command

```bash
adb shell [cmd]
```

List processes

```bash
adb shell ps
```

List Android devices connected to your machine

```bash
adb devices
```

Dump the log messages from Android system

```bash
adb logcat
```

Copy local file to Android device

```bash
adb push <local> <device>
```

Copy file from the Android device

```bash
adb pull <remote> <local>
```

Install APK file on the Android device

```bash
adb install <APK_file>
```

Install an App Bundle

```bash
adb install-multiple <APK_file1> <APK_file2> <APK_file3> ...
```

Set-up port forwarding using TCP protocol from host to Android device

```bash
adb forward tcp:<local_port> tcp:remote_port
```

List all packages on the device

```bash
adb shell pm list packages
```

Find the path where the APK is stored for the selected package name

```bash
adb shell pm path <package-name>
```

List only installed apps (not system apps) and the associated path

```bash
adb shell pm list packages -f -3
```

List packages names matching the specified pattern

```bash
adb shell pm list packages -f -3 [pattern]
```

### Application Signing

For signing your APK file, you have 2 options

* [jarsigner](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/jarsigner.html): Only supports v1 signature scheme (JAR signature)

  ```terminal
  jarsigner -verbose -keystore <keystore_name> -storepass <keystore_password> <APK_file> <alias_name>
  ```

* [apksigner](https://developer.android.com/studio/command-line/apksigner): Official tool from Android SDK (since version 24.0.3), which supports all the signature schemes (from v1 to v4)

  ```terminal
  apksigner sign --ks <keystore_name> --ks-pass pass:<keystore_password> <APK_file> 
  ```

To create your own keystore, the following one-liner can be used:

```bash
keytool -genkeypair -dname "cn=John Doe, ou=Security, o=Randorisec, c=FR" -alias <alias_name> 
-keystore <keystore_name> -storepass <keystore_password> -validity <days> -keyalg RSA -keysize 2048 -sigalg SHA1withRSA
```

### Code Tampering

To tamper an APK file, the foolowing steps should be performed:

1. Disassemble the app with `apktool` and save the smali code into output directory

    ```bash
    apktool d <APK_file> -o <directory_output>
    ```

1. Modify the smali code of your app (or the resource files if needed)

1. Build the modified APK with `apktool`

    ```bash
    apktool b <directory_output> -o <new_APK_file> 
    ```

1. Sign the APK (see [Application Signing](application-signing))

1. (Optional) Use `zipalign` to provide optimization to the APK file

    ```bash
    zipalign -fv 4 <input_APK> <output_APK>
    ```

### Frida

#### Installation

Install Frida and Python bindings on your system using `pip`

```bash
pip install frida frida-tools
```

Download the Frida server binary matching the targeted architecture and your Frida version

```bash
VER=`frida --version`
ABI=`adb shell getprop ro.product.cpu.abi`
wget https://github.com/frida/frida/releases/download/$VER/frida-server-$VER-android-$ABI.xz
xz -d frida-server-$VER-android-$ABI.xz
```

Upload and execute the Frida server binary on your Android device (root privileges are needed)

```bash
VER=`frida --version`
ABI=`adb shell getprop ro.product.cpu.abi`
adb root
adb push frida-server-$VER-android-$ABI /data/local/tmp/frida
adb shell "chmod 755 /data/local/tmp/frida" 
adb shell "/data/local/tmp/frida"
```

#### Tools

List running processes (emulators or devices connected through USB)

```bash
frida-ps -U 
```

List only installed applications

```bash
frida-ps -U  -i
```

Attach Frida client to the specified application (emulator or device connected through USB)

```bash
frida -U <package_name>
```

Spawn the specified application (emulator or device connected through USB)

```bash
frida -U -f <package_name> 
```

Spawn the specified application without any pause at the beginning (emulator or device connected through USB)

```bash
frida -U -f <package_name> --no-pause
```

Load a Frida script when attaching to the specified application

```bash
frida -U -l <script_file> <package_name>
```

### Objection

Inject Frida Gadget library inside an APK file by specifying the targeted architecture (if emulator not running or device not connected)

```bash
objection patchapk --source <APK_file> -V <frida_version> --architecture <arch>
```

Inject Frida Gadget library inside an APK file using lastest Frida version available on Github (if emulator running or device connected to the device)

```bash
objection patchapk --source <APK_file>
```

### SSL/TLS Interception with BurpSuite

#### Before Android 7

1. Launch `BurpSuite` and modify Proxy settings in order to listen on "All interfaces" (or a specific interface)
1. Edit the Wireless network settings in your device or the emulator proxy settings (Android Studio)
1. Export the CA certificate from Burp and save it with ".cer" extension
1. Push the exported certificate on the device with adb (into the SD card)
1. Go to "Settings->Security" and select "Install from device storage"
1. Select for "Credentials use" select "VPN and apps"

References:

* [Configuring an Android device to work with Burp](https://portswigger.net/support/configuring-an-android-device-to-work-with-burp)
* [Installing BurpSuite's CA certificate in an Android device](https://portswigger.net/support/installing-burp-suites-ca-certificate-in-an-android-device)

#### After Android 7

From Android 7, the Android system no longer trusts the user supplied CA certificates. To be able to intercept SSL/TLS communication, you have 3 options:

1. Use an older version of Android
1. Use a rooted device and install the BurpSuite CA certificate inside the sytem store certificate
1. Tamper the targeted application in order to re-enable the user store certificate

In order to tamper the targeted Android application, we are going to add or modify the network security configuration file. This file on recent Android versions allows to force the application to trust the user supplied CA certificates. The following steps should be performed:

1. Install the Burpsuite's CA certificate on your Android device (see [Before Android 7](before-android-7))
1. Disassemble the targeted app (APK file) with `apktool`
1. Add or modify the `network_security_config.xml` file (usually on `res/xml/` folder). The content of the file should be:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <network-security-config>
      <base-config>
        <trust-anchors>
          <certificates src="system" />
          <certificates src="user" />
        </trust-anchors>
      </base-config>
    </network-security-config>

1. If the `network_security_config.xml` file is not present on your app, the `AndroidManifest.xml`also need to be modified by adding the `networkSecurityConfig` tag as follow:

    ```xml
    <application android:name="AppName" android:networkSecurityConfig="@xml/network_security_config">
    ```

1. Build the modified app with `apktool` and then sign the newly created APK file (see [Application Signing](application-signing))

### Content Provider

Query a Content Provider

```bash
adb shell content query --uri content://<provider_authority_name>/<table_name>
```

Insert an element on a Content Provider

```bash
adb shell content insert --uri content://<provider_authority_name>/<table_name> 
--bind <param_name>:<param_type>:<param_value>
```

Delete a row on a Content Provider

```bash
adb shell content delete --uri content://<provider_authority_name>/<table_name> 
--where "<param_name>='<param_value>'"
```

### Activity Manager

Start an Activity with the specified Intent

```bash
adb shell am start -n <package_name/activity_name> -a <intent_action>
```

Start an Activity with the specified Intent and extra parameters

```bash
adb shell am start -n <package_name/activity_name> -a <intent_action> --es <param_name> <string_value> --ez <param_name> <boolean_value>  --ei <param_name> <int_value> …
```

## iOS CheatSheet

### Filesystem

* App list database

```bash
/User/Library/FrontBoard/applicationState.db 
```

* Binary directory: include all the static resources of the app

```bash
/private/var/containers/Bundle/Application/UUID/App.app 
```

* Path of the binary (executable)

```bash
/private/var/containers/Bundle/Application/UUID/App.app/App
```

* App metadata: configuration of the app (icon to display, supported document types, etc.)

```bash
/private/var/containers/Bundle/Application/UUID/App.app/Info.plist
```

* Data directory

```bash
/private/var/mobile/Containers/Data/Application/Data-UUID
```

*UUID (Universally Unique Identifier): random 36 alphanumeric characters string unique to the app
Data-UUID: random 36 alphanumeric characters string unique to the app*

### Bundle ID

* The bundle ID represents the app’s unique identifier (e.g. for YouTube)

```
com.google.ios.youtube
```

### How to find the data and binary directories

Grep is the not-so-quick ‘n dirty way to find where are the data and binary directories of your app

```bash
iPhone:~ root# grep -r <App_name> /private/var/*
```

### How to find the data and binary directories and the Bundle ID

By launching Frida with the ios-app-info script

```bash
# frida -U <App_name> -c dki/ios-app-info
```

And then

```bash
[iPhone::App]-> appInfo()
```

Or manually by opening the app list database

```bash
iPhone:~ root# sqlite3 /User/Library/FrontBoard/applicationState.db
```

And displaying the key_tab table to get the binary directories

```bash
sqlite> select * from key_tab;
```

Or displaying the application_identifier_tab table to get the bundle IDs

```bash
sqlite> select * from application_identifier_tab;
```

### App decryption

1. Add [https://level3tjg.xyz/repo/](https://level3tjg.xyz/repo/) src to Cydia and install bfdecrypt tool
2. Go to bfdecrypt pref pane in Settings and set the app to decrypt
3. Launch the app to decrypt: decrypted IPA is stored in the Documents folder of the app

### Dynamic analysis with Frida

List all processes

```bash
# frida-ps –U
```

Analyse the calls to a method by launching Frida with the objc-method-observer script

```bash
# frida -U <App_name> –c mrmacete/objc-method-observer
```

And then using the command ```observeSomething```

```bash
[iPhone::App]-> observeSomething('*[* *<Method_name>*]’);
```

Hook the calls to the method <Method_name>

```bash
# frida-trace -U <App_name> -m "-[* <Method_name>*]"
```

Then open the JavaScript handler file to edit the ```onEnter``` or ```onLeave``` functions to manipulate the behavior of the app

### Dynamic analysis with Objection

Inject objection

```bash
objection -g "<App_name>" explore
```

List the classes (output will contain thousands of lines)

```bash
ios hooking list classes
```

List the methods of a class

```bash
ios hooking list class_methods <Class_name>
```

Search for classes|methods names containing <String>

```bash
ios hooking search classes|methods <String>
```

Analyse the calls to the method <Method_name>

```bash
ios hooking watch method "-[<Class_name> <Method_name>]"
```

Hook the <Method_name> and return true to each call

```bash
ios hooking set return_value "-[<Class_name> <Method_name>]" true
```

### Get the NSLog (syslog)

Impactor (http://www.cydiaimpactor.com) let you display the NSLog (syslog) on command line

```bash
# ./Impactor idevicesyslog -u <UDID>
```

### SSL Interception with BurpSuite

1. Launch Burp and modify proxy settings in order to listen on “All interfaces”
2. Browse to the IP/port of your Burp proxy using Safari
3. Tap on the “CA Certificate” at the top right of the screen
4. Tap on “Allow” on the pop-up asking to download a configuration profile
5. Go to “Settings->Profile Downloaded” and select the “PortSwigger CA” profile
6. Tap on “Install” then “Install” again and then “Install” one last time
7. Edit the wireless network settings on your device to set a proxy (“Settings->Wi-Fi” then tap on the blue “i”, slide to the bottom of the screen and tap on “Configure Proxy”)
8. Tap on ”Manual”, set the IP/port of your Burp proxy, tap on “Save”
9. Go to “Settings->General->About->Certificate Trust Settings” & toggle on the PortSwiggerCA

### Bypass SSL Pinning using SSL Kill Switch 2

Download and install SSL Kill Switch 2 tweak

```bash
# wget https://github.com/nabla-c0d3/ssl-kill-switch2/releases/download/0.14/com.nablac0d3.sslkillswitch2_0.14.deb
# dpkg -i com.nablac0d3.sslkillswitch2_0.14.deb
# killall -HUP SpringBoard 
```

Go to “Settings->SSL Kill Switch 2” to ”Disable Certificate Validation”

### UDID (Unique Device Identifier)

UDID is a string that is used to identify a device. Needed for some operations like signature, app installation, network monitoring.

* Get UDID with MacOS

```bash
# idevice_id –l
```

or

```bash
# ioreg -p IOUSB -l | grep "USB Serial"
```

or by launching Impactor without parameters

* Get UDID with Linux

```bash
# usbfluxctl list 
```

or

```bash
# lsusb -s :`lsusb | grep iPhone | cut -d ' ' -f 4 | sed 's/://'` -v | grep iSerial | awk '{print $3}'
```

or by launching Impactor without parameters

### Network capture (works also on non jailbroken devices)

* With MacOS (install Xcode and additional tools and connect the device with USB)

```bash
# rvictl -s <UDID>
# tcpdump or tshark or wireshark –i rvi0
```

* With Linux (get https://github.com/gh2o/rvi_capture and connect the device with USB) (works with iOS <= 12)

```bash
# ./rvi_capture.py --udid <UDID> iPhone.pcap
```

### Sideloading an app

Sideloading an app including an instrumentation library like Frida let you interact with the app even if it’s installed on a non jailbroken device.

#### With IPAPatch

Here’s the process to do it with IPAPatch:
Clone the IPAPatch project

```bash
# git clone https://github.com/Naituw/IPAPatch
```

Move the IPA of the app you want to sideload to the Assets directory

```bash
# mv <IPAfile> IPAPatch/Assets/
```

Download the FridaGadget library (in Assets/Dylibs/FridaGadget.dylib)

```bash
# curl -O https://build.frida.re/frida/ios/lib/FridaGadget.dylib
```

Select the identity to sign the app

```bash
# security find-identity -p codesigning –v
```

Sign FridaGadget library

```bash
# codesign -f -s <IDENTITY> FridaGadget.dylib
```

Then open IPAPatch Xcode project, Build and Run.

#### With Objection

Here’s the process to do it with Objection (detailed steps on https://github.com/sensepost/objection/wiki/Patching-iOS-Applications)

```bash
# security find-identity -p codesigning –v
# objection patchipa --source <IPAfile> --codesign-signature <IDENTITY>
# unzip <patchedIPAfile>
# ios-deploy --bundle Payload/my-app.app -W –d
# objection explore
```

### Data Protection Class

Four levels are provided by iOS to encrypt automatically files on the device:

* ```NSProtectionComplete```: file is only accessible when device is unlocked (files are encrypted with a key derived from the user PIN code & an AES key generated by the device)
* ```NSProtectionCompleteUntilFirstUserAuthentication```: (defaut class) same except as before, but the decryption key is not deleted when the device is locked
* ```ProtectedUnlessOpen```: file is accessible until open
* ```NoProtection```: file is accessible even if device is locked

### Get Data Protection Class

By launching Frida with the ios-dataprotection script

```bash
# frida -U <App_name> -c ay-kay/ios-dataprotection
```

## License

The Mobile Hacking CheatSheet is an open source project released under the [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.fr) licence.
