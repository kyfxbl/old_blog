title: cxf配置方式打开日志和https
date: 2013-09-24 11:32
categories: java 
---
本文介绍通过配置方式打开cxf的日志和https功能。用编码方式也可以实现，但是就存在代码重复的问题，用配置方式会比较好 
<!--more-->

# 开启日志

cxf内置了日志拦截器，但是默认并没有打开

## 编码方式开启

```
<jaxws:client id="client" serviceClass="xxx.xxx.xxx" address="${webservice_address}" />
```

```
WebserviceInterface client = (WebserviceInterface)ApplicationContext.getBean("client");
Client proxy = ClientProxy.getClient(client);
proxy.getInInterceptors().add(new LoggingInInterceptor());
proxy.getOutInterceptors().add(new LoggingOutInterceptor());
```

## 配置文件开启

```
<jaxws:client id="client" serviceClass="xxx.xxx.xxx" address="${webservice_address}">
		<jaxws:outInterceptors>
			<bean class="org.apache.cxf.interceptor.LoggingOutInterceptor" />
		</jaxws:outInterceptors>
		<jaxws:inInterceptors>
			<bean class="org.apache.cxf.interceptor.LoggingInInterceptor" />
		</jaxws:inInterceptors>
</jaxws:client>
```

这样就可以把web service请求和响应的日志打出来了 

## 日志格式

```
Outbound Message 
--------------------------- 
ID: 1 
Address: https://www.remedy-dummy.com:443/remedy/webservice/RemedySA 
Encoding: UTF-8 
Content-Type: text/xml 
Headers: {SOAPAction=["http://cz.o2.com/systems/integrationinfrastructure/CIP-B2B/CIP-B2B_ServiceAssuranceWorkForceClientManagement/1.0/acknowledge"], Accept=[*/*]} 
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:AcknowledgeRequest xmlns="http://cz.o2.com/cip/svc/IntegrationMessage-2.0" xmlns:ns2="http://cz.o2.com/systems/integrationinfrastructure/CIP-B2B/CIP-B2B_ServiceAssuranceWorkForceClientManagement/1.0"><ns2:requestBody><ns2:messageType>wolegequ</ns2:messageType><ns2:correlationId>0</ns2:correlationId></ns2:requestBody></ns2:AcknowledgeRequest></soap:Body></soap:Envelope> 
-------------------------------------- 
```

Outbound Message，是发出去的消息，对于客户端来说，是发出去的请求；对于服务端来说，是发出去的响应 

```
Inbound Message 
---------------------------- 
ID: 1 
Encoding: UTF-8 
Content-Type: text/xml;charset=UTF-8 
Headers: {content-type=[text/xml;charset=UTF-8], Date=[Fri, 20 Apr 2012 15:25:11 GMT], Content-Length=[478], X-Powered-By=[Servlet 2.4; JBoss-4.2.3.GA (build: SVNTag=JBoss_4_2_3_GA date=200807181417)/JBossWeb-2.0], Server=[Apache-Coyote/1.1]} 
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:AcknowledgeResponse xmlns="http://cz.o2.com/cip/svc/IntegrationMessage-2.0" xmlns:ns2="http://cz.o2.com/systems/integrationinfrastructure/CIP-B2B/CIP-B2B_ServiceAssuranceWorkForceClientManagement/1.0"><ns2:responseBody><ns2:status>true</ns2:status><ns2:errorDescription>call acknowledge() success</ns2:errorDescription></ns2:responseBody></ns2:AcknowledgeResponse></soap:Body></soap:Envelope> 
-------------------------------------- 
```

Inbound Message，是收到的消息，对于客户端来说，是收到的响应；对于服务端来说，是收到的请求 

ID是成对出现的，一个请求必有一个响应 

Address只有请求的Outbound才有，表示发送的地址，也就是web service的endpoint 
Headers是http请求头或响应头 
Payload是日志的关键，其中就是soap正文的内容 

# 发送https请求 

主要是要把证书的信息放到http消息中

## 编码方式实现

```
WebserviceInterface client = (WebserviceInterface) ApplicationContext
				.getBean("client");
		Client proxy = ClientProxy.getClient(client);

		HTTPConduit conduit = (HTTPConduit) proxy.getConduit();

		TLSClientParameters tlsParams = conduit.getTlsClientParameters();
		tlsParams.setKeyManagers();
		tlsParams.setTrustManagers();
		tlsParams.setDisableCNCheck(true);
		tlsParams.setSecureSocketProtocol("SSL");

		conduit.setTlsClientParameters(tlsParams);
```

## 配置文件实现

```
<http:conduit name="*.http-conduit">

		<http:tlsClientParameters disableCNCheck="true"
			secureSocketProtocol="SSL">

			<!-- 对方的证书 -->
			<sec:trustManagers>
				<sec:keyStore type="JKS" password="changeit"
					file="trust.keystore" />
			</sec:trustManagers>

			<!-- 己方的证书 -->
			<sec:keyManagers keyPassword="changeit">
				<sec:keyStore type="JKS" password="changeit"
					file="self.keystore" />
			</sec:keyManagers>

			<sec:cipherSuitesFilter>
				<sec:include>.*_EXPORT_.*</sec:include>
				<sec:include>.*_EXPORT1024_.*</sec:include>
				<sec:include>.*_WITH_DES_.*</sec:include>
				<sec:include>.*_WITH_NULL_.*</sec:include>
				<sec:exclude>.*_DH_anon_.*</sec:exclude>
			</sec:cipherSuitesFilter>

		</http:tlsClientParameters>

	</http:conduit>
```

用配置文件的方式，可以节省很多重复代码 