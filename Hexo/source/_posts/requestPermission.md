---
title: Android 6.0 后的新授权机制
date: 2016-03-15 19:50:25
category: Android 进阶
---
以前我们需要获得某个权限的时候，需要在Android 配置文件中注册该权限，只要注册了该权限，在用户安装的时候，会弹出所有的权限列表，并且默认是同意的，用户可以在安装前，通过手动关闭某个权限。这样虽然对我们开发者挺好，但对用户而言，其实，存在很大的安全隐患。从Android 6.0(API level 23)开始，用户授权变成了在app运行的时候请求权限。

![Activity异常生命周期](/uploads/runtimepermission.jpg)

从android6.0开始，权限主要分为 **normal** 和 dangerous两类:

**Normal** 权限不直接涉及用户的隐私，如果在Android Manifest文件中注册了normal权限，系统自动授予许可。
**Dangerous** 权限可以给应用程序访问用户的机密数据，如果你在ANdroid Manifest文件中注册了一个危险的权限，应用程序必须经过用户审批同意过后，才能获得该权限.
**注意:**从android6.0(api level 23)开始，用户需要审批权限，低于6.0，仍然使用的以前的权限机制，所以，不需要担心，自己以前的app因为没有向用户请求权限而报错.

### Normal权限列表
> android.permission.ACCESS_LOCATION_EXTRA_COMMANDS
android.permission.ACCESS_NETWORK_STATE
android.permission.ACCESS_NOTIFICATION_POLICY
android.permission.ACCESS_WIFI_STATE
android.permission.ACCESS_WIMAX_STATE
android.permission.BLUETOOTH
android.permission.BLUETOOTH_ADMIN
android.permission.BROADCAST_STICKY
android.permission.CHANGE_NETWORK_STATE
android.permission.CHANGE_WIFI_MULTICAST_STATE
android.permission.CHANGE_WIFI_STATE
android.permission.CHANGE_WIMAX_STATE
android.permission.DISABLE_KEYGUARD
android.permission.EXPAND_STATUS_BAR
android.permission.FLASHLIGHT
android.permission.GET_ACCOUNTS
android.permission.GET_PACKAGE_SIZE
android.permission.INTERNET
android.permission.KILL_BACKGROUND_PROCESSES
android.permission.MODIFY_AUDIO_SETTINGS
android.permission.NFC
android.permission.READ_SYNC_SETTINGS
android.permission.READ_SYNC_STATS
android.permission.RECEIVE_BOOT_COMPLETED
android.permission.REORDER_TASKS
android.permission.REQUEST_INSTALL_PACKAGES
android.permission.SET_TIME_ZONE
android.permission.SET_WALLPAPER
android.permission.SET_WALLPAPER_HINTS
android.permission.SUBSCRIBED_FEEDS_READ
android.permission.TRANSMIT_IR
android.permission.USE_FINGERPRINT
android.permission.VIBRATE
android.permission.WAKE_LOCK
android.permission.WRITE_SYNC_SETTINGS
com.android.alarm.permission.SET_ALARM
com.android.launcher.permission.INSTALL_SHORTCUT
com.android.launcher.permission.UNINSTALL_SHORTCUT
注:除了以上列举的权限，其余没列举的，属于Dangerous权限.

### 检查是否有权限
```java
//Assume this Activity is the current activity
int permissionCheck = ContextCompat.checkSelfPermission(thisActivity,Manifest.permission.WRITE_CALENDAR);
```
如果应用程序请求授权，用户同意授权，该方法返回PackageManager.PERMISSION_GRANTED(一个值为0的常量)。如果用户拒绝授权，该方法返回PackageManager.PERMISSION_DENIED(一个值为-1的常量)

### 请求权限
如果你的应用需要一个dangerous权限，就需要向用户请求授权，Android提供了集中方法可用于请求许可。调用这些方法可以提供一个标准的Android对话框。注意:无法对请求授权对话款进行定制。

**以下代码为检查应用程序是否允许读取用户的联系人示例代码,如果未授权，则请求授权**
```java
// Here, thisActivity is the current activity
if (ContextCompat.checkSelfPermission(thisActivity,
                Manifest.permission.READ_CONTACTS)
        != PackageManager.PERMISSION_GRANTED) {

    // Should we show an explanation?
    if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity,
            Manifest.permission.READ_CONTACTS)) {

        // Show an expanation to the user *asynchronously* -- don't block
        // this thread waiting for the user's response! After the user
        // sees the explanation, try again to request the permission.

    } else {

        // No explanation needed, we can request the permission.

        ActivityCompat.requestPermissions(thisActivity,
                new String[]{Manifest.permission.READ_CONTACTS},
                MY_PERMISSIONS_REQUEST_READ_CONTACTS);

        // MY_PERMISSIONS_REQUEST_READ_CONTACTS is an
        // app-defined int constant. The callback method gets the
        // result of the request.
    }
}

```
注意:当应用程序调用requestPermissions(),系统会显示一个标准的对话框(该对话框无法定制)给用户。如果需要提前向用户解释为什么需要该权限，你可以在调用requestPermission()之前弹出提示解释。

