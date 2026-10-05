# Advanced Java (MCC-2.3.1) — Unit II Notes
**PG MCA, Ravenshaw University (CBCS) | Semester III**
**Unit II:** JDBC · Applet · AWT · Swing (MVC) · Enterprise Java · JSP

---

## Table of Contents
1. [JDBC](#1-jdbc)
2. [JDBC Programs: Insert, Update, Delete, Select](#2-jdbc-programs-insert-update-delete-select)
3. [JDBC in an Application Program](#3-jdbc-use-in-an-application-program)
4. [Applet](#4-applet)
5. [AWT](#5-awt-abstract-window-toolkit)
6. [Swing and MVC](#6-swing-and-mvc)
7. [Enterprise Java Programming](#7-enterprise-java-programming)
8. [JSP](#8-jsp-javaserver-pages)
9. [Quick Revision](#9-quick-revision)

---

## 1. JDBC

**JDBC (Java Database Connectivity):** A standard Java API (package `java.sql`, `javax.sql`) to connect to relational databases, run SQL, and process results. Database-independent: the same code works with any DB through a **driver**.

### 1.1 JDBC Architecture
```
Java Application → JDBC API → JDBC Driver Manager → JDBC Driver → Database
```

### 1.2 JDBC Driver Types
| Type | Name | How it works | Notes |
|---|---|---|---|
| 1 | JDBC–ODBC Bridge | Translates JDBC calls to ODBC | Needs ODBC; slow; **removed in Java 8** |
| 2 | Native-API (partly Java) | Calls DB's native client library | Needs native library on client |
| 3 | Network Protocol (Middleware) | Calls go to a middleware server, which talks to DB | Pure Java client |
| 4 | Thin / Native-Protocol | Pure Java; talks directly to DB protocol | **Most used** (MySQL, Oracle thin) |

### 1.3 Core Classes / Interfaces
| Name | Purpose |
|---|---|
| `DriverManager` | Manages drivers; creates connections |
| `Connection` | Session with the database |
| `Statement` | Executes static SQL |
| `PreparedStatement` | Precompiled SQL with `?` parameters (faster, **prevents SQL injection**) |
| `CallableStatement` | Calls stored procedures |
| `ResultSet` | Table of data returned by a SELECT |
| `ResultSetMetaData`, `DatabaseMetaData` | Info about columns / the database |
| `SQLException` | Database error |

### 1.4 Steps to Connect (5 Steps)
1. **Load/register driver** – `Class.forName("com.mysql.cj.jdbc.Driver");` *(optional in JDBC 4+, auto-loaded from classpath)*
2. **Create connection** – `DriverManager.getConnection(url, user, pwd);`
3. **Create statement** – `con.createStatement()` / `con.prepareStatement(sql)`
4. **Execute query** – `executeQuery()` (SELECT) / `executeUpdate()` (INSERT, UPDATE, DELETE)
5. **Process result & close** – iterate `ResultSet`; close `ResultSet`, `Statement`, `Connection`

**Connection URL formats**
```
MySQL  : jdbc:mysql://localhost:3306/dbname
Oracle : jdbc:oracle:thin:@localhost:1521:xe
```

### 1.5 Statement Execution Methods
| Method | Used for | Returns |
|---|---|---|
| `executeQuery(sql)` | SELECT | `ResultSet` |
| `executeUpdate(sql)` | INSERT / UPDATE / DELETE / DDL | `int` (rows affected) |
| `execute(sql)` | Any SQL | `boolean` |

---

## 2. JDBC Programs: Insert, Update, Delete, Select

Sample table (MySQL):
```sql
CREATE TABLE student(id INT PRIMARY KEY, name VARCHAR(50), marks INT);
```

### 2.1 INSERT
```java
import java.sql.*;
public class InsertDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/college";
        try (Connection con = DriverManager.getConnection(url, "root", "root");
             PreparedStatement ps = con.prepareStatement(
                 "INSERT INTO student(id, name, marks) VALUES (?,?,?)")) {
            ps.setInt(1, 101);
            ps.setString(2, "Amit");
            ps.setInt(3, 85);
            int n = ps.executeUpdate();
            System.out.println(n + " row inserted");
        } catch (SQLException e) { e.printStackTrace(); }
    }
}
```

### 2.2 UPDATE
```java
PreparedStatement ps = con.prepareStatement("UPDATE student SET marks=? WHERE id=?");
ps.setInt(1, 90);
ps.setInt(2, 101);
System.out.println(ps.executeUpdate() + " row updated");
```

### 2.3 DELETE
```java
PreparedStatement ps = con.prepareStatement("DELETE FROM student WHERE id=?");
ps.setInt(1, 101);
System.out.println(ps.executeUpdate() + " row deleted");
```

### 2.4 SELECT
```java
Statement st = con.createStatement();
ResultSet rs = st.executeQuery("SELECT * FROM student");
while (rs.next()) {
    System.out.println(rs.getInt("id") + " " + rs.getString("name") + " " + rs.getInt("marks"));
}
rs.close(); st.close(); con.close();
```
- `rs.next()` moves the cursor to the next row (returns `false` at end).
- Getters: `getInt`, `getString`, `getDouble`, `getDate` (by column name or index starting at **1**).

> **try-with-resources** (used above) auto-closes `Connection`/`Statement`/`ResultSet`.

### 2.5 Transactions (extra, commonly asked)
```java
con.setAutoCommit(false);
try { /* several updates */ con.commit(); }
catch (SQLException e) { con.rollback(); }
```

---

## 3. JDBC Use in an Application Program

Typical practice: separate DB logic into a reusable class (**DAO – Data Access Object**).

```java
// DBConnection.java – reusable connection utility
public class DBConnection {
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/college", "root", "root");
    }
}

// StudentDAO.java
public class StudentDAO {
    public void add(int id, String name, int marks) throws SQLException {
        try (Connection con = DBConnection.getConnection();
             PreparedStatement ps = con.prepareStatement("INSERT INTO student VALUES(?,?,?)")) {
            ps.setInt(1, id); ps.setString(2, name); ps.setInt(3, marks);
            ps.executeUpdate();
        }
    }
    public List<String> all() throws SQLException {
        List<String> list = new ArrayList<>();
        try (Connection con = DBConnection.getConnection();
             ResultSet rs = con.createStatement().executeQuery("SELECT name FROM student")) {
            while (rs.next()) list.add(rs.getString(1));
        }
        return list;
    }
}
```
**Best practices:** use `PreparedStatement`; always close resources; handle `SQLException`; don't hard-code credentials; use **connection pooling** in real web apps.

---

## 4. Applet

**Applet:** A small Java program embedded in an HTML page, downloaded from a server and run **inside a browser (applet viewer)** in a restricted sandbox. Extends `java.applet.Applet` (AWT) or `javax.swing.JApplet` (Swing).

> ⚠ **Note:** Applets are **deprecated** (since Java 9; deprecated for removal from Java 17) and no longer supported by modern browsers. Still part of the syllabus.

### 4.1 Life Cycle
```
 init() → start() → [running/paint()] → stop() → destroy()
              ↑___________________________|  (start ↔ stop repeat when page revisited)
```
| Method | When called |
|---|---|
| `init()` | **Once**, when applet is first loaded; initialise variables/GUI |
| `start()` | After `init()` and **each time** the page becomes visible again |
| `paint(Graphics g)` | Whenever the display needs drawing/redrawing (from `java.awt.Component`/Container) |
| `stop()` | When user leaves the page; suspend threads/animation |
| `destroy()` | **Once**, when applet is unloaded; release resources |

### 4.2 Example
```java
import java.applet.Applet;
import java.awt.Graphics;

public class HelloApplet extends Applet {
    public void paint(Graphics g) {
        g.drawString("Hello Applet", 50, 50);
    }
}
```
**HTML to run it**
```html
<applet code="HelloApplet.class" width="300" height="200"></applet>
```
**Run:** `javac HelloApplet.java` → `appletviewer HelloApplet.html`
*(Alternatively add `/* <applet code="HelloApplet" width=300 height=200></applet> */` in the .java file and run `appletviewer HelloApplet.java`.)*

### 4.3 Applet vs Application
| Applet | Application |
|---|---|
| Runs in browser/appletviewer | Runs standalone with `java` |
| No `main()`; uses life-cycle methods | Starts at `main()` |
| Restricted (no local file access, limited network) | Full system access |
| Extends `Applet`/`JApplet` | Any class |

### 4.4 Passing Parameters
```html
<applet code="P.class" width="200" height="100">
  <param name="user" value="Amit">
</applet>
```
```java
String u = getParameter("user");
```

---

## 5. AWT (Abstract Window Toolkit)

**AWT:** Java's original GUI API (package `java.awt`). Components are **heavyweight** and **platform-dependent** — they use native OS peer widgets, so look differs per OS.

### 5.1 Hierarchy
`Object → Component → Container → Window → Frame / Dialog` ; `Container → Panel → Applet`

### 5.2 Common Components
`Label`, `Button`, `TextField`, `TextArea`, `Checkbox`, `CheckboxGroup` (radio), `Choice` (dropdown), `List`, `Scrollbar`, `Menu`, `MenuBar`, `Canvas`.

**Containers:** `Frame` (top-level window), `Panel` (grouping), `Dialog`, `Window`.

### 5.3 Layout Managers
| Layout | Behaviour |
|---|---|
| `FlowLayout` (default for Panel/Applet) | Left→right, wraps rows |
| `BorderLayout` (default for Frame) | NORTH, SOUTH, EAST, WEST, CENTER |
| `GridLayout` | Equal-size grid cells |
| `CardLayout` | Stack of cards, one visible |
| `GridBagLayout` | Flexible grid |

### 5.4 Event Handling (Delegation Event Model)
**Event source** (button) → generates **event object** → sent to registered **listener** → handler method runs.

| Listener | Event | Method(s) |
|---|---|---|
| `ActionListener` | Button click, Enter | `actionPerformed()` |
| `ItemListener` | Checkbox/Choice | `itemStateChanged()` |
| `MouseListener` | Mouse | `mouseClicked/Pressed/Released/Entered/Exited` |
| `KeyListener` | Keyboard | `keyPressed/Released/Typed` |
| `WindowListener` | Window | `windowClosing()` etc. |

### 5.5 Example
```java
import java.awt.*;
import java.awt.event.*;

public class AwtDemo extends Frame implements ActionListener {
    TextField tf = new TextField(15);
    Button b = new Button("Show");
    Label l = new Label("            ");
    AwtDemo() {
        setLayout(new FlowLayout());
        add(tf); add(b); add(l);
        b.addActionListener(this);
        addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) { dispose(); }
        });
        setSize(300, 150);
        setVisible(true);
    }
    public void actionPerformed(ActionEvent e) { l.setText("Hello " + tf.getText()); }
    public static void main(String[] a) { new AwtDemo(); }
}
```

---

## 6. Swing and MVC

**Swing:** Part of **JFC (Java Foundation Classes)**, package `javax.swing`. Components are **lightweight** (written in pure Java, drawn by Java itself) and **platform-independent**, with a pluggable look-and-feel.

### 6.1 AWT vs Swing
| AWT | Swing |
|---|---|
| Heavyweight (native peers) | Lightweight (pure Java) |
| Platform-dependent look | Platform-independent, pluggable L&F |
| Fewer components | Rich set (`JTable`, `JTree`, `JTabbedPane`…) |
| Does not follow MVC | Follows (modified) MVC |
| `java.awt` | `javax.swing` (built on AWT) |
| Faster/legacy | Slightly heavier but more flexible |

### 6.2 Swing Components (prefix **J**)
`JFrame`, `JPanel`, `JLabel`, `JButton`, `JTextField`, `JPasswordField`, `JTextArea`, `JCheckBox`, `JRadioButton` (+`ButtonGroup`), `JComboBox`, `JList`, `JTable`, `JTree`, `JMenuBar/JMenu/JMenuItem`, `JScrollPane`, `JTabbedPane`, `JOptionPane`, `JDialog`, `JApplet`.
Add components to the frame's **content pane**: `frame.add(c)` (or `getContentPane().add(c)`).

### 6.3 Example
```java
import javax.swing.*;
import java.awt.event.*;

public class SwingDemo {
    public static void main(String[] args) {
        JFrame f = new JFrame("Swing Demo");
        JTextField tf = new JTextField(15);
        JButton b = new JButton("Greet");
        JLabel l = new JLabel();
        f.setLayout(new java.awt.FlowLayout());
        f.add(tf); f.add(b); f.add(l);
        b.addActionListener(e -> l.setText("Hello " + tf.getText()));
        f.setSize(300, 150);
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        f.setVisible(true);
    }
}
```
> Best practice: create GUI on the Event Dispatch Thread → `SwingUtilities.invokeLater(() -> ...)`.

### 6.4 MVC (Model–View–Controller)
Design pattern that splits an application into three parts:

| Part | Responsibility | Swing example |
|---|---|---|
| **Model** | Data/state + business logic | `ButtonModel`, `ListModel`, `TableModel` |
| **View** | What the user sees (presentation) | Component's drawn appearance |
| **Controller** | Takes user input, updates model/view | Event handling (listeners) |

**Flow:** User acts → *Controller* handles → updates *Model* → Model notifies → *View* refreshes.

**In Swing (Model-Delegate / separable model architecture):** the **View and Controller are combined into a "UI delegate"** (`javax.swing.plaf`), while the **Model** stays separate. This is why Swing is called a *loosely/modified* MVC.

**Benefits:** separation of concerns, multiple views for the same data, easier maintenance and testing, pluggable look-and-feel.

---

## 7. Enterprise Java Programming

**Enterprise Java (Java EE → now Jakarta EE):** A set of specifications for building **large, distributed, multi-tier, scalable and secure** enterprise/web applications.

### 7.1 Multi-tier Architecture
```
Client Tier (browser/GUI)  →  Web Tier (Servlet, JSP, JSF)
      →  Business Tier (EJB, Spring)  →  EIS/Data Tier (DB via JDBC/JPA)
```

### 7.2 Key Technologies
| Technology | Purpose |
|---|---|
| Servlet | Server-side request handling |
| JSP | Dynamic web pages (view layer) |
| EJB | Business logic components |
| JDBC / JPA | Database access / ORM |
| JMS | Messaging |
| JNDI | Naming & directory lookup |
| JTA | Transactions |
| JavaMail | Email |
| JSF, JSTL | UI framework, standard tag library |
| RMI, Web Services (JAX-RS/JAX-WS) | Remote communication |

### 7.3 Web Container / Server
A **web container** (Tomcat, Jetty) runs Servlets & JSP; an **application server** (WildFly, GlassFish, WebLogic) additionally provides EJB, JMS etc.

> **Note:** Since Jakarta EE 9, packages moved from `javax.*` to `jakarta.*` (e.g., `jakarta.servlet`). The syllabus books use `javax.*`.

---

## 8. JSP (JavaServer Pages)

**JSP:** Server-side technology for creating **dynamic web pages** by embedding Java code in HTML using special tags. A JSP is **translated into a Servlet** by the container, then compiled and executed.

### 8.1 Features
- Easy to write — HTML + Java tags; separates presentation from logic
- Translated to and runs as a servlet (so all servlet features available)
- Compiled **only on first request** (or when modified) → fast afterwards
- Platform independent; access to all Java APIs
- Supports implicit objects, custom tags, JSTL, Expression Language (EL), JavaBeans
- Built-in session, error handling and database access

**Servlet vs JSP (brief):** Servlet = Java code with HTML inside `out.println()` (good for logic/controller); JSP = HTML with Java inside (good for view).

### 8.2 Syntax Structure (Basic Tags)
| Element | Syntax |
|---|---|
| Scriptlet | `<% java code %>` |
| Expression | `<%= expression %>` |
| Declaration | `<%! declaration %>` |
| Directive | `<%@ directive attribute="value" %>` |
| Comment | `<%-- JSP comment --%>` |
| Action | `<jsp:include page="a.jsp" />` |
| EL | `${expression}` |

**First JSP (`hello.jsp`)**
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" %>
<html>
<body>
  <h2>Hello JSP</h2>
  <% String name = "Amit"; %>
  Welcome, <%= name %> <br>
  Time: <%= new java.util.Date() %>
</body>
</html>
```
Place in the web app folder (Tomcat: `webapps/myapp/hello.jsp`) → open `http://localhost:8080/myapp/hello.jsp`.

### 8.3 JSP Life Cycle
```
Translation (.jsp → .java) → Compilation (.java → .class) → Class loading & instantiation
   → jspInit() → _jspService() [per request] → jspDestroy()
```
| Phase | Method / Action |
|---|---|
| 1. **Translation** | Container checks syntax, converts JSP to servlet source (`hello_jsp.java`) |
| 2. **Compilation** | Servlet source compiled to `.class` |
| 3. **Loading & instantiation** | Class loaded, object created |
| 4. **Initialization** | `jspInit()` — called **once** |
| 5. **Request processing** | `_jspService(request, response)` — called **per request** |
| 6. **Destruction** | `jspDestroy()` — called **once** before removal |

- You may **override** `jspInit()` and `jspDestroy()` (via declarations); you **cannot override** `_jspService()` (generated by container).
- Translation + compilation happen on first request or when the JSP changes.

### 8.4 Dynamic Web Page Creation by JSP
Static HTML + Java produces output that changes per request.

**Form page `login.html`**
```html
<form action="welcome.jsp" method="post">
  Name: <input type="text" name="uname">
  <input type="submit" value="Go">
</form>
```
**`welcome.jsp`**
```jsp
<% String n = request.getParameter("uname"); %>
<h3>Hello, <%= n %>!</h3>
<% for (int i = 1; i <= 3; i++) { %>
    <p>Line <%= i %></p>
<% } %>
```

### 8.5 Anatomy of a JSP Page
A JSP page is made of:
1. **Template text** – static HTML/XML sent as-is
2. **Directives** – page-level instructions (`<%@ … %>`)
3. **Scripting elements** – declarations, scriptlets, expressions
4. **Standard actions** – `<jsp:useBean>`, `<jsp:include>`, `<jsp:forward>`
5. **Implicit objects** – built-in objects (request, session…)
6. **Comments**, **EL**, **custom tags / JSTL**

### 8.6 Scripting Elements
| Element | Syntax | Placement in generated servlet | Example |
|---|---|---|---|
| **Declaration** | `<%! … %>` | Class level (fields/methods) | `<%! int count = 0; %>` |
| **Scriptlet** | `<% … %>` | Inside `_jspService()` | `<% count++; %>` |
| **Expression** | `<%= … %>` | Argument to `out.print()` (no `;`) | `<%= count %>` |

```jsp
<%! int hits = 0;
    int square(int x) { return x * x; } %>
<% hits++; %>
Visits: <%= hits %> | 5² = <%= square(5) %>
```
Declared variables are **instance (shared across requests)**; scriptlet variables are **local (per request)**.

### 8.7 Implicit Objects (9)
Created automatically by the container; usable in scriptlets/expressions without declaration.

| Object | Type | Scope | Use |
|---|---|---|---|
| `request` | `HttpServletRequest` | request | Read form data, headers, cookies (`getParameter()`) |
| `response` | `HttpServletResponse` | page | Send output/redirect/cookies (`sendRedirect()`) |
| `out` | `JspWriter` | page | Write to output (`out.println()`) |
| `session` | `HttpSession` | session | Per-user data (`setAttribute/getAttribute`) |
| `application` | `ServletContext` | application | App-wide shared data |
| `config` | `ServletConfig` | page | Servlet init parameters |
| `pageContext` | `PageContext` | page | Access to all scopes & other objects |
| `page` | `Object` (= `this`) | page | Current servlet instance |
| `exception` | `Throwable` | page | Only in **error pages** (`isErrorPage="true"`) |

**Scopes:** page < request < session < application.

### 8.8 Directive Elements
Instruct the container during **translation**. Syntax: `<%@ directive attr="val" %>`

| Directive | Purpose | Example |
|---|---|---|
| **page** | Page-wide settings | `<%@ page import="java.util.*" contentType="text/html" errorPage="err.jsp" session="true" %>` |
| **include** | Include a file **statically at translation time** | `<%@ include file="header.jsp" %>` |
| **taglib** | Declare custom/JSTL tag library | `<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>` |

**Common `page` attributes:** `import`, `contentType`, `language`, `session`, `buffer`, `autoFlush`, `isThreadSafe`, `errorPage`, `isErrorPage`, `isELIgnored`, `pageEncoding`.

> Static include (`<%@ include %>`) = merged at translation. Dynamic include (`<jsp:include page="">`) = included at **request time**.

**Standard actions:** `<jsp:include>`, `<jsp:forward page="x.jsp"/>`, `<jsp:param>`, `<jsp:useBean>`, `<jsp:setProperty>`, `<jsp:getProperty>`.

### 8.9 MVC in JSP
JSP supports two design approaches:

| Model 1 | Model 2 (MVC) |
|---|---|
| JSP handles request, logic and view | **Servlet = Controller**, **JavaBean/DAO = Model**, **JSP = View** |
| Simple, small apps | Large, maintainable apps |
| Logic mixed with HTML | Clean separation |

**Model 2 flow**
```
Browser → Servlet (Controller) → Model (JavaBean/DAO/DB)
                 ↓ sets data in request
           RequestDispatcher.forward() → JSP (View) → Browser
```
```java
// Controller (Servlet)
String n = request.getParameter("uname");
Student s = new Student(n);                       // Model
request.setAttribute("student", s);
request.getRequestDispatcher("result.jsp").forward(request, response);
```
```jsp
<%-- View: result.jsp --%>
<% Student s = (Student) request.getAttribute("student"); %>
Hello <%= s.getName() %>      <%-- or ${student.name} --%>
```

### 8.10 Session Tracking
HTTP is **stateless** — each request is independent. **Session tracking** maintains a user's state across multiple requests.

| Technique | How | Pros / Cons |
|---|---|---|
| **Cookies** | Small text data stored in browser | Persistent; fails if cookies disabled |
| **URL Rewriting** | Append session id to URL (`response.encodeURL(url)`) → `page.jsp;jsessionid=…` | Works without cookies; ugly/insecure URLs |
| **Hidden Form Fields** | `<input type="hidden" name="id" value="5">` | Simple; only for form submits; visible in page source |
| **HttpSession** | Server-side object, tracked via cookie/URL id (JSESSIONID) | Most used and secure |

```jsp
<%-- page1.jsp --%>
<% session.setAttribute("user", "Amit");
   session.setMaxInactiveInterval(600);     // seconds %>

<%-- page2.jsp --%>
Welcome <%= session.getAttribute("user") %>

<%-- logout.jsp --%>
<% session.invalidate(); %>

<%-- Cookie --%>
<% Cookie c = new Cookie("uname", "Amit");
   c.setMaxAge(3600);
   response.addCookie(c); %>
```
Useful `HttpSession` methods: `getId()`, `setAttribute()`, `getAttribute()`, `removeAttribute()`, `invalidate()`, `isNew()`. Disable with `<%@ page session="false" %>`.

### 8.11 JSP Database Access
Use JDBC inside JSP (fine for learning; in real projects put it in DAO/Servlet).

**Insert**
```jsp
<%@ page import="java.sql.*" %>
<%
  String id = request.getParameter("id");
  String name = request.getParameter("name");
  try {
      Class.forName("com.mysql.cj.jdbc.Driver");
      Connection con = DriverManager.getConnection(
          "jdbc:mysql://localhost:3306/college", "root", "root");
      PreparedStatement ps = con.prepareStatement("INSERT INTO student(id,name) VALUES(?,?)");
      ps.setInt(1, Integer.parseInt(id));
      ps.setString(2, name);
      int n = ps.executeUpdate();
      out.println(n + " record saved");
      con.close();
  } catch (Exception e) { out.println(e); }
%>
```
**Select (display table)**
```jsp
<%@ page import="java.sql.*" %>
<table border="1">
<%
  Class.forName("com.mysql.cj.jdbc.Driver");
  Connection con = DriverManager.getConnection(
      "jdbc:mysql://localhost:3306/college", "root", "root");
  ResultSet rs = con.createStatement().executeQuery("SELECT * FROM student");
  while (rs.next()) {
%>
  <tr><td><%= rs.getInt("id") %></td><td><%= rs.getString("name") %></td></tr>
<% } con.close(); %>
</table>
```
> Place the DB driver JAR (e.g., `mysql-connector-j.jar`) in `WEB-INF/lib`. Update/Delete work the same way using `UPDATE`/`DELETE` SQL with `executeUpdate()`.

### 8.12 JSP Exceptions
Ways to handle run-time errors in JSP:

**1. try–catch inside scriptlet**
```jsp
<% try { int x = 10 / 0; }
   catch (ArithmeticException e) { out.println("Error: " + e.getMessage()); } %>
```
**2. Page-level error page (`errorPage` / `isErrorPage`)** — recommended
```jsp
<%-- main.jsp --%>
<%@ page errorPage="error.jsp" %>
<% int x = 10 / 0; %>

<%-- error.jsp --%>
<%@ page isErrorPage="true" %>
<h3>Oops! <%= exception.getMessage() %></h3>
```
**3. Application-wide in `web.xml`**
```xml
<error-page>
  <exception-type>java.lang.Exception</exception-type>
  <location>/error.jsp</location>
</error-page>
<error-page>
  <error-code>404</error-code>
  <location>/notfound.jsp</location>
</error-page>
```
- The `exception` implicit object is available **only** when `isErrorPage="true"`.

---

## 9. Quick Revision

- **JDBC steps:** load driver → connect → create statement → execute → process `ResultSet` → close.
- `executeQuery` → SELECT; `executeUpdate` → INSERT/UPDATE/DELETE. **Type 4** = pure Java, most used.
- **Applet life cycle:** `init → start → paint → stop → destroy` (`init`/`destroy` once). Applets are deprecated.
- **AWT:** heavyweight, platform-dependent. **Swing:** lightweight, pure Java, MVC (View+Controller merged into UI delegate).
- **MVC:** Model = data, View = display, Controller = input handling.
- **JSP life cycle:** translate → compile → `jspInit()` → `_jspService()` → `jspDestroy()`.
- **Scripting:** `<%! %>` declaration, `<% %>` scriptlet, `<%= %>` expression.
- **9 implicit objects:** request, response, out, session, application, config, pageContext, page, exception.
- **3 directives:** page, include, taglib.
- **Session tracking:** Cookies, URL rewriting, Hidden fields, HttpSession.
- **JSP MVC (Model 2):** Servlet (controller) + JavaBean (model) + JSP (view).

### Likely Exam Questions
1. Explain JDBC architecture and the types of JDBC drivers.
2. Write a JDBC program to insert / update / delete / display records.
3. Explain the life cycle of an applet with an example.
4. Differentiate AWT and Swing. Explain MVC in Swing.
5. What is JSP? Explain its features and life cycle.
6. Explain JSP scripting elements with examples.
7. List and explain JSP implicit objects.
8. Explain JSP directives (page, include, taglib).
9. Explain MVC (Model 2) architecture in JSP.
10. Explain session tracking techniques in JSP.
11. How are exceptions handled in JSP? (`errorPage` / `isErrorPage`)
12. Write a JSP page that accesses a database and displays data.

---
*Reference books (per syllabus): Ivan Bayross – Web Technologies Pt.1 & 2 (BPB); Steven Holzner – Java 8 Programming Black Book (Dreamtech); Eric Jendrock – Java EE 6 Basic Concepts (Pearson).*
