

#  开发文档

[Android API reference | Android Developers](https://developer.android.google.cn/reference?hl=zh-cn)


# App开发

# Intent

## 1、显式 Intent (明显显示出活动名称的)

```java
startActivity(new Intent(MainActivity.this,MainActivity2.class));
```

## 2、隐式 Intent

```xml
<activity>
	<intent-filter>
		<action android:name="...."/>
		<category android:name="...."/>
		<!--此句一般都要加 -->
		<category android:name="android.intent.category.DEFAULT"/>	
		<data android:scheme="..." android:host="..." android:path="/..." android:type="..."/>
	</intent-filter>
</activity>
```

# 数据存储

### 1. 写入本地文件

### 2. SharedPreferences（key-value 形式）

> 有三种方式拿到 SP 实例

```
/**
 * getSharedPreferences()
 * getPreferences()
 * PreferenceManager()
 */
```

> 拿到以后再通过 Editor 对象对文件进行读写

```
/*
		第一个参数为: 文件名称；
		第二个参数为: 读写权限
            进程共享的 ： MODE_MULTI_PROCESS = 4;
            进程独享的 ： MODE_PRIVATE = 0;
*/

SharedPreferences.Editor edit = getSharedPreferences("fileName", MODE_PRIVATE).edit();
```

> 使用 put 方法将数据存入

![image-20220311152957512](D:\backup\n024851\AppData\Roaming\Typora\typora-user-images\image-20220311152957512.png)



> 提交！

- commit和apply虽然都是原子性操作，但是原子的操作不同，commit是原子提交到数据库，所以从提交数据到存在Disk中都是同步过程，中间不可打断。
- 而apply方法的原子操作是原子提交的内存中，而非数据库，所以在提交到内存中时不可打断，之后再异步提交数据到数据库中，因此也不会有相应的返回值。
- 所有commit提交是同步过程，效率会比apply异步提交的速度慢，但是apply没有返回值，永远无法知道存储是否失败。
- 在不关心提交结果是否成功的情况下，优先考虑apply方法。

```java
edit.commit()/edit.apply();
```

### 3. SqlLite

1. 通过重写 SQLiteOpenHelper 抽象类的 onCreate() 和 onUpgrade() 方法 ，实现对数据库的创建和修改升级

   ```java
   /**
   		 *  @param context
   		 *  @param name:数据库名称
   		 *  @param factory 可以为 null ， 将默认使用 SQLiteCursor
   		 *  @param version：数据库版本
   		 */
   class sqlLiteHelper extends SQLiteOpenHelper {
   		// real 浮点
   		private final Context mcontext;
   		public static final String createShell =
   				"create table user("
   						+ "username integer,"
   						+ "password text,"
   						+ "age real" 
   						+ ")";
   
   		/**
   		 * @param context
   		 * @param name:数据库名称
   		 * @param factory
   		 * @param version：数据库版本
   		 */
   		public sqlLiteHelper(@Nullable Context context, @Nullable String name, @Nullable SQLiteDatabase.CursorFactory factory, int version) {
   			super(context, name, factory, version);
   			mcontext = context;
   		}
   
   		/**
   		 * 创建数据库：
   		 *
   		 * @param sqLiteDatabase
   		 */
   		@Override
   		public void onCreate(SQLiteDatabase sqLiteDatabase) {
   			sqLiteDatabase.execSQL(createShell);
   			Toast.makeText(mcontext, "create Success", Toast.LENGTH_SHORT).show();
   		}
   
   		/**
   		 * 升级数据库：
   		 * @param sqLiteDatabase
   		 * @param oldversion
   		 * @param newVersion
   		 */
   		@Override
   		public void onUpgrade(SQLiteDatabase sqLiteDatabase, int oldversion, int newVersion) {
   
   		}
   	}
   ```

2. 拿到一个数据库实例进行后续的逻辑

   ```SQLiteDatabase db
   SQLiteDatabase db = sqlLiteHelper.getWritableDatabase()
   
   SQLiteDatabase db =	sqlLiteHelper.getReadableDatabase()
   ```

3. 增删改查

   ```java
    /**
       * query()
       *      @param boolean distinct,
       *      @param String table,
       *      @param String[] columns,
       *      @param String selection,
       *      @param String[] selectionArgs,
       *      @param String groupBy,
       *      @param String having,
       *      @param String orderBy,
       *      @param String limit,
       *      @param CancellationSignal cancellationSignal
       */
   /*
      怎样判段是否为空
            1. query.next == null;
            2. query.at == -2;
   */
   
       queryData.setOnClickListener(v -> {
                db.query();
       });
       
   ```

4. Sql语句的封装（ ContentValues ）

5. 事务（确保某一系列的操作要么就全部完成，无法完成就全部不完成）



# 驱动机制

## 事件监听

## Handler(过时)

> 用于异步消息的处理： 有点类似辅助类，封装了消息投递、消息处理等接口。当发出一个消息之后，首先进入一个消息队列，发送消息的函数即刻返回，而另外一个部分在消息队列中逐一将消息取出，然后对消息进行处理，也就是发送消息和接收消息不是同步的处理。 这种机制通常用来处理相对耗时比较长的操作。

- MessageQueue
- Looper
- HandlerManager

```java
MyHandler handler = new MyHandler(this);

	@SuppressLint("HandlerLeak")
	class MyHandler extends Handler {
		WeakReference<handlerTest> mactivity;
		MyHandler(Context context) {
			context = getApplicationContext();
		}

		@Override
		public void handleMessage(@NonNull Message msg) {
			super.handleMessage(msg);

			handlerTest nactivity=mactivity.get();
			if(nactivity == null) {
				return ;
			}

			switch (msg.what) {
				case 0:
					//在这里通过nactivity引用外部类
					nactivity.m_var=0;
					break;
				default:
					break;
			}

		}
	}
```

## AsyncTask（过时）

> 效率不高而且资源占用较大，只适合少量的操作

 **`java.util.concurrent`包下的相关类**，如[Executor](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.com%2Freference%2Fjava%2Futil%2Fconcurrent%2FExecutor)，[ThreadPoolExecutor](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.com%2Freference%2Fjava%2Futil%2Fconcurrent%2FThreadPoolExecutor)，[FutureTask](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.com%2Freference%2Fjava%2Futil%2Fconcurrent%2FFutureTask)。





# ArrayAdapter

> ArrayAdapter绑定的数据是集合或数组，比较单一。视图是列表形式，ListView 或 Spinner.

- 自定义Adapter

  <img src="D:\backup\n024851\AppData\Roaming\Typora\typora-user-images\image-20220311150523352.png" alt="image-20220311150523352" style="zoom:25%;" />

```java
public class msgAdapter extends ArrayAdapter<Msg> {
	//绑定控件
    class ViewHolder {
    }

    public msgAdapter(@NonNull Context context, int textViewResourceId, @NonNull List<Msg> objects) {
        super(context, textViewResourceId, objects);
        recsourceId = textViewResourceId;
    }

    @NonNull
    @Override
    public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
        Msg msg = getItem(position);
        View view;
        ViewHolder viewHolder;
        if (convertView == null) {
            Log.d(TAG, "getView: convertView == null");
            view = LayoutInflater.from(getContext()).inflate(recsourceId, null);
            viewHolder = new ViewHolder();
            viewHolder.leftLinearLayout = (LinearLayout) view.findViewById(R.id.left_layout);
            viewHolder.rightLinearLayout = (LinearLayout) view.findViewById(R.id.right_layout);

            viewHolder.leftMsg = view.findViewById(R.id.left_msg);
            viewHolder.rightMsg = view.findViewById(R.id.right_msg);
            view.setTag(viewHolder);
        } else {
            view = convertView;
            viewHolder = (ViewHolder) view.getTag();
        }
        if (msg.getMsgType() == Msg.TYPE_RECREIVED) {
            Log.d(TAG, "getView: init TURE");
            viewHolder.leftLinearLayout.setVisibility(View.VISIBLE);
            viewHolder.rightLinearLayout.setVisibility(View.GONE);
            viewHolder.leftMsg.setText(msg.getMsgContent());
        } else {
            Log.d(TAG, "getView: init TURE");
            viewHolder.rightLinearLayout.setVisibility(View.VISIBLE);
            viewHolder.leftLinearLayout.setVisibility(View.GONE);
            viewHolder.rightMsg.setText(msg.getMsgContent());
        }
        return view;
    }
}
```

- 默认Adapter

  <img src="D:\backup\n024851\AppData\Roaming\Typora\typora-user-images\image-20220311150809438.png" alt="image-20220311150809438" style="zoom:25%;" />

```java
ArrayAdapter<String> adapter = new ArrayAdapter(context, android.R.layout.simple_list_item_1, strings);			 	adapter.notifyDataSetChanged();
spinner.setAdapter(adapter);
```

#  蓝牙

- 扫描其他蓝牙设备

- 查询本地蓝牙是诶器的配对蓝牙设备
- 建立RFCOMM通道
- 通过服务发现连接到其他设备
- 与其他设备进行双向数据传输
- 管理多个连接

## 创建一个蓝牙连接

### 1、申请蓝牙权限

```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
```

### 2、拿到一个 蓝牙适配器对象

```java
BluetoothAdapter adapter = BluetoothAdapter.getDefaultAdapter();
```

### 3、扫描其他蓝牙设备

- 所需GPS权限

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="ACCESS_COARSE_LOCATION" />
```

- 判断是否有使用权限

```java
LocationManager locationManager = (LocationManager) getSystemService(Context.LOCATION_SERVICE);
boolean isGpsEnabled = locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER);
```

- 开始扫描

```java
bluetoothAdapter.startDiscovery()
```

- 注册广播(要记得`unregisterReceiver()`)

```java
public Boolean registBroadCast() {
        // 找到设备的广播
        IntentFilter filter = new IntentFilter(BluetoothDevice.ACTION_FOUND);
        // 注册广播
        registerReceiver(broadcastReceiver, filter);
        PackageManager pm = getApplicationContext().getPackageManager();
        // 搜索完成的广播
        filter = new IntentFilter(BluetoothAdapter.ACTION_DISCOVERY_FINISHED);
        // 注册广播
        registerReceiver(broadcastReceiver, filter);

        return true;
    }
