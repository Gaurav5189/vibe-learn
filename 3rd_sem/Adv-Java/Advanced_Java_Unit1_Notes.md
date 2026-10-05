# Advanced Java (MCC-2.3.1) — Unit I Notes
**PG MCA, Ravenshaw University (CBCS) | Semester III**
**Unit I:** Web fundamentals · HTML · DHTML · JavaScript · CSS · XHTML · XML

---

## Table of Contents
1. [Network Basics](#1-network-basics)
2. [Web Terminology](#2-web-terminology)
3. [Web Programming](#3-web-programming)
4. [HTML](#4-html)
5. [DHTML](#5-dhtml)
6. [JavaScript](#6-javascript)
7. [Writing & Running HTML/DHTML Programs](#7-process-of-writing-and-running-html--dhtml-programs)
8. [CSS](#8-css)
9. [XHTML & XML](#9-xhtml-and-xml)
10. [Quick Revision](#10-quick-revision)

---

## 1. Network Basics

**Network:** Two or more computers/devices connected to share data and resources.

### Types of Networks
| Type | Full Form | Coverage | Example |
|---|---|---|---|
| PAN | Personal Area Network | A few metres | Bluetooth phone↔laptop |
| LAN | Local Area Network | Room / building / campus | College lab |
| MAN | Metropolitan Area Network | City / large campus | Cable TV, city Wi-Fi |
| WAN | Wide Area Network | Country / continent | **Internet** (largest WAN) |

### Features of a Network
- **Resource sharing** – files, printers, internet
- **Communication** – email, chat, video calls
- **Centralised data** & backup
- **Scalability** – add devices easily
- **Cost saving** – shared hardware/software
- **Drawbacks:** security risk, setup/maintenance cost, dependence on the server

### Network Architectures
- **Client–Server:** dedicated server provides services; clients request them.
- **Peer-to-Peer (P2P):** every node acts as both client and server.

---

## 2. Web Terminology

| Term | Meaning |
|---|---|
| **Browser** | Software to request, retrieve and display web pages (Chrome, Firefox, Edge). |
| **Client** | Machine/program that *requests* a service (e.g., browser). |
| **Server** | Machine/program that *provides* resources/services (web server: Apache, Tomcat, Nginx). |
| **URL** | *Uniform Resource Locator* – the address of a resource on the web. |
| **Web page** | A single document (usually HTML) shown in a browser. |
| **Website** | Collection of related web pages under one domain. |

### URL Structure
```
https://www.example.com:8080/folder/page.html?id=5#top
 |        |               |    |                |    |
protocol  domain/host    port  path           query fragment
```

### Static vs Dynamic Web Page
| Static | Dynamic |
|---|---|
| Fixed content, plain HTML | Content generated at run time (JSP, Servlet, JS) |
| Same for all users | Can vary per user/request |

### How the Web Works (Request–Response)
`Browser (client)` → HTTP request → `Web Server` → HTTP response (HTML) → `Browser renders page`

---

## 3. Web Programming

Creating web applications in two parts:
- **Client-side** – runs in browser: HTML, CSS, JavaScript, DHTML, AJAX.
- **Server-side** – runs on server: Servlet, JSP, PHP, ASP.NET (covered in Units II–IV).

---

## 4. HTML

**HTML (HyperText Markup Language):** Standard markup language for creating web pages. Uses **tags** to describe structure; browser interprets them. Current version: **HTML5**.

### 4.1 Basic Structure
```html
<!DOCTYPE html>            <!-- declares HTML5 -->
<html lang="en">
  <head>                   <!-- metadata, not displayed -->
    <meta charset="UTF-8">
    <title>My Page</title>
  </head>
  <body>                   <!-- visible content -->
    <h1>Hello</h1>
  </body>
</html>
```
- **Tag:** `<p>` ; **Element:** `<p>Text</p>` ; **Attribute:** `<a href="...">` (name="value").
- Empty (void) tags: `<br>`, `<hr>`, `<img>`.

### 4.2 Basic HTML Commands (Tags)
| Purpose | Tag(s) |
|---|---|
| Paragraph / line break / rule | `<p>`, `<br>`, `<hr>` |
| Bold / italic / underline | `<b>` `<strong>`, `<i>` `<em>`, `<u>` |
| Sub / superscript | `<sub>`, `<sup>` |
| Preformatted | `<pre>` |
| Comment | `<!-- text -->` |
| Generic containers | `<div>` (block), `<span>` (inline) |

### 4.3 Heading
Six levels: `<h1>` (largest) … `<h6>` (smallest).
```html
<h1>Main Title</h1>
<h3>Sub heading</h3>
```

### 4.4 Lists
```html
<!-- Unordered (bullets) -->
<ul type="disc|circle|square"> <li>Java</li> <li>JSP</li> </ul>

<!-- Ordered (numbers) -->
<ol type="1|A|a|I|i" start="1"> <li>HTML</li> <li>CSS</li> </ol>

<!-- Definition list -->
<dl> <dt>HTML</dt> <dd>Markup language</dd> </dl>
```
Lists can be **nested** (a `<ul>` inside an `<li>`).

### 4.5 Table
```html
<table border="1">
  <caption>Marks</caption>
  <thead><tr><th>Name</th><th>Marks</th></tr></thead>
  <tbody>
    <tr><td>Amit</td><td>85</td></tr>
    <tr><td colspan="2">Merged cell</td></tr>
  </tbody>
</table>
```
- `<tr>` row, `<th>` header cell, `<td>` data cell.
- `colspan` / `rowspan` merge cells; `<thead> <tbody> <tfoot>` group sections.

### 4.6 Image
```html
<img src="logo.png" alt="Logo" width="200" height="100">
```
- `src` = path/URL, `alt` = alternate text (accessibility). Formats: JPG, PNG, GIF, SVG, WebP.

### 4.7 Hyperlink
```html
<a href="page2.html">Internal</a>
<a href="https://google.com" target="_blank">External (new tab)</a>
<a href="#section1">Jump to anchor</a>       <!-- target: <h2 id="section1"> -->
<a href="mailto:me@x.com">Email</a>
<a href="page.html"><img src="b.png" alt="btn"></a>   <!-- image link -->
```

### 4.8 Marquee
Scrolling text/image.
```html
<marquee behavior="scroll|slide|alternate" direction="left|right|up|down"
         scrollamount="5" loop="-1">Welcome!</marquee>
```
> ⚠ **Note:** `<marquee>` is **deprecated/non-standard in HTML5** (still works in browsers). Modern alternative: CSS animation. Syllabus includes it, so know both.

### 4.9 Form and Components
**Form:** collects user input and sends it to a server.
```html
<form action="submit.jsp" method="post">
  Name: <input type="text" name="uname" required><br>
  Password: <input type="password" name="pwd"><br>
  Gender:
    <input type="radio" name="g" value="M"> M
    <input type="radio" name="g" value="F"> F<br>
  Hobbies:
    <input type="checkbox" name="h" value="read"> Reading<br>
  Course:
    <select name="course">
      <option value="mca">MCA</option>
      <option value="mba">MBA</option>
    </select><br>
  Comments: <textarea name="c" rows="3" cols="20"></textarea><br>
  <input type="file" name="f"><br>
  <input type="hidden" name="id" value="101">
  <input type="submit" value="Send">
  <input type="reset"  value="Clear">
  <button type="button">Click</button>
</form>
```
| Attribute | Meaning |
|---|---|
| `action` | URL that processes the form |
| `method` | `GET` (data in URL, visible, limited) / `POST` (data in body, secure, large) |

**Components:** text, password, radio, checkbox, select/option, textarea, file, hidden, submit, reset, button. HTML5 adds `email`, `number`, `date`, `range`, `color`, `url`, `tel`.

### 4.10 Frames
Divide the browser window into multiple independent sections.
```html
<!-- Old method (obsolete in HTML5) -->
<frameset cols="30%,70%">
  <frame src="menu.html" name="left">
  <frame src="main.html" name="right">
</frameset>

<!-- Current method: inline frame -->
<iframe src="https://example.com" width="400" height="300"></iframe>
```
> `<frameset>/<frame>` are **obsolete in HTML5**; use `<iframe>` (valid).

### 4.11 Audio and Video (HTML5)
```html
<audio controls autoplay loop>
  <source src="song.mp3" type="audio/mpeg">
  <source src="song.ogg" type="audio/ogg">
  Browser does not support audio.
</audio>

<video width="320" height="240" controls poster="thumb.jpg">
  <source src="movie.mp4" type="video/mp4">
  <source src="movie.webm" type="video/webm">
  Browser does not support video.
</video>
```
Attributes: `controls`, `autoplay`, `loop`, `muted`, `preload`, `poster` (video). Multiple `<source>` give fallback formats.

---

## 5. DHTML

**DHTML (Dynamic HTML):** *Not a language* — a combination of technologies that make a web page **dynamic and interactive** on the client side without reloading.

**DHTML = HTML + CSS + JavaScript + DOM**

| Component | Role |
|---|---|
| HTML | Structure / content |
| CSS | Presentation / style |
| JavaScript | Behaviour / logic |
| **DOM** (Document Object Model) | Tree-like interface letting scripts access & change elements, attributes, styles |

### HTML vs DHTML
| HTML | DHTML |
|---|---|
| Static pages | Dynamic, interactive pages |
| Content fixed after load | Content/style change after load |
| Markup language | Combination of technologies |

### Uses
Drop-down menus, form validation, animations, show/hide content, mouseover effects, real-time updates.

### Example
```html
<p id="demo" style="color:blue">Hello</p>
<button onclick="
  document.getElementById('demo').style.color='red';
  document.getElementById('demo').innerHTML='Changed!';">
  Click
</button>
```

---

## 6. JavaScript

**JavaScript (JS):** Lightweight, interpreted, **client-side scripting language** (standard: ECMAScript) that adds behaviour to web pages. Case-sensitive, dynamically typed. *Not the same as Java.*

| Java | JavaScript |
|---|---|
| Compiled to bytecode, runs on JVM | Interpreted by the browser engine |
| Strongly typed, class-based OOP | Dynamically typed, prototype-based (ES6 adds `class`) |
| Standalone apps | Runs in web pages (and Node.js) |

### 6.1 Syntax Structure
```html
<!-- Internal -->
<script type="text/javascript">
  document.write("Hello JS");   // single-line comment
  /* multi-line comment */
  alert("Hi");
</script>

<!-- External -->
<script src="app.js"></script>
```
- Statements end with `;` (optional but recommended). Blocks use `{ }`.
- Identifiers: letters, `$`, `_`, digits (not first); case-sensitive.
- Variables: `var` (function scope), `let` (block scope), `const` (constant).

### 6.2 Data Types
| Category | Types |
|---|---|
| **Primitive (7)** | `String`, `Number`, `BigInt`, `Boolean`, `undefined`, `null`, `Symbol` |
| **Non-primitive** | `Object` (includes Array, Function, Date…) |

```javascript
let name = "Ravi";      // String
let age = 25;           // Number
let ok = true;          // Boolean
let x;                  // undefined
let y = null;           // null
console.log(typeof age);   // "number"
```

### 6.3 Tokens
Smallest individual units of a program:
1. **Keywords** – `var`, `if`, `function`, `return`…
2. **Identifiers** – variable/function names
3. **Literals** – `10`, `"abc"`, `true`
4. **Operators** – `+ - * / ==`
5. **Punctuators/Separators** – `; , ( ) { } [ ]`
6. *(Comments & whitespace are ignored)*

### 6.4 Operators
| Type | Operators |
|---|---|
| Arithmetic | `+ - * / % ** ++ --` |
| Assignment | `= += -= *= /= %=` |
| Comparison | `== != === !== > < >= <=` (`===` checks value **and** type) |
| Logical | `&& \|\| !` |
| Bitwise | `& \| ^ ~ << >> >>>` |
| Conditional (ternary) | `cond ? a : b` |
| Others | `typeof`, `instanceof`, `delete`, `new` |

### 6.5 Control Structures
```javascript
// Decision
if (a > b) { ... } else if (a == b) { ... } else { ... }
switch (day) { case 1: ...; break; default: ...; }

// Loops
for (let i = 0; i < 5; i++) { ... }
while (cond) { ... }
do { ... } while (cond);
for (let k in obj) { ... }      // keys
for (let v of arr) { ... }      // values

// Jump
break; continue; return;
```

### 6.6 Array
Ordered collection; index starts at 0; can hold mixed types.
```javascript
let a = [10, 20, 30];
let b = new Array("x", "y");
a.push(40);        // add end
a.pop();           // remove end
a.shift();         // remove first
a.unshift(5);      // add first
a.length;          // size
a.slice(1,3); a.splice(1,1); a.sort(); a.reverse(); a.indexOf(20); a.join("-");
```

### 6.7 Function
Reusable block of code.
```javascript
function add(a, b) { return a + b; }        // declaration
let sub = function(a, b) { return a - b; }; // expression
let mul = (a, b) => a * b;                  // arrow (ES6)
console.log(add(2, 3));
```
- Parameters are optional/untyped. Functions are first-class (can be passed around).
- Built-in: `parseInt()`, `parseFloat()`, `isNaN()`, `eval()`.

### 6.8 Events and Types
**Event:** an action/occurrence (user or browser) that JS can respond to via an **event handler**.
```html
<button onclick="greet()">Click</button>
<script>
  function greet(){ alert("Hello"); }
  // or: element.addEventListener("click", greet);
</script>
```
| Type | Events |
|---|---|
| Mouse | `onclick`, `ondblclick`, `onmouseover`, `onmouseout`, `onmousedown`, `onmouseup`, `onmousemove` |
| Keyboard | `onkeydown`, `onkeyup`, `onkeypress` |
| Form | `onsubmit`, `onreset`, `onchange`, `onfocus`, `onblur`, `oninput`, `onselect` |
| Window/Document | `onload`, `onunload`, `onresize`, `onscroll` |

### 6.9 Class
ES6 syntax for object blueprints (built on prototypes).
```javascript
class Student {
  constructor(name, roll) { this.name = name; this.roll = roll; }
  show() { return this.name + " - " + this.roll; }
}
let s = new Student("Ravi", 1);
console.log(s.show());
// Inheritance: class PG extends Student { super(...) }
```
JS also has built-in objects: `Math`, `Date`, `String`, `Array`, `RegExp`, `window`, `document`.

### 6.10 Exception Handling
Handles run-time errors so the program doesn't crash.
```javascript
try {
  let r = riskyCall();
  if (r < 0) throw new Error("Negative value");
} catch (e) {
  console.log(e.name + ": " + e.message);
} finally {
  console.log("Always runs");
}
```
- `try` – code to test; `catch` – handles error; `finally` – always executes; `throw` – raise custom error.
- Error types: `SyntaxError`, `ReferenceError`, `TypeError`, `RangeError`.

---

## 7. Process of Writing and Running HTML / DHTML Programs

1. **Write** code in a text editor (Notepad, VS Code, Sublime).
2. **Save** with extension `.html` or `.htm` (e.g., `index.html`), encoding UTF-8.
3. **Open** in a browser (double-click, or right-click → Open with…).
4. **View output** – browser parses HTML, applies CSS, executes JS.
5. **Debug** – browser *Developer Tools* (F12) → Console/Elements.
6. **Edit → Save → Refresh (F5)** to see changes.

> Hosting on a server (Apache/Tomcat) is only required for server-side code (JSP/Servlet).

**Complete DHTML Example**
```html
<!DOCTYPE html>
<html>
<head>
  <title>DHTML Demo</title>
  <style> #box{width:100px;height:100px;background:orange;} </style>
  <script>
    function change(){
      var b = document.getElementById("box");
      b.style.background = "green";
      b.style.width = "200px";
    }
  </script>
</head>
<body>
  <div id="box" onmouseover="change()"></div>
</body>
</html>
```

---

## 8. CSS

**CSS (Cascading Style Sheets):** Language that describes the **presentation** (colour, layout, fonts) of HTML documents, separating content from design. Current standard: **CSS3**.

**Advantages:** reusability, consistent design, smaller HTML files, easy maintenance, responsive design.

### 8.1 CSS Syntax
```
selector {
    property : value;
    property : value;
}
```
```css
h1 { color: blue; font-size: 24px; text-align: center; }
```
- `selector` = element to style; `{ }` = declaration block; `property: value;` = declaration.
- Comment: `/* text */`

### 8.2 CSS Selectors
| Selector | Syntax | Selects |
|---|---|---|
| Universal | `* { }` | All elements |
| Element/Type | `p { }` | All `<p>` |
| Class | `.note { }` | `class="note"` (reusable) |
| ID | `#main { }` | `id="main"` (unique) |
| Group | `h1, h2, p { }` | Multiple selectors |
| Descendant | `div p { }` | `<p>` anywhere inside `<div>` |
| Child | `div > p { }` | `<p>` direct child of `<div>` |
| Adjacent sibling | `h1 + p { }` | `<p>` immediately after `<h1>` |
| General sibling | `h1 ~ p { }` | All `<p>` siblings after `<h1>` |
| Attribute | `input[type="text"] { }` | By attribute |
| Pseudo-class | `a:hover { }`, `li:first-child` | By state/position |
| Pseudo-element | `p::first-line { }`, `::before` | Part of an element |

**Specificity (low→high):** element < class/pseudo-class < ID < inline style (`!important` overrides).

### 8.3 CSS Properties (common)
| Group | Properties |
|---|---|
| Text | `color`, `text-align`, `text-decoration`, `text-transform`, `line-height`, `letter-spacing` |
| Font | `font-family`, `font-size`, `font-weight`, `font-style` |
| Background | `background-color`, `background-image`, `background-repeat`, `background-size` |
| Box model | `width`, `height`, `margin`, `padding`, `border` |
| Layout | `display`, `position`, `float`, `top/left`, `z-index`, `overflow`, `flex`, `grid` |
| List/Table | `list-style-type`, `border-collapse` |
| Visual | `opacity`, `visibility`, `cursor`, `box-shadow` |

**Box model (outside → inside):** Margin → Border → Padding → Content.

### 8.4 Types of CSS
| Type | Where | Example |
|---|---|---|
| **Inline** | `style` attribute of one element | `<p style="color:red">` |
| **Internal (Embedded)** | `<style>` inside `<head>` | `<style> p{color:red} </style>` |
| **External** | Separate `.css` file linked via `<link>` | `<link rel="stylesheet" href="style.css">` |

**Priority order:** Inline > Internal > External (for equal specificity, the later rule wins).
External is best for multi-page sites.

---

## 9. XHTML and XML

### 9.1 XHTML
**XHTML (Extensible HyperText Markup Language):** A stricter, XML-based version of HTML 4. Pages must be **well-formed XML**.

**Rules**
1. Proper `<!DOCTYPE>` and root `<html xmlns="http://www.w3.org/1999/xhtml">`
2. Tags and attribute names in **lowercase**
3. All tags must be **closed** (`<br />`, `<img ... />`)
4. Properly **nested** tags
5. Attribute values must be **quoted** (`width="100"`)
6. No attribute minimisation (`checked="checked"`, not `checked`)
7. Exactly one root element

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
 "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <head><title>XHTML</title></head>
  <body><p>Hello<br /></p></body>
</html>
```
| HTML | XHTML |
|---|---|
| Lenient syntax | Strict, XML rules |
| Case-insensitive tags | Lowercase required |
| Closing tags optional (some) | Mandatory |
| Errors tolerated | Errors can stop rendering |

### 9.2 XML
**XML (eXtensible Markup Language):** W3C markup language to **store and transport data**; self-descriptive; you define your **own tags**. Platform and language independent.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<students>
  <student id="1">
    <name>Ravi</name>
    <marks>85</marks>
  </student>
</students>
```
**Rules (well-formed):** one root element · closed tags · proper nesting · case-sensitive · quoted attributes · special characters escaped (`&lt; &gt; &amp; &quot; &apos;`).

**Parts:** Prolog (`<?xml ...?>`), Elements, Attributes, Text, Comments `<!-- -->`, CDATA `<![CDATA[ ... ]]>`.

**Validation:** *Well-formed* = follows syntax rules. *Valid* = also conforms to a **DTD** or **XML Schema (XSD)**.

**Related technologies:** XSLT (transform), XPath (query), DOM/SAX parsers (read XML in Java), CSS (style).

| HTML | XML |
|---|---|
| Displays data | Stores/transports data |
| Predefined tags | User-defined tags |
| Not strict | Strict |
| Case-insensitive | Case-sensitive |

---

## 10. Quick Revision

- **Network types:** PAN < LAN < MAN < WAN (Internet = largest WAN).
- **URL:** protocol + domain + port + path + query + fragment.
- **HTML5 additions:** `<audio>`, `<video>`, semantic tags, new input types. **Obsolete:** `<frameset>`, `<frame>`, `<marquee>` (non-standard).
- **DHTML = HTML + CSS + JS + DOM.**
- **JS:** 7 primitives + object; `===` strict equality; `let/const` block scoped; `try-catch-finally` for exceptions.
- **CSS types:** Inline > Internal > External. Selector priority: ID > class > element.
- **XHTML:** strict, lowercase, closed, nested, quoted. **XML:** user-defined tags, stores data.

### Likely Exam Questions
1. Explain LAN, MAN, WAN with examples.
2. Differentiate client and server; explain URL structure.
3. Write HTML for a registration form / table / frameset.
4. What is DHTML? Differentiate HTML and DHTML.
5. Explain JavaScript data types, operators, and events.
6. Explain exception handling in JavaScript with example.
7. What are CSS selectors? Explain types of CSS.
8. Differentiate HTML, XHTML and XML.

---
*Reference books (per syllabus): Ivan Bayross – Web Technologies Pt.1 & 2 (BPB); Thomas A. Powell – HTML & CSS: The Complete Reference.*
