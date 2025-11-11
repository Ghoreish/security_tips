# TypeScript Security Keywords / Patterns

Below is a checklist-style table of risky keywords/patterns to search for in a TypeScript codebase, why each is risky, what to inspect.

| Keyword / Pattern | Why it's risky | What to check (how to inspect) | Example (pattern) |
|---|---|---|---|
| `eval` | Executes a string as code → RCE / XSS risk | Find all `eval` usages; replace with safe parsing or explicit APIs; ensure arguments are not user-controlled. | `eval(userInput)` |
| `new Function(...)` | Same risk as `eval` — dynamic code execution | Search for `new Function`; avoid constructing functions from user input. | `new Function("a","b", userCode)` |
| `setTimeout(string, ...)` / `setInterval(string, ...)` | Strings are executed like `eval` | Ensure first argument is a function, not a string. | `setTimeout("doThing()", 1000)` |
| `innerHTML` / `insertAdjacentHTML` / `outerHTML` | Direct HTML insertion → DOM XSS | Sanitize input or use `textContent`/`createTextNode`. Verify source of inserted HTML. | `el.innerHTML = resp.html` |
| `document.write` | Writes raw HTML into page → XSS/DOM injection | Avoid; if necessary sanitize input thoroughly. | `document.write(userHtml)` |
| `dangerouslySetInnerHTML` (React) | Explicit bypass of React escaping → XSS | Audit all uses; only set from safe/whitelisted sources or sanitized HTML. | `<div dangerouslySetInnerHTML={{__html: html}}/>` |
| `location.href =` / `window.open` with user input | Open-redirect, phishing, SSRF-like navigation | Validate and whitelist target URLs; parse with `URL` constructor. | `window.open(req.query.url)` |
| `innerText` vs `textContent` | Misuse can lead to inconsistent escaping | Prefer `textContent` for raw text; ensure no HTML injection vector remains. | `el.innerText = user` |
| `JSON.parse()` on unvalidated input | Unvalidated data may lead to logic bugs; in some flows can enable prototype pollution | Validate schema before/after parse; avoid merging parsed objects into sensitive prototypes. | `JSON.parse(req.body)` |
| Dynamic `require()` / `import()` with user input | Can load arbitrary modules → RCE / information leak | Whitelist modules or use fixed imports; never require paths from user input. | `require(req.query.module)` |
| `child_process.exec` / `spawn` | Executes shell commands → RCE | Use `execFile` with arg arrays, validate/sanitize args, avoid shell interpolation. | `exec("cmd " + userArg)` |
| File APIs (`fs.readFile`, `fs.writeFile`) with user paths | Path traversal (`../`) or unauthorized file access | Canonicalize path, `path.resolve`, enforce a whitelist/base directory. | `fs.readFile("/data/" + req.path)` |
| String-building SQL queries (concatenation / template strings) | SQL injection | Use parameterized queries / prepared statements / ORM parameter APIs. | `db.query("... WHERE id=" + id)` |
| MongoDB `$where` or dynamic query injection | NoSQL injection or JS execution inside DB | Avoid `$where`; use parameterized query objects and validate inputs. | `collection.find({ $where: userInput })` |
| `Object.assign` / object spread from untrusted input | Prototype pollution (e.g., `__proto__`) | Filter keys (`__proto__`, `constructor`, `prototype`), deep-clone safely, or use safe-merge libs. | `Object.assign({}, JSON.parse(body))` |
| Excessive `any` / `as any` / `// @ts-ignore` | Circumvents type safety → hidden logic/security bugs | Audit and replace with proper types or `unknown` + runtime validation. | `const x: any = JSON.parse(s)` |
| Dangerous casts (`as unknown as T`) | Masks type mismatches that can cause incorrect assumptions | Find casts and add runtime guards/checks for the assumed shape. | `const req = body as unknown as RequestType` |
| `new RegExp(user)` or user-controlled regex | ReDoS (Regular Expression DOS) — catastrophic backtracking | Avoid building regexes from user input; enforce length/complexity limits or use safe-regex libs. | `new RegExp(req.query.r)` |
| ESLint / TS disables (`// eslint-disable`, `// @ts-ignore`) | Hides warnings/errors, may bypass safety rules | Search for disable comments and review why they were added. | `// eslint-disable-next-line no-eval` |
| Hard-coded secrets in source | Credential leak (API keys, passwords) | Search for obvious keys (`API_KEY`, `SECRET`, `PASSWORD`) and remove; use env vars / secret manager. | `const token = "abcd1234"` |
| Weak or custom crypto usage | Broken encryption → data compromise | Prefer vetted crypto APIs, modern algorithms (AES-GCM, RSA-PSS); avoid MD5/SHA1 for integrity/security. | `createHash('md5')` |
| CORS `Access-Control-Allow-Origin: *` or dynamic echo | Overly permissive cross-origin access → data leak | Review server CORS policy; prefer specific origins and credential-aware rules. | `res.setHeader('Access-Control-Allow-Origin', req.headers.origin)` |
| `fetch()` / HTTP calls with user URL | SSRF (server-side request forgery) risk | Whitelist hosts, block internal IP ranges (127.0.0.1, 10.*, 172.16-31.*, 192.168.*), validate scheme/port. | `fetch(req.query.url)` |
| Server-side rendering (SSR) outputting unescaped values | Server-generated XSS | Escape output in templates or sanitize server-side HTML. | `res.send(\`<div>\${data}</div>\`)` |
| `postMessage` without `origin` check | Message spoofing or data leakage between windows/frames | Always validate `event.origin` and check message shape/type. | `otherWindow.postMessage(data, '*')` |
| Using known-vulnerable library versions | Known CVEs exploitable in dependencies | Run `npm audit`, Snyk, Dependabot; update/patch vulnerable libs. | `lodash@<vulnerable>` |
| Missing input validation / insufficient sanitization | Foundation for XSS, injection, logic bugs | Add schema validation (Zod/Joi/io-ts) on all external inputs. | `if (!req.body.name) ...` (without full validation) |
| Non-null assertion (`!`) | Hides possible `null`/`undefined` → runtime crashes or logic flaws | Inspect all `!` usages, add runtime null checks where necessary. | `const x = obj!.prop` |

