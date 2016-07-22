# notes
开发笔记


1.Android工具下载
http://www.androiddevtools.cn/  

2.gradle各个版本下载地址
链接: http://pan.baidu.com/s/1hqjIVlE 密码: 8ccb

3.android studio 打包xml中文提示is not translated in "zh" 解决办法 
```
<resources
  xmlns:tools="http://schemas.android.com/tools"
  tools:ignore="MissingTranslation" >
</resources>
```


4.android studio 打包混淆提示specified twice.
proguard-project.txt  里面-libraryjars jar注释掉  eclipse不要注释


5.如果只有一套图片，最好全都放在drawable-xxhdpi，可以节省图片内存

6.Gson集合使用方法  
List<Person> peoples = new Gson().fromJson(jsonArray.toString(), new TypeToken<List<Person>>() {}.getType()); 


7.解决百度地图 service 配置 android:process=":remote" 导致application.oncreate创建多次
定位SDK会在单独进程中运行，因此会触发APPLICATION中的oncreat方法，只需要对该进程的名字进行判定就行了。。。。 
 
/**
* 判断进程名是否是自己的
*/
```     
private String getCurProcessName(Context context) {
    int pid = android.os.Process.myPid();
    ActivityManager mActivityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    for (ActivityManager.RunningAppProcessInfo appProcess : mActivityManager.getRunningAppProcesses()) {
        if (appProcess.pid == pid) {
          return appProcess.processName;
        }
    }
    return null;
}

if ("packageName".equals(getCurProcessName(this))) {
    SDKInitializer.initialize(this);
}
```

8.listview设置完数据之后scrollview就自动从顶部跳到中间
```
scrollview   最上面的一个控件  加这2个属性 
android:focusable="true"
android:focusableInTouchMode="true" 
```
