# 使用JSONP解决跨域问题
## 什么是跨域

		浏览器对ajax请求的限制，不允许跨域请求资源。
		
		http://www.a.com   http://www.b.com       是跨域
		http://www.a.com   http://www.a.com:8080  是跨域
		http://a.a.com   http://b.a.com  是跨域
		http://www.a.com   http://www.a.com/api   不是

### 总结：
**不同的域名或不同的端口都是跨域请求。**

### 如何解决跨域问题？-----jsonp
## 定义一个jsp文件输出json数据
		<%@ page language="java" contentType="text/html; charset=UTF-8"
			pageEncoding="UTF-8"%>
		<%
		    String callback = request.getParameter("callback");
			if(null != callback || !"".equals(callback)) {
			    out.print(callback + "({\"abc\":\"123\"})");
			} else {
			    out.print("{\"abc\":\"123\"}");
			}
		   
		%>
## 在另一个工程下添加一个html文件并通过ajax方式解析该数据
		<!DOCTYPE html>
		<html>
		<head>
		<meta charset="UTF-8">
		<title>Insert title here</title>
		</head>
		<body>
			<script type="text/javascript" src="http://localhost:8181/js/jquery-1.12.3.min.js"></script>
			<script type="text/javascript">
				alert($);
				$(function(){
					$.ajax({
						url : "http://localhost:8181/index.jsp",
						type : "get",
						dataType : "jsonp",
						//jsonp : "callback",
						success : function(data) {
							alert(data.abc);
						} 
					});
				});
			</script>
		</body>
		</html>

## 运行之后看到屏幕弹框显示如下内容
123

### 解释

		    	Jsonp的原理：
		    	1、jsonp通过script标签的src可以跨域请求的特性，加载资源
		    	2、将加载的资源（通过一个方法名将数据进行包裹）当做是js脚本解析
		   		3、定义一个回调函数，获取传入的数据
		   		翻看了一下Jquery文档发现jsonp:”callback”, 
		   		jsonpCallback:”success_jsonpCallback”,
		   		传递这两个参数是有原因的，jsonp的返回数据格式应该是: 
		   		    “客户端传递的回调方法名称(json数据)”
				可以看到，php文件返回的结果是
 				
				success_jsonpCallback({“abc”:”123”}) ，
				这才是正确的jsonp返回格式，而success_jsonpCallback这是传递过去的参数。