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
```
在application的onCreate里面这样判断即可
```
if ("你的包名".equals(getCurProcessName(this))) {
    SDKInitializer.initialize(this);
}
```

8.listview设置完数据之后scrollview就自动从顶部跳到中间
```
scrollview   最上面的一个控件  加这2个属性 
android:focusable="true"
android:focusableInTouchMode="true" 
```
9.Imageloader框架加载图片及时释放Bitmap
```
public class ReleaseBitmap implements ImageLoadingListener {

	private List<Bitmap> bitmaps;

	public ReleaseBitmap() {
		// TODO Auto-generated constructor stub
		bitmaps = new ArrayList<Bitmap>();
	}

	@Override
	public void onLoadingCancelled(String arg0, View arg1) {
		// TODO Auto-generated method stub

	}

	@Override
	public void onLoadingComplete(String imageUri, View view, Bitmap bitmap) {
		// TODO Auto-generated method stub
		bitmaps.add(bitmap);
	}

	@Override
	public void onLoadingFailed(String arg0, View arg1, FailReason arg2) {
		// TODO Auto-generated method stub

	}

	@Override
	public void onLoadingStarted(String arg0, View arg1) {
		// TODO Auto-generated method stub

	}

	public void clear() {
		if (bitmaps.size() > 0) {
			for (int i = 0; i < bitmaps.size(); i++) {
				Bitmap b = bitmaps.get(i);
				if (b != null && !b.isRecycled()) {
					b.recycle();
					b = null;
					bitmaps.remove(i);
				}
			}
		}
	}
}
```
只需要在加载的时候把releaseBitmap类传进去就OK了
```
ReleaseBitmap releaseBitmap = new ReleaseBitmap();
mImageLoader.displayImage(url, imageview, options, releaseBitmap);
```
在activity销毁的时候调用releaseBitmap.clear();<br/>
adapter也可以写哦，在adapter写一个clear函数
```
public void clear(){
		releaseBitmap.clear();
}
```
在activity销毁的时候调用adapter.clear();<br/>

10.ViewPager 切换报错 The specified child already has a parent. You must call removeView() on the child's <br/>
在adapter里如下面代码这么写即可解决
```
@Override
public Object instantiateItem(ViewGroup view, int position) {
  if(list.get(position).getParent()==null){
    view.addView(list.get(position));
  }else{
    ((ViewGroup)list.get(position).getParent()).removeView(list.get(position));
                view.addView(list.get(position));
  }
    return list.get(position);
}
```

11.Fresco图片加载框架,Facebook出品，必然强大。<br/>
Fresco 中设计有一个叫做 image pipeline的模块。它负责从网络，从本地文件系统，本地资源加载图片。为了最大限度节省空间和CPU时间，它含有3级缓存设计（2级内存，1级文件）。<br/>
Fresco 中设计有一个叫做 Drawees 模块，方便地显示loading图，当图片不再显示在屏幕上时，及时地释放内存和空间占用。<br/>
Fresco 默认配置是在5.0系统以下自动回收不显示图片内存。5.0以上的话需要配置<br/>
在application的onCreate初始化配置<br/>
```
ImagePipelineConfig config = ImagePipelineConfig.newBuilder(this)
        .setBitmapMemoryCacheParamsSupplier(new LollipopBitmapMemoryCacheParamsSupplier((ActivityManager) getSystemService(Context.ACTIVITY_SERVICE)))
        .build();
Fresco.initialize(this,config);

public class LollipopBitmapMemoryCacheParamsSupplier implements Supplier {

    private ActivityManager activityManager;

    public LollipopBitmapMemoryCacheParamsSupplier(ActivityManager activityManager) {
        this.activityManager = activityManager;
    }

    @Override
    public MemoryCacheParams get() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            return new MemoryCacheParams(getMaxCacheSize(), 56, Integer.MAX_VALUE,
                    Integer.MAX_VALUE,
                    Integer.MAX_VALUE);
        } else {
            return new MemoryCacheParams(
                    getMaxCacheSize(),
                    256,
                    Integer.MAX_VALUE,
                    Integer.MAX_VALUE,
                    Integer.MAX_VALUE);
        }
    }

    private int getMaxCacheSize() {
        final int maxMemory = Math.min(activityManager.getMemoryClass() * ByteConstants.MB, Integer.MAX_VALUE);

        if (maxMemory < 32 * ByteConstants.MB) {
            return 4 * ByteConstants.MB;
        } else if (maxMemory < 64 * ByteConstants.MB) {
            return 6 * ByteConstants.MB;
        } else {
            if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.GINGERBREAD) {
                return 8 * ByteConstants.MB;
            } else {
                return maxMemory / 4;
            }
        }
    }
}

```

12.github框架排名前100源码 http://www.devstore.cn/essay/essayInfo/5839.html

13.好用的相册选择器  https://github.com/wqandroid/wqgallery

14.轮播banner https://github.com/youth5201314/banner

14.图片压缩利器 https://tinypng.com/