```

> 返回一个回调函数,接受设备返回的广播

```java
BroadcastReceiver broadcastReceiver = new BroadcastReceiver() {
        @SuppressLint("MissingPermission")
        @Override
        public void onReceive(Context context, Intent intent) {

            BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
            ArrayAdapter<BluetoothDevice> adapter = new ArrayAdapter(context, android.R.layout.simple_spinner_dropdown_item, strings);

            // 发现设备的广播
            if (device != null) {
                devicesMap.put(device.getName(), device);
                strings.remove(device.getName());
                strings.add(device.getName());
            }
            adapter.notifyDataSetChanged();
            spinner.setAdapter(adapter);
        }
    };
```

- 选择设备发起配对请求

```java
BlueToothDevice.createBond();
```

### 5、配对成功后，开始准备通信

<aside>
💡 Socket : 套接字/接口，连接两个BT设备，形成一个信道
<aside>
💡 UUID : 通过listenUsingRfcommWithServiceRecord()方法监听指定UUID和NAME设备的请求
**由于连接建立的过程会阻塞进程，属于耗时操作，因此连接的建立和管理都需要在单独的线程中进行。在实际的工程开发过程中，建立蓝牙连接的技巧是将每个蓝牙设备初始化为服务器端并监听连接，这样每个设备都可以自动在服务器端和客户端之间进行转化。**


> 服务端线程

```java
@SuppressLint("MissingPermission") 
					Runnable server = ()->{
                    try {
						Toast.makeText(this, "创建一个蓝牙服务器 参数分别：服务器名称、UUID", Toast.LENGTH_SHORT).show();
                        // 创建一个蓝牙服务器 参数分别：服务器名称、UUID

                        mServerSocket = defaultAdapter.listenUsingRfcommWithServiceRecord(btDevice.getName(),
                                uuid);
							Toast.makeText(this, "请稍候，正在等待客户端的连接..", Toast.LENGTH_SHORT).show();
                        /* 接受客户端的连接请求 */
                        mSocket = mServerSocket.accept();

						Toast.makeText(this, "客户端已经连接上！可以发送信息。", Toast.LENGTH_SHORT).show();
                        // 启动接受数据
                        new ReadThread().start();
                    } catch (IOException e) {
                        e.printStackTrace();
						Toast.makeText(this, "server  IOException", Toast.LENGTH_SHORT).show();
                    }
                };
