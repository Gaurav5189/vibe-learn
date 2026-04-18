---
name: security-audit-full-analysis
description: Comprehensive security vulnerability detection, review, and remediation guidance for all software projects
metadata:
  model: inherit
---

## Core Capabilities

### 1. Vulnerability Detection
**Inputs:**
- Source code files (any language/framework)
- Configuration files (Docker, Kubernetes, CI/CD, cloud configs)
- Dependency manifests (package.json, requirements.txt, Cargo.toml, pom.xml, etc.)
- Infrastructure-as-code templates
- API specifications (OpenAPI/Swagger)

**Outputs:**
- Structured vulnerability report with severity levels (Critical, High, Medium, Low, Info)
- Line-by-line code references
- Exploit scenario descriptions
- CVSS scores where applicable

**Execution Steps:**
1. Parse project structure and identify technology stack
2. Run static analysis patterns for OWASP Top 10 (2026) vulnerabilities
3. Check dependency vulnerabilities against known CVE databases
4. Identify hardcoded secrets, API keys, and credentials
5. Detect insecure configurations and misconfigurations
6. Analyze authentication/authorization flows
7. Review input validation and output encoding
8. Check cryptographic implementations
9. Generate prioritized remediation recommendations

### 2. Security Review & User Validation
**Process:**
- Present findings in organized categories
- Require user confirmation before suggesting changes
- Provide risk assessment for each finding
- Allow user to mark findings as "accepted risk" or "false positive"
- Generate final report with user annotations

### 3. Latest Security Implementation Recommendations
**Based on April 2026 Standards:**
- OWASP Top 10 2026 compliance checks
- Modern cryptographic standards (post-quantum readiness where applicable)
- Zero-trust architecture patterns
- Supply chain security (SBOM, signing, verification)
- AI/ML security considerations (prompt injection protection)
- Container and cloud security hardening
- API security best practices
- Privacy-by-design implementations

---

## Security Checklist (2026 Standards)

### Authentication & Authorization
- [ ] Multi-factor authentication (MFA) enforced
- [ ] Passwordless authentication options available
- [ ] Session management prevents fixation/hijacking
- [ ] Role-based access control (RBAC) implemented
- [ ] Principle of least privilege applied
- [ ] OAuth 2.1 / OIDC used for third-party auth
- [ ] Token expiration and refresh mechanisms secure

### Input Validation & Output Encoding
- [ ] All user inputs validated and sanitized
- [ ] Parameterized queries prevent SQL injection
- [ ] Output encoding prevents XSS attacks
- [ ] Content Security Policy (CSP) headers configured
- [ ] Rate limiting implemented on sensitive endpoints
- [ ] Prompt injection protection for AI features

### Cryptography
- [ ] TLS 1.3 enforced for all communications
- [ ] Modern cipher suites (AES-GCM, ChaCha20-Poly1305)
- [ ] Keys managed via secure vault (not hardcoded)
- [ ] Regular key rotation schedule implemented
- [ ] Hashing uses Argon2 or bcrypt (not MD5/SHA1)
- [ ] Digital signatures for critical data integrity

### Secrets Management
- [ ] No secrets in source code or version control
- [ ] Environment variables or secret managers used
- [ ] Secrets encrypted at rest
- [ ] Access logging for secret retrieval
- [ ] Automated secret rotation where possible

### Dependency & Supply Chain Security
- [ ] Software Bill of Materials (SBOM) maintained
- [ ] Dependencies scanned for known CVEs
- [ ] Reproducible builds implemented
- [ ] Release signing and hash verification
- [ ] Minimum viable dependency tree
- [ ] Automated dependency update alerts

### API Security
- [ ] Authentication required on all endpoints
- [ ] Input validation on all parameters
- [ ] Rate limiting and throttling configured
- [ ] API versioning with deprecation policy
- [ ] CORS properly configured
- [ ] GraphQL introspection disabled in production

### Infrastructure & Configuration
- [ ] Default-deny security posture
- [ ] Container images scanned for vulnerabilities
- [ ] Minimal base images used
- [ ] Non-root container execution
- [ ] Network segmentation implemented
- [ ] Security groups/firewalls minimized

### Logging & Monitoring
- [ ] Security-relevant events logged
- [ ] Tamper-evident logging implemented
- [ ] Real-time alerting on anomalies
- [ ] Log retention policy defined
- [ ] Centralized log aggregation
- [ ] Incident response procedures documented

### Business Logic Security
- [ ] Threat modeling completed for user journeys
- [ ] Race condition protections implemented
- [ ] Idempotency keys for critical operations
- [ ] Business logic bypass testing performed
- [ ] Audit trails for sensitive actions

---

## Output Format

### Vulnerability Report Structure
```markdown
## [SEVERITY] Vulnerability Title

**Location:** `file/path:line_number`  
**Category:** OWASP Category / CWE ID  
**CVSS Score:** X.X (if applicable)

### Description
[Clear explanation of the vulnerability]

### Risk Assessment
- **Exploitability:** [Easy/Medium/Hard]
- **Impact:** [Critical/High/Medium/Low]
- **Affected Users:** [All/Admin/Specific roles]

### Evidence
```code
// Vulnerable code snippet