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

11.Fresco图片框架,Facebook出品，必然强大。<br/>
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
//混淆
-dontwarn com.facebook.**
```

12.github框架排名前100源码 http://www.devstore.cn/essay/essayInfo/5839.html

13.好用的相册选择器  https://github.com/wqandroid/wqgallery

14.轮播banner https://github.com/youth5201314/banner

14.图片压缩利器 https://tinypng.com/

15.微信支付的坑,签名的顺序一定要按照它的来,搞乱一个就不行了,注意金钱单位为分。
```
public void wxpay(PayInfo payInfos){
	final String appid = WxConfig.WX_APPID;
	final String mch_id = WxConfig.PARTNER;
	final String nonce_str = genNonceStr();
	String body = payInfos.getBody();
	String out_trade_no = payInfos.getOut_trade_no();
	String total_fee = payInfos.getTotal_fee()+"";
	String spbill_create_ip = CommonUtil.getLocalHostIp();
	String notify_url = payInfos.getNotify_url();
	String trade_type = "APP";
	
	
	List<NameValuePair> packageParams = new LinkedList<NameValuePair>();
	packageParams.add(new BasicNameValuePair("appid", appid));
	packageParams.add(new BasicNameValuePair("body", body));
	packageParams.add(new BasicNameValuePair("mch_id", mch_id));
	packageParams.add(new BasicNameValuePair("nonce_str", nonce_str));
	packageParams.add(new BasicNameValuePair("notify_url", notify_url));
	packageParams.add(new BasicNameValuePair("out_trade_no", out_trade_no));
	packageParams.add(new BasicNameValuePair("spbill_create_ip", spbill_create_ip));
	packageParams.add(new BasicNameValuePair("total_fee", total_fee));
	packageParams.add(new BasicNameValuePair("trade_type", trade_type));
	// 生成package 
	StringBuilder sb = new StringBuilder(); 
	 
	for (int i = 0; i < packageParams.size(); i++) { 
	  sb.append(packageParams.get(i).getName()); 
	  sb.append('='); 
	  sb.append(packageParams.get(i).getValue()); 
	  sb.append('&');   
	} 
	sb.append("key="); 
	sb.append(WxConfig.PARTNER_KEY);
	// 进行md5摘要前，params内容为原始内容，未经过url encode处理 
	String packageSign = MD5.getMessageDigest(sb.toString().getBytes()).toUpperCase(); 
	packageParams.add(new BasicNameValuePair("sign", packageSign));
	HttpForRequest.wxPay(buildXMLUnifiedOrder(packageParams), new RequestCallBack<String>() {
				
			@Override
			public void onSuccess(ResponseInfo<String> arg0) {
				// TODO Auto-generated method stub
				dismissProgressDialog();
				String return_code = null;
				String return_msg = null;
				String prepay_id = null;
				String sign = null;
				try {
					XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
		            XmlPullParser pull = factory.newPullParser();
		            pull.setInput(new StringReader(arg0.result));
		            int type=pull.getEventType();
		    		while(type!=XmlPullParser.END_DOCUMENT){
		    			String nodeName =pull.getName();
		    			if("return_code".equals(nodeName)){
	    					return_code = pull.nextText();
	    				}
		    			if("return_msg".equals(nodeName)){
		    				return_msg = pull.nextText();
		    			}
		    			if("prepay_id".equals(nodeName)){
		    				prepay_id = pull.nextText();
		    			}
		    			type=pull.next();
		    		}
				} catch (Exception e) {
					// TODO: handle exception
					e.printStackTrace();
				}
				String timeStamp = String.valueOf(genTimeStamp());
				
				if(return_code.equals("SUCCESS")&&return_msg.equals("OK")){
					PayReq req = new PayReq();
					req.appId			= appid;
					req.partnerId		= mch_id;
					req.prepayId		= prepay_id;
					req.nonceStr		= nonce_str;
					req.timeStamp		= timeStamp;
					req.packageValue	= "Sign=WXPay";
					
					List<NameValuePair> packageParams = new LinkedList<NameValuePair>();
					packageParams.add(new BasicNameValuePair("appid", req.appId));
					packageParams.add(new BasicNameValuePair("noncestr", req.nonceStr));
					packageParams.add(new BasicNameValuePair("package", "Sign=WXPay"));
					packageParams.add(new BasicNameValuePair("partnerid", req.partnerId));
					packageParams.add(new BasicNameValuePair("prepayid", req.prepayId));
					packageParams.add(new BasicNameValuePair("timestamp", req.timeStamp));
					// 生成package 
					StringBuilder sb = new StringBuilder(); 
					 
					for (int i = 0; i < packageParams.size(); i++) { 
					  sb.append(packageParams.get(i).getName()); 
					  sb.append('='); 
					  sb.append(packageParams.get(i).getValue()); 
					  sb.append('&');   
					} 
					sb.append("key="); 
					sb.append(WxConfig.PARTNER_KEY);
					sign = MD5.getMessageDigest(sb.toString().getBytes()).toUpperCase(); ;
					req.sign	= sign;
					// 在支付之前，如果应用没有注册到微信，应该先调用IWXMsg.registerApp将应用注册到微信
					api.sendReq(req);
				}else{
					showToast("微信支付失败！");
				}
				
			}
			
			@Override
			public void onFailure(HttpException arg0, String arg1) {
				// TODO Auto-generated method stub
				dismissProgressDialog();
				showToast("微信支付失败！");
			}
	});
}
```
16.ScrollView中包含Listview的时候,要给Listview重新计算下高度,注意要在setAdapter()之后
```
/***
 * 给list重新设置高度在scrollview显示
 * 
 * @param listView
 */
