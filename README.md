# 🛠️ 10 Essential `curl` Commands to Debug an API or Ingress

A quick-reference cheat sheet for troubleshooting HTTP responses, latency bottlenecks, TLS/DNS routing, and ingress configurations directly from the terminal.

---

## 1. Inspecting Responses ("What came back?")

Use these commands to verify status codes, inspect response headers, and trace redirect chains.

* **Include response headers with the body:**
  ```bash
  curl -i https://api.example.com/health
  ```
  > Shows HTTP status codes and response headers alongside the payload.

* **Fetch headers only (`HEAD` request):**
  ```bash
  curl -I https://api.example.com/health
  ```
  > Quickly checks endpoint availability, cache headers, and status codes without downloading the body.

* **Follow HTTP redirects (`301` / `302` → `200`):**
  ```bash
  curl -L http://api.example.com/health
  ```
  > Automatically follows `Location` headers until reaching the final destination.

---

## 2. Diagnosing Latency & Connection Issues ("Where is it slow?")

Pinpoint whether delays happen during DNS lookup, TCP handshake, TLS negotiation, or server processing.

* **Verbose connection & TLS trace:**
  ```bash
  curl -v api.example.com
  ```
  > Prints every step of the connection, TLS handshake, request headers, and response headers.

* **Measure total request timing:**
  ```bash
  curl -w "%{time_total}\n" -so /dev/null api.example.com
  ```
  > Silences the output body (`-so /dev/null`) and prints the exact total time taken in seconds.

* **Fail fast if a connection cannot be established:**
  ```bash
  curl --connect-timeout 3 api.example.com
  ```
  > Aborts if the TCP connection isn't established within `3` seconds.

* **Cap the total request duration:**
  ```bash
  curl --max-time 10 api.example.com
  ```
  > Forces `curl` to exit if the entire operation takes longer than `10` seconds.

---

## 3. Testing Ingress, Routing & Payloads ("Test it directly")

Bypass DNS propagation, test specific ingress rules, and simulate authenticated client requests.

* **Pin a specific IP address and skip TLS verification:**
  ```bash
  curl -k --resolve api.example.com:443:10.0.0.5 https://api.example.com/health
  ```
  > Resolves `api.example.com` directly to `10.0.0.5` while preserving SNI (Server Name Indication) for HTTPS and ignoring self-signed/mismatched certs (`-k`).

* **Target a specific Ingress / VirtualHost rule via the `Host` header:**
  ```bash
  curl -H "Host: api.example.com" http://10.0.0.5/health
  ```
  > Sends the request directly to the load balancer or node IP while routing to the matching host rule.

* **Simulate an authenticated `POST` API call:**
  ```bash
  curl -X POST -d '{"user":"a"}' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $TOKEN" \
    https://api.example.com/login
  ```
  > Sends a JSON payload with a Bearer token to test endpoints just like a real client.

---

## 📋 Quick Flag Reference

| Flag | Purpose |
| :--- | :--- |
| `-i` | Include HTTP response headers in the output |
| `-I` | Fetch headers only (`HEAD` request) |
| `-L` | Follow HTTP redirects |
| `-v` | Enable verbose output (DNS, TCP, TLS, headers) |
| `-w` | Format and display custom timing/transfer metrics |
| `--connect-timeout` | Max seconds allowed for the initial connection |
| `--max-time` | Max seconds allowed for the entire transfer |
| `-k` | Allow insecure TLS/SSL connections without cert validation |
| `--resolve` | Force a custom `HOST:PORT:ADDRESS` mapping |
| `-H` | Pass custom HTTP headers (e.g., `Host`, `Authorization`) |
| `-X` / `-d` | Specify HTTP method (`POST`, `PUT`, etc.) and request body data |
