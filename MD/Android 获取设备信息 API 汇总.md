> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/J9ceAuvxrg1DcSFX3u7VOg)

判断系统模式
======

> *   系统模式分为 eng，user，userdebug 模式
>     

<table><thead><tr><th><section>user</section></th><th><section>userdebug</section></th><th><section>eng</section></th></tr></thead><tbody><tr><td><section>仅安装标签为 user 的模块</section></td><td><section>安装标签为 user、debug 的模块</section></td><td><section>安装标签为 user、debug、eng 的模块</section></td></tr><tr><td><section>设定属性 ro.secure=1，打开安全检查功能</section></td><td><section>设定属性 ro.secure=1，打开安全检查功能</section></td><td><section>设定属性 ro.secure=0，关闭安全检查功能</section></td></tr><tr><td><section>设定属性 ro.debuggable=0，关闭应用调试功能</section></td><td><section>设定属性 ro.debuggable=1，启用应用调试功能</section></td><td><section>设定属性 ro.debuggable=1，启用应用调试功能</section></td></tr><tr><td><section><br></section></td><td><section><br></section></td><td><section>设定属性 ro.kernel.android.checkjni=1，启用 JNI 调用检查</section></td></tr><tr><td><section>默认关闭 adb 功能</section></td><td><section>默认打开 adb 功能</section></td><td><section>默认打开 adb 功能</section></td></tr><tr><td><section>打开 Proguard 混淆器</section></td><td><section>打开 Proguard 混淆器</section></td><td><section>关闭 Proguard 混淆器</section></td></tr><tr><td><section>打开 DEXPREOPT 预先编译优化</section></td><td><section>打开 DEXPREOPT 预先编译优化</section></td><td><section>关闭 DEXPREOPT 预先编译优化</section></td></tr></tbody></table>

> *   判断`当前系统模式`，根据`不同模式`，`执行不同`的`操作`。
>     

```
getprop ro.build.type
userdebug


```

![](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtDjYtJ3PzLA3hWlhahiax4KnM1Xk2Jq28T74YZo87MKhyGaj9udY2hNceKYum6WNwNGibCIIu0nniaQ/640?wx_fmt=png&from=appmsg)

判断系统模式

```
// 获取系统模式  eng模式，user模式，userdebug模式
public static String getDeviceModelType(){
    return Build.TYPE;
}


```

获取设备信息
======

> 获取系统上对于调试与开发有用的所有设备信息。
> 
> *   设备型号
>     
> *   SN 编码
>     
> *   安全补丁
>     
> *   CPU
>     
> *   linux 内核
>     
> *   设备类型
>     
> *   内存
>     
> *   …..
>     

头文件
---

```
import java.io.BufferedReader;
import java.io.File;
import java.io.FileFilter;
import java.io.FileInputStream;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;


```

获取设备型号
------

```
/**
* 获取设备型号
* @return String 类型 获取设备型号
*/
public static String getDeviceModel() {
    return Build.MODEL;
}


```

获取 SN 编码
--------

```
/**
 * 获取主板设备的SN编号 : 通过反射 来获取 SN编码
 * @return
 */
public static String getDeviceSN_Num()
{
    String serial = null;
    try {
        Class<?> c =Class.forName("android.os.SystemProperties");
        Method get =c.getMethod("get", String.class);
        serial = (String)get.invoke(c, "ro.serialno");
    } catch (Exception e) {
        e.printStackTrace();
    }
    return serial;

}


```

获取设备制造商
-------

```
/**
* 获取设备制造商
* @return String 类型 获取设备制造商
*/
public static String getManufacturer() {
    return Build.MANUFACTURER;
}


```

获取 Android 版本号
--------------

```
/**
* 获取Android版本号
* @return String 类型 Android版本号
*/
public static String getAndroidVersion() {
    return Build.VERSION.RELEASE;
}


```

获取 SDK 版本信息
-----------

```
 /**
* 获取 Android SDK的版本 信息
* @return int 类型 Android SDK的版本 信息
*/
public static int getDeviceSDK(){
    return android.os.Build.VERSION.SDK_INT;
}


```

