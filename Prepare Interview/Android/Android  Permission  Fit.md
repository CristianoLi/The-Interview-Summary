### Android版本适配


**在Android开发中新版本的推出会带来一些版本的差异，而为了保障我们的应用能够在我们所支持的各个Android版本手机下正常运行，这就需要我们针对一些版本来进行适配。且今年所有应用需要按照[《移动应用软件高API等级预置与分发自律公约》](https://baike.baidu.com/item/%E7%A7%BB%E5%8A%A8%E5%BA%94%E7%94%A8%E8%BD%AF%E4%BB%B6%E9%AB%98API%E7%AD%89%E7%BA%A7%E9%A2%84%E7%BD%AE%E4%B8%8E%E5%88%86%E5%8F%91%E8%87%AA%E5%BE%8B%E5%85%AC%E7%BA%A6/22759862?fr=aladdin)的约定，应用应基于Android O开发，否则无法通过应用商店的上架。**


 [TOC]


--------
#### 在此之前我们需要明确两点：
#####  1. APP向前兼容和APP向后兼容
 ①. 向前兼容比较容易理解，即使我们所开发的应用程序可以在Android更高版本的手机系统上运行。 至于APP如何向前兼容？ 这需要我们来通过改变`targetSdkVersion`参数来实现。
通过设置该参数，如果实际的手机版本号 > 应用设置的`targetSdkVersion`,那么手机会按照`targetSdkVersion`设置的版本号来运行。当实际的手机版本号 <= 应用设置的`targetSdkVersion`时按照实际的手机版本号来运行。 

	1、targetSDKVersion < 23 & API(手机系统) < 6.0 ：安装时默认获得权限，且用户无法在安装App之后取消权限。 
	2、targetSDKVersion < 23 & API(手机系统) >= 6.0 ：安装时默认获得权限，但是用户可以在安装App完成后取消授权。
	3、targetSDKVersion >= 23 & API(手机系统) < 6.0 ：安装时默认获得权限，且用户无法在安装App之后取消权限。
	4、targetSDKVersion >= 23 & API(手机系统) >= 6.0 ：安装时不会获得权限，可以在运行时向用户申请权限。用户授权以后仍然可以在设置界面中取消授权。
	
	
 ②. 向后兼任即我们所开发的应用程序可以在Android较低版本的手机系统上运行。我们通过改变`minSdkVersion`可以设置最低支持的系统版本，通过支持库(`support library`)可以使得低版本手机可以支持高版本的内容。例如appcompat-v4、appcompat-v7使得低版本手机可以支持`Material Design`。
	
##### 2. compileSdkVersion、minSdkVersion 和 targetSdkVersion 这三个参数
- **compileSdkVersion**：
告诉Gradle，我们是用哪一版本的Android SDK去`编译程序`的。推荐总是使用最新的 SDK 进行编译（默认就是最新的）修改 compileSdkVersion `不会改变运行时的行为`。当修改了 compileSdkVersion 的时候，可能会出现新的编译警告、编译错误，但新的 compileSdkVersion 不会被包含到 APK 中：它纯粹只是在编译的时候使用。

	*（如果使用 Support Library ，那么使用最新发布的 Support Library 就需要使用最新的 SDK 编译）*
	
- **minSdkVersion**:
指明应用程序运行所需的最小API level。(我们应用程序所使用的minSdkVersion 需要大于等于所使用库的 minSdkVersion。)

	*（设置uses-sdk可突破第三方库的 minSdkVersion 限制，不建议使用）*

- **targetSdkVersion**：
targetSdkVersion 是 Android 提供向前兼容的主要依据。
将 target 更新为最新的 SDK 是所有应用都应该优先处理的事情。但这不意味着你一定要使用所有新引入的功能，也不意味着你可以不做任何`测试`就盲目地更新 targetSdkVersion ，请一定在更新 targetSdkVersion 之前做测试！如果平台的API Level高于你的应用程序中的targetSdkVersion属性指定的值，系统会开启`兼容行为`来确保你的应用程序继续以期望的形式来运行。

	*（`minSdkVersion <= targetSdkVersion <= compileSdkVersion`）*

--------


####  1. Android 6.0适配 （API 23）
#####  [Google官方文档](https://developer.android.google.cn/about/versions/marshmallow/android-6.0-changes)

主要的变化:
- ` 运行时权限 `
- 增加低电耗模式和应用待机模式 
- `移除对Apache HTTP客户端的支持 `
- 移除硬件标识符访问权 
- WLAN 和网络连接变更 
- 相机服务变更

- #####  运行时权限
	**从Android6.0开始，Android系统对权限的处理产生了很大的变化。如果APP运行的设备系统版本为Android6.0或更高，并且target在23或更高，那么dangerious级别的权限将由之前的安装时授予变成运行时动态申请。**
	
	**根据权限的敏感程度，这些权限一般分为三种：**

	（1）`Normal Permissions 普通权限`：直接manifest清单文件中写上注册就行了 。
			  正常权限涵盖应用需要访问其沙盒外部数据或资源，但对用户隐私或其他应用操作风险很小的区域。
	（2）`Dangerous Permissions 危险权限`：需要动态申请 。
			危险权限涵盖应用需要涉及用户隐私信息的数据或资源，或者可能对用户存储的数据或其他应用的操作产生影响的区域。例如，能够读取用户的联系人属于危险权限。如果应用声明其需要危险权限，则用户必须明确向应用授予该权限。
	（3）`Special Permissions 特殊权限`： 需在清单中声明该权限，且发送请求用户授权的 intent。
			其行为方式与正常权限及危险权限都不同。`SYSTEM_ALERT_WINDOW` 和 `WRITE_SETTINGS` 特别敏感，因此大多数应用不应该使用它们。如果某应用需要其中一种权限，必须在清单中声明该权限，并且发送请求用户授权的 intent。系统将向用户显示详细管理屏幕，以响应该 intent。
		
	[参考：危险权限列表](https://developer.android.google.cn/guide/topics/permissions/overview#permission-groups)
	
	[参考：运行时权限请求方式](https://developer.android.google.cn/training/permissions/requesting)

	##### 权限请求库 [RxPermissions](https://github.com/tbruyelle/RxPermissions)
	
	**`RxPermissions`是基于RxJava的动态权限框架。目前默认分支基于`RxJava2`。下面是简单的使用示例：**

	```
	RxPermissions rxPermission = new RxPermissions(this);
	rxPermission
		.requestEach(Manifest.permission.ACCESS_FINE_LOCATION,
	                        Manifest.permission.WRITE_EXTERNAL_STORAGE,
	                        Manifest.permission.READ_PHONE_STATE,
	                        Manifest.permission.READ_SMS,
	                        Manifest.permission.CAMERA,
	                        Manifest.permission.CALL_PHONE,
	                        Manifest.permission.SEND_SMS)
	                .subscribe(new Consumer<Permission>() {
	                    @Override
	                    public void accept(Permission permission) throws Exception {
	                        if (permission.granted) {
	                            // 用户已经同意该权限
	                            Log.d(TAG, permission.name + " is granted.");
	                        } else if (permission.shouldShowRequestPermissionRationale) {
	                            // 用户拒绝了该权限，没有选中『不再询问』（Never ask again）,那么下次再次启动时。还会提示请求权限的对话框
	                            Log.d(TAG, permission.name + " is denied. More info should be provided.");
	                        } else {
	                            // 用户拒绝了该权限，而且选中『不再询问』
	                            Log.d(TAG, permission.name + " is denied.");
	                        }
	                    }
	                });
	```


- ##### 移除对 `Apache HTTP`客户端的支持

	**Android 6.0 版移除了对 Apache HTTP 客户端的支持。如果您的应用使用该客户端，并以 Android 2.3（API 级别 9）或更高版本为目标平台，请改用 `HttpURLConnection` 类。此 API 效率更高，因为它可以通过透明压缩和响应缓存减少网络使用，并可最大限度降低耗电量。要继续使用 Apache HTTP API，您必须先在 build.gradle 文件中声明以下编译时依赖项 :**

	```
	android {
	    useLibrary 'org.apache.http.legacy'
	}
	```

----

#### 2. Android 7.0适配 （API 24）

#####  [Google官方文档](https://developer.android.google.cn/about/versions/nougat/android-7.0-changes)

主要变化：
- `私有文件访问权限更改`
- `多窗口支持（分屏显示）`
- `后台优化：移除三项隐式广播`
- 通知增强功能
- 随时随地低电耗模式
- 多语言区域支持，更多语言
- 新增的表情符号
- Chrome 和 WebView 配合使用
- `APK signature scheme v2`

- ##### 私有文件访问权限更改
**在targetSdkVersion >= 24的App中，但是如果我们没有去适配7.0。那么在调用安装页面，或修改用户头像操作时，就会失败。那么就需要你去适配7.0或是将targetSdkVersion改为24以下（不推荐）。**

	当通过`URI`传递文件会触发 FileUriExposedException。这就需要使用`FileProvider`。
	 

	**示例：调用系统相机拍照，裁切照片**
在Android7.0之前，假设你想调用系统相机拍照能够通过下面代码来进行：

	```
	File file=new File(Environment.getExternalStorageDirectory(), "/temp/"+System.currentTimeMillis() + ".jpg");
	if (!file.getParentFile().exists()){
		file.getParentFile().mkdirs();
	}
	Uri imageUri = Uri.fromFile(file);
	Intent intent = new Intent();
	intent.setAction(MediaStore.ACTION_IMAGE_CAPTURE);//设置Action为拍照
	intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);//将拍取的照片保存到指定URI
	startActivityForResult(intent,1006);
	```
在Android7.0上使用上述方式调用系统相拍照会抛出异常：`android.os.FileUriExposedException`
所以我们使用FileProvider 来操作：

	**1.在manifest清单文件里注冊provider**
	
	android:name：一般使用系统包提供的FileProvider。
	android:authorities：类似schema，命名空间之类，后面会用到。
	android:exported：false表示我们的provider不需要对外开放。
	android:grantUriPermissions：申明为true，你才能获取临时共享权限。


	```
	<provider
	    android:name="android.support.v4.content.FileProvider"
	    android:authorities="com.xxx.fileprovider"
	    android:grantUriPermissions="true"
	    android:exported="false">
	    <meta-data
	        android:name="android.support.FILE_PROVIDER_PATHS"
	        android:resource="@xml/file_paths" />
	</provider>
	```

	**2.指定共享的文件夹**
		
	物理路径相当于Environment.getExternalStorageDirectory() + /path/
	物理路径相当于Context.getFilesDir() + /path/
	物理路径相当于Context.getCacheDir() + /path/
	物理路径相当于Context.getExternalFilesDir(String) + /path/
	物理路径相当于Context.getExternalCacheDir() + /path/

	/storage/emulated/0 
	 /data/user/0/packname/files 
	 /data/user/0/packname/cache
	  /storage/emulated/0/Android/data/packname/files
	  /storage/emulated/0/Android/data/packname/cache
		
	```
	<xml version="1.0" encoding="utf-8"?>
		<resources>
		    <paths>
		        <external-path path="" name="camera_photos" />
		        <files-path path="" name="name" />    
		        <cache-path path="" name="name"  />
		        <external-files-path path="" name="name"  />
		        <external-cache-path  path="" name="name"/>
		    </paths>
		</resources>
	</xml>
	```
	**3.使用FileProvider **

	```
	File file=new File(Environment.getExternalStorageDirectory(), "/temp/"+System.currentTimeMillis() + ".jpg");
	if (!file.getParentFile().exists()){
		file.getParentFile().mkdirs();
	}
	Uri imageUri = FileProvider.getUriForFile(context, "com.xxx.fileprovider", file);//通过FileProvider创建一个content类型的Uri
	Intent intent = new Intent();
	intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION 
               | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);  //加入这一句表示授予目录临时共享权限
               
	intent.setAction(MediaStore.ACTION_IMAGE_CAPTURE);//设置Action为拍照
	intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);//将拍取的照片保存到指定URI
	startActivityForResult(intent,1006);
	```

- ##### 多窗口支持
Android 的SDK版本中给AndroidManifest里的Application（Activity）新增了一个属性：
	```
	android:resizeableActivity=["true" | "false"]
	```
该属性为true时，将支持多窗口，为false时不支持，展示时只能是全屏模式（PS：该属性不配置时，默认为true）
当你开发编译使用的SDK版本低于Android N时，如果你不希望系统去强制修改你的页面，比较偷懒的解决方案是：在AndroidMainifest中配置Applcaiton（或Activity任务栈里的首个入栈Activity）的android:screenOrientation属性。配了该属性后，你的页面将不支持多窗口，但是缺点就是Activity的朝向会被固定。

	[多窗口支持详细实现-参考Google官方文档](https://developer.android.google.cn/guide/topics/ui/multi-window.html)


- ##### 移除三项隐式广播
	**Android 7.0 移除了三项隐式广播，以帮助优化内存使用和电量消耗。此项变更很有必要，因为隐式广播会在后台频繁启动已注册侦听这些广播的应用。删除这些广播可以显著提升设备性能和用户体验。**

	- 在 Android 7.0上 应用不会收到 `CONNECTIVITY_ACTION` 广播，即使你在manifest清单文件中设置了请求接受这些事件的通知。 但在前台运行的应用如果使用BroadcastReceiver 请求接收通知，则仍可以在主线程中侦听 CONNECTIVITY_CHANGE。
	- 在 Android 7.0上应用无法发送或接收 `ACTION_NEW_PICTURE` 或`ACTION_NEW_VIDEO` 类型的广播。

	**应对策略：Android 框架提供多个解决方案来缓解对这些隐式广播的需求。 例如，JobScheduler API**


- ##### APK signature scheme v2
Android 7.0 引入一项新的应用签名方案 APK Signature Scheme v2，它能提供更快的应用安装时间和更多针对未授权 APK 文件更改的保护。在默认情况下，Android Studio 2.2 和 Android Plugin for Gradle 2.2 会使用 APK Signature Scheme v2 和传统签名方案来签署您的应用。

	1）只勾选`v1`签名就是传统方案签署，但是在7.0上不会使用V2安全的验证方式。
	2）只勾选`V2`签名`7.0以下`会显示未安装，`7.0上`则会使用了`V2`安全的验证方式。
	3）同时勾选V1和V2则所有版本都没问题。(但当使用给已有的`apk文件中添加空的渠道文件`时,将`不适用于V2`的签名方案)

----

#### 3. Android 8.0适配 （API 26）
#####  [Google官方文档](https://developer.android.google.cn/about/versions/oreo/android-8.0-changes)
主要变化：
- `后台执行限制`
- `权限授予变化`
- 启动图标
- `悬浮窗适配`
- `安装APK`
- `透明主题的Activity`
- 视图焦点
- 集合的处理
- `通知渠道`

- ##### 后台执行限制

	应用在两个方面受到限制：
	1). `后台服务限制`：处于空闲状态时，应用可以使用的后台服务存在限制。 这些限制不适用于前台服务，因为前台服务更容易引起用户注意。
	2). `广播限制`：除了有限的例外情况，应用无法使用清单注册隐式广播。 它们仍然可以在运行时注册这些广播，并且可以使用清单注册专门针对它们的显式广播。
	
	在大多数情况下，应用都可以使用` JobScheduler `克服这些限制。 这种方式让应用安排为在未活跃运行时执行工作，不过仍能够使系统可以在不影响用户体验的情况下安排这些作业。eg: [android-JobScheduler](https://github.com/googlesamples/android-JobScheduler)。
	后台任务google推荐方案使用 `WorkManager`，WorkManager可以自动维护后台任务，同时可适应不同的条件，同时满足后台Service和静态广播，内部维护着JobScheduler，而在6.0以下系统版本则可自动切换为`AlarmManager`。

- ##### 权限授予变化

	在 Android 8.0 之前，如果应用在运行时请求权限并且被授予该权限，系统会错误地将属于同一权限组并且在清单中注册的其他权限也一起授予应用。
	
	对于针对 Android 8.0 的应用，此行为已被纠正。`系统只会授予应用明确请求的权限`。然而，`之前一旦用户为应用授予某个权限，则所有后续对该权限组中权限的请求都将被自动批准`。
	
	假设某个应用在其清单中列出 READ_EXTERNAL_STORAGE 和 WRITE_EXTERNAL_STORAGE。应用请求 READ_EXTERNAL_STORAGE，并且用户授予了该权限。如果该应用针对的是 API 级别 24 或更低级别，系统还会同时授予 WRITE_EXTERNAL_STORAGE，因为该权限也属于同一 STORAGE 权限组并且也在清单中注册过。如果该应用针对的是 Android 8.0，则系统此时仅会授予 READ_EXTERNAL_STORAGE；不过，如果该应用后来又请求 WRITE_EXTERNAL_STORAGE，则系统会立即授予该权限，而不会提示用户。

- ##### 悬浮窗适配

	使用 `SYSTEM_ALERT_WINDOW` 权限的应用无法再使用以下窗口类型来在其他应用和系统窗口上方显示提醒窗口：
	
	TYPE_PHONE
	TYPE_PRIORITY_PHONE
	TYPE_SYSTEM_ALERT
	TYPE_SYSTEM_OVERLAY
	TYPE_SYSTEM_ERROR
	相反，应用必须使用名为 `TYPE_APPLICATION_OVERLAY` 的新窗口类型。
	
	使用 TYPE_APPLICATION_OVERLAY 窗口类型显示应用的提醒窗口时，请记住新窗口类型的以下特性：
	
	应用的提醒窗口`始终显示在状态栏和输入法等关键系统窗口的下面`。
	系统可以移动使用 TYPE_APPLICATION_OVERLAY 窗口类型的窗口或调整其大小，以改善屏幕显示效果。
	通过打开通知栏，用户可以访问设置来阻止应用显示使用 TYPE_APPLICATION_OVERLAY 窗口类型显示的提醒窗口。

	也就是说需要在之前的基础上判断一下：
	
	```
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
	    mWindowParams.type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY
	}else {
	    mWindowParams.type = WindowManager.LayoutParams.TYPE_SYSTEM_ALERT
	}
	```
	
	当然记得需要有权限

	```
	<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
	<uses-permission android:name="android.permission.SYSTEM_OVERLAY_WINDOW" />
	```


- ##### 安装APK
	Android 8.0去除了“允许未知来源”选项，所以如果我们的App有安装App的功能（检查更新之类的），那么会无法正常安装。
	首先在AndroidManifest文件中添加安装未知来源应用的权限:

	```
	<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
	```
	这样系统会自动询问用户完成授权。当然你也可以先使用 `canRequestPackageInstalls()`查询是否有此权限，如果没有的话使用`Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES`这个action将用户引导至安装未知应用权限界面去授权。


	

	```
	private static final int REQUEST_CODE_UNKNOWN_APP = 100;
	
	    private void installAPK(){
	
	        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
	            boolean hasInstallPermission = getPackageManager().canRequestPackageInstalls();
	            if (hasInstallPermission) {
	                //安装应用
	            } else {
	                //跳转至“安装未知应用”权限界面，引导用户开启权限
	                Uri selfPackageUri = Uri.parse("package:" + this.getPackageName());
	                Intent intent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES, selfPackageUri);
	                startActivityForResult(intent, REQUEST_CODE_UNKNOWN_APP);
	            }
	        }else {
	            //安装应用
	        }
	
	    }
	
	    //接收“安装未知应用”权限的开启结果
	    @Override
	    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	        super.onActivityResult(requestCode, resultCode, data);
	        if (requestCode == REQUEST_CODE_UNKNOWN_APP) {
	            installAPK();
	        }
	    }
	```


- ##### 透明主题的Activity
这个是在 `targetSdk=27`，Android为8.0的手机时，出现的`BUG`（因为官方已经在`8.1修复`）。问题的探究可以查看这里。
只有全屏不透明的activity才可以设置方向。否则报错：
	```
	java.lang.IllegalStateException: Only fullscreen opaque activities can request orientation
	```
	解决办法：
	要么去掉对应activity中的 `screenOrientation` 属性，或者对应设置方向的代码。
	要么舍弃透明效果，在它的Theme中添加：
	```
	<item name="android:windowIsTranslucent">false</item>
	```


- ##### 通知渠道

	从 Android 8.0（API 级别 26）开始，Google引入通知渠道，每个通知对应一个渠道。必须为所有通知分配渠道，否则通知将不会显示。渠道还可用于指定通知的重要程度等级。每个app都能创建当前app拥有哪些渠道，但这些渠道的控制权在用户手上。用户可以自由选择这些通知渠道的重要程度，是否响铃，是否振动啊之类的。
	
	以下是注册通知渠道的代码，记得先判断版本号是不是大于或者等于8.0,因为只有8.0以上才有通知渠道API。
	
	```
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
	    //创建通知渠道
	    CharSequence name = "渠道名称1";
	    String description = "渠道描述1";
	    String channelId="channelId1";//渠道id     
	    int importance = NotificationManager.IMPORTANCE_DEFAULT;//重要性级别
	    NotificationChannel mChannel = new NotificationChannel(channelId, name, importance);
	    mChannel.setDescription(description);//渠道描述
	    mChannel.enableLights(true);//是否显示通知指示灯
	    mChannel.enableVibration(true);//是否振动
	    
	    NotificationManager notificationManager = (NotificationManager) getSystemService(
	            NOTIFICATION_SERVICE);
	    notificationManager.createNotificationChannel(mChannel);//创建通知渠道
	}
	```
	然后我们在之前创建的渠道上发送一个通知。
	```
	//第二个参数与channelId对应
	Notification.Builder builder = new Notification.Builder(this,channelId);
	//icon title text必须包含，不然影响桌面图标小红点的展示
	builder.setSmallIcon(android.R.drawable.stat_notify_chat)
	        .setContentTitle("通知渠道1->标题")
	        .setContentText("通知渠道1->内容")
	        .setNumber(3); //久按桌面图标时允许的此条通知的数量
	
	Intent intent=new Intent(this,NotificationActivity.class);
	PendingIntent ClickPending = PendingIntent.getActivity(this, 0, intent, 0);
	builder.setContentIntent(ClickPending);
	
	notificationManager.notify(id,builder.build());
	```
	
	渠道重要性级别影响该渠道中所有通知的显示，在创建NotificationChannel对象的构造方法中必须要指定级别，一共有5个重要性界别，范围从 IMPORTANCE_NONE(0)至 IMPORTANCE_HIGH(4)。

 * IMPORTANCE_NONE 关闭通知
 * IMPORTANCE_MIN 开启通知，不会弹出，但没有提示音，状态栏中无显示
 * IMPORTANCE_LOW 开启通知，不会弹出，不发出提示音，状态栏中显示
 * IMPORTANCE_DEFAULT 开启通知，不会弹出，发出提示音，状态栏中显示
 * IMPORTANCE_HIGH 开启通知，会弹出，发出提示音，状态栏中显示


#### 4. Android 9.0适配 （API 28）
##### [Google官方文档](https://developer.android.google.cn/about/versions/pie/android-9.0-changes-all)
- `刘海屏适配`
- `默认情况下启用网络传输层安全协议 (TLS)`
- `隐私权变更`
- `限制非SDK接口 -Veridex 扫描工具`
- `前台服务- FOREGROUND_SERVICE 权限`
- `彻底取消了对Apache HTTPClient的支持`






