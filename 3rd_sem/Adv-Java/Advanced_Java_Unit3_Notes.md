# Advanced Java (MCC-2.3.1) — Unit III Notes
**PG MCA, Ravenshaw University (CBCS) | Semester III**
**Unit III:** Servlet · Session Tracking · Cookies · RMI

---

## Table of Contents
1. [Servlet: Introduction & Features](#1-servlet-introduction--features)
2. [JSP vs Servlet](#2-jsp-vs-servlet)
3. [Servlet Syntax Structure](#3-servlet-syntax-structure)
4. [Servlet Life Cycle](#4-servlet-life-cycle)
5. [Types of Servlets](#5-types-of-servlets)
6. [Servlet API and Packages](#6-servlet-api-and-packages)
7. [Dynamic Page Creation by Servlet](#7-dynamic-page-creation-by-servlet)
8. [Database Access Using Servlet](#8-database-access-using-servlet)
9. [Session Tracking](#9-session-tracking)
10. [Creating Cookies](#10-creating-cookies)
11. [RMI](#11-rmi-remote-method-invocation)
12. [Quick Revision](#12-quick-revision)

---

## 1. Servlet: Introduction & Features

**Servlet:** A Java class that runs on a **web server (servlet container)** and handles client **requests**, generating dynamic **responses** (usually HTML) over HTTP. It is the Java replacement for CGI.

**Servlet container (web container):** Software (Tomcat, Jetty, GlassFish) that loads servlets, manages their life cycle, and maps URLs to servlets.

### Request–Response Flow
```
Browser → HTTP request → Web Server/Container → Servlet (doGet/doPost)
Browser ← HTTP response (HTML) ←───────────────────┘
```

### Features
- **Platform independent** (written in Java)
- **Efficient** – loaded once; each request runs in a **new thread**, not a new process
- **Persistent** – stays in memory between requests
- **Robust & secure** – Java memory management, exception handling, security manager
- **Portable** across servers; **extensible** (full Java API access)
- Built-in **session management** (cookies, `HttpSession`)
- Supports **multithreading**; integrates with JDBC, JSP, EJB, RMI

### Servlet vs CGI
| Servlet | CGI |
|---|---|
| One process, **threads** per request | New **OS process** per request |
| Fast, stays in memory | Slower (process creation overhead) |
| Platform independent | Platform dependent |
| Built-in session support | Manual |

---

## 2. JSP vs Servlet

| Servlet | JSP |
|---|---|
| Java code with HTML inside `out.println()` | HTML with Java code in tags |
| Best for **logic / controller** | Best for **presentation / view** |
| Must be compiled manually; needs `web.xml`/annotation mapping | Auto-translated to servlet & compiled by container |
| Harder for designers | Easy for designers |
| Faster to execute (already a class) | First request slower (translation) |
| Has no implicit objects (get via parameters) | Has 9 implicit objects |
| `doGet()` / `doPost()` | `_jspService()` |

> A JSP **is converted into a servlet**. In MVC: Servlet = Controller, JSP = View.

---

## 3. Servlet Syntax Structure

### 3.1 Skeleton
```java
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.WebServlet;

@WebServlet("/hello")                         // URL mapping (Servlet 3.0+)
public class HelloServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html");           // 1. set MIME type
        PrintWriter out = response.getWriter();         // 2. get writer
        out.println("<html><body>");                    // 3. write output
        out.println("<h2>Hello Servlet</h2>");
        out.println("</body></html>");
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }
}
```
**Structure steps:** (1) import packages → (2) extend `HttpServlet` → (3) override `doGet()`/`doPost()` → (4) set content type → (5) get `PrintWriter` → (6) write response → (7) map URL.

### 3.2 Deployment Descriptor `web.xml` (alternative to `@WebServlet`)
```xml
<web-app>
  <servlet>
    <servlet-name>Hello</servlet-name>
    <servlet-class>HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>Hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>
```
### 3.3 Folder Structure (Tomcat)
```
myapp/
 ├── index.html / *.jsp
 └── WEB-INF/
      ├── web.xml
      ├── classes/   (compiled .class files)
      └── lib/       (JAR files, e.g. JDBC driver)
```
**Run:** `http://localhost:8080/myapp/hello`

### 3.4 HTTP Methods → Servlet methods
| HTTP | Method | Use |
|---|---|---|
| GET | `doGet()` | Fetch data; params in URL (visible, limited) |
| POST | `doPost()` | Send data in body (secure, large) |
| PUT / DELETE | `doPut()` / `doDelete()` | Update / delete resource |

---

## 4. Servlet Life Cycle

Managed entirely by the **container**. Defined by the `javax.servlet.Servlet` interface.

```
Load class → Instantiate → init() → service() [many times] → destroy() → Garbage collected
```
| Phase | Method | Called | Purpose |
|---|---|---|---|
| 1. Loading & instantiation | constructor | Once | Container loads class, creates **one instance** |
| 2. Initialization | `init(ServletConfig)` | **Once** | One-time setup (DB connection, config) |
| 3. Request handling | `service(req, res)` → `doGet()/doPost()` | **Every request** (new thread) | Process request, send response |
| 4. Destruction | `destroy()` | **Once** | Release resources |

- `service()` in `HttpServlet` reads the HTTP method and **dispatches** to `doGet()`, `doPost()`, etc.
- `init()` must finish successfully before any request is served.
- After `destroy()`, `service()` is never called again.
- Servlets are **multithreaded** → avoid shared instance variables (or synchronise).
- `loadOnStartup` (in `@WebServlet` / `web.xml`) loads the servlet at server start instead of first request.

```java
public class LifeDemo extends HttpServlet {
    public void init() throws ServletException { System.out.println("init"); }
    protected void doGet(HttpServletRequest rq, HttpServletResponse rs) throws IOException {
        rs.getWriter().println("service/doGet");
    }
    public void destroy() { System.out.println("destroy"); }
}
```

---

## 5. Types of Servlets

| Type | Class | Description |
|---|---|---|
| **Generic Servlet** | `javax.servlet.GenericServlet` | Protocol-**independent**; override `service()`; works with any protocol |
| **HTTP Servlet** | `javax.servlet.http.HttpServlet` | HTTP-**specific**; override `doGet()/doPost()`; most commonly used |

**Hierarchy:**
```
Servlet (interface) → GenericServlet (abstract class) → HttpServlet (abstract class) → YourServlet
```
```java
// Generic servlet
public class MyGeneric extends GenericServlet {
    public void service(ServletRequest req, ServletResponse res) throws IOException {
        res.getWriter().println("Generic Servlet");
    }
}
```
**Other ways servlets are classified:**
- **Servlet implementing `Servlet` interface** directly (must implement all 5 methods).
- **By creation:** *Annotation-based* (`@WebServlet`) vs *Descriptor-based* (`web.xml`).

---

## 6. Servlet API and Packages

### 6.1 Packages
| Package | Contents |
|---|---|
| `javax.servlet` | Generic, protocol-independent classes/interfaces |
| `javax.servlet.http` | HTTP-specific classes/interfaces |
| `javax.servlet.annotation` | Annotations (`@WebServlet`, `@WebFilter`, `@WebListener`, `@WebInitParam`) |
| `javax.servlet.descriptor` | Deployment-descriptor config API |

> From Jakarta EE 9+, names are `jakarta.servlet.*` (same API, new namespace).

### 6.2 Key Interfaces & Classes
**`javax.servlet`**
| Name | Role |
|---|---|
| `Servlet` (interface) | Defines `init()`, `service()`, `destroy()`, `getServletConfig()`, `getServletInfo()` |
| `ServletConfig` | Per-servlet init parameters (`getInitParameter()`) |
| `ServletContext` | Per-application info shared by all servlets |
| `ServletRequest` / `ServletResponse` | Generic request / response |
| `RequestDispatcher` | `forward()` / `include()` to another resource |
| `Filter`, `FilterChain` | Pre/post-processing of requests |
| `GenericServlet` (class) | Protocol-independent base servlet |
| `ServletException`, `UnavailableException` | Exceptions |

**`javax.servlet.http`**
| Name | Role |
|---|---|
| `HttpServlet` | Base class for HTTP servlets |
| `HttpServletRequest` | `getParameter()`, `getParameterValues()`, `getHeader()`, `getCookies()`, `getSession()`, `getMethod()`, `getRequestDispatcher()` |
| `HttpServletResponse` | `setContentType()`, `getWriter()`, `sendRedirect()`, `addCookie()`, `setStatus()`, `sendError()` |
| `HttpSession` | Session tracking object |
| `Cookie` | HTTP cookie |
| `HttpSessionListener`, `HttpSessionBindingListener` | Session event listeners |

### 6.3 Passing Data (request ↔ servlet)
```html
<form action="hello" method="post">
  Name: <input type="text" name="uname"> <input type="submit">
</form>
```
```java
String n = request.getParameter("uname");
```
**`ServletConfig` vs `ServletContext`:** Config = one servlet's parameters; Context = whole web app's shared data.

### 6.4 Redirect vs Forward
| `response.sendRedirect(url)` | `RequestDispatcher.forward(req, res)` |
|---|---|
| Client-side; new request; URL changes | Server-side; same request; URL unchanged |
| Can go to other apps/sites | Only within same app |

---

## 7. Dynamic Page Creation by Servlet

Output varies by request data, time, user, etc.

**`login.html`**
```html
<form action="welcome" method="post">
  Name: <input type="text" name="uname">
  Marks: <input type="number" name="m">
  <input type="submit" value="Result">
</form>
```
**`WelcomeServlet.java`**
```java
@WebServlet("/welcome")
public class WelcomeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest req, HttpServletResponse res)
            throws ServletException, IOException {
        res.setContentType("text/html");
        PrintWriter out = res.getWriter();
        String name = req.getParameter("uname");
        int m = Integer.parseInt(req.getParameter("m"));
        out.println("<html><body>");
        out.println("<h2>Hello, " + name + "</h2>");
        out.println(m >= 40 ? "<p style='color:green'>Pass</p>"
                            : "<p style='color:red'>Fail</p>");
        out.println("<p>Time: " + new java.util.Date() + "</p>");
        out.println("</body></html>");
    }
}
```

---

## 8. Database Access Using Servlet

**Flow:** HTML form → Servlet (JDBC) → DB → Servlet builds HTML result.
Put the JDBC driver JAR in `WEB-INF/lib`.

### 8.1 Insert (registration)
```java
@WebServlet("/register")
public class RegisterServlet extends HttpServlet {
    protected void doPost(HttpServletRequest req, HttpServletResponse res)
            throws ServletException, IOException {
        res.setContentType("text/html");
        PrintWriter out = res.getWriter();
        int id = Integer.parseInt(req.getParameter("id"));
        String name = req.getParameter("name");
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            try (Connection con = DriverManager.getConnection(
                     "jdbc:mysql://localhost:3306/college", "root", "root");
                 PreparedStatement ps = con.prepareStatement(
                     "INSERT INTO student(id,name) VALUES(?,?)")) {
                ps.setInt(1, id);
                ps.setString(2, name);
                int n = ps.executeUpdate();
                out.println("<h3>" + n + " record inserted</h3>");
            }
        } catch (Exception e) { out.println("Error: " + e); }
    }
}
```
### 8.2 Select (display all)
```java
protected void doGet(HttpServletRequest req, HttpServletResponse res)
        throws ServletException, IOException {
    res.setContentType("text/html");
    PrintWriter out = res.getWriter();
    try {
        Class.forName("com.mysql.cj.jdbc.Driver");
        try (Connection con = DriverManager.getConnection(
                 "jdbc:mysql://localhost:3306/college", "root", "root");
             Statement st = con.createStatement();
             ResultSet rs = st.executeQuery("SELECT * FROM student")) {
            out.println("<table border='1'><tr><th>ID</th><th>Name</th></tr>");
            while (rs.next())
                out.println("<tr><td>" + rs.getInt(1) + "</td><td>" + rs.getString(2) + "</td></tr>");
            out.println("</table>");
        }
    } catch (Exception e) { out.println("Error: " + e); }
}
```
**Update / Delete:** same pattern with `UPDATE ... SET ... WHERE id=?` / `DELETE ... WHERE id=?` and `executeUpdate()`.

**Tips:** use `PreparedStatement` (prevents SQL injection); load the driver once in `init()`; use a **connection pool** (`DataSource` via JNDI) in production; keep DB code in a DAO class.

---

## 9. Session Tracking

**Problem:** HTTP is **stateless** — the server forgets the client after each request.
**Session tracking:** Maintaining a user's state/data across multiple requests (login, shopping cart).

### 9.1 Techniques
| # | Technique | Mechanism | Pros | Cons |
|---|---|---|---|---|
| 1 | **Cookies** | Small data stored in browser, sent with every request | Simple, persistent | Can be disabled; size limit; not secure |
| 2 | **URL Rewriting** | Append session id to URL (`page;jsessionid=ABC` or `?id=5`) via `response.encodeURL()` | Works when cookies are disabled | Ugly URLs, exposes data, only works with links |
| 3 | **Hidden Form Fields** | `<input type="hidden" name="uid" value="5">` | Browser independent, simple | Only for form submissions; visible in page source |
| 4 | **HttpSession** | Server-side object per user, identified by `JSESSIONID` | Secure, stores any object | Uses server memory; times out |

### 9.2 HttpSession API
```java
HttpSession session = request.getSession();          // get or create
HttpSession s2 = request.getSession(false);          // get existing, else null
session.setAttribute("user", "Amit");                // store
String u = (String) session.getAttribute("user");    // retrieve
session.removeAttribute("user");
session.setMaxInactiveInterval(600);                 // timeout in seconds
session.getId(); session.isNew(); session.getCreationTime();
session.invalidate();                                // destroy (logout)
```
### 9.3 Example (Login → Profile → Logout)
```java
// LoginServlet
String u = request.getParameter("uname");
request.getSession().setAttribute("user", u);
response.sendRedirect("profile");

// ProfileServlet
HttpSession s = request.getSession(false);
if (s == null || s.getAttribute("user") == null) {
    response.sendRedirect("login.html");
} else {
    response.getWriter().println("Welcome " + s.getAttribute("user"));
}

// LogoutServlet
request.getSession().invalidate();
response.sendRedirect("login.html");
```
**Hidden field example:**
```java
out.println("<form action='next' method='post'>");
out.println("<input type='hidden' name='uid' value='" + uid + "'>");
out.println("<input type='submit'></form>");
```
**URL rewriting example:**
```java
out.println("<a href='" + response.encodeURL("page2") + "'>Next</a>");
```

---

## 10. Creating Cookies

**Cookie:** Small piece of text data (name–value pair) created by the server, stored by the browser, and sent back with every subsequent request to that site. Class: `javax.servlet.http.Cookie`.

### 10.1 Types
- **Session (non-persistent) cookie:** deleted when browser closes (no max-age).
- **Persistent cookie:** stored on disk until expiry (`setMaxAge(seconds)`).

### 10.2 Cookie Methods
| Method | Purpose |
|---|---|
| `new Cookie(name, value)` | Create |
| `getName()`, `getValue()`, `setValue()` | Read/modify |
| `setMaxAge(int sec)` | Lifetime (`0` = delete, negative = until browser closes) |
| `setPath()`, `setDomain()` | Scope |
| `setSecure(true)` | Send only over HTTPS |
| `setHttpOnly(true)` | Hidden from JavaScript |

### 10.3 Steps
1. **Create** cookie object → 2. **Set** max age/attributes → 3. **Add** to response (`response.addCookie(c)`, *before* writing output) → 4. **Read** later using `request.getCookies()`.

### 10.4 Example
```java
// SetCookieServlet
@WebServlet("/setcookie")
public class SetCookieServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        Cookie c = new Cookie("username", "Amit");
        c.setMaxAge(60 * 60);              // 1 hour
        res.addCookie(c);                  // before getWriter()
        res.getWriter().println("Cookie created");
    }
}

// GetCookieServlet
@WebServlet("/getcookie")
public class GetCookieServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        PrintWriter out = res.getWriter();
        Cookie[] cs = req.getCookies();    // may be null
        if (cs != null)
            for (Cookie c : cs)
                out.println(c.getName() + " = " + c.getValue());
        else out.println("No cookies");
    }
}
```
**Delete a cookie:** `Cookie c = new Cookie("username", ""); c.setMaxAge(0); res.addCookie(c);`

**Limits:** ~4 KB per cookie, ~20 per site (browser dependent); user can disable; don't store passwords/sensitive data.

---

## 11. RMI (Remote Method Invocation)

**RMI:** A Java API (`java.rmi`) that lets an object in one JVM **invoke methods on an object in another JVM** (same or different machine) as if it were local. Used to build **distributed applications** in pure Java.

### 11.1 Features
- **Object-oriented** remote calls (pass Java objects, not just data)
- **Location transparency** – remote call looks like a local call
- **Pure Java**, platform independent (needs JVM both sides)
- Automatic **marshalling/unmarshalling** of parameters via Java **serialization**
- **Dynamic class loading**; built-in **RMI Registry** for lookup
- Supports **security** (security manager/policy), **distributed garbage collection**
- Uses **JRMP** (Java Remote Method Protocol) over TCP/IP (or RMI-IIOP for CORBA)
- *Limitation:* Java-to-Java only; slower than local calls

### 11.2 RMI Structure (Architecture)
```
CLIENT JVM                                        SERVER JVM
Client App                                         Remote Object (Impl)
   ↓                                                   ↑
 Stub  (proxy of remote object)                    Skeleton (older versions)
   ↓                                                   ↑
Remote Reference Layer (RRL)  ←────────────→  Remote Reference Layer
   ↓                                                   ↑
Transport Layer (TCP/IP, JRMP)  ←────────────→ Transport Layer
                       ↘                         ↙
                       RMI Registry (naming service, port 1099)
```
| Component | Role |
|---|---|
| **Remote interface** | Extends `java.rmi.Remote`; declares methods callable remotely; each method `throws RemoteException` |
| **Remote object (Impl)** | Server-side class implementing the interface (`UnicastRemoteObject`) |
| **Stub** | Client-side **proxy**; marshals parameters, forwards call, unmarshals result |
| **Skeleton** | Server-side counterpart: unmarshals, calls the real method, marshals result *(not needed since Java 1.2; stubs are generated dynamically)* |
| **RRL** | Manages references and invocation semantics |
| **Transport layer** | Network connection between JVMs (TCP) |
| **RMI Registry** | Naming service: server `bind()`s an object under a name; client `lookup()`s it |

### 11.3 Marshalling and Unmarshalling
- **Marshalling:** Converting method **arguments / return values / exceptions** into a **byte stream** (serialization) to send over the network. Done by the *stub* (client) for arguments and by the *skeleton/server runtime* for results.
- **Unmarshalling:** Reconstructing the original objects from the byte stream at the receiving end.
- Primitives are copied by value; **serializable objects** are passed by **copy**; **remote objects** are passed by **reference** (as a stub).
- Objects passed must implement `java.io.Serializable`.

```
Client: args → [marshal] → bytes ══network══▶ bytes → [unmarshal] → args : Server
Client: result ← [unmarshal] ← bytes ◀══network══ bytes ← [marshal] ← result : Server
```

### 11.4 RMI Application (Steps)
1. **Define remote interface** (extends `Remote`, methods throw `RemoteException`)
2. **Implement** the interface (remote object)
3. **Write server** – create object, export, bind to registry
4. **Write client** – lookup in registry, invoke methods
5. **Compile** all (`javac *.java`)
6. **Start registry** – `rmiregistry` (port 1099) *(or `LocateRegistry.createRegistry(1099)` in server)*
7. **Run server**, then **run client**

> Stubs are auto-generated at runtime (Java 5+); the `rmic` tool is not required for modern Java.

**Example: Calculator**

*Calculator.java* (remote interface)
```java
import java.rmi.*;
public interface Calculator extends Remote {
    int add(int a, int b) throws RemoteException;
}
```
*CalculatorImpl.java* (remote object)
```java
import java.rmi.*;
import java.rmi.server.UnicastRemoteObject;
public class CalculatorImpl extends UnicastRemoteObject implements Calculator {
    public CalculatorImpl() throws RemoteException { super(); }
    public int add(int a, int b) throws RemoteException { return a + b; }
}
```
*Server.java*
```java
import java.rmi.registry.*;
public class Server {
    public static void main(String[] args) {
        try {
            Registry reg = LocateRegistry.createRegistry(1099);
            reg.rebind("CalcService", new CalculatorImpl());
            System.out.println("Server ready");
        } catch (Exception e) { e.printStackTrace(); }
    }
}
```
*Client.java*
```java
import java.rmi.registry.*;
public class Client {
    public static void main(String[] args) {
        try {
            Registry reg = LocateRegistry.getRegistry("localhost", 1099);
            Calculator c = (Calculator) reg.lookup("CalcService");
            System.out.println("Sum = " + c.add(10, 20));
        } catch (Exception e) { e.printStackTrace(); }
    }
}
```
**Run:** `javac *.java` → `java Server` (new terminal) → `java Client` → **Output:** `Sum = 30`

### 11.5 RMI Database Program
The **server** accesses the DB through JDBC; the **client** calls remote methods without knowing DB details.

*StudentService.java*
```java
import java.rmi.*;
public interface StudentService extends Remote {
    String addStudent(int id, String name, int marks) throws RemoteException;
    String getStudent(int id) throws RemoteException;
}
```
*StudentServiceImpl.java*
```java
import java.rmi.*;
import java.rmi.server.UnicastRemoteObject;
import java.sql.*;

public class StudentServiceImpl extends UnicastRemoteObject implements StudentService {
    private static final String URL = "jdbc:mysql://localhost:3306/college";
    public StudentServiceImpl() throws RemoteException { super(); }

    public String addStudent(int id, String name, int marks) throws RemoteException {
        try (Connection con = DriverManager.getConnection(URL, "root", "root");
             PreparedStatement ps = con.prepareStatement(
                 "INSERT INTO student(id,name,marks) VALUES(?,?,?)")) {
            ps.setInt(1, id); ps.setString(2, name); ps.setInt(3, marks);
            return ps.executeUpdate() + " record inserted";
        } catch (SQLException e) { throw new RemoteException("DB error", e); }
    }

    public String getStudent(int id) throws RemoteException {
        try (Connection con = DriverManager.getConnection(URL, "root", "root");
             PreparedStatement ps = con.prepareStatement("SELECT * FROM student WHERE id=?")) {
            ps.setInt(1, id);
            try (ResultSet rs = ps.executeQuery()) {
                return rs.next() ? rs.getInt("id") + " " + rs.getString("name") + " " + rs.getInt("marks")
                                 : "Not found";
            }
        } catch (SQLException e) { throw new RemoteException("DB error", e); }
    }
}
```
*DBServer.java*
```java
import java.rmi.registry.*;
public class DBServer {
    public static void main(String[] args) throws Exception {
        Registry reg = LocateRegistry.createRegistry(1099);
        reg.rebind("StudentService", new StudentServiceImpl());
        System.out.println("DB Server ready");
    }
}
```
*DBClient.java*
```java
import java.rmi.registry.*;
public class DBClient {
    public static void main(String[] args) throws Exception {
        Registry reg = LocateRegistry.getRegistry("localhost", 1099);
        StudentService s = (StudentService) reg.lookup("StudentService");
        System.out.println(s.addStudent(101, "Amit", 85));
        System.out.println(s.getStudent(101));
    }
}
```
> Requirements: MySQL JDBC JAR on the **server's classpath**; table `student(id, name, marks)`; client needs only the `StudentService` interface.

---

## 12. Quick Revision

- **Servlet** = server-side Java class; runs in a container; one instance, many threads.
- **Life cycle:** `init()` (once) → `service()`→`doGet/doPost` (every request) → `destroy()` (once).
- **Types:** `GenericServlet` (protocol-independent), `HttpServlet` (HTTP). Packages: `javax.servlet`, `javax.servlet.http`.
- **Servlet vs JSP:** servlet = logic (Java with HTML); JSP = view (HTML with Java); JSP → translated into servlet.
- **Mapping:** `web.xml` or `@WebServlet`.
- **Session tracking:** Cookies, URL rewriting, Hidden fields, `HttpSession`.
- **Cookie:** `new Cookie` → `setMaxAge` → `response.addCookie` → `request.getCookies`.
- **`sendRedirect`** = client-side, URL changes; **`forward`** = server-side, same request.
- **RMI:** call methods of remote JVM objects; **Stub** (client proxy) + **Skeleton** (server) + **RRL** + **Transport** + **Registry** (port 1099).
- **Marshalling** = objects → byte stream; **Unmarshalling** = byte stream → objects (Serializable required).
- Remote interface: extends `Remote`, methods `throws RemoteException`; impl extends `UnicastRemoteObject`.
- RMI steps: interface → impl → server (`bind/rebind`) → client (`lookup`) → registry → run.

### Likely Exam Questions
1. What is a servlet? Explain its features. Differentiate servlet and JSP / servlet and CGI.
2. Explain the servlet life cycle with a diagram and example.
3. Explain types of servlets (`GenericServlet` vs `HttpServlet`).
4. Explain the Servlet API and important packages/interfaces.
5. Write a servlet to display dynamic content / handle a form.
6. Write a servlet for database access (insert / select).
7. What is session tracking? Explain its techniques with examples.
8. How do you create and read cookies in a servlet? Write a program.
9. What is RMI? Explain its architecture and features.
10. Explain marshalling and unmarshalling.
11. Write an RMI application (e.g., calculator) with server and client.
12. Write an RMI program with database connectivity.

---
*Reference books (per syllabus): Ivan Bayross – Web Technologies Pt.1 & 2 (BPB); Steven Holzner – Java 8 Programming Black Book (Dreamtech); Eric Jendrock – Java EE 6 Basic Concepts (Pearson).*
