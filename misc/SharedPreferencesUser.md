# SharedPreferences使用

# 1.前言

SharedPreferences是一种轻型的数据存储方式，它的本质是基于XML文件存储key-value键值对数据，通常用来存储一些简单的配置信息。其存储位置在/data/data/<包名>/shared_prefs目录下。SharedPreferences对象本身只能获取数据而不支持存储和修改，存储修改是通过Editor对象实现。比较经典的使用方式例如用户输入框对过往登录账户的存储。实现SharedPreferences存储的步骤如下：
1、根据Context获取SharedPreferences对象
2、利用edit()方法获取Editor对象。
3、通过Editor对象存储key-value键值对数据。
4、通过commit()方法提交数据。

# 2.SharedPreferences和Editor 的关系

1、SharedPreferences

### public abstract SharedPreferences getSharedPreferences (String name, int mode)

方法得到一个sharedpreferences对象，参数name是preference文件的名字，mode则是方式，默认为0。
2、Editor 

Editor可用于SharedPreferences数据的添加，删除，修改和查询。

Public abstract SharedPreferences.Editor  putString (String key,String value)

通过执行commit（）或是apply（）方法，将会应用更改。

### 三、SharedPreferences的代码片段

##### 存储sharedpreferences	  

```
   public void setSharedPreference() {  
    sharedPreferences = getSharedPreferences("loginInfo", Context.MODE_PRIVATE);  
    Editor editor = sharedPreferences.edit();  
    editor.putString("username", text1.getText().toString());  
    editor.putInt("password", getpw());  
    editor.commit();// 提交修改  
    }  
```



##### 清除sharedpreferences的数据    

	    // 清除sharedpreferences的数据  
	    public void removeSharedPreference() {  
	    sharedPreferences = getSharedPreferences("loginInfo", Context.MODE_PRIVATE);  
	    Editor editor = sharedPreferences.edit();  
	    editor.remove("username");  
	    editor.remove("password");  
	    editor.commit();// 提交修改  
	    }  
#####  获得sharedpreferences的数据       

    // 获得sharedpreferences的数据  
    public void getSahrePreference() {
    SharedPreferences = getSharedPreferences("loginInfo", Context.MODE_PRIVATE);
    String username = sharedPreferences.getString("username", "");  
    int password = sharedPreferences.getInt("password", 0);  
    String str = String.valueOf(password);  
    text1.setText(username);  
    text2.setText(str);  
    }  