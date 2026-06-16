# okhx.github.io/tools

---

## `❤️ Made with love`

## 🛠 Modules, functions

### 1. `img://tools` — analyze, graphics protection
A comprehensive tool for working with pixel structure and hidden image data:

**EXIF Metadata Parser & Mapping:** Complete extraction of hidden metadata (camera model, date, shooting parameters). If GPS coordinates are available, an interactive map is automatically rendered (integration with Leaflet.js).

**LSB Steganography:** Hiding text arrays inside the Least Significant Bit of color channels and instantly extracting data from containers.
  
**Anti-AI Cloak:** Generate custom noise and micro-patterns to protect images from unauthorized parsing and train neural networks (protection from AI scraping). 

**Perceptual Hashing (pHash):** Analyze image structure, generate persistent hashes, and calculate Hamming distance to find visual duplicates and traces of editing.

### 2. 🛠️ `px://hook` — Advanced Session Intercept & Security Diagnostics
A comprehensive, specialized environment for capturing traffic, debugging web requests, automating payload injection, and auditing client-side application security.
px://hook acts as a full-featured web proxy and security analysis hub directly within your ecosystem, designed for security engineers and developers.

### 🚀 Core Modules
**🎛️ Intruder (Automated Attacks):** A highly customizable automation engine for fuzzing, brute-forcing, and structured payload injection to stress-test endpoints and uncover vulnerabilities.

**📊 Sequencer (Randomness Analysis):** Advanced statistical analysis tool to evaluate the unpredictability and randomness of session tokens, CSRF tokens, and cryptographic nonces.

**🔄 Decoder & Comparer:** Integrated utilities for rapid data encoding/decoding (Hex, Base64, URL, etc.) and executing precise, visual differential analysis (diff) between separate requests or responses.

**📂 Organizer & Session Hub:** A centralized workspace to catalog, filter, tag, and structure captured requests and analysis results for streamlined testing workflows.

**⚡ Request Interception & Modification:** Real-time monitoring, capturing, and hot-swapping of HTTP/HTTPS traffic, headers, payloads, and cookies on the fly.

**🛡️ DOM & XSS Vector Testing:** An isolated sandbox for evaluating structural payload mutations, analyzing DOM injection vectors, and auditing client-side security mechanisms.

**🔍 Anomaly & Signature Detection:** Deep analysis of code structures and network traffic to identify non-standard patterns, broken packet signatures, and structural anomalies.

### ⚙️ Targeted Configuration
#### 🌐 Scope Settings
Strict target filtering definitions to isolate your traffic. Ensure your diagnostics, history, and automated attacks are strictly confined to specific domains, paths, ports, or protocols, eliminating background noise from unrelated browser traffic.

#### 🔄 Match and Replace
Rule-based, automated modification of headers, request/response bodies, or parameters on the fly. Use this to effortlessly analyze and swap site behavior under altered conditions, such as:
 * Emulating different user roles by replacing Auth headers.
 * Forcing specific client-side behaviors by modifying response payloads.
 * Stripping security headers (CSP, HSTS) in a test environment to isolate vulnerabilities.
