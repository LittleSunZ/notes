# notes
开发笔记


Android工具下载
http://www.androiddevtools.cn/  

gradle各个版本下载地址
链接: http://pan.baidu.com/s/1hqjIVlE 密码: 8ccb

android studio 打包xml中文提示is not translated in "zh" 解决办法
<resources
  xmlns:tools="http://schemas.android.com/tools"
  tools:ignore="MissingTranslation" >
</resources>


android studio 打包混淆提示specified twice.
proguard-project.txt  里面-libraryjars jar注释掉  eclipse不要注释


//如果只有一套图片，最好全都放在drawable-xxhdpi，可以节省图片内存

//Gson集合使用方法  
List<Person> peoples = new Gson().fromJson(jsonArray.toString(), new TypeToken<List<Person>>() {}.getType()); 