获取当前设备安全补丁级别日期
--------------

```
/**
* 获取当前设备的安全补丁级别 日期
* @return String 类型 安全补丁级别 日期
*/
public static String getSecurityPatchLevel() {
    return Build.VERSION.SECURITY_PATCH;
}


```

获取设备制造商
-------

```
/**
* 获取设备制造商
* @return String 类型 获取设备制造商
*/
public static String getDeviceMANUFACTURER() {
    return Build.SOC_MANUFACTURER;
}


```

获取构建的内部版本
---------

> 内部版本`Build ID`在`/build/make/core/build_id.mk` 目录下 (`RK平台`)

```
/**
* @return String 类型 内部版本 Build ID
*/
public static String getBuildID() {
    return Build.ID;
}


```

获取显示信息
------

```
/**
* 获取显示信息
* @return String 类型 显示信息
*/
public static String getDisplay() {
    return Build.DISPLAY;
}


```

获取设备硬件名
-------

```
/**
* 获取设备硬件名
* @return String 类型 硬件名称
*/
public static String getHardware() {
    return Build.HARDWARE;
}


```

获取设备 CPU 架构
-----------

```
/**
* 获取设备CPU架构
* @return String 类型 CPU架构
*/
public static String getCpuArchitecture() {
    return Build.CPU_ABI;
}


```

获取 EMMC 信息
----------

> EMMC ： 是一种集成控制器和存储芯片的存储设备，采用了 NAND Flash 技术，eMMC 适合于移动设备、嵌入式系统和一些消费电子产品等领域的应用。

```
/**
* 获取 EMMC 信息
* @return String 类型 EMMC 信息
*/
public static String getEMMC(Context context) {
    File dataPath = Environment.getDataDirectory();
    StatFs stat = new StatFs(dataPath.getPath());
    long blockSize = stat.getBlockSize();
    long availableBlocks = stat.getAvailableBlocks();
    return Formatter.formatFileSize(context, availableBlocks * blockSize);
}


```

获取 CPU 名称
---------

> 通过读取`Linux文件`的形式获取

```
public static String getCpuName() {
    String str1 = "/proc/cpuinfo";
    String str2 = "";
    String cpuName = "";
    try {
        FileReader fileReader = new FileReader(str1);
        BufferedReader bufferedReader = new BufferedReader(fileReader);
        while ((str2 = bufferedReader.readLine()) != null) {
            // 为空跳过
            if (TextUtils.isEmpty(str2)) {
                continue;
            }
            // 进行 分割 对比
            // 使用split(":\\s+", 2)方法将字符串str2按冒号和后续空格分割成最多两部分。例如，"Hardware: Intel Core i7"会被分成["Hardware", "Intel Core i7"]。
            String[] arrayOfString = str2.split(":\\s+", 2);
            if (TextUtils.equals(arrayOfString[0].trim(), "Hardware")) {
                cpuName = arrayOfString[1];
                break;
            }
        }
        bufferedReader.close();
        fileReader.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return cpuName;
}


```

获取 CPU 核数
---------

```
 /**
* 获取CPU 核数
* @return 返回处理input之后的结果
*/
public static String getCpuCores() {
    return Build.CPU_ABI;
}


```

获取 CPU 频率
---------