```

> 客户端线程

```java
private class ReadThread extends Thread {
		public void run() {
            byte[] buffer = new byte[1024];
            int bytes;
            InputStream is = null;
            try {
                is = mSocket.getInputStream();
                while (true) {
                    if ((bytes = is.read(buffer)) > 0) {
                        byte[] buf_data = new byte[bytes];
                        for (int i = 0; i < bytes; i++) {
                            buf_data[i] = buffer[i];
                        }
                        String s = new String(buf_data);

                        Log.d(TAG, "run: accept--->"+s);
                        runOnUiThread(()->{
                            Toast.makeText(blueTooth.this, s, Toast.LENGTH_SHORT).show();
                        });
                    }
                }
            } catch (IOException e1) {
                e1.printStackTrace();
            } finally {
                try {
                    is.close();
                } catch (IOException e2) {
                    e2.printStackTrace();
                }
            }

        }
}
```

# 遇到的问题

### 1. Manifest.permission.BLUETOOTH_SCAN

### 2. Spinner在初始化时会自动调用一次OnItemSelectedListener事件

```java
	spinner.setSelection(0, true)
```

### 3. 判断蓝牙绑定状态

```java
	btDevice.getBondState()

    public static final int BOND_BONDED = 12;
    public static final int BOND_BONDING = 11;
    public static final int BOND_NONE = 10;
