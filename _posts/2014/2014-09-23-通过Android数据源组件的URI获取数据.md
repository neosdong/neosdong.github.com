---
layout: post
category : program
tagline: "Supporting tagline2"
tags : Android

---

## 概念

数据源组件派生自抽象类android.content.ContentProvider，需要实现其中的query、update、insert和delete等抽象接口。

数据源组件的数据存储方式没有任何限制，可以通过

* 数据库
* 文件
* 甚至其他数据源

来获取。

整个数据源组件的接口设计集合了REST标准和数据库设计的概念，通过URI进行定位，像数据库一样，通过SQL语句描述具体的操作。

例如，Andorid通信录提供`Phone.CONTENT_URI`这个URI来提供数据。当然，还要仔细研究`Phone`类的使用。

## 代码

获取手机联系人的关键代码

	/**获取库Phone表字段**/
    private static final String[] PHONES_PROJECTION = new String[] {
	    Phone.DISPLAY_NAME, Phone.NUMBER,Photo.PHOTO_ID,Phone.CONTACT_ID };
	
	// 获取手机联系人
	Cursor phoneCursor = resolver.query(Phone.CONTENT_URI,PHONES_PROJECTION, null, null, null);


	if (phoneCursor != null) {
	    
	    while (phoneCursor.moveToNext()) {

			//得到手机号码
			String phoneNumber = phoneCursor.getString(PHONES_NUMBER_INDEX);
			//当手机号码为空的或者为空字段 跳过当前循环
			if (TextUtils.isEmpty(phoneNumber))
		    	continue;
		
			//得到联系人名称
			String contactName = phoneCursor.getString(PHONES_DISPLAY_NAME_INDEX);
		
			//...
		
		}
		
		//添加数据到数组，方便后续adapter等的使用。
		mContactsName.add(contactName);
		//...
		
	    }

	    phoneCursor.close();
	}

## Demo
- [读取通信录到列表的Demo](https://github.com/CocaMelon/Android_DEMO/tree/master/PRVD_Contacts)
