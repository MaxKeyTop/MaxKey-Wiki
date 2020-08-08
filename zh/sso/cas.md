---
layout: zh/default
---
<h2>CAS应用集成</h2>
本文介绍CAS应用如何与MaxKey进行集成。

<h2>应用注册</h2>

应用在MaxKey管理系统进行注册，注册的配置信息如下

<img src="{{ "/static/images/sso/sso_cas_conf.png" | prepend: site.baseurl }}?{{ site.time | date: "%Y%m%d%H%M" }}"  alt=""/>


<h2>CAS客户端配置</h2>

本文使用JAVA WEB程序为例

源代码地址

https://github.com/MaxKeyTop/MaxKey-Demo/blob/master/maxkey-demo-cas


<h3> 第一步，引入cas 客户端所需包</h3>

jar包依赖如下

<pre><code class="ini hljs">
cas-client-core-3.2.1.jar
commons-codec-1.9.jar
commons-io-2.2.jar
commons-logging-1.1.1.jar
</code></pre>


<h3> 第二步，web.xml安全及CAS配置</h3>

<pre><code class="xml hljs">
&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;web-app&gt;
	&lt;display-name&gt;&lt;/display-name&gt;
	&lt;listener&gt;
		&lt;listener-class&gt;org.jasig.cas.client.session.SingleSignOutHttpSessionListener&lt;/listener-class&gt;
	&lt;/listener&gt;
	&lt;filter&gt;
		&lt;filter-name&gt;CAS Single Sign Out Filter&lt;/filter-name&gt;
		&lt;filter-class&gt;org.jasig.cas.client.session.SingleSignOutFilter&lt;/filter-class&gt;
	&lt;/filter&gt;
	&lt;filter-mapping&gt;
		&lt;filter-name&gt;CAS Single Sign Out Filter&lt;/filter-name&gt;
		&lt;url-pattern&gt;/index.jsp&lt;/url-pattern&gt;
	&lt;/filter-mapping&gt;
	&lt;filter&gt;
		&lt;filter-name&gt;CAS Filter&lt;/filter-name&gt;
		&lt;filter-class&gt;org.jasig.cas.client.authentication.AuthenticationFilter&lt;/filter-class&gt;
		&lt;!-- cas server login url --&gt;
		&lt;init-param&gt;
			&lt;param-name&gt;casServerLoginUrl&lt;/param-name&gt;
			&lt;param-value&gt;&gt;https://sso.maxkey.top/maxkey/authz/cas/login&lt;/param-value&gt;
		&lt;/init-param&gt;
		&lt;!-- cas client url, in end of url / is required --&gt;
		&lt;init-param&gt;
			&lt;param-name&gt;serverName&lt;/param-name&gt;
			&lt;param-value&gt;http://cas.demo.maxkey.top:8080/&lt;/param-value&gt;
		&lt;/init-param&gt;
	&lt;/filter&gt;
	&lt;filter-mapping&gt;
		&lt;filter-name&gt;CAS Filter&lt;/filter-name&gt;
		&lt;url-pattern&gt;/index.jsp&lt;/url-pattern&gt;
	&lt;/filter-mapping&gt;

	&lt;!-- Cas10TicketValidationFilter Cas20ProxyReceivingTicketValidationFilter --&gt;
	&lt;filter&gt;
		&lt;filter-name&gt;CAS Validation Filter&lt;/filter-name&gt;
		&lt;filter-class&gt;org.jasig.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter&lt;/filter-class&gt;
		&lt;!-- cas server Validation url --&gt;
		&lt;init-param&gt;
			&lt;param-name&gt;casServerUrlPrefix&lt;/param-name&gt;
			&lt;param-value&gt;https://sso.maxkey.top/maxkey/authz/cas/&lt;/param-value&gt;
		&lt;/init-param&gt;
		&lt;!-- cas client url --&gt;
		&lt;init-param&gt;
			&lt;param-name&gt;serverName&lt;/param-name&gt;
			&lt;param-value&gt;http://cas.demo.maxkey.top:8080/&lt;/param-value&gt;
		&lt;/init-param&gt;
	&lt;/filter&gt;
	&lt;filter-mapping&gt;
		&lt;filter-name&gt;CAS Validation Filter&lt;/filter-name&gt;
		&lt;url-pattern&gt;/index.jsp&lt;/url-pattern&gt;
	&lt;/filter-mapping&gt;
	&lt;filter&gt;
		&lt;filter-name&gt;CAS HttpServletRequest Wrapper Filter&lt;/filter-name&gt;
		&lt;filter-class&gt;
			org.jasig.cas.client.util.HttpServletRequestWrapperFilter
		&lt;/filter-class&gt;
	&lt;/filter&gt;
	&lt;filter-mapping&gt;
		&lt;filter-name&gt;CAS HttpServletRequest Wrapper Filter&lt;/filter-name&gt;
		&lt;url-pattern&gt;/index.jsp&lt;/url-pattern&gt;
	&lt;/filter-mapping&gt;
	&lt;filter&gt;
		&lt;filter-name&gt;CAS Assertion Thread Local Filter&lt;/filter-name&gt;
		&lt;filter-class&gt;org.jasig.cas.client.util.AssertionThreadLocalFilter&lt;/filter-class&gt;
	&lt;/filter&gt;
	&lt;filter-mapping&gt;
		&lt;filter-name&gt;CAS Assertion Thread Local Filter&lt;/filter-name&gt;
		&lt;url-pattern&gt;/index.jsp&lt;/url-pattern&gt;
	&lt;/filter-mapping&gt;
	&lt;welcome-file-list&gt;
		&lt;welcome-file&gt;index.jsp&lt;/welcome-file&gt;
	&lt;/welcome-file-list&gt;
&lt;/web-app&gt;
</code></pre>

<h3> 第三步，JSP获取登录名及用户属性</h3>

<pre><code class="jsp hljs">
&lt;%@ page language="java" import="java.util.*" pageEncoding="utf-8"%&gt;
&lt;%@ page language="java" import="java.util.Map.Entry" %&gt;
&lt;%@ page language="java" import="org.apache.commons.codec.binary.Base64" %&gt;
&lt;%@ page language="java" import="org.jasig.cas.client.authentication.AttributePrincipal" %&gt;
&lt;%@ page language="java" import="org.jasig.cas.client.validation.Assertion" %&gt;
&lt;%@ page language="java" import="org.jasig.cas.client.util.AbstractCasFilter" %&gt;
&lt;%
	String path = request.getContextPath();
	String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
	System.out.println("CAS Assertion Success . ");
	Assertion assertion = (Assertion) request.getSession().getAttribute(AbstractCasFilter.CONST_CAS_ASSERTION);
	                          
	String username=     assertion.getPrincipal().getName();
%&gt;

&lt;!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;base href="&lt;%=basePath%&gt;"&gt;
    
    &lt;title&gt;Demo CAS&lt;/title&gt;
	&lt;meta http-equiv="pragma" content="no-cache"&gt;
	&lt;meta http-equiv="cache-control" content="no-cache"&gt;
	&lt;meta http-equiv="expires" content="0"&gt;    
	&lt;meta http-equiv="keywords" content="keyword1,keyword2,keyword3"&gt;
	&lt;meta http-equiv="description" content="CAS Demo"&gt;
	&lt;link rel="shortcut icon" type="image/x-icon" href="&lt;%=basePath %&gt;/static/images/favicon.ico"/&gt;

	&lt;style type="text/css"&gt;
		body{
			margin: 0;
			margin-top: 0px;
			margin-left: auto;
			margin-right: auto;
			padding: 0 0 0 0px;
			font-size: 12px;
			text-align:center;
			float:center;
			font-family: "Arial", "Helvetica", "Verdana", "sans-serif";
		}
		.container {
			width: 990px;
			margin-left: auto;
			margin-right: auto;
			padding: 0 10px
		}
		table.datatable {
			border: 1px solid #d8dcdf;
			border-collapse:collapse;
			border-spacing:0;
			width: 100%;
		}
		
		table.datatable th{
			border: 1px solid #d8dcdf;
			border-collapse:collapse;
			border-spacing:0;
			height: 40px;
		}
		
		
		table.datatable td{
			border: 1px solid #d8dcdf;
			border-collapse:collapse;
			border-spacing:0;
			height: 40px;
		}
		
		table.datatable td.title{
			text-align: center;
			font-size: 20px;
			font-weight: bold;
		}
	&lt;/style&gt;
  &lt;/head&gt;
  
  &lt;body&gt;
  		&lt;div class="container"&gt;
	  		&lt;table class="datatable"&gt;
	  			&lt;tr&gt;
	  				&lt;td colspan="2" class="title"&gt;CAS Demo for MaxKey&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;CAS Logo&lt;/td&gt;
	  				&lt;td&gt; &lt;img src="&lt;%=basePath %&gt;/static/images/cas.png"/&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td width="50%"&gt;CAS Assertion&lt;/td&gt;
	  				&lt;td&gt;&lt;%=username %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;CAS Has Attributes &lt;/td&gt;
	  				&lt;td&gt;&lt;%=!assertion.getPrincipal().getAttributes().isEmpty() %&gt; size : &lt;%=assertion.getPrincipal().getAttributes().size() %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;%
		  			Map&lt;String, Object&gt; attMap = assertion.getPrincipal().getAttributes();  
		            for (Entry&lt;String, Object&gt; entry : attMap.entrySet()) {   
		            	String attributeValue=entry.getValue()==null?"":entry.getValue().toString();
		            	System.out.println("attributeValue : "+attributeValue);
		            	if(attributeValue.startsWith("base64:")){
		            		attributeValue=new String(Base64.decodeBase64(attributeValue.substring("base64:".length())),"UTF-8");
		            	}
		        %&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;CAS &lt;%=entry.getKey() %&gt; &lt;/td&gt;
	  				&lt;td&gt;&lt;%=attributeValue %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;%}%&gt;
	  		&lt;/table&gt;
  		&lt;/div&gt;
  &lt;/body&gt;
&lt;/html&gt;
</code></pre>


<h2>基于SpringBoot CAS客户端配置</h2>

源代码地址

https://github.com/MaxKeyTop/MaxKey-SpringBoot4CAS-demo


demo分别写了三个请求:拦截请求 test1/index,test1/index1 以及不拦截请求test1/index2,

<h3> 第一步，引入cas 客户端所需包</h3>

<pre><code class="xml hljs">
&lt;dependency&gt;
	&lt;groupId&gt;net.unicon.cas&lt;/groupId&gt;
	&lt;artifactId&gt;cas-client-autoconfig-support&lt;/artifactId&gt;
	&lt;version&gt;2.3.0-GA&lt;/version&gt;
&lt;/dependency&gt;
</code></pre>	  

<h3>第二步，配置spring boot 配置文件</h3>

<pre><code class="ini hljs">
server:
  port: 8989
cas:
  # cas服务端地址
  server-url-prefix: http://sso.maxkey.top/maxkey/authz/cas/
  # cas服务端登陆地址
  server-login-url: http://sso.maxkey.top/maxkey/authz/cas/login
  # 客户端访问地址
  client-host-url: http://localhost:8989/
  # 认证方式，默认cas
  validation-type: cas
  #  客户端需要拦截的URL地址
  authentication-url-patterns:
    - /test1/index
    - /test1/index1
</code></pre>	

扩展配置项
<pre><code class="ini hljs">
cas.authentication-url-patterns
cas.validation-url-patterns
cas.request-wrapper-url-patterns
cas.assertion-thread-local-url-patterns
cas.gateway
cas.use-session
cas.redirect-after-validation
cas.allowed-proxy-chains
cas.proxy-callback-url
cas.proxy-receptor-url
cas.accept-any-proxy
server.context-parameters.renew
</code></pre>

<h3>第三步 在application启动类上加上 @EnableCasClient 注解</h3>

<pre><code class="java hljs">
@SpringBootApplication
@EnableCasClient
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
</code></pre>

<h3>第四步 在代码中获取登录用户信息</h3>

<pre><code class="java hljs">
    @GetMapping("test1/index1")
    public String index1(HttpServletRequest request){
        String token =request.getParameter("token");
        System.out.println("token : "+token);
        Assertion assertion = (Assertion) request.getSession().getAttribute(AbstractCasFilter.CONST_CAS_ASSERTION);

        String username=     assertion.getPrincipal().getName();
        System.out.println(username);

        return "test index cas拦截正常,登录账号:"+username;
    }
</code></pre>
