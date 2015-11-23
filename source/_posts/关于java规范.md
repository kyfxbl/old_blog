title: 关于java规范
date: 2013-09-24 11:06
categories: java 
---
规范（Specification），有时也叫做“标准”，一般称为JSR-xxx，我的理解是：为了解决特定范围的问题，而设计的一系列接口的集合
<!--more--> 
 
有了规范之后，开发人员就可以自由选择实现。使用者的代码，不与具体的提供商绑定，以实现松耦合 

比如说JSR315，也就是Servlet3.0规范，定义了Filter、Servlet、ServletRequest、ServletResponse等一系列接口，并规定了这些接口应该提供哪些功能，然后tomcat、jboss、WAS等servlet容器，就实现了这个规范

开发人员开发的web应用，如果仅依赖于servlet-api，不依赖某个厂商的特定实现类，那么这个web应用，应该是无论在哪个servlet容器中，都能部署的 

单个或者几个接口，还不足以成为规范。比如spring框架中定义了一组接口ApplicationContext、BeanFactory、BeanDefinitionReader等。spring自己提供了实现，开发人员也可以对spring框架进行扩展。但是这还不能称为一种规范。如果高度上升一点，比如称为JSR-xxx，Java Dependence Injection，然后规定一系列的接口，由不同的框架提供商来实现，大概就可以称为一种规范了 

规范有3种角色的参与者，规范制定者（也就是JCP），提供商（Provider），以及使用者（开发人员） 

```
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package javax.servlet;

import java.io.IOException;

/**
 * Defines methods that all servlets must implement.
 * 
 * <p>
 * A servlet is a small Java program that runs within a Web server. Servlets
 * receive and respond to requests from Web clients, usually across HTTP, the
 * HyperText Transfer Protocol.
 * 
 * <p>
 * To implement this interface, you can write a generic servlet that extends
 * <code>javax.servlet.GenericServlet</code> or an HTTP servlet that extends
 * <code>javax.servlet.http.HttpServlet</code>.
 * 
 * <p>
 * This interface defines methods to initialize a servlet, to service requests,
 * and to remove a servlet from the server. These are known as life-cycle
 * methods and are called in the following sequence:
 * <ol>
 * <li>The servlet is constructed, then initialized with the <code>init</code>
 * method.
 * <li>Any calls from clients to the <code>service</code> method are handled.
 * <li>The servlet is taken out of service, then destroyed with the
 * <code>destroy</code> method, then garbage collected and finalized.
 * </ol>
 * 
 * <p>
 * In addition to the life-cycle methods, this interface provides the
 * <code>getServletConfig</code> method, which the servlet can use to get any
 * startup information, and the <code>getServletInfo</code> method, which allows
 * the servlet to return basic information about itself, such as author,
 * version, and copyright.
 * 
 * @version $Version$
 * 
 * @see GenericServlet
 * @see javax.servlet.http.HttpServlet
 */
public interface Servlet
```

可以看到，JSR315的API是由Apache基金会提供的 