```
public staticfinalint DEVICEINFO_UNKNOWN = -1;
/**
* Method for reading the clock speed of a CPU core on the device. Will read from either
* {@code /sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_max_freq} or {@code /proc/cpuinfo}.
*
* @return Clock speed of a core on the device, or -1 in the event of an error.
* 获取 CPU 频率
*/
public static ArrayList<Integer> getCPUFreqMHzs() {
    int maxFreq = DEVICEINFO_UNKNOWN;
    int curFreq = 0;
    ArrayList<Integer> arrayList = new ArrayList<Integer>();
    try {
        int coreNum = getNumberOfCPUCores();
        for (int i = 0; i < coreNum; i++) {
            String filename =
                "/sys/devices/system/cpu/cpu" + i + "/cpufreq/cpuinfo_max_freq";
            File cpuInfoMaxFreqFile = new File(filename);
            if (cpuInfoMaxFreqFile.exists() && cpuInfoMaxFreqFile.canRead()) {
                byte[] buffer = newbyte[128];
                FileInputStream stream = new FileInputStream(cpuInfoMaxFreqFile);
                try {
                    stream.read(buffer);
                    int endIndex = 0;
                    // Trim the first number out of the byte buffer.
                    while (Character.isDigit(buffer[endIndex]) && endIndex < buffer.length) {
                        endIndex++;
                    }
                    String str = new String(buffer, 0, endIndex);
                    // 频率是按照1000计算
                    curFreq = Integer.parseInt(str) / 1000;
                    arrayList.add(curFreq);
                } catch (NumberFormatException e) {
                } catch (IOException e) {
                    thrownew RuntimeException(e);
                } finally {
                    stream.close();
                }
            }
        }
        if (maxFreq == DEVICEINFO_UNKNOWN && arrayList.size() == 0) {
            FileInputStream stream = new FileInputStream("/proc/cpuinfo");
            try {
                int freqBound = parseFileForValue("cpu MHz", stream);
                curFreq = freqBound;
                arrayList.add(curFreq);
            } finally {
                stream.close();
            }
        }
    } catch (IOException e) {
    }
    return arrayList;
}
/**
     * Reads the number of CPU cores from the first available information from
     * {@code /sys/devices/system/cpu/possible}, {@code /sys/devices/system/cpu/present},
     * then {@code /sys/devices/system/cpu/}.
     *
     * @return Number of CPU cores in the phone, or DEVICEINFO_UKNOWN = -1 in the event of an error.
     */
public static int getNumberOfCPUCores() {
    int coreNumber = -1;
    if (coreNumber != DEVICEINFO_UNKNOWN) {
        return coreNumber;
    }
    if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.GINGERBREAD_MR1) {
        // Gingerbread doesn't support giving a single application access to both cores, but a
        // handful of devices (Atrix 4G and Droid X2 for example) were released with a dual-core
        // chipset and Gingerbread; that can let an app in the background run without impacting
        // the foreground application. But for our purposes, it makes them single core.
        coreNumber = 1;
        return coreNumber;
    }
    int cores;
    try {
        cores = getCoresFromFileInfo("/sys/devices/system/cpu/present");
        if (cores == DEVICEINFO_UNKNOWN) {
            cores = new File("/sys/devices/system/cpu/").listFiles(CPU_FILTER).length;;
        }
    } catch (SecurityException e) {
        cores = DEVICEINFO_UNKNOWN;
    } catch (NullPointerException e) {
        cores = DEVICEINFO_UNKNOWN;
    }
    coreNumber = cores;
    return coreNumber;
}

/**
     * Tries to read file contents from the file location to determine the number of cores on device.
     * @param fileLocation The location of the file with CPU information
     * @return Number of CPU cores in the phone, or DEVICEINFO_UKNOWN = -1 in the event of an error.
     */
private static int getCoresFromFileInfo(String fileLocation) {
    InputStream is = null;
    try {
        is = new FileInputStream(fileLocation);
        BufferedReader buf = new BufferedReader(new InputStreamReader(is));
        String fileContents = buf.readLine();
        buf.close();
        return getCoresFromFileString(fileContents);
    } catch (IOException e) {
        return DEVICEINFO_UNKNOWN;
    } finally {
        if (is != null) {
            try {
                is.close();
            } catch (IOException e) {
                // Do nothing.
            }
        }
    }
}

/**
     * Converts from a CPU core information format to number of cores.
     * @param str The CPU core information string, in the format of "0-N"
     * @return The number of cores represented by this string
     */
private static int getCoresFromFileString(String str) {
    if (str == null || !str.matches("0-[\\d]+$")) {
        return DEVICEINFO_UNKNOWN;
    }
    return Integer.valueOf(str.substring(2)) + 1;
}

privatestaticfinal FileFilter CPU_FILTER = new FileFilter() {
    @Override
    public boolean accept(File pathname) {
        String path = pathname.getName();
        //regex is slow, so checking char by char.
        if (path.startsWith("cpu")) {
            for (int i = 3; i < path.length(); i++) {
                if (!Character.isDigit(path.charAt(i))) {
                    returnfalse;
                }
            }
            returntrue;
        }
        returnfalse;
    }
};

/**
     * Helper method for reading values from system files, using a minimised buffer.
     *
     * @param textToMatch - Text in the system files to read for.
     * @param stream      - FileInputStream of the system file being read from.
     * @return A numerical value following textToMatch in specified the system file.
     * -1 in the event of a failure.
     */
private static int parseFileForValue(String textToMatch, FileInputStream stream) {
    byte[] buffer = newbyte[1024];
    try {
        int length = stream.read(buffer);
        for (int i = 0; i < length; i++) {
            if (buffer[i] == '\n' || i == 0) {
                if (buffer[i] == '\n') i++;
                for (int j = i; j < length; j++) {
                    int textIndex = j - i;
                    //Text doesn't match query at some point.
                    if (buffer[j] != textToMatch.charAt(textIndex)) {
                        break;
                    }
                    //Text matches query here.
                    if (textIndex == textToMatch.length() - 1) {
                        return extractValue(buffer, j);
                    }
                }
            }
        }
    } catch (IOException e) {
        //Ignore any exceptions and fall through to return unknown value.
    } catch (NumberFormatException e) {
    }
    return DEVICEINFO_UNKNOWN;
}

/**
     * Helper method used by {@link #parseFileForValue(String, FileInputStream) parseFileForValue}. Parses
     * the next available number after the match in the file being read and returns it as an integer.
     * @param index - The index in the buffer array to begin looking.
     * @return The next number on that line in the buffer, returned as an int. Returns
     * DEVICEINFO_UNKNOWN = -1 in the event that no more numbers exist on the same line.
     */
private static int extractValue(byte[] buffer, int index) {
    while (index < buffer.length && buffer[index] != '\n') {
        if (Character.isDigit(buffer[index])) {
            int start = index;
            index++;
            while (index < buffer.length && Character.isDigit(buffer[index])) {
                index++;
            }
            String str = new String(buffer, 0, start, index - start);
            return Integer.parseInt(str);
        }
        index++;
    }
    return DEVICEINFO_UNKNOWN;
}


```

