# Client Side

the following table contains the keywords that are important and should be checked in js source files:

|Keyword / Function|Description|
|------------------|-----------|
| `__proto__` |Property allowing manipulation of an object's prototype chain, leading to prototype pollution vulnerabilities.|
|`JSON.parse()`|Parses JSON strings into JavaScript objects. Improper usage can result in prototype pollution vulnerabilities.|
|`$.extend()`|Merges contents of two or more objects together, potentially leading to prototype pollution if untrusted data is used.|
|`Object.assign()`|Copies values of enumerable own properties from one or more source objects to a target object. Improper usage can lead to prototype pollution vulnerabilities.|
|`eval()`|Executes JavaScript code represented as a string. Improper usage with user-controlled input can result in code injection vulnerabilities.|
|`Function() constructor`|Dynamically creates functions from strings, making it susceptible to code injection if not used carefully.|
|`innerHTML`|Sets HTML content of an element, leading to XSS vulnerabilities if untrusted data is included.|
|`document.write()`|Dynamically writes content to the document, potentially leading to XSS vulnerabilities if untrusted data is included.|
|`setTimeout() / setInterval()`|Executes code after a specified delay or at regular intervals. If the code executed contains untrusted data, it can lead to security vulnerabilities.|
|`addEventListener() / attachEvent()`|Attaches event handlers to HTML elements, potentially leading to XSS vulnerabilities if untrusted data is included.|
|`location`|Objects like window.location or document.location can be manipulated to perform XSS attacks, especially with user-supplied data.|
|`window`|Access to the window object can be abused to perform XSS attacks or manipulate the prototype chain if not handled securely.|
|`Reflect.construct()`|Creates instances of objects. If constructor function or arguments are derived from untrusted sources, it can lead to security vulnerabilities.|
|`new operator`|Creates instances of objects. If constructor function or arguments are derived from untrusted sources, it can lead to security vulnerabilities.|
|`RegExp() constructor`|Creating regular expressions from user input can lead to security vulnerabilities, including denial of service or injection attacks.|
|`HTML parsing and DOM manipulation methods`|Functions like createElement(), createAttribute(), appendChild(), etc., can lead to XSS vulnerabilities if user-controlled data is used without proper sanitation.|
