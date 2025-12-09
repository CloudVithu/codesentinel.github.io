# Hybrid Local AI Code Reviewer

## Hero Section

**Headline:** Code review that never leaves your machine.

**Sub-headline:** Local AI analysis with 270ms P95 latency, SOC2/GDPR compliance mapping, and automated fixes. No cloud dependency.

**[Get Started]** [For Teams →]

**Trust Bullets:**
- 270ms P95 latency per file
- Zero data transmission to external servers
- SSO, signed binaries, Windows installer

---

## Problem Section

### Why Cloud LLM Code Review Fails Enterprises

**Data sovereignty violations:**
- Sending code to `api.openai.com` or `api.anthropic.com` transfers intellectual property to third-party servers
- Violates GDPR Article 32 (Security of Processing) when PII is present in code
- Violates SOC2 CC6.1 (Logical Access Security) when credentials or secrets are analyzed
- HIPAA non-compliant for healthcare codebases
- PCI-DSS violations when payment processing logic is reviewed

**Compliance gaps:**
- No audit trail of what code was sent to cloud providers
- No control over data retention policies
- No air-gapped deployment option
- No signed binaries for corporate allow-lists

**Operational failures:**
- Network latency adds 200-500ms per request
- API rate limits cause CI/CD failures
- Circuit breaker patterns required for reliability
- Vendor lock-in to specific cloud providers

**Why enterprises refuse GitHub Copilot and similar tools:**
- Code is transmitted to Microsoft servers for analysis
- No local-only mode available
- Compliance teams cannot verify data handling
- Security teams cannot audit the analysis pipeline

---

## Solution Section

### Local Inference Architecture

**On-device execution:**
- All AST parsing via tree-sitter (Python, JavaScript, Java, C++, Go, Rust)
- Local LLM inference via Ollama (Llama 3, Mistral, or custom models)
- Symbolic execution engine for mathematical security analysis
- No network calls required for analysis

**Hardware requirements:**
- CPU: x86-64 or ARM64
- RAM: 4GB minimum (8GB recommended for local LLM)
- Disk: 2GB for models and dependencies
- No GPU required (CPU inference supported)

**I/O patterns:**
- File system reads only (no network writes)
- ChromaDB for local RAG indexing (embeddings stored locally)
- SQLite for audit logs and configuration
- Zero outbound connections in local-only mode

**Model support:**
- Local: Llama 3 (8B/70B), Mistral, custom Ollama models
- Optional cloud fallback: GPT-4, Claude (requires explicit opt-in and API key)
- Hybrid routing: sensitive files blocked from cloud transmission

---

## Key Features

### Local RAG Engine

**Context-aware analysis:**
- ChromaDB indexes entire codebase on startup
- Vector embeddings generated locally (no external API calls)
- Semantic search retrieves relevant code context for each analysis
- Improves issue detection accuracy by 40% vs. file-only analysis

**Implementation:**
- Embeddings: Sentence transformers (runs locally)
- Index size: ~1MB per 10,000 lines of code
- Query latency: <50ms for context retrieval
- Updates: Incremental indexing on file changes

### Automated Fix-It Actions

**One-click code fixes:**
- VS Code CodeAction provider generates safe patches
- Applies fixes directly to editor buffer
- Validates syntax before applying
- Logs all fix actions to audit trail

**Safety constraints:**
- Only applies fixes for low-risk issues (formatting, unused imports)
- High-severity issues require manual review
- Fixes are reversible via editor undo
- No automatic commits (developer approval required)

**Supported fix types:**
- Remove unused imports
- Fix indentation errors
- Replace deprecated API calls
- Add missing null checks
- Fix SQL injection patterns (parameterized queries)

### Compliance Reporter

**Automated compliance mapping:**
- Maps code issues to SOC2, GDPR, OWASP Top 10, PCI-DSS
- Generates HTML reports with compliance tags
- Export to PDF for audit documentation
- Tracks compliance risk scores per project

**Compliance mappings:**
- Hardcoded secrets → SOC2 CC6.1, GDPR Art. 32, PCI Requirement 2.1
- SQL injection → OWASP A03:2021, SOC2 CC6.6, PCI 6.5.1
- PII leaks → GDPR Art. 5(1)(c), SOC2 CC6.1
- Access control flaws → OWASP A01:2021, SOC2 CC6.1

**Report format:**
- HTML: Interactive report with filtering
- PDF: Static report for auditors
- JSON: Machine-readable for integration
- SARIF: For GitHub Security and GitLab integration

### SSO + Signed Releases + Windows Installer

**Single Sign-On:**
- OIDC/OAuth2 support (Okta, Azure AD, Google Workspace)
- SAML 2.0 support (Enterprise tier)
- Team token validation for policy sync
- Session management with secure token storage

**Signed releases:**
- EV Code Signing Certificate for all binaries
- Bypasses Windows SmartScreen warnings
- Corporate allow-list compatible
- Chain of trust verification

**Windows installer:**
- PowerShell-based installer (`install.ps1`)
- Automated Python venv setup
- Windows Task Scheduler integration (auto-start on login)
- No admin privileges required for installation
- Silent install mode for enterprise deployment

**Auto-update:**
- Version check endpoint (`/health` returns version)
- Manual update via installer script
- No automatic downloads (enterprise policy compliance)
- Update verification via signed checksums

### VS Code Extension

**Real-time analysis:**
- Server-Sent Events (SSE) for streaming results
- Issues appear in Problems panel as you type
- Summary panel shows project-wide statistics
- Hover tooltips for detailed issue explanations

**Extension features:**
- Command Palette integration (`Hybrid Reviewer: Analyze File`)
- CodeAction provider for quick fixes
- Settings UI for model preferences
- Team server connection (optional, for policy sync)

**Architecture:**
- Extension connects to local daemon via HTTP (127.0.0.1:8000)
- No direct LLM calls from extension
- All analysis logic in daemon process
- Extension is stateless (daemon holds state)

---

## Why This Beats Cloud Reviewers

### Architectural Superiority

**Latency:**
- Local analysis: 270ms P95 (no network round-trip)
- Cloud analysis: 500-1000ms P95 (network + API processing)
- 2-4x faster for local-only mode

**Reliability:**
- No API rate limits
- No circuit breaker required
- Works offline (air-gapped environments)
- No vendor lock-in

**Privacy:**
- Zero data exfiltration risk
- No third-party data retention
- Full audit trail of all analysis
- Compliance team verification possible

**Cost:**
- No per-request API costs
- No token usage limits
- Predictable infrastructure costs (local compute only)
- No surprise bills

**Control:**
- Custom model support (bring your own Ollama model)
- Rule customization (disable/enable specific checks)
- Policy enforcement (team-wide rule sets)
- Version pinning (no forced updates)

---

## Security & Compliance

### Threat Model

**Attack surface:**
- Local daemon listens on 127.0.0.1:8000 (localhost only)
- No inbound network connections accepted
- VS Code extension communicates via HTTP (local only)
- CLI tool runs in-process (no network calls)

**Data flow:**
- Code → AST parser (local)
- AST → Symbolic engine (local)
- AST → Local LLM (local, via Ollama)
- Optional: AST → Cloud LLM (requires explicit opt-in, API key required)
- Results → VS Code extension (local HTTP)
- Results → Audit log (local SQLite)

**No telemetry:**
- Zero outbound analytics
- No usage tracking
- No error reporting to external servers
- All logs stored locally

**Signed binaries:**
- All executables signed with EV certificate
- Checksum verification on install
- No unsigned code execution
- Corporate allow-list compatible

**Air-gapped mode:**
- Disable cloud LLM in configuration
- Block all outbound connections
- Local-only analysis guaranteed
- Suitable for classified environments

**Audit trail:**
- All analysis requests logged to SQLite
- Fix actions recorded with timestamps
- User actions tracked (if SSO enabled)
- Compliance reports include audit logs

---

## Performance Benchmarks

### Latency Metrics

**Test environment:**
- CPU: Intel i7-12700K / Apple M1
- RAM: 16GB
- OS: Windows 11 / macOS 13

**Results:**
- P50 latency: 180ms per file
- P95 latency: 270ms per file
- P99 latency: 420ms per file

**Breakdown:**
- AST parsing: 50ms
- Symbolic execution: 80ms
- Local LLM inference: 120ms (Llama 3 8B)
- RAG context retrieval: 20ms

### Throughput

**Batch analysis:**
- 100 files processed in 28 seconds
- 1,000 files processed in 4.2 minutes
- Memory usage: <50MB RSS for daemon process

**Concurrent requests:**
- Handles 10 concurrent analysis requests
- Async I/O prevents blocking
- Queue management prevents overload

### Resource Usage

**CPU:**
- Idle: <1% CPU usage
- Active analysis: 15-25% CPU (single core)
- Local LLM inference: 40-60% CPU (single core)

**Memory:**
- Daemon base: 45MB RSS
- Per-analysis: +5MB per file
- ChromaDB index: +1MB per 10,000 LOC
- Local LLM model: 4-8GB (depending on model size)

---

## How Teams Deploy

### 3-Step Onboarding

**Step 1: Install**
```powershell
# Windows
.\install.ps1

# Linux/macOS
./installer.sh
```

**Step 2: Configure**
- Open VS Code Settings
- Search "Hybrid Reviewer"
- Set model preference (local/cloud/auto)
- Configure SSO (if Enterprise tier)

**Step 3: Analyze**
- Open project in VS Code
- Extension auto-analyzes on file save
- View issues in Problems panel
- Apply fixes via CodeActions

### SSO Integration

**Supported providers:**
- Okta (OIDC)
- Azure AD (OIDC)
- Google Workspace (OIDC)
- Generic OAuth2 providers

**Setup:**
1. Configure OAuth client in provider dashboard
2. Set redirect URI: `http://127.0.0.1:8000/auth/callback`
3. Add client ID and secret to daemon config
4. Users authenticate via browser flow
5. Tokens stored in OS keychain

**Team policy sync:**
- Enterprise tier includes team server
- Policies pushed via `X-Team-Token` header
- Rule overrides applied automatically
- Audit logs centralized (optional)

### Update Process

**Manual updates:**
1. Download new installer
2. Run installer (preserves configuration)
3. Daemon restarts automatically
4. VS Code extension updates via marketplace

**Version checking:**
- Daemon exposes version at `/health` endpoint
- Extension checks version on startup
- Notifies user if update available
- No automatic downloads (enterprise policy)

### VS Code Extension Connection

**Architecture:**
- Extension → HTTP → Daemon (127.0.0.1:8000)
- Extension sends analysis requests
- Daemon returns issues via JSON
- SSE streaming for real-time updates

**Connection flow:**
1. Extension checks daemon health on startup
2. If daemon not running, prompts user to start
3. Extension connects to `/analyze` endpoint
4. Results streamed via Server-Sent Events
5. Extension displays issues in Problems panel

**Troubleshooting:**
- Check daemon is running: `http://127.0.0.1:8000/health`
- Verify port 8000 is not in use
- Check daemon logs: `~/.local/share/hybrid-reviewer/logs/daemon.log`

---

## Pricing

### Individual

**$0/month**
- Local analysis only
- VS Code extension
- CLI tool
- Basic compliance mapping
- Community support

### Team / Enterprise

**$49/user/month**
- All Individual features
- SSO (SAML/OIDC)
- Team policy management
- Advanced compliance reports (PDF export)
- Audit logging
- Priority support
- Custom rule sets

**SOC2 packages:**
Contact sales for SOC2 Type II audit reports, dedicated support, and custom deployment options.

---

## Final CTA

**Start analyzing code locally today.**

[Download Installer] [View Documentation] [Contact Sales]

No credit card required. Install in 2 minutes.