```

### 4. 通过反射解除与device的配对

```java
	try {
            device.getClass().getMethod("removeBond", (Class[]) null).invoke(device, (Object[]) null);
        } catch (Exception e) {
            e.printStackTrace();
        }
```

### 5.为了简化context的使用方法，现在有这么一种方法，就是在Application类里面维护一个弱引用：

> - 在垃圾回收器线程扫描它所管辖的内存区域的过程中, 一旦发现了**只具有弱引用**的对象，不管当前内存空间足够与否，都会回收它的内存
>
> - 避免activity中的任何对象的生命周期长过activity，避免由于对象对activity的引用导致activity不能正常被销毁。
>
> - application context伴随application的一生，与activity的生命周期无关。
>
> - Context.getApplicationContext()或者Activity.getApplicationContext()方法获取

```java
/** 用来保存当前该Application的context */ 
private static Context instance; 
/** 用来保存最新打开页面的context */ 
private volatile static WeakReference<Context> instanceRef = null;
```



# Rom 开发

[源码在线网站](http://androidxref.com/)

![image-20220328093614120](D:\backup\n024851\AppData\Roaming\Typora\typora-user-images\image-20220328093614120.png)

## Kernel

- Android系统由上至下大致可分为五层，分别是Applications层，Framework层，Native层，Hardware层，Kernel层。

- 系统镜像文件是存储在flash中的，而不是在CPU芯片或者DDR中。

  > 然后每个mtk芯片都有个boot rom（可以看成是一段指令），在上电时刻，boot rom开始启动，boot rom加载preloader到内部的SRAM中，为什么是加载到内部的SRAM中，而不是外部RAM中呢，是因为这个时候外部RAM还没有被初始化好，preloader被加载完成之后，程序就从boot rom跳转到preloader处开始执行，preloader初始化好外部RAM之后，preloader将lk（或uboot）加载外部RAM中，然后跳转到lk（或uboot）中去执行，lk（或uboot）紧接着就加载bootimage（包括kernel和ramdisk）到外部RAM中，然后去执行kernel部分。

- ```
  系统上电->Rom boot->Pre-loader->lk(u-boot)->kernel
  ```

### Kernel组成



## C++ --> JAVA

#### Zygote作用（可以看作是Linux的一个应用）

- 启动SystemServer

- 孵化进程

#### createZygout

1.解析init.rc文件；
2.执行init.rc文件中各个阶段的动作，而**创建zygote**的工作就是其中的某个阶段完成的；
3.调用property_init初始化属性相关的资源，并且通过property_start_service启动属性服务；
4.init进入一个无限循环，并且等待一些事情的发生。

1. 创建 Zygote

   ```java
   //init.rc
   import /init.${ro.zygote}.rc
   ```

      ```
   zygote32：纯32位模式
   zygote32_64：混32位模式(即32位为主，64位为辅)模式
   zygote64：纯64位模式
   zygote64_32：混64位模式(即 64位为主，32位为辅)模式
      ```

2. fork 出一个 Zygote进程

   ```java
   service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
   ```

3. App_main.cpp

   ```java
   /system/bin/app_process
   ```

4. ```
   /frameworks/base/core/jni/AndroidRuntime.cpp
   1.创建虚拟机---startVm；
   2.注册JNI函数---startReg；
   3.通过JNI调用Java函数，调用的函数是ZygoteInit的main函数。
   ```

   

#### initZygote

- registerZygoteSocket

- SystemServer

  - AMS（Activity Manage Service）

  - WMS（Windows Manage Service）

  - PMS（Package Manage Service）

  - .....

  - 启动 Binder 线程池 `C++`

    ```java
    ZygoteInit.nativeZygoteInit();
    ```

  - SystemServerManager(对系统服务进行创建，启动，生命周期的管理)

- Looper



#### Launcher

- PackageManagementService( 返回已安装app的信息 )

```java
SystemServer.startOtherServices(){
			AMS.systemReady() {
			ActivityStackSupervisor.resumeFocussedStackTopActivityLocked()						
	}
}---> Laucher
```

































ADB shell 命令

```java
// 连接设备
adb shell
    