获取设备内存信息
--------

```
// 获取设备总内存
public static String getTotalMemory(Context context) {
    ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    ActivityManager.MemoryInfo mi = new ActivityManager.MemoryInfo();
    am.getMemoryInfo(mi);
    return Formatter.formatFileSize(context, mi.totalMem);// 将获取的内存大小规格化
}

// 获取设备剩余内存
public static String getAvailableMemory(Context context) {
    ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    ActivityManager.MemoryInfo mi = new ActivityManager.MemoryInfo();
    am.getMemoryInfo(mi);
    return Formatter.formatFileSize(context, mi.availMem);// 将获取的内存大小规格化
}

// 格式化文件大小，单位为MB
private static String formatFileSize(long sizeInBytes) {
    return Formatter.formatFileSize(null, sizeInBytes);
}



```

获取主板品牌
------

```
public static String getBRAND() {
    return Build.BRAND;
}


```

获取 Kernel 版本
------------

> 获取 `Kernel版本` 主要 通过读取 `/proc/version` 文件 来`进行读取`。

```
/**
* 获取kernel 版本 通过读取 /proc/version 的 方法 来 进行读取
*/
public static String getFormattedKernelVersion() {
    StringBuilder kernelVersion = new StringBuilder();
    BufferedReader reader = null;

    try {
        // 读取 /proc/version 文件
        reader = new BufferedReader(new FileReader("/proc/version"));
        String line = reader.readLine();
        if (line != null) {
            // 去除括号及其中的内容
            line = removeTextInParentheses(line);
            kernelVersion.append(line);
        }
    } catch (IOException e) {
        Log.e("KernelInfo", "Error reading /proc/version", e);
    } finally {
        if (reader != null) {
            try {
                reader.close();
            } catch (IOException e) {
                Log.e("KernelInfo", "Error closing reader", e);
            }
        }
    }

    return kernelVersion.toString();
}
/**
* 移除 input 括号及其内容
*
* @param input: 文本处理内容
* @return 返回处理input之后的结果
*/
private static String removeTextInParentheses(String input) {
    // 正则表达式匹配括号及其中内容
    Pattern pattern = Pattern.compile("\\(.*?\\)");
    Matcher matcher = pattern.matcher(input);
    return matcher.replaceAll("");  // 替换括号中的内容为空
}


```

