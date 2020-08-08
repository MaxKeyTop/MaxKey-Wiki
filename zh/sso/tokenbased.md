---
layout: zh/default
---

<h2>TokenBased应用集成</h2>
本文介绍TokenBased应用如何与MaxKey进行集成。

<h2>应用注册</h2>

应用在MaxKey管理系统进行注册，注册的配置信息如下

<img src="{{ "/static/images/sso/sso_token_conf.png" | prepend: site.baseurl }}?{{ site.time | date: "%Y%m%d%H%M" }}"  alt=""/>

LTPA使用Cookie传输令牌

<img src="{{ "/static/images/sso/sso_token_ltpa_conf.png" | prepend: site.baseurl }}?{{ site.time | date: "%Y%m%d%H%M" }}"  alt=""/>


<h2>TokenBased客户端集成</h2>

本文使用JAVA WEB程序为例

<h3> 第一步，引入客户端所需包</h3>

<pre><code class="ini hljs">
gson-2.2.4.jar
maxkey-client-sdk.jar
nimbus-jose-jwt-8.10.jar
commons-codec-1.9.jar
commons-io-2.2.jar
commons-logging-1.1.1.jar
</code></pre>

<h3> 第二步，JSP实现Code</h3>

<h4> 简单令牌</h4>

<pre><code class="jsp hljs">
&lt;%@ page language="java" import="java.util.*" pageEncoding="utf-8"%&gt;
&lt;%@ page language="java" import="org.maxkey.client.tokenbase.*"%&gt;
&lt;%@ page language="java" import="org.maxkey.client.crypto.*"%&gt;
&lt;%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";

String token =request.getParameter("token");
System.out.println("token : "+token);
String tokenString=TokenUtils.decode(token, "x8zPbCya", ReciprocalUtils.Algorithm.DES);
String parseToken[]=TokenUtils.parseSimpleBasedToken(tokenString);
%&gt;
&lt;!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;base href="&lt;%=basePath%&gt;"&gt;
    
   &lt;title&gt;SimpleBasedToken Demo&lt;/title&gt;
	&lt;meta http-equiv="pragma" content="no-cache"&gt;
	&lt;meta http-equiv="cache-control" content="no-cache"&gt;
	&lt;meta http-equiv="expires" content="0"&gt;    
	&lt;meta http-equiv="keywords" content="keyword1,keyword2,keyword3"&gt;
	&lt;meta http-equiv="description" content="SimpleBasedToken Demo"&gt;
	&lt;link rel="shortcut icon" type="image/x-icon" href="&lt;%=basePath %&gt;/static/images/favicon.ico"/&gt;
	&lt;link   type="text/css" rel="stylesheet"  href="&lt;%=basePath %&gt;/demo.css"/&gt;
	
  &lt;/head&gt;
  
  &lt;body&gt;
  		&lt;div class="container"&gt;
	  		&lt;table class="datatable"&gt;
	  			&lt;tr&gt;
	  				
	  				&lt;td colspan="2" class="title"&gt;SimpleBasedToken Demo&lt;/td&gt;
	  			&lt;/tr&gt;
	  			
	  			&lt;tr&gt;
	  				&lt;td&gt;SimpleBasedToken Logo&lt;/td&gt;
	  				&lt;td&gt; &lt;img src="&lt;%=basePath %&gt;images/simple.png" width="124px" height="124px"/&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;UserName&lt;/td&gt;
	  				&lt;td&gt;&lt;%=parseToken[0]%&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Authentication at Time&lt;/td&gt;
	  				&lt;td&gt;&lt;%=parseToken[1]%&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  		&lt;/table&gt;
  		&lt;/div&gt;
  &lt;/body&gt;
&lt;/html&gt;
</code></pre>

<h4>基于JSON令牌</h4>

<pre><code class="jsp hljs">
&lt;%@ page language="java" import="java.util.*" pageEncoding="utf-8"%&gt;
&lt;%@ page language="java" import="org.maxkey.client.tokenbase.*"%&gt;
&lt;%@ page language="java" import="org.maxkey.client.crypto.*"%&gt;
&lt;%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";

String token =request.getParameter("token");
System.out.println("token : "+token);
String tokenString=TokenUtils.decode(token, "lEWhDLTo", ReciprocalUtils.Algorithm.DES);
Map tokenMap=TokenUtils.parseJsonBasedToken(tokenString);

%&gt;
&lt;!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;base href="&lt;%=basePath%&gt;"&gt;
    
   &lt;title&gt;JsonBasedToken Demo&lt;/title&gt;
	&lt;meta http-equiv="pragma" content="no-cache"&gt;
	&lt;meta http-equiv="cache-control" content="no-cache"&gt;
	&lt;meta http-equiv="expires" content="0"&gt;    
	&lt;meta http-equiv="keywords" content="keyword1,keyword2,keyword3"&gt;
	&lt;meta http-equiv="description" content="JsonBasedToken Demo"&gt;
	&lt;link rel="shortcut icon" type="image/x-icon" href="&lt;%=basePath %&gt;/static/images/favicon.ico"/&gt;
	&lt;link   type="text/css" rel="stylesheet"  href="&lt;%=basePath %&gt;/demo.css"/&gt;
	
  &lt;/head&gt;
  
  &lt;body&gt;
  		&lt;div class="container"&gt;
	  		&lt;table class="datatable"&gt;
	  			&lt;tr&gt;
	  				
	  				&lt;td colspan="2" class="title"&gt;JsonBasedToken Demo&lt;/td&gt;
	  			&lt;/tr&gt;
	  			
	  			&lt;tr&gt;
	  				&lt;td&gt;JsonBasedToken Logo&lt;/td&gt;
	  				&lt;td&gt; &lt;img src="&lt;%=basePath %&gt;images/json.png" width="124px" height="124px"/&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;UID&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("uid") %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;UserName&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("username") %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Department&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("department") %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Email&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("email") %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Authentication at Time&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("at")%&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Expires&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("expires")%&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  		&lt;/table&gt;
  		&lt;/div&gt;
  &lt;/body&gt;
&lt;/html&gt;

</code></pre>

<h4>基于LTPA JSON(COOKIE JSON)令牌</h4>
<pre><code class="jsp hljs">
&lt;%@ page language="java" import="java.util.*" pageEncoding="utf-8"%&gt;
&lt;%@ page language="java" import="org.maxkey.client.ltpa.*"%&gt;
&lt;%@ page language="java" import="org.maxkey.client.crypto.*"%&gt;
&lt;%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";

String ltpaVaule=LtpaUtils.readLtpa(request, "maxkey.top", "ltpa");
System.out.println("============ ltpaVaule "+ltpaVaule);
Map tokenMap=null;
if(ltpaVaule!=null){
	String ltpaString=LtpaUtils.decode(ltpaVaule, "k1tk41Ng", ReciprocalUtils.Algorithm.DES);
	tokenMap=LtpaUtils.parseLtpaJson(ltpaString);
}
%&gt;

&lt;!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;base href="&lt;%=basePath%&gt;"&gt;
    
   &lt;title&gt;LTPA Demo&lt;/title&gt;
	&lt;meta http-equiv="pragma" content="no-cache"&gt;
	&lt;meta http-equiv="cache-control" content="no-cache"&gt;
	&lt;meta http-equiv="expires" content="0"&gt;    
	&lt;meta http-equiv="keywords" content="keyword1,keyword2,keyword3"&gt;
	&lt;meta http-equiv="description" content="LTPA Demo"&gt;
	&lt;link rel="shortcut icon" type="image/x-icon" href="&lt;%=basePath %&gt;/static/images/favicon.ico"/&gt;
	&lt;link   type="text/css" rel="stylesheet"  href="&lt;%=basePath %&gt;/demo.css"/&gt;
	
  &lt;/head&gt;
  
  &lt;body&gt;
  		&lt;div class="container"&gt;
	  		&lt;table class="datatable"&gt;
	  			&lt;tr&gt;
	  				
	  				&lt;td colspan="2" class="title"&gt;LTPA Demo&lt;/td&gt;
	  			&lt;/tr&gt;
	  			
	  			&lt;tr&gt;
	  				&lt;td&gt;LTPA Logo&lt;/td&gt;
	  				&lt;td&gt; &lt;img src="&lt;%=basePath %&gt;images/ltpa.png" width="124px" height="124px"/&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;UID&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("uid") %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;UserName&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("username") %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Department&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("department") %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Email&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("email") %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Authentication at Time&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("at")%&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Expires&lt;/td&gt;
	  				&lt;td&gt;&lt;%=tokenMap.get("expires")%&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  		&lt;/table&gt;
  		&lt;/div&gt;
  &lt;/body&gt;
&lt;/html&gt;

</code></pre>