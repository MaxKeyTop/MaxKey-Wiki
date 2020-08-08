---
layout: zh/default
---
<h2>OpenID Connect应用集成</h2>
本文介绍OpenID Connect应用如何与MaxKey进行集成。

<h2>认证流程</h2>

请参照OAuth2认证流程

<h2>应用注册</h2>
应用在MaxKey管理系统进行注册，注册的配置信息如下

<img src="{{ "/static/images/sso/sso_oidc_conf.png" | prepend: site.baseurl }}?{{ site.time | date: "%Y%m%d%H%M" }}"  alt=""/>

<h2>集成和接口</h2>
<h4>3)用户属性接口/api/connect/v10/userinfo</h4>

通过访问token 获取登录用户信息及签名信息，在程序中必须验证相关的签名信息。
 
<table  border="0" class="table table-striped table-bordered ">
 	   <tr>
	    <th> 接口名称 </th>
	    <th> OIDC授权用户信息查询接口 </th>
  	  </tr>
	  <tr>
	    <td> url </td>
	    <td>https://sso.maxkey.org/maxkey/api/connect/v10/userinfo</td>
	  </tr>
	  <tr>
	    <td> 请求方式 </td>
	    <td> http get/post </td>
	  </tr>
</table>
 	 
<h5>请求参数</h5>

<table  border="0" class="table table-striped table-bordered ">
 	   <tr>
	    <th>参数 </th>
	    <th> 说明 </th>
  	  </tr>
	  <tr>
	    <td> access_token </td>
	    <td> 调用sso/ token获得的token值。 </td>
	  </tr>
	  <tr align="left">
	  	<td colspan="2" align="left">
	  				实际请求如下：
<pre><code class="http hljs"> 
POST /oauth/ userinfo HTTP/1.1
Host: sso.maxkey.org/openapi
Content-Type: application/x-www-form-urlencoded
access_token= PQ7q7W91a-oMsCeLvIaQm6bTrgtp7
</code></pre>
	  	</td>
	  </tr>
	  <tr>
	  		<td colspan="2" align="left">
	  		返回数据/ response data
	  		</td>
	  </tr>
	  <tr>
	  		<td colspan="2">
	  		<p>成功返回JSON数据，如下：</p>
<pre><code class="json hljs"> 
{
userid     :  “zhangs”,
				…
}</code></pre>
<br/>
zhangs是认证的用户ID
	  		</td>
	  </tr>
 	 </table> 	



OAuth认证接口属性列表

<table   border="0" class="table table-striped table-bordered ">
   <tr >
	<th> 属性名(Attribute) </th>
	<th> 描述 </th>
	<th>数据类型</th>
  </tr>
  <tr>
	<td>uid</td>
	<td>uid</td>
	<td>字符串</td>
  </tr>
 </table> 	

其他请参照OAuth2


<h2>OIDC V1客户端集成</h2>

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

<h3> 第二步，认证跳转</h3>

<pre><code class="jsp hljs">
&lt;%@ page language="java" import="java.util.*" pageEncoding="ISO-8859-1"%&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.oauth.*" %&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.builder.*" %&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.builder.api.MaxkeyApi20" %&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.model.Token" %&gt;

&lt;%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+path+"/";

String callback="http://oauth.demo.maxkey.top:8080/demo-oauth/oauth20callback.jsp";
OAuthService service = new ServiceBuilder()
                            .provider(MaxkeyApi20.class)
                            .apiKey("ae20330a-ef0b-4dad-9f10-d5e3485ca2ad")
                            .apiSecret("KQY4MDUwNjIwMjAxNTE3NTM1OTEYty")
                            .callback(callback)
                            .build();
Token EMPTY_TOKEN = null;
String authorizationUrl = service.getAuthorizationUrl(EMPTY_TOKEN);

request.getSession().setAttribute("oauthv20service", service);

%&gt;

&lt;!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;base href="&lt;%=basePath%&gt;"&gt;
    
    &lt;title&gt;OIDC V1 SSO&lt;/title&gt;
	&lt;meta http-equiv="pragma" content="no-cache"&gt;
	&lt;meta http-equiv="cache-control" content="no-cache"&gt;
	&lt;meta http-equiv="expires" content="0"&gt;    
	&lt;meta http-equiv="keywords" content="keyword1,keyword2,keyword3"&gt;
	&lt;meta http-equiv="description" content="This is my page"&gt;
  &lt;/head&gt;
  
  &lt;body&gt;
    &lt;a href="&lt;%=authorizationUrl%&gt;&approval_prompt=auto"&gt;OIDC V1 SSO&lt;/a&gt;
  &lt;/body&gt;
&lt;/html&gt;

</code></pre>


<h3> 第三步，获取令牌和用户信息及验证签名 (id_token及用户信息)</h3>

<pre><code class="jsp hljs">
&lt;%@ page language="java" import="java.util.*" pageEncoding="utf-8"%&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.oauth.*" %&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.builder.*" %&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.builder.api.MaxkeyApi20" %&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.model.*" %&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.*" %&gt;
&lt;%@ page language="java" import="org.maxkey.client.oauth.domain.*" %&gt;
&lt;%@ page language="java" import="org.maxkey.client.utils.*" %&gt;
&lt;%@ page language="java" import="com.nimbusds.jwt.JWTClaimsSet" %&gt;
&lt;%@ page language="java" import="com.nimbusds.jose.*" %&gt;
&lt;%@ page language="java" import="com.nimbusds.jwt.*" %&gt;
&lt;%@ page language="java" import="com.connsec.oidc.jose.keystore.*" %&gt;
&lt;%@ page language="java" import="com.nimbusds.jose.jwk.*" %&gt;
&lt;%@ page language="java" import="java.io.File" %&gt;
&lt;%@ page language="java" import="com.nimbusds.jose.crypto.*" %&gt;
&lt;%@ page language="java" import="com.google.gson.*" %&gt;