获取存储信息
------

```
// 获取设备存储 信息
// 获取StorageManager实例
private String GetStorageInfo(Map<String,String> m_Device_Map) throws IOException {
    String Storege_info = "";
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        StorageManager storageManager = (StorageManager) getSystemService(Context.STORAGE_SERVICE);
        StorageStatsManager storageStatsManager = (StorageStatsManager)getSystemService(STORAGE_STATS_SERVICE);
        List<StorageVolume> volumeList = storageManager.getStorageVolumes();

        Log.i(Tag,"获取的存储数量" + volumeList.size());
        for (StorageVolume volume : volumeList) {
            if (null != volume ) {
                String label = volume.getDescription(this);   //这个其实就是U盘的名称
                String status = volume.getState();                   //设备挂载的状态，如:mounted、unmounted
                boolean isEmulated = volume.isEmulated();            //是否是内部存储设备
                boolean isRemovable = volume.isRemovable();          //是否是可移除的外部存储设备
                String mPath="";                                     //设备的路径
                Log.i(Tag,"name:"+label);
                Log.i(Tag,"status:"+status);
                Log.i(Tag,"isEmulated:"+isEmulated);
                Log.i(Tag,"isRemovable:"+isRemovable);

                File file;
                try {
                    long totalSpace = 0;
                    long availSpace = 0;
                    file = volume.getDirectory();
                    mPath = file.getPath();
                    Log.i(Tag,"mPath:"+mPath);
                    // 判断存储路径是否指向系统盘
                    String path = volume.getDirectory().getAbsolutePath();
                    if (mPath.contains("/storage/emulated/0") ) {
                        // 读取系统所在占用空间
                        try {
                            totalSpace = storageStatsManager.getTotalBytes(StorageManager.UUID_DEFAULT);//总空间大小
                            availSpace = storageStatsManager.getFreeBytes(StorageManager.UUID_DEFAULT);//可用空间大小
                            long systemBytes = totalSpace - availSpace;//系统所占不可用空间大小
                            Log.i(Tag,"totalBytes:"+ formatFileSize(totalSpace));
                            Log.i(Tag,"isEmulated:"+ formatFileSize(availSpace));
                            Log.i(Tag,"systemBytes:"+ formatFileSize(systemBytes));
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    else {
                        // 通过读取挂载文件夹的形式读取文件空间
                        totalSpace = file.getTotalSpace();
                        availSpace = file.getFreeSpace();
                        Log.i(Tag, "可用的block数目：:" + SDCardUtils.formatFileSize(totalSpace) + ",剩余空间:" + SDCardUtils.formatFileSize(availSpace));

                    }
                    // 设备名称 对应的空间容量
                    m_Device_Map.put(label,"Storage:" + formatFileSize(totalSpace) );
                }catch (Exception e) {
                    e.toString();
                }
            }
            else
            {
                Log.i(Tag,"Not volume");
            }
        }
    }
    //想看外部存储时，替换uuid即可
    return Storege_info;
}

/**
     * 格式化文件大小并转换为可视化单位（B, KB, MB, GB, TB）
     */
private   String formatFileSize(long sizeInBytes) {
    if (sizeInBytes <= 0) {
        return"0 B";
    }
    // 定义单位 列表
    final String[] units = new String[] {"B", "KB", "MB", "GB", "TB"};

    // 计算单位的指数
    int unitIndex = 0;
    double size = sizeInBytes;

    // 根据大小调整单位
    while (size >= 1000 && unitIndex < units.length - 1) {
        size /= 1000;
        unitIndex++;
    }
    // 保留两位小数
    return String.format("%.2f %s", size, units[unitIndex]);
}


```

