Nutz集成nutz-integration-nettice</的插件
======================

简介(可用性:生产,维护者:Rekoe)
==================================
![design](https://github.com/cyfonly/nettice/blob/master/pictures/nettice.png "nettice")  
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/cyfonly/nettice/blob/master/LICENSE)  [![Built with Maven](http://maven.apache.org/images/logos/maven-feather.png)](http://search.maven.org/#search%7Cga%7C1%7Ccyfonly)  
基于netty http协议栈的轻量级 MVC 组件
  
# 特性
1. 接收装配请求数据、流程控制和渲染数据
2. `URI` 到方法直接映射，以及命名空间

  
# 功能
1. 对 `HttpRequest` 的流程控制
2. 像普通方法一样处理 `http` 请求
3. 对请求的数据自动装配，支持`基本类型`、`List`、`Array` 和 `Map`
4. 提供 `Render` 方法渲染并写回响应，支持多种 `Content-Type`
5. 支持可配置的命名空间
  
  
# 请求分发整体设计
![design](https://github.com/cyfonly/nettp/blob/master/pictures/design.png "design.png")
  
  
# Action请求处理
![action_process](https://github.com/cyfonly/nettp/blob/master/pictures/action_process.png "action_process.png")  
  
# Usage  
#### 1. Maven   

```
<dependency>
  <groupId>org.nutz</groupId>
  <artifactId>nutz-integration-nettice</artifactId>
  <version>1.r.63-SNAPSHOT</</version>
</dependency>
```   
#### 2. In your project code
```
.addLast("dispatcher",new ActionDispatcher())
```  

例如 `com.server.action.DemoAction` 提供了 `returnTextUseNamespace()` 方法，`com.server.action.sub.SubDemoAction` 也提供了 `returnTextUseNamespace()` 方法，但两个方法实现不同功能。`nettice` 组件默认使用方法名进行 `URI` 映射，那么上述两个 `returnTextUseNamespace()` 方法会产生冲突，开发者可以使用 `@Namespace` 注解修改 `URI` 映射：  
```
package com.server.action;
public class DemoAction extends BaseAction{
  	@At("/nettp/demo/")
  	public Render returnTextUseNamespace(@Param("id") Integer id, @Param("proj") String proj){
    		//do something
    		return new Render(RenderType.TEXT, "returnTextUseNamespace in [DemoAction]");
  	}
}
``` 
  
```
package com.server.action.sub;
public class SubDemoAction extends BaseAction{
  	@At("/nettp/subdemo/")
	public Render returnTextUseNamespace(@Param("ids") Integer[] ids, @Param("names") List<String> names){
		//do something
		return new Render(RenderType.TEXT, "returnTextUseNamespace in [SubDemoAction]");
	}
}
```

```

# 接收装配请求数据
使用Read注解可以自动装配请求数组，支持不同的类型（`基本类型`、`List`、`Array`  和 `Map`），可以设置默认值（**目前仅支持基本类型设置 defaultValue**）。  
这个例子演示了从 `HttpRequest` 中获取基本类型的方法，如果没有值会自动设置默认值：
```
public Render primTypeTest(@Param(value="id", df="1" ) Integer id, @Param("proj") String proj, @Param("author") String author){
	System.out.println("Receive parameters: id=" + id + ",proj=" + proj + ",author=" + author);
	return new Render(RenderType.TEXT, "Received your primTypeTest request.[from primTypeTest]");
}
```  
这个例子演示了从 `HttpRequest` 中获取 `List/Array` 类型的方法：
```
public Render arrayListTypeTest(@Param("ids") Integer[] ids, @Param("names") List<String> names){
	System.out.println("server output ids:");
	for(int i=0; i<ids.length; i++){
		System.out.println(ids[i]);
	}
		
	System.out.println("server output names：");
	for(String item : names){
		System.out.println(item);
	}
		
	NutMap obj = new NutMap();
	obj.put("code", 0);
	obj.put("msg", "Received your Array/List request.[from arrayListTypeTest()]");
		
	return new Render(RenderType.JSON, Json.toJson(obj));
}
```
这个例子演示了从 `HttpRequest` 中获取 `Map` 类型的方法（**注意，使用 Map 时限定了只能存在一个 Map<String,String> 参数**）：
```
public Render mapTypeTest(@Read(key="srcmap") Map<String,String> srcmap){
	System.out.println("server output srcmap:");
	for(String key : srcmap.keySet()){
		System.out.println(key + "=" + srcmap.get(key));
	}
		
	NutMap obj = new NutMap();
	obj.put("code", 0);
	obj.put("msg", "Received your Map request.[from mapTypeTest]");
		
	return new Render(RenderType.JSON, Json.toJson(obj));
}
```  
  
# 渲染数据
处理方法可以通过返回 `Render` 对象向客户端返回特定格式的数据，一个 `Render` 对象由枚举类型 `RenderType` 和 `data` 两部分组成。  
`nettice` 组件会通过 `RenderType` 来为 `Response` 设置合适的 `Content-Type`，开发者也可以扩展 `Render` 以及相关类来实现更多的类型支持。  
例如这是一个返回 `JSON` 对象的例子，客户端将收到一个 `Json` 对象：
```
public Render postPriMap(){
	NutMap obj = new NutMap();
	obj.put("code", 0);
	obj.put("msg", "had received your request.");
	
	return new Render(RenderType.JSON, Json.toJson(obj));
}
```  
  
# TODO LIST 
1. java bean支持  
  
  
# License
基于 Apache License 2.0 发布。有关详细信息，请参阅 [LICENSE](https://github.com/cyfonly/nettice/blob/master/LICENSE)。