---

## Quick practical checklist

- ✅ **Run static checks**: `eslint` with security plugins, `tsc --noEmit` (strict).  
- ✅ **Add runtime validation**: use `zod`, `io-ts`, or `Joi` to validate all external inputs.  
- ✅ **Prevent XSS**: always escape server-side and sanitize any HTML input (e.g., `DOMPurify` for browser HTML).  
- ✅ **Use parameterized DB queries**: no string concatenation for SQL/NoSQL.  
- ✅ **Protect file access**: `path.resolve` + whitelist base directories.  
- ✅ **Avoid dynamic code execution**: remove `eval`, `new Function`, string `setTimeout`, dynamic `require` from user input.  
- ✅ **Audit dependencies periodically**: `npm audit`, Dependabot/Snyk.  
- ✅ **Block SSRF attempts**: whitelist hosts, disallow internal IP ranges.  
- ✅ **Harden CORS**: prefer specific origins; avoid `*` if credentials are used.  
- ✅ **Review all `// @ts-ignore` and `any`**: add types or runtime checks.

---

## How to use this file 
1. Use simple `grep`/`rg`/`git grep` to find these patterns across the codebase. Example:
   ```bash
   rg "eval|innerHTML|dangerouslySetInnerHTML|new Function|child_process|fs.readFile|new RegExp|// @ts-ignore|as any" --hidden
   
> [!NOTE]
> This content was created by ChatGPT, and since I found it interesting and useful, I decided to share it. I hope it helps you.