&lt;%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";

OAuthService service = (OAuthService)request.getSession().getAttribute("oauthv20service");

if(service==null){
	String callback="http://oauth.demo.maxkey.top:8080/demo-oauth/oidc10callback.jsp";
	service = new ServiceBuilder()
     .provider(MaxkeyApi20.class)
     .apiKey("ae20330a-ef0b-4dad-9f10-d5e3485ca2ad")
     .apiSecret("KQY4MDUwNjIwMjAxNTE3NTM1OTEYty")
     .callback(callback)
     .build();
}

Token EMPTY_TOKEN = null;
Verifier verifier = new Verifier(request.getParameter("code"));
Token accessToken = service.getAccessToken(EMPTY_TOKEN, verifier);

//JWTClaimsSet idClaims = JWTClaimsSet.parse(accessToken.getId_token());
SignedJWT signedJWT=null;

//JWKSetKeyStore jwkSetKeyStore=new JWKSetKeyStore();

File jwksFile=new File(PathUtils.getInstance().getClassPath()+"jwk.jwks");
JWKSet jwkSet=JWKSet.load(jwksFile);

RSASSAVerifier rsaSSAVerifier = new RSASSAVerifier(((RSAKey) jwkSet.getKeyByKeyId("maxkey_rsa")).toRSAPublicKey());
try {
    signedJWT = SignedJWT.parse(accessToken.getId_token());
} catch (java.text.ParseException e) {
    // Invalid signed JWT encoding
}
;

OAuthClient restClient=new OAuthClient("https://sso.maxkey.top/maxkey/api/connect/v10/userinfo",accessToken.getToken());
 
OIDCUserInfo userInfo=restClient.getOIDCUserInfo(accessToken.getToken());
 
%&gt;

&lt;!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;base href="&lt;%=basePath%&gt;"&gt;

   &lt;title&gt;OpenID Connect 1.0 Demo&lt;/title&gt;
	&lt;meta http-equiv="pragma" content="no-cache"&gt;
	&lt;meta http-equiv="cache-control" content="no-cache"&gt;
	&lt;meta http-equiv="expires" content="0"&gt;    
	&lt;meta http-equiv="keywords" content="keyword1,keyword2,keyword3"&gt;
	&lt;meta http-equiv="description" content="OpenID Connect 1.0 Demo"&gt;
	&lt;link rel="shortcut icon" type="image/x-icon" href="&lt;%=basePath %&gt;/static/images/favicon.ico"/&gt;
	&lt;script type="text/javascript" src="&lt;%=basePath %&gt;/jquery-3.5.0.min.js"&gt;&lt;/script&gt;
	&lt;script type="text/javascript" src="&lt;%=basePath %&gt;/jsonformatter.js"&gt;&lt;/script&gt;
	&lt;link   type="text/css" rel="stylesheet"  href="&lt;%=basePath %&gt;/demo.css"/&gt;

  &lt;/head&gt;
  
  &lt;body&gt;
  		&lt;div class="container"&gt;
	  		&lt;table class="datatable"&gt;
	  			&lt;tr&gt;
	  				
	  				&lt;td colspan="2" class="title"&gt;OpenID Connect 1.0 Demo&lt;/td&gt;
	  			&lt;/tr&gt;
	  			
	  			&lt;tr&gt;
	  				&lt;td&gt;OpenID Connect 1.0 Logo&lt;/td&gt;
	  				&lt;td&gt; &lt;img src="&lt;%=basePath %&gt;/static/images/openid.png"  width="124px" height="124px"/&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Login&lt;/td&gt;
	  				&lt;td&gt;&lt;%=userInfo.getSub() %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;DisplayName&lt;/td&gt;
	  				&lt;td&gt;&lt;%=userInfo.getName()%&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Department&lt;/td&gt;
	  				&lt;td&gt;&lt;%=userInfo.getGender() %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			
	  			&lt;tr&gt;
	  				&lt;td&gt;email&lt;/td&gt;
	  				&lt;td&gt;&lt;%=userInfo.getEmail() %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;ResponseString&lt;/td&gt;
	  				&lt;td style="word-wrap: break-word;"&gt;
						&lt;textarea cols="68" rows="20" v-model="text2"&gt;&lt;%=userInfo.getResponseString() %&gt;&lt;/textarea&gt;
					&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Id_token&lt;/td&gt;
	  				&lt;td style="word-wrap: break-word;"&gt;&lt;%=accessToken.getId_token() %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
				&lt;tr&gt;
	  				&lt;td&gt;Verify&lt;/td&gt;
	  				&lt;td style="word-wrap: break-word;"&gt;&lt;%=signedJWT.verify(rsaSSAVerifier) %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;Issuer&lt;/td&gt;
	  				&lt;td style="word-wrap: break-word;"&gt;&lt;%=signedJWT.getJWTClaimsSet().getIssuer() %&gt;&lt;/td&gt;
	  			&lt;/tr&gt;
	  			&lt;tr&gt;
	  				&lt;td&gt;JWTClaims&lt;/td&gt;
	  				&lt;td style="word-wrap: break-word;"&gt;
						&lt;textarea cols="68" rows="20" v-model="text2"&gt;&lt;%=signedJWT.getPayload() %&gt;&lt;/textarea&gt;
					&lt;/td&gt;
	  			&lt;/tr&gt;
	  			
	  		&lt;/table&gt;
  		&lt;/div&gt; 
		&lt;script type="text/javascript"&gt;
			FormatTextarea();
		&lt;/script&gt;
  &lt;/body&gt;
&lt;/html&gt;

</code></pre>
