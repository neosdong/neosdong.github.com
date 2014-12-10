---
layout: post
category : program
tags : Android XML解析

---


![Mou icon](http://s1.5km.co/201410/2416/47797_z.gif)

## 解析XML的三种方式

### DOM

通用性强，它会将XML文件的所有内容读取到内存中，然后允许您使用DOM API遍历XML树、检索所需的数据；

简单直观，但需要将文档读取到内存，并不太适合移动设备。

### SAX

SAX是一个解析速度快并且占用内存少的xml解析器；

采用事件驱动，它并不需要解析整个文档；

### PULL

Android自带的XML解析器，和SAX基本类似，也是事件驱动，不同的是PULL事件返回的是数值型；推荐使用。

基于事件响应，定位到某一节点。遍历子树。

## PULL解析XML框架代码

    XmlPullParser parser = XmlPullParserFactory.newInstance().newPullParser();
    parser.setInput(fileInputStream, “utf-8”);//设置数据源编码
    int eventCode = parser.getEventType();//获取事件类型(封闭方法)
    while(eventCode != XmlPullParser.END_DOCUMENT)  {   
        switch (eventCode){   
            case XmlPullParser.START_DOCUMENT: //开始读取XML文档  
		    //实例化集合类  
		    break;   
    		case XmlPullParser.START_TAG://开始读取某个标签		
				if("person".equals(parser.getName())) {   
				//通过getName判断读到哪个标签，然后通过nextText()获取文本节点值，或通过getAttributeValue(i)获取属性节点值
			}   
			break;
			case XmlPullParser.END_TAG://读完一个Person，可以将其添加到集合类中
	    	break;
		}
		parser.next();
	}


## PULL解析XML思路

读取一部分内容，根据内容，得到XML节点类型（此处定义为事件类型）ID：

1. START_DOCUMENT (1)
2. START_TAG (2)
3. END_TAG (3)
4. END_DOCUMENT (0)

对应响应

1. 实例化集合类

        persons = new ArrayList<Person>();

2. 读取便签属性、内容赋值给响应的类：
    1. 开始读标签
    2. 读内容（注意currentPerson标识变量的使用） 

        	String name = parser.getName();

        	if(name.equalsIgnoreCase("person")) {
        	//通过getName判断读到哪个标签，然后通过nextText()获取文本节点值，或通过getAttributeValue(i)获取属性节点值
            	currentPerson = new Person();
            	currentPerson.setId(new Integer(parser.getAttributeValue(null,"id")));
        	}	
        	else if (currentPerson!=null){
            	if (name.equalsIgnoreCase("name")){
                	currentPerson.setName(parser.nextText());
            	}
            	else if (name.equalsIgnoreCase("age")){
                	currentPerson.setAge(new
                	Short(parser.nextText()));
            	}
        	}

    3. 读完一个便签，添加到集合类中




## 参考
* 【Android】PULL解析XML文件 - CSDN.NET : http://blog.csdn.net/jueblog/article/details/13164349
* XML 树结构 : http://www.w3school.com.cn/xml/xml_tree.asp
