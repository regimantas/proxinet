# ProxiNet

**ProxiNet** is a modern, dynamic, and lightweight reverse proxy server for HTTP and HTTPS.  
It supports automatic certificate management, real-time configuration reloads, load balancing with weights, WebSocket proxying, multi-language logging, and can be easily run as a Linux systemd or Windows service.

---

## Features

- **Dynamic configuration:**  
  Reloads `redirects.json` and `conf.json` automatically at runtime, without restarting the proxy.
- **HTTP & HTTPS support:**  
  Supports both protocols, with automatic HTTP↔HTTPS redirection based on configuration.
- **Automatic TLS certificates:**  
  Uses Let's Encrypt (autocert) for domains with `useHTTPS: true` and no manual certs.
- **Manual certificate support:**  
  Use your own certificates by specifying `certFile` and `keyFile`.
- **Load balancing:**  
  Supports multiple backend targets per route, with weighted round-robin balancing.
- **WebSocket proxying:**  
  Fully supports WebSocket connections.
- **Multi-language logging:**  
  Logs and errors are translated (English, Lithuanian, Polish, Spanish, German) and language can be changed at runtime.
- **Systemd integration:**  
  Can generate and enable a Linux systemd service automatically.
- **Hot reload:**  
  Add, remove, or change domains, targets, or certificates in `redirects.json` and changes take effect in seconds.

---

## Minimal Example

Here is a minimal `redirects.json` example for a single domain, no weights, and a single backend target:

```json
[
  {
    "host": "example.com",
    "path": "",
    "targets": [
      "http://localhost:8080"
    ]
  }
]
```

## Minimal Example with HTTPS
Here is a minimal `redirects.json` example for a single domain with automatic HTTPS certificate (Let's Encrypt), no weights, and a single backend target:

```json
[
  {
    "host": "yourdomain.com",
    "path": "",
    "targets": [
      "http://localhost:8080"
    ],
    "useHTTPS": true
  }
]
```

**How to use:**
1. Replace `"yourdomain.com"` with your real domain (it must point to your server's IP).
2. Make sure port 80 and 443 are open and not blocked by a firewall.
3. Run ProxiNet (no flags needed, language will be auto-selected from `conf.json` or default to English):
   ```sh
   ./proxinet
   ```
4. Optionally, you can specify language with a flag:
   ```sh
   ./proxinet -lang=lt
   ```
5. ProxiNet will automatically obtain and renew a free HTTPS certificate for your domain.

**No weights or extra settings are needed for this minimal setup. Just the domain, path, and target.**

---

## Usage

### 1. Run ProxiNet

```sh
./proxinet
```

- You can also use `-lang=lt`, `-lang=pl`, `-lang=es`, or `-lang=de` for other languages:
  ```sh
  ./proxinet -lang=lt
  ```
- The selected language is saved in `conf.json` for next runs.
- If you run without the `-lang` flag, the language will be automatically selected from the last used value in `conf.json` (if it exists), or default to English.

---

### 2. Run as a systemd service for Linux or Windows

```sh
# For Linux needs root privileges
sudo ./proxinet -service
# or to specify language and service name
sudo ./proxinet -service -lang=en -svcname=proxinet

# For Windows, run in an elevated command prompt (as Administrator)
./proxinet -service
# or to specify language and service name
./proxinet -service -lang=en -svcname=proxinet
```

- This will create `/etc/systemd/system/proxinet.service`, enable and start the service.

---

## Configuration

### `redirects.json` example

```json
[
  {
    "host": "example.com",
    "path": "",
    "targets": [
      "http://localhost:8081",
      "http://localhost:8082"
    ],
    "weights": [3, 1],
    "useHTTPS": true,
    "preservePath": false
  },
  {
    "host": "example.com",
    "path": "/api",
    "targets": [
      "http://localhost:9000"
    ],
    "weights": [1],
    "useHTTPS": false,
    "preservePath": true
  },
  {
    "host": "anotherdomain.com",
    "path": "",
    "targets": [
      "http://192.168.1.100:8080"
    ],
    "weights": [1],
    "certFile": "/etc/ssl/certs/anotherdomain.crt",
    "keyFile": "/etc/ssl/private/anotherdomain.key",
    "useHTTPS": true,
    "preservePath": true
  }
]
```

**Field descriptions:**
- `host`: Domain name to match.
- `path`: URL path prefix to match ("" for root).
- `targets`: List of backend servers.
- `weights`: (optional) List of weights for load balancing (same order as targets).
- `certFile`/`keyFile`: (optional) Use your own certificate for this domain.
- `useHTTPS`: If true, serve this host/path over HTTPS.
- `preservePath`: If true, do not strip the path prefix when proxying.

---

### `conf.json` are automatically created and managed by ProxiNet. 
When ProxiNer runed with language flag, it will create `conf.json` with the selected language.
It be remembered for future runs. and saved as:
```json
{
  "lang": "en"
}
```

---

## How it works

- **Dynamic reload:**  
  The proxy watches `redirects.json` and `conf.json` for changes every 5 seconds.
  - If you edit these files (add/remove domains, change targets, change language), the proxy reloads them automatically and applies changes in real time.
- **Load balancing:**  
  Each route can have multiple targets and weights.
  - Requests are distributed using weighted round-robin.
- **WebSocket support:**  
  WebSocket requests are detected and proxied to the correct backend.
- **Automatic certificates:**  
  If `useHTTPS` is true and no manual certs are set, Let's Encrypt certificates are requested and renewed automatically for each domain.
  - New domains added to `redirects.json` are picked up and certificates are issued without restarting the proxy.
- **Manual certificates:**  
  If `certFile` and `keyFile` are set, those certificates are used for the domain.
- **HTTP↔HTTPS redirection:**  
  If a route is set to `useHTTPS: true`, HTTP requests are redirected to HTTPS and vice versa.
- **Multi-language:**  
  All logs and error messages are translated.
  - Change language at runtime by editing `conf.json` or using the `-lang` flag.

---

## Advanced

- **Zero downtime config reload:**  
  You can update backends, add/remove domains, or change HTTPS settings live.
- **Systemd integration:**  
  Use `-service` flag to generate and enable a systemd service for ProxiNet.
- **Minimal resource usage:**  
  File watching is efficient and does not impact performance.

---

## Example: Add a new HTTPS domain live

1. Edit `redirects.json` and add a new entry with `"host": "newdomain.com", "useHTTPS": true`.
2. Save the file.
3. ProxiNet will detect the change, reload config, and automatically request a certificate for `newdomain.com` (no restart needed).

---

## License

MIT

---

**ProxiNet** – a modern, dynamic, and reliable reverse proxy for your