public static void setListViewHeightBasedOnChildren(ListView listView) {
	ListAdapter listAdapter = listView.getAdapter();
	if (listAdapter == null) {
		return;
	}
	int totalHeight = 0;
	for (int i = 0; i < listAdapter.getCount(); i++) {
		View listItem = listAdapter.getView(i, null, listView);
		listItem.measure(0, 0);
		totalHeight += listItem.getMeasuredHeight();
	}
	ViewGroup.LayoutParams params = listView.getLayoutParams();
	params.height = totalHeight
			+ (listView.getDividerHeight() * (listAdapter.getCount() - 1));
	listView.setLayoutParams(params);
}
```

17.当项目中有webView有JS要跟activity交互时,
```
webView.addJavascriptInterface(new JsInterface(), "aaa");
```
混淆的时候记得加上,包名.MainActivity是这个类的路径,JsInterface是你的类名
```
-keepattributes *Annotation*
-keepattributes *JavascriptInterface*
-keepclassmembers  class 包名.MainActivity$JsInterface  {
   public *;
}
```

18.Fresco跟百度的SO文件冲突了,启动的时候会报这行的错
```
SDKInitializer.initialize(this);
```
解决办法是把所有类型CPU的SO文件都拷贝到libs下,但是有些库没有这么多SO文件的时候，最好的办法是在build.gradle加上
```
ndk {
    abiFilters "armeabi", "armeabi-v7a", "x86", "mips"
}
```

```
android {
    compileSdkVersion 21
    buildToolsVersion "22.0.1"

    defaultConfig {
        applicationId "com.test.app"
        minSdkVersion 10
        targetSdkVersion 21
        multiDexEnabled true

        ndk {
            abiFilters "armeabi", "armeabi-v7a", "x86", "mips"
        }
    }
}
```
意为系统么有固定只匹配这几个类型的so文件，没有匹配到就找默认的：armeabi或者armeabi-v7a

19.Fresco Gif播放 0.9.0以上需要依赖
```
compile 'com.facebook.fresco:animated-gif:0.10.0'
```