### 处理权限请求响应
当应用程序请求权限，系统向用户提供了一个对话框。当用户响应，系统调用应用程序的onRequestPermissionsResult()方法，并传递响应参数。你可以重写该方法，在该方法中，查看是否请求授权成功。
示例代码:
```java
@Override
public void onRequestPermissionsResult(int requestCode,
        String permissions[], int[] grantResults) {
    switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                // permission was granted, yay! Do the
                // contacts-related task you need to do.

            } else {

                // permission denied, boo! Disable the
                // functionality that depends on this permission.
            }
            return;
        }

        // other 'case' lines to check for other
        // permissions this app might request
    }
}
```

### 一次请求多个权限.
有的时候，我们需要一次性请求多个权限。代码如下:
```java
final private int REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS = 124;
 
private void insertDummyContactWrapper() {
    List<String> permissionsNeeded = new ArrayList<String>();
 
    final List<String> permissionsList = new ArrayList<String>();
    if (!addPermission(permissionsList, Manifest.permission.ACCESS_FINE_LOCATION))
        permissionsNeeded.add("GPS");
    if (!addPermission(permissionsList, Manifest.permission.READ_CONTACTS))
        permissionsNeeded.add("Read Contacts");
    if (!addPermission(permissionsList, Manifest.permission.WRITE_CONTACTS))
        permissionsNeeded.add("Write Contacts");
 
    if (permissionsList.size() > 0) {
        if (permissionsNeeded.size() > 0) {
            // Need Rationale
            String message = "You need to grant access to " + permissionsNeeded.get(0);
            for (int i = 1; i < permissionsNeeded.size(); i++)
                message = message + ", " + permissionsNeeded.get(i);
            showMessageOKCancel(message,
                    new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            requestPermissions(permissionsList.toArray(new String[permissionsList.size()]),
                                    REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS);
                        }
                    });
            return;
        }
        requestPermissions(permissionsList.toArray(new String[permissionsList.size()]),
                REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS);
        return;
    }
 
    insertDummyContact();
}
 
private boolean addPermission(List<String> permissionsList, String permission) {
    if (checkSelfPermission(permission) != PackageManager.PERMISSION_GRANTED) {
        permissionsList.add(permission);
        // Check for Rationale Option
        if (!shouldShowRequestPermissionRationale(permission))
            return false;
    }
    return true;
}
```
如果所有权限被授权，就会回调onRequestPermissionsResult.代码如下:
```java
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    switch (requestCode) {
        case REQUEST_CODE_ASK_MULTIPLE_PERMISSIONS:
            {
            Map<String, Integer> perms = new HashMap<String, Integer>();
            // Initial
            perms.put(Manifest.permission.ACCESS_FINE_LOCATION, PackageManager.PERMISSION_GRANTED);
            perms.put(Manifest.permission.READ_CONTACTS, PackageManager.PERMISSION_GRANTED);
            perms.put(Manifest.permission.WRITE_CONTACTS, PackageManager.PERMISSION_GRANTED);
            // Fill with results
            for (int i = 0; i < permissions.length; i++)
                perms.put(permissions[i], grantResults[i]);
            // Check for ACCESS_FINE_LOCATION
            if (perms.get(Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED
                    && perms.get(Manifest.permission.READ_CONTACTS) == PackageManager.PERMISSION_GRANTED
                    && perms.get(Manifest.permission.WRITE_CONTACTS) == PackageManager.PERMISSION_GRANTED) {
                // All Permissions Granted
                insertDummyContact();
            } else {
                // Permission Denied
                Toast.makeText(MainActivity.this, "Some Permission is Denied", Toast.LENGTH_SHORT)
                        .show();
            }
            }
            break;
        default:
            super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }
}
```

### 总结
只有很少的一部分权限需要在运行时请求授权，比如:录音权限。大多数常用权限在安装时会自动授权，当然，需要在Android配置文件中配置，例如:网络访问权限。
注意：如果你的代码的代码不支持新权限，却是android 6.0的系统，请不要设置targetSdkVersion 23，请将targetSdkVersion 设置为23一下。

### 参考文献:
[Android官方文档:http://developer.android.com/intl/zh-cn/training/permissions/requesting.html](http://developer.android.com/intl/zh-cn/training/permissions/requesting.html)