// 查看adb状态
adb devices
    
// restarting adbd as root    
adb root
    
// 将日志信息输出到指定文件中（该文件不存在，则会新建）
adb logcat > logcat.log
```

Linux

```shell
gitk 
    
grep [-abcEFGhHilLnqrsvVwxy][-A<显示行数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
```



切换国家码

```java
*#*#6030#
// 查看内部版本号和国家码
####5993#  
```



## App 启动流程









## 1. preLoad App

- 把 需要预装的 apk 放到指定路径

- 改写编译脚本(mk)





![image-20220323102413863](D:\backup\n024851\AppData\Roaming\Typora\typora-user-images\image-20220323102413863.png)

## 2. SElinux(法无授权不可为~~,法不禁止即自由~~)

[Android 中的安全增强型 Linux]: https://source.android.google.cn/security/selinux?hl=zh-cn
[SELINUX 用户和管理员指南]:https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/selinux_users_and_administrators_guide/index



> SELinux 按照默认拒绝的原则运行：任何未经明确     允许的行为都会被拒绝。SELinux 可按两种全局模式运行：

- 宽容模式：权限拒绝事件会被记录下来，但不会被强制执行。
- 强制模式：权限拒绝事件会被记录下来**并**强制执行。

###  1. SELinux Context

进程和文件都有属于自己的Context信息，Context分为几个部分，分别是 SELinux User、Role、Type 和一个可选的Level信息。SELinux在运行过程中将使用这些信息查询安全策略进行决策。

- SELinux User：每一个Linux用户都会映射到SELinux用户，每一个SELinux User都会对应相应的Role。
- SELinux Role：每个Role也对应几种SELinux Type，并且充当了User和Type的‘中间人’
- SELinux Type：安全策略使用SELinux Type制定规则，定义何种Domian（Type）的Subject，可以接入何种Type的Object。

### 2.常用命令

- 查看当前的运行状态

```shell
~]# getenforce
```

- 使用`sestatus`可以查看完整的状态信息

```java
~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      30
```

### 3.SELinux Log

>SELinux 的Log日志默认记录在`/var/log/audit/audit.log`

```shell
~]# cat /var/log/audit/audit.log
```

> `/var/log/message` 也会记录相应的信息

























