统一输出日志
------

```
// 统一日志输出方法
public static void logDeviceInfo(Context context) {
    if (true) { // 仅在 Debug 模式打印日志，避免正式版泄露设备信息
        Log.i(TAG, "===== 设备信息 =====");
        Log.i(TAG, String.format("设备型号: %s", getDeviceModel()));
        Log.i(TAG, String.format("Android 版本: %s (SDK %d)", getAndroidVersion(), getDeviceSDK()));
        Log.i(TAG, String.format("安全补丁级别: %s", getSecurityPatchLevel()));
        Log.i(TAG, String.format("Build 号: %s", getBuildID()));
        Log.i(TAG, String.format("屏幕显示信息: %s", getDisplay()));
        Log.i(TAG, "===== 产品信息 =====");
        Log.i(TAG, String.format("硬件信息: %s", getHardware()));
        Log.i(TAG, String.format("制造商: %s", getDeviceMANUFACTURER()));
        Log.i(TAG, String.format("品牌: %s", getBRAND()));

        Log.i(TAG, "===== 内存信息 =====");
        Log.i(TAG, String.format("总内存: %s", getTotalMemory(context)));
        Log.i(TAG, String.format("可用内存: %s", getAvailableMemory(context)));
        Log.i(TAG, "======CPU信息=======");
        Log.i(TAG, String.format("CPU 名称: %s", getCpuName()));
        Log.i(TAG, String.format("CPU 架构: %s", getCpuArchitecture()));
        Log.i(TAG, String.format("CPU 核数: %s", getNumberOfCPUCores()));
        Log.i(TAG, String.format("CPU 频率: %s", getCPUFreqMHzs()));
    }
}


```

源码分析
====

设备设备基础信息
--------

> `build\make\core\sysprop.mk`

![](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtDjYtJ3PzLA3hWlhahiax4K2TKdu3TN166hpg3Wf3ewUq59OvIHcDwjF2jqicgZrBqfjv8P8iaq7Zmw/640?wx_fmt=png&from=appmsg)

源代码位置

![](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtDjYtJ3PzLA3hWlhahiax4KuIKav0WAPJE9PGn8CLwrTJRrzY10zJroQwPadmtAZvLef22FZb80vQ/640?wx_fmt=png&from=appmsg)

查看依赖关系

> *   搜索到数据很多，根据当前使用的`Rockchip 3568`的芯片，找到 `3568`对应的文件夹下的 `device/rockchip/rk356x/rk3568_t/rk3568_t.mk`
>     

![](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtDjYtJ3PzLA3hWlhahiax4Kyzs10ZVlEOhM9bkPn65bleKIBreDzLQGdNGkG5j7r4o4r0746tdTdA/640?wx_fmt=png&from=appmsg)

搜索到很多

> *   根据需求修改相应的参数
>     

![](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtDjYtJ3PzLA3hWlhahiax4KpUDGLibF6DrGhhWbPtaG3jv7PqPHuNut2UnZb3WicEic7vU6I94QNN1iaA/640?wx_fmt=png&from=appmsg)

根据需求进行 DIY 的修改

修改 Build_ID
-----------

![](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtDjYtJ3PzLA3hWlhahiax4KaAeUK6a1aY7WUaRa0OdLlAL2mScZ7p1GJBJxEJiaxurMaNNpzmcUXrw/640?wx_fmt=png&from=appmsg)

build_id 存储位置

参考资料
====

> android 利用 StorageStatsManager 获取应用程序的存储信息 [1]

参考资料

[1] 

android 利用 StorageStatsManager 获取应用程序的存储信息: _https://blog.csdn.net/CJohn1994/article/details/126534844_