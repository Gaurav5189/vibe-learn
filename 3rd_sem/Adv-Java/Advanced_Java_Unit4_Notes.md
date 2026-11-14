# Advanced Java (MCC-2.3.1) — Unit IV Notes
**PG MCA, Ravenshaw University (CBCS) | Semester III**
**Unit IV:** JavaBean · EJB · AJAX · Spring · Struts · JSF · Facelets · JSTL · JMS · Hibernate

---

## Table of Contents
1. [JavaBean](#1-javabean)
2. [EJB](#2-ejb-enterprise-javabeans)
3. [AJAX](#3-ajax)
4. [Spring Framework](#4-spring-framework)
5. [Struts](#5-struts)
6. [JSF and Facelets](#6-jsf-and-facelets)
7. [JSTL](#7-jstl)
8. [JMS](#8-jms-java-message-service)
9. [Hibernate](#9-hibernate-framework)
10. [Quick Revision](#10-quick-revision)

---

## 1. JavaBean

**JavaBean:** A reusable, serializable Java class that encapsulates data (and optionally behaviour) following fixed conventions, so tools/frameworks/JSP can discover and use it. Package: `java.beans`.

### 1.1 Conventions (Rules)
1. **Public no-argument constructor**
2. Properties are **private**; accessed via public **getters/setters**
3. Naming: `getXxx()` / `setXxx()`; for boolean, `isXxx()`
4. Implements **`java.io.Serializable`**
5. (Optional) Supports **event listeners** for bound/constrained properties

### 1.2 Features
- **Reusable & encapsulated** component
- **Introspection** – tools analyse a bean's properties/methods/events (via reflection, `Introspector`)
- **Customization** – property editors/sheets in IDEs
- **Persistence** – state saved via serialization
- **Events** – communicate through the event model (`PropertyChangeListener`)
- **Properties** – exposed attributes (get/set)
- **Methods** – public methods callable by other components
- Platform independent ("write once, run anywhere")

### 1.3 Properties
| Type | Description | Methods |
|---|---|---|
| **Simple** | Single value | `getX()`, `setX()` |
| **Read-only / Write-only** | Only getter / only setter | — |
| **Indexed** | Array of values | `getX(int i)`, `setX(int i, T v)`, `getX()`, `setX(T[])` |
| **Bound** | **Notifies listeners** when value changes | `PropertyChangeSupport`, `addPropertyChangeListener()` |
| **Constrained** | Listeners can **veto** (reject) a change | `VetoableChangeSupport`, throws `PropertyVetoException` |

### 1.4 Types of JavaBeans
| Type | Meaning |
|---|---|
| **Visual (GUI) bean** | Has a visible UI; extends `Component` (e.g., custom button) |
| **Non-visual (Invisible) bean** | No UI; holds data/business logic (e.g., `Student`, `Employee`) — most common in JSP/servlets |

### 1.5 Program: Create and Use a Bean
**Student.java**
```java
package mypack;
import java.io.Serializable;

public class Student implements Serializable {
    private int id;
    private String name;
    private int marks;

    public Student() { }                         // no-arg constructor

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getMarks() { return marks; }
    public void setMarks(int marks) { this.marks = marks; }
}
```
**Compile into `WEB-INF/classes/mypack/`.**

**Use in JSP (`bean.jsp`)**
```jsp
<jsp:useBean id="s" class="mypack.Student" scope="session"/>
<jsp:setProperty name="s" property="id" value="101"/>
<jsp:setProperty name="s" property="name" value="Amit"/>
<jsp:setProperty name="s" property="marks" value="85"/>

ID: <jsp:getProperty name="s" property="id"/><br>
Name: <jsp:getProperty name="s" property="name"/><br>
Marks: <jsp:getProperty name="s" property="marks"/>
```
- `<jsp:setProperty name="s" property="*"/>` auto-maps all matching **form fields** to bean properties.
- `scope`: `page` (default), `request`, `session`, `application`.

**Use as a plain Java class**
```java
Student s = new Student();
s.setName("Amit");
System.out.println(s.getName());
```
**Bound property example**
```java
import java.beans.*;
public class Counter {
    private int value;
    private PropertyChangeSupport pcs = new PropertyChangeSupport(this);
    public void addPropertyChangeListener(PropertyChangeListener l) { pcs.addPropertyChangeListener(l); }
    public void setValue(int v) { int old = value; value = v; pcs.firePropertyChange("value", old, v); }
    public int getValue() { return value; }
}
```
**JavaBean vs EJB:** JavaBean = simple local reusable class; EJB = server-side, container-managed enterprise component (transactions, security, remote access).

---

## 2. EJB (Enterprise JavaBeans)

**EJB:** A server-side component architecture (part of Java/Jakarta EE) for building **scalable, transactional, secure, distributed business components**. Beans run inside an **EJB container**, which provides system services automatically.

### 2.1 Features
- Encapsulates **business logic** on the server
- **Container-managed services:** transactions (JTA), security, concurrency, lifecycle, resource pooling, persistence
- **Remote & local access** (RMI-based remote clients)
- **Scalability** via instance pooling; **portability** across servers
- **Dependency injection** (`@EJB`, `@Resource`) and annotations (EJB 3+) — no complex descriptors
- **Asynchronous messaging** via Message-Driven Beans
- **Timer service**, interceptors, web-service exposure

### 2.2 Types of EJB
| Type | Purpose | State |
|---|---|---|
| **Session Bean** | Business logic invoked by clients | — |
| &nbsp;&nbsp;• **Stateless** | One call = one task; no client memory; pooled and shared | No conversational state |
| &nbsp;&nbsp;• **Stateful** | Maintains client-specific state across calls (e.g., shopping cart) | Per-client state |
| &nbsp;&nbsp;• **Singleton** (EJB 3.1) | **One shared instance** per application | Shared state |
| **Message-Driven Bean (MDB)** | Asynchronously consumes **JMS** messages (`onMessage()`) | Stateless, no client interface |
| **Entity Bean** *(legacy)* | Persistent data object mapped to DB row | Replaced by **JPA `@Entity`** in EJB 3 |

### 2.3 EJB Architecture
```
Client (Servlet / JSP / Java app / Remote client)
        │  JNDI lookup / @EJB injection
        ▼
┌──────────────── Application Server ─────────────────┐
│  EJB Container                                       │
│   ├─ Enterprise Beans (Session, MDB)                 │
│   ├─ Services: Transaction · Security · Pooling ·    │
│   │            Persistence · Concurrency · Naming    │
│   └─ EJB Proxy/Stub (business interface)             │
└───────────────┬─────────────────────────────────────┘
                ▼
     Database (JDBC/JPA)   JMS Provider   Other systems
```
**Components:**
- **EJB Server/Application Server** – hosts the container (WildFly, GlassFish, WebLogic)
- **EJB Container** – manages bean life cycle & services
- **Enterprise Beans** – business logic
- **Business interface** – `@Local` (same JVM) / `@Remote` (different JVM)
- **Client** – servlet, JSP, standalone app
- **JNDI** – naming directory used to locate beans
- **Deployment descriptor** `ejb-jar.xml` (optional in EJB 3)

### 2.4 Life Cycle of EJB
| Bean | States | Callbacks |
|---|---|---|
| **Stateless** | Does not exist ⇄ **Ready (pooled)** | `@PostConstruct` (after creation/injection), `@PreDestroy` |
| **Stateful** | Does not exist → **Ready** ⇄ **Passivated** → removed | `@PostConstruct`, `@PrePassivate`, `@PostActivate`, `@PreDestroy`, `@Remove` |
| **Singleton** | Does not exist ⇄ **Ready** | `@PostConstruct`, `@PreDestroy` |
| **Message-Driven** | Does not exist ⇄ **Ready (pooled)** | `@PostConstruct`, `onMessage()`, `@PreDestroy` |

```
Stateless / MDB / Singleton:   [Does not exist] ──create + inject + @PostConstruct──▶ [Ready]
                                                 ◀────────── @PreDestroy ───────────┘

Stateful:  [Does not exist] ─▶ [Ready] ⇄ (@PrePassivate / @PostActivate) [Passivated]
                                  │ @Remove / timeout (@PreDestroy)
                                  ▼
                            [Does not exist]
```
- **Passivation:** container saves an idle stateful bean's state to disk to free memory; **activation** restores it.
- Only stateful beans are passivated.

### 2.5 Program: Creating an EJB (EJB 3.x)
**1. Business interface – `HelloBeanRemote.java`**
```java
import javax.ejb.Remote;
@Remote
public interface HelloBeanRemote {
    String sayHello(String name);
    int add(int a, int b);
}
```
**2. Bean class – `HelloBean.java`**
```java
import javax.ejb.Stateless;
@Stateless
public class HelloBean implements HelloBeanRemote {
    public String sayHello(String name) { return "Hello, " + name; }
    public int add(int a, int b) { return a + b; }
}
```
**3. Client – servlet using injection**
```java
import javax.ejb.EJB;
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.WebServlet;
import java.io.*;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @EJB
    private HelloBeanRemote bean;

    protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.getWriter().println(bean.sayHello("Amit") + " | 2+3=" + bean.add(2, 3));
    }
}
```
**Stateful bean example (cart)**
```java
@Stateful
public class CartBean {
    private List<String> items = new ArrayList<>();
    public void add(String i) { items.add(i); }
    public List<String> getItems() { return items; }
    @Remove public void checkout() { items.clear(); }
}
```
**Deploy** the JAR/WAR/EAR on an application server (WildFly, GlassFish/Payara, WebLogic). *Tomcat alone does not support EJB.*

> Jakarta EE 9+ uses `jakarta.ejb.*` instead of `javax.ejb.*`.

---

## 3. AJAX

**AJAX (Asynchronous JavaScript And XML):** A technique (not a language) to **exchange data with the server in the background and update parts of a web page without reloading the whole page**.

**Technologies:** HTML/CSS (UI) + JavaScript + **XMLHttpRequest** (or `fetch()`) + DOM + data format (**XML / JSON / text**).

### 3.1 Features
- **Asynchronous** – page stays responsive during request
- **Partial page update** – only needed part refreshed
- **Faster, less bandwidth**; better user experience
- Works with any server technology (Servlet, JSP, PHP…)
- Supports XML, JSON, HTML, plain text
- Disadvantages: needs JavaScript; harder debugging/back-button & SEO issues; security (XSS/CSRF) care needed

### 3.2 Working of AJAX
```
1. Event occurs on page (click/keyup)
2. JavaScript creates XMLHttpRequest object
3. open(method, url, async) + send()  ───────▶  Server (Servlet/JSP)
                                                  4. Processes request (DB etc.)
5. onreadystatechange fires          ◀─────────  Sends response (text/XML/JSON)
6. JavaScript reads responseText/responseXML
7. DOM updated — page NOT reloaded
```
**`readyState` values:** `0` UNSENT · `1` OPENED · `2` HEADERS_RECEIVED · `3` LOADING · `4` DONE
**`status`:** `200` OK, `404` Not Found, `500` Server error.

| Method/Property | Use |
|---|---|
| `open(method, url, async)` | Configure request |
| `send()` / `send(data)` | Send (GET / POST) |
| `setRequestHeader()` | e.g., Content-Type for POST |
| `onreadystatechange` | Callback on state change |
| `responseText` / `responseXML` | Response data |

### 3.3 AJAX Request & Response Example
**`index.html`**
```html
<html><body>
<h3>Hello AJAX</h3>
<button onclick="loadData()">Get Server Time</button>
<div id="result"></div>

<script>
function loadData() {
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = function () {
    if (xhr.readyState == 4 && xhr.status == 200) {
      document.getElementById("result").innerHTML = xhr.responseText;
    }
  };
  xhr.open("GET", "TimeServlet", true);   // true = asynchronous
  xhr.send();
}
</script>
</body></html>
```
**`TimeServlet.java`** (response)
```java
@WebServlet("/TimeServlet")
public class TimeServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/plain");
        res.getWriter().print("Server time: " + new java.util.Date());
    }
}
```
**Modern alternative (`fetch`)**
```javascript
fetch("TimeServlet").then(r => r.text()).then(t => document.getElementById("result").innerHTML = t);
```
**POST request**
```javascript
xhr.open("POST", "Save", true);
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhr.send("name=Amit&id=101");
```

### 3.4 AJAX Database Connectivity
Browser never talks to the DB directly — AJAX calls a **server program** (Servlet/JSP) which uses **JDBC**.

**`search.html`** — live student search
```html
Enter ID: <input type="text" id="sid" onkeyup="getStudent(this.value)">
<div id="info"></div>
<script>
function getStudent(id) {
  if (id.length == 0) { document.getElementById("info").innerHTML = ""; return; }
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = function () {
    if (xhr.readyState == 4 && xhr.status == 200)
      document.getElementById("info").innerHTML = xhr.responseText;
  };
  xhr.open("GET", "StudentServlet?id=" + encodeURIComponent(id), true);
  xhr.send();
}
</script>
```
**`StudentServlet.java`**
```java
@WebServlet("/StudentServlet")
public class StudentServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.setContentType("text/html");
        PrintWriter out = res.getWriter();
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            try (Connection con = DriverManager.getConnection(
                     "jdbc:mysql://localhost:3306/college", "root", "root");
                 PreparedStatement ps = con.prepareStatement("SELECT name, marks FROM student WHERE id=?")) {
                ps.setInt(1, Integer.parseInt(req.getParameter("id")));
                try (ResultSet rs = ps.executeQuery()) {
                    if (rs.next())
                        out.print("Name: " + rs.getString("name") + " | Marks: " + rs.getInt("marks"));
                    else out.print("No record found");
                }
            }
        } catch (Exception e) { out.print("Error: " + e.getMessage()); }
    }
}
```
**Flow:** `onkeyup` → XHR GET → Servlet → JDBC SELECT → text/JSON response → JS updates `<div>`.

---

## 4. Spring Framework

**Spring:** A lightweight, open-source **application framework** for Java (enterprise) development, built on **Inversion of Control (IoC)** and **Dependency Injection (DI)**.

- **IoC:** Control of object creation/wiring shifts from your code to the **Spring container**.
- **DI:** Container *injects* an object's dependencies (via constructor or setter) instead of the object creating them → **loose coupling**.

### 4.1 Features
- **Lightweight** & non-invasive (POJO-based)
- **IoC container & Dependency Injection** (loose coupling, easy testing)
- **AOP** – separates cross-cutting concerns (logging, security, transactions)
- **MVC web framework** (Spring MVC), REST support
- **Data access abstraction** – JDBC template, ORM (Hibernate/JPA), declarative **transaction management**
- **Modular** – use only the modules you need
- Easy **testing** (JUnit integration); integrates with other frameworks (Struts, Hibernate, JSF)
- Annotation and Java-config support; **Spring Boot** for auto-configuration

### 4.2 Spring Modules
| Group | Modules |
|---|---|
| **Core Container** | `spring-core`, `spring-beans` (BeanFactory), `spring-context` (ApplicationContext), `spring-expression` (SpEL) |
| **AOP & Instrumentation** | `spring-aop`, `spring-aspects` (AspectJ), `spring-instrument` |
| **Data Access/Integration** | `spring-jdbc`, `spring-tx` (transactions), `spring-orm`, `spring-oxm`, `spring-jms` |
| **Web** | `spring-web`, `spring-webmvc`, `spring-webflux`, `spring-websocket` |
| **Test** | `spring-test` |

### 4.3 Steps for Creating a Spring Application
1. **Add dependencies** – `spring-context` (Maven/Gradle or JARs: core, beans, context, expression, commons-logging)
2. **Create bean class** (POJO)
3. **Create configuration** – XML (`applicationContext.xml`) *or* Java config (`@Configuration`/`@Bean`) *or* annotations (`@Component`)
4. **Create the container** – `ApplicationContext` (e.g., `ClassPathXmlApplicationContext`)
5. **Get bean** – `context.getBean("id", Class)`
6. **Run & use** the bean

### 4.4 Example Program (XML + DI)
**Maven dependency**
```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>6.1.x</version>
</dependency>
```
**Address.java** and **Student.java**
```java
public class Address {
    private String city;
    public void setCity(String city) { this.city = city; }
    public String getCity() { return city; }
}

public class Student {
    private int id;
    private String name;
    private Address address;                         // dependency

    public void setId(int id) { this.id = id; }
    public void setName(String name) { this.name = name; }
    public void setAddress(Address address) { this.address = address; }

    public void display() {
        System.out.println(id + " " + name + " " + address.getCity());
    }
}
```
**applicationContext.xml** (src/main/resources)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="addr" class="Address">
    <property name="city" value="Cuttack"/>
  </bean>

  <bean id="stu" class="Student">
    <property name="id" value="101"/>
    <property name="name" value="Amit"/>
    <property name="address" ref="addr"/>          <!-- setter injection -->
  </bean>
</beans>
```
**MainApp.java**
```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        Student s = (Student) ctx.getBean("stu");
        s.display();                                  // Output: 101 Amit Cuttack
    }
}
```
**Annotation / Java-config version**
```java
@Configuration
@ComponentScan("com.demo")
public class AppConfig { }

@Component
public class Greeter {
    @Autowired private Messenger messenger;
    public String greet() { return messenger.msg(); }
}

ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
```
**DI types:** *Constructor injection* (`<constructor-arg>`) — mandatory dependencies; *Setter injection* (`<property>`) — optional dependencies.
**Bean scopes:** `singleton` (default), `prototype`, `request`, `session`, `application`.
**BeanFactory vs ApplicationContext:** BeanFactory = basic, lazy; ApplicationContext = advanced (events, i18n, AOP, eager singleton loading).

---

## 5. Struts

**Apache Struts:** An open-source **MVC web framework** for building Java web applications. **Struts 2** = WebWork + Struts 1 (a complete redesign, not an upgrade of Struts 1).

### 5.1 Features (Struts 2)
- **MVC architecture** – Action (Controller/Model), JSP (View)
- **POJO-based actions** (no need to extend special classes; `ActionSupport` optional)
- **Interceptors** – reusable pre/post processing (validation, logging, file upload)
- **Powerful tag library** (`<s:form>`, `<s:textfield>`, `<s:property>`) and **OGNL** expression language
- **Configurable via XML or annotations**
- **Multiple result types** (JSP, redirect, JSON, FreeMarker…), **AJAX/REST/JSON plugins**
- Built-in **validation** and **internationalisation (i18n)**
- **Integration** with Spring, Hibernate, JSF, SiteMesh
- **Theme & template** support; pluggable architecture

### 5.2 Struts 2 Architecture / Request Flow
```
Browser → Servlet Filter (StrutsPrepareAndExecuteFilter) → ActionMapper
   → ActionProxy (reads struts.xml) → ActionInvocation
   → Interceptors (before) → Action.execute() → Result code ("success")
   → Result (JSP/View) → Interceptors (after) → Response → Browser
```
| Component | Role |
|---|---|
| **Filter (FilterDispatcher)** | Front controller; intercepts all requests |
| **ActionMapper** | Decides whether the request maps to an action |
| **ActionProxy / ActionInvocation** | Execute interceptors + action |
| **Action** | Handles request, calls model; returns result string |
| **Interceptor** | Cross-cutting tasks |
| **Result / View** | JSP that renders output |
| `struts.xml` | Maps action → class → result |

### 5.3 Creating a Struts 2 Application (Login Example)
**Steps:** (1) Create *Dynamic Web Project* → (2) add Struts 2 JARs / Maven dependency → (3) configure filter in `web.xml` → (4) create Action class → (5) create `struts.xml` → (6) create JSP views → (7) deploy & run.

**1. `web.xml`**
```xml
<filter>
  <filter-name>struts2</filter-name>
  <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>struts2</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```
*(In older Struts 2.x the class is `org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter`; very old versions used `FilterDispatcher`.)*

**2. `LoginAction.java`**
```java
package com.demo;
import com.opensymphony.xwork2.ActionSupport;

public class LoginAction extends ActionSupport {
    private String username;
    private String password;
    // getters & setters (form fields are auto-populated)
    public String getUsername() { return username; }
    public void setUsername(String u) { this.username = u; }
    public String getPassword() { return password; }
    public void setPassword(String p) { this.password = p; }

    public String execute() {
        if ("admin".equals(username) && "123".equals(password)) return SUCCESS;
        return ERROR;
    }
}
```
**3. `struts.xml`** (in `src/` or `WEB-INF/classes`)
```xml
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
  "http://struts.apache.org/dtds/struts-2.5.dtd">
<struts>
  <package name="default" extends="struts-default">
    <action name="login" class="com.demo.LoginAction">
      <result name="success">welcome.jsp</result>
      <result name="error">login.jsp</result>
    </action>
  </package>
</struts>
```
**4. `login.jsp`**
```jsp
<%@ taglib prefix="s" uri="/struts-tags" %>
<s:form action="login">
  <s:textfield name="username" label="User"/>
  <s:password  name="password" label="Password"/>
  <s:submit value="Login"/>
</s:form>
```
**5. `welcome.jsp`**
```jsp
<%@ taglib prefix="s" uri="/struts-tags" %>
Welcome, <s:property value="username"/>
```
Run: `http://localhost:8080/app/login.jsp` → submit → `login.action`.

> Struts 1 vs 2: Struts 2 uses POJO actions, interceptors, OGNL, and is far more flexible; Struts 1 used `ActionForm`/`Action` classes and is end-of-life.

---

## 6. JSF and Facelets

### 6.1 JSF (JavaServer Faces)
**JSF:** A **component-based, server-side MVC UI framework** for building web UIs in Java EE/Jakarta EE.

**Basic features**
- **Reusable UI components** (`h:inputText`, `h:commandButton`, `h:dataTable`) with server-side state
- **MVC:** Model = **managed (backing) beans**, View = **Facelets (XHTML)**, Controller = **FacesServlet**
- **Event handling** (action, value-change listeners)
- **Navigation** (implicit outcome or `faces-config.xml`)
- **Built-in validation & conversion** (`f:validateLength`, `f:convertNumber`)
- **Expression Language** `#{bean.property}` binds UI to beans
- **Managed beans** with scopes (`@RequestScoped`, `@SessionScoped`, `@ViewScoped`, `@ApplicationScoped`); CDI `@Named`
- **Internationalisation**, **AJAX support** (`<f:ajax>`), templating, custom components
- **JSF Life Cycle (6 phases):** Restore View → Apply Request Values → Process Validations → Update Model Values → Invoke Application → Render Response

**Example**
```xml
<!-- hello.xhtml -->
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html">
<h:body>
  <h:form>
    <h:inputText value="#{userBean.name}"/>
    <h:commandButton value="Greet" action="#{userBean.greet}"/>
    <h:outputText value="#{userBean.message}"/>
  </h:form>
</h:body>
</html>
```
```java
@Named @RequestScoped
public class UserBean {
    private String name, message;
    public String greet() { message = "Hello " + name; return null; }
    // getters/setters
}
```
### 6.2 Facelets
**Facelets:** The **default view declaration language (VDL)** of JSF 2.x (replaced JSP as the view technology). Pages are **XHTML** files (`.xhtml`).

**Features**
- **Templating** – master layout with `<ui:insert>`, pages fill via `<ui:composition template>` + `<ui:define>`
- **Composite components** – reusable custom components (`<composite:interface/implementation>`)
- **Faster** than JSP; compile-time EL checking, better error reporting
- **Reuse** with `<ui:include>`, `<ui:repeat>`, `<ui:param>`
- Works with XHTML, EL, and JSF/PrimeFaces tag libraries

```xml
<!-- template.xhtml -->
<html xmlns:ui="http://xmlns.jcp.org/jsf/facelets">
<body>
  <div id="header"><ui:insert name="header">Default header</ui:insert></div>
  <div id="content"><ui:insert name="content"/></div>
</body></html>

<!-- page.xhtml -->
<ui:composition template="template.xhtml"
                xmlns:ui="http://xmlns.jcp.org/jsf/facelets">
  <ui:define name="content">Page specific content</ui:define>
</ui:composition>
```
> Jakarta Faces 3+/4 uses namespaces `jakarta.faces.*` / `jakarta.faces.html` instead of `javax.faces` / `xmlns.jcp.org`.

---

## 7. JSTL

**JSTL (JSP Standard Tag Library):** A collection of **standard custom tags** that replace scriptlets (`<% %>`) in JSP, making pages cleaner. Works with **Expression Language (EL)** `${...}`.

**Setup:** Add JSTL JAR to `WEB-INF/lib`; declare the taglib.

### 7.1 Tag Libraries
| Library | Prefix | URI (JSTL 1.2) | Use |
|---|---|---|---|
| **Core** | `c` | `http://java.sun.com/jsp/jstl/core` | Variables, conditions, loops, URLs |
| **Formatting (fmt)** | `fmt` | `http://java.sun.com/jsp/jstl/fmt` | i18n, number/date formatting |
| **SQL** | `sql` | `http://java.sun.com/jsp/jstl/sql` | Database queries (`setDataSource`, `query`, `update`) |
| **XML** | `x` | `http://java.sun.com/jsp/jstl/xml` | XML parsing/XPath/transform |
| **Functions** | `fn` | `http://java.sun.com/jsp/jstl/functions` | String functions (`fn:length`, `fn:toUpperCase`) |

*(JSTL 3.0/Jakarta uses `jakarta.tags.core` etc.)*

### 7.2 Core Tags
| Tag | Purpose |
|---|---|
| `<c:set var="x" value="10"/>` | Set variable |
| `<c:out value="${x}"/>` | Print (escapes XML) |
| `<c:remove var="x"/>` | Remove variable |
| `<c:if test="${x gt 5}">…</c:if>` | Condition |
| `<c:choose> <c:when> <c:otherwise>` | If–else / switch |
| `<c:forEach var="i" begin="1" end="5">` | Loop (also over collections) |
| `<c:forTokens>` | Loop over delimited string |
| `<c:import>`, `<c:redirect>`, `<c:url>` | URL operations |
| `<c:catch var="e">` | Exception handling |

### 7.3 Example
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>

<c:set var="marks" value="72"/>
<c:choose>
  <c:when test="${marks >= 60}">First Division</c:when>
  <c:otherwise>Below 60</c:otherwise>
</c:choose>

<c:forEach var="s" items="${studentList}">
  ${s.id} - ${s.name}<br>
</c:forEach>

<fmt:formatNumber value="1234.5" type="currency"/>
```
**SQL tags example**
```jsp
<%@ taglib prefix="sql" uri="http://java.sun.com/jsp/jstl/sql" %>
<sql:setDataSource var="db" driver="com.mysql.cj.jdbc.Driver"
    url="jdbc:mysql://localhost:3306/college" user="root" password="root"/>
<sql:query dataSource="${db}" var="rs">SELECT * FROM student</sql:query>
<c:forEach var="r" items="${rs.rows}">${r.id} ${r.name}<br></c:forEach>
```
**Benefits:** no Java in JSP, reusable, readable, designer-friendly. *(SQL tags are for prototyping only; use DAO/Servlet in real apps.)*

---

## 8. JMS (Java Message Service)

**JMS:** A Java API for sending and receiving **messages asynchronously** between applications/components through a **message broker (JMS provider)** — e.g., ActiveMQ, Artemis, IBM MQ. Enables **loose coupling** (sender and receiver need not run at the same time).

### 8.1 Messaging Models
| Model | Description | Destination | Receivers |
|---|---|---|---|
| **Point-to-Point (PTP)** | One sender → **one receiver** per message; message consumed once | **Queue** | Single consumer |
| **Publish/Subscribe (Pub/Sub)** | Publisher → **many subscribers** | **Topic** | All active subscribers |

### 8.2 Architecture & Components
```
Producer ──▶ [ Destination: Queue/Topic in JMS Provider ] ──▶ Consumer
```
| Component | Role |
|---|---|
| **JMS Provider** | Message broker implementing JMS |
| **ConnectionFactory** | Creates connections (looked up via JNDI) |
| **Connection / Session** | Link to broker / single-threaded context for producing & consuming |
| **Destination** | `Queue` or `Topic` |
| **MessageProducer / MessageConsumer** | Send / receive |
| **Message** | `TextMessage`, `ObjectMessage`, `MapMessage`, `BytesMessage`, `StreamMessage` |
| **MessageListener** | `onMessage()` for async receipt (used by **MDB**) |

**Message structure:** Header + Properties + Body.
**Delivery:** synchronous (`receive()`) or asynchronous (`MessageListener`). Features: reliability, transactions, acknowledgements, persistent/non-persistent delivery.

### 8.3 Example (JMS 2.0 simplified API)
```java
// Sender
@Resource(lookup = "jms/ConnectionFactory") ConnectionFactory cf;
@Resource(lookup = "jms/MyQueue") Queue queue;

try (JMSContext ctx = cf.createContext()) {
    ctx.createProducer().send(queue, "Hello JMS");
}

// Receiver (Message-Driven Bean)
@MessageDriven(activationConfig = {
  @ActivationConfigProperty(propertyName = "destinationType", propertyValue = "javax.jms.Queue"),
  @ActivationConfigProperty(propertyName = "destinationLookup", propertyValue = "jms/MyQueue")
})
public class MyMDB implements MessageListener {
    public void onMessage(Message m) {
        try { System.out.println("Got: " + ((TextMessage) m).getText()); }
        catch (JMSException e) { e.printStackTrace(); }
    }
}
```
**Use cases:** order processing, notifications, decoupled microservices. *(Jakarta Messaging uses `jakarta.jms.*`.)*

---

## 9. Hibernate Framework

**Hibernate:** An open-source **ORM (Object-Relational Mapping)** framework that maps Java classes ↔ database tables, so you work with objects instead of writing JDBC/SQL. It is a **JPA** implementation.

### 9.1 Features
- **ORM** – object ↔ table mapping (XML or annotations like `@Entity`, `@Table`, `@Id`, `@Column`)
- **Database independent** – generates DB-specific SQL via *dialect*
- **HQL** (Hibernate Query Language, object-oriented), Criteria API, native SQL
- **Automatic table creation** (`hbm2ddl.auto`), **lazy loading**, **caching** (1st-level default, 2nd-level optional)
- **Less boilerplate** than JDBC; **transaction management**; **relationships** (one-to-one, one-to-many, many-to-many); inheritance mapping
- Integrates with Spring, Struts, JSF

### 9.2 Architecture / Core Interfaces
```
Application (POJO / Entities)
      ↓
Configuration (hibernate.cfg.xml) → SessionFactory (heavy, one per DB)
      ↓
Session (light, per unit-of-work) ─ Transaction ─ Query/Criteria
      ↓
JDBC / JTA  →  Database
```
| Interface | Role |
|---|---|
| `Configuration` | Reads `hibernate.cfg.xml` & mappings |
| `SessionFactory` | Immutable, thread-safe factory of `Session` (created once) |
| `Session` | Single-threaded; wraps JDBC connection; `save/persist`, `get/load`, `update`, `delete`, `createQuery` |
| `Transaction` | `begin → commit / rollback` |
| `Query` / `Criteria` | HQL / criteria queries |

### 9.3 Example
**`hibernate.cfg.xml`**
```xml
<hibernate-configuration>
  <session-factory>
    <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
    <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/college</property>
    <property name="hibernate.connection.username">root</property>
    <property name="hibernate.connection.password">root</property>
    <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
    <property name="hibernate.hbm2ddl.auto">update</property>
    <property name="hibernate.show_sql">true</property>
    <mapping class="com.demo.Student"/>
  </session-factory>
</hibernate-configuration>
```
**Entity**
```java
import javax.persistence.*;          // jakarta.persistence.* in Hibernate 6+

@Entity
@Table(name = "student")
public class Student {
    @Id private int id;
    @Column(name = "name") private String name;
    private int marks;
    public Student() { }
    // getters & setters
}
```
**Client (CRUD)**
```java
SessionFactory sf = new Configuration().configure().buildSessionFactory();
Session session = sf.openSession();
Transaction tx = session.beginTransaction();

Student s = new Student(); s.setId(101); s.setName("Amit"); s.setMarks(85);
session.persist(s);                                       // INSERT (older: session.save)

Student f = session.get(Student.class, 101);              // SELECT by id
f.setMarks(90);                                           // UPDATE (dirty checking)
List<Student> all = session.createQuery("from Student", Student.class).list();   // HQL
// session.remove(f);                                     // DELETE (older: session.delete)

tx.commit();
session.close();
sf.close();
```
**Steps:** add dependencies → create `hibernate.cfg.xml` → create entity → build `SessionFactory` → open `Session` → begin `Transaction` → perform operations → commit → close.

**Hibernate vs JDBC:** Hibernate = object-oriented, auto SQL, caching, portable; JDBC = manual SQL & mapping, boilerplate, tied to DB.

---

## 10. Quick Revision

- **JavaBean:** no-arg constructor + private fields + getters/setters + `Serializable`. Properties: simple, indexed, **bound** (notify), **constrained** (veto). Types: visual / non-visual.
- **EJB:** server-side business components in an **EJB container**. Types: **Stateless, Stateful, Singleton** (session), **MDB**, Entity (legacy → JPA). Stateful bean = only one that passivates.
- **AJAX:** `XMLHttpRequest`; `readyState==4 && status==200`; updates page partially. DB access via Servlet + JDBC.
- **Spring:** IoC + DI; modules: Core Container, AOP, Data Access, Web, Test. Steps: bean → config → `ApplicationContext` → `getBean`.
- **Struts 2:** MVC; filter → ActionMapper → ActionProxy → Interceptors → Action → Result; `struts.xml`.
- **JSF:** component-based UI; managed beans + `#{}` EL; 6-phase life cycle. **Facelets** = XHTML view/templating (`ui:insert/define`).
- **JSTL:** Core, fmt, sql, xml, functions; `<c:forEach>`, `<c:if>`, `<c:choose>`.
- **JMS:** async messaging; **Queue** (point-to-point, one consumer) vs **Topic** (pub/sub, many); MDB consumes.
- **Hibernate:** ORM; `SessionFactory` → `Session` → `Transaction`; HQL; `hibernate.cfg.xml`.

### Likely Exam Questions
1. What is a JavaBean? Explain its features, properties and types. Write a program to create and use a bean.
2. What is EJB? Explain its features, types and architecture.
3. Explain the life cycle of stateless / stateful session beans and MDB.
4. Write a program to create an EJB (session bean + client).
5. What is AJAX? Explain features and working with an example.
6. Write an AJAX program with database connectivity.
7. What is Spring? Explain IoC/DI, features and modules.
8. Write steps and a sample program for a Spring application.
9. Explain Struts features and architecture; create a Struts application.
10. Explain basic features of JSF and Facelets.
11. What is JSTL? Explain core tags with examples.
12. What is JMS? Explain Queue vs Topic and its components.
13. Explain Hibernate framework, architecture and write a sample program.

---
*Reference books (per syllabus): Ivan Bayross – Web Technologies Pt.1 & 2 (BPB); Steven Holzner – Java 8 Programming Black Book (Dreamtech); Eric Jendrock – Java EE 6 Basic Concepts (Pearson); Java (J2EE) and Spring Framework – Ayan International.*
