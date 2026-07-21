# Secure Software Development Lectures - Cheat Sheet

[← Back to the complete lab reference](index.html)

This page condenses Lectures 1-P1 through 9 and shows where each lecture concept is applied in the labs. Use it to move from **theory** to **evidence**:

```text
Concept -> Requirement -> Threat -> Design control -> Implementation
        -> Test or scan -> Evidence -> Residual risk
```

> The lecture slides are labelled “SIT Internal.” This page summarizes the concepts without publishing the original decks. The slides and lecturer remain authoritative for assessment wording.

## Fast lecture-to-lab map

| Lecture | Core question | Main concepts | Closest labs |
|---|---|---|---|
| 1-P1 Module Overview | Why build security into the lifecycle? | SSDLC, quality gates, security in sprints | [Lab 1](CHEAT_SHEET.md#lab-1---docker-and-docker-compose), [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements), all later labs |
| 1-P2 Secure Software Concepts I | What commonly goes wrong? | OWASP Top 10, vulnerabilities, insecure design | [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements), [X06](CHEAT_SHEET.md#x06---software-composition-analysis), [X08](CHEAT_SHEET.md#x08---static-code-analysis), [X11a](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) |
| 2-P1 Secure Software Concepts II | How do we track weaknesses and improve maturity? | CVE, NVD, CVSS, CAPEC, ATT&CK, BSIMM, SAMM, Microsoft SDL | [X06](CHEAT_SHEET.md#x06---software-composition-analysis), [X08](CHEAT_SHEET.md#x08---static-code-analysis), [X09](CHEAT_SHEET.md#x09---sonarqube) |
| 2-P2 Secure Requirements I | What must the system protect? | Functional/non-functional security requirements, CIA, authentication | [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements) |
| 3-P1 Secure Requirements II | How do we make requirements adversary-aware and testable? | Authorization, accountability, session/error/configuration management, anti-requirements, misuse cases | [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements), [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions) |
| 3-P2 Threat Modeling | What can attack this design? | Trust boundaries, DFDs, mis-actors, attack trees, STRIDE | [X03](CHEAT_SHEET.md#x03---microsoft-threat-modeling-tool), [X04](CHEAT_SHEET.md#x04---owasp-threat-dragon) |
| 4 Secure Design I | Which principles prevent design flaws? | Attack surface, secure defaults, least privilege, defense in depth, fail securely | [X03](CHEAT_SHEET.md#x03---microsoft-threat-modeling-tool), [X04](CHEAT_SHEET.md#x04---owasp-threat-dragon), [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions) |
| 5 Secure Design II | How do CIA and AAA become architecture? | Cryptography, integrity, availability, RBAC, audit design, secure architecture | [Lab 1](CHEAT_SHEET.md#lab-1---docker-and-docker-compose), [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions), [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions), [X09](CHEAT_SHEET.md#x09---sonarqube) |
| 5.5 Second-half introduction | What evidence connects design, implementation, and verification? | D2/D3 evidence, CI/CD history, login/session/access-control implementation, repository freeze | [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions), [X06](CHEAT_SHEET.md#x06---software-composition-analysis), [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions) |
| 6 Secure Implementation I | How does security become part of delivery and authentication? | DevSecOps, CI/CD, TLS/HSTS/PFS, password storage, passwordless, FIDO/passkeys | [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements), [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions), [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions) |
| 7 Secure Implementation II | Which coding practices prevent common weaknesses? | OWASP Proactive Controls, validation, ReDoS, parameterized SQL, dependency security, safe logging | [X06](CHEAT_SHEET.md#x06---software-composition-analysis), [X08](CHEAT_SHEET.md#x08---static-code-analysis), [X09](CHEAT_SHEET.md#x09---sonarqube), [X11a](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) |
| 8 Secure Implementation III | How do we verify security requirements? | Testing pyramid, unit/integration/UI tests, ASVS, WSTG, authentication and logging tests | [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions), [X11a](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap), [X11b](CHEAT_SHEET.md#x11b---fuzzing-with-burp-suite) |
| 9 Static Code Analysis I | What can code analysis prove without running the program? | SAST spectrum, data/control flow, abstract interpretation, model checking, false positives/negatives | [X08](CHEAT_SHEET.md#x08---static-code-analysis), [X09](CHEAT_SHEET.md#x09---sonarqube) |

## Lecture 1-P1 - Module overview and the SSDLC

### The central idea

Security cannot be added only after the functional features are finished. Secure software development places security activities throughout the software development lifecycle:

1. Secure concepts and governance
2. Secure requirements
3. Secure design and threat modeling
4. Secure implementation and coding
5. Secure testing and verification
6. Secure deployment, vulnerability management, and incident response

The module’s practical work follows the same order. Lab 1 establishes the development environment; Lab 2 defines security requirements; X03/X04 model threats; X05-X09 automate controls and analysis; X11 verifies a running application.

### Security quality gates

A **quality gate** is an explicit pass/fail decision before work moves forward. Security gates should use evidence rather than intuition.

| Lifecycle point | Example gate | Evidence |
|---|---|---|
| Requirements | High-risk assets have testable security requirements | Requirement and traceability matrix |
| Design | High-priority threats have approved controls | Threat-model report and design review |
| Pull request | Tests and agreed static checks pass | CI workflow run and SARIF results |
| Release candidate | Dependencies and application are assessed | SCA, SAST, DAST, and test reports |
| Deployment | Secrets, TLS, logging, recovery, and monitoring are ready | Deployment checklist and operational test |

### Security inside a sprint

Security work belongs in the backlog and definition of done:

- refine security acceptance criteria with the feature;
- update threats when architecture or data flows change;
- implement controls with the feature rather than as a separate late task;
- add positive, negative, and abuse-case tests;
- run automated checks in CI;
- record residual risk and follow-up work.

### Learning outcomes translated into deliverables

| Learning outcome | Concrete deliverable |
|---|---|
| Recognize security across the lifecycle | SSDLC plan and security gates |
| Model requirements and threats | Requirement matrix, misuse cases, DFD, STRIDE report |
| Apply design principles | Architecture decisions and control mapping |
| Implement secure code | Reviewed code, dependency controls, CI checks |
| Verify security | Automated tests, SAST/SCA/DAST reports, remediation evidence |

### How it links to the labs

- [Lab 1](CHEAT_SHEET.md#lab-1---docker-and-docker-compose) provides repeatable infrastructure for later controls.
- [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements) begins the first major security gate.
- [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions) turns repeated build/test activities into CI.
- [X06-X09](CHEAT_SHEET.md#x06---software-composition-analysis) supply automated evidence.
- [X11a/X11b](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) verify the running system from an attacker-facing perspective.

Source basis: Lecture 1-P1, Module Overview, AY2025T3.

---

## Lecture 1-P2 - Secure software concepts I

### Why application security is necessary

Firewalls and intrusion detection cannot correct insecure application logic. Application security must handle risks such as:

- missing object-level authorization;
- unsafe password recovery;
- injection caused by mixing instructions and untrusted data;
- insecure defaults and exposed administrative functions;
- weak secrets or cryptography;
- vulnerable dependencies;
- missing logging, alerting, and safe error handling.

The case-study pattern is consistent: security was omitted from the design, a developer made an implementation mistake, or delivery pressure prevented adequate review.

### Vulnerability, threat, risk, flaw, and bug

| Term | Meaning |
|---|---|
| Asset | Something valuable that needs protection |
| Threat | A potential cause of harm, including an actor or event |
| Vulnerability | A weakness that could be exploited or triggered |
| Exploit | A method or action that uses a vulnerability |
| Impact | The harm if the event occurs |
| Likelihood | How plausible/frequent the event is in context |
| Risk | A contextual combination of likelihood and impact |
| Design flaw | Security weakness in architecture or intended behavior |
| Implementation bug | Incorrect code or configuration implementing a design |

### OWASP Top 10 lecture summary

The lecture uses the 2025 categories. Memorize the idea and control family, not just the number.

| Category | Recognize it | Design/implementation response | Lab evidence |
|---|---|---|---|
| A01 Broken Access Control | User reaches another user’s object or an admin function | Deny by default; server-side role and ownership checks; test every action | Lab 2 authorization matrix; X07 negative tests; X11a alerts |
| A02 Security Misconfiguration | Defaults, samples, listings, stack traces, open cloud storage | Hardened baseline; minimum services; safe errors; reviewed configuration | Lab 1 Compose; X05 Nginx/TLS; X09 review |
| A03 Software Supply Chain Failures | Vulnerable/malicious component or update pipeline | Inventory, pinning, integrity, trusted sources, SCA, controlled updates | X06 Dependency-Check/Dependabot; X05 CI |
| A04 Cryptographic Failures | Cleartext data, weak algorithms, poor key handling, invalid certificate acceptance | Classify data; TLS; approved primitives; secret/key lifecycle | Lab 2 confidentiality; X05 TLS |
| A05 Injection | Untrusted input changes SQL, commands, HTML, or other interpreter behavior | Parameterization, contextual output encoding, validation, least privilege | X08 static rules; X11a/X11b testing |
| A06 Insecure Design | Missing control or unsafe workflow even if code matches design | Threat modeling, misuse cases, secure patterns, business limits | Lab 2; X03/X04; Lectures 3-P2 and 4 |
| A07 Authentication Failures | Credential stuffing, weak recovery, persistent/unsafe sessions | MFA, secure recovery, rate controls, robust session lifecycle | Lab 2 authentication requirements; X07/X11b tests |
| A08 Software and Data Integrity Failures | Unsigned update, compromised CI, unsafe deserialization | Verify provenance/signatures; least-privilege CI; safe formats | X05 Actions; X06 SCA; X08 CodeQL |
| A09 Security Logging and Alerting Failures | Important events missing or never acted upon | Audit design, central monitoring, alerts, protected logs, response process | Lab 2 accountability; X09 findings; X11 scans should trigger monitoring |
| A10 Mishandling Exceptional Conditions | Failing open, leaked stack trace, incomplete rollback, resource leakage | Fail closed; bounded resources; rollback; safe user errors; internal diagnostics | Lecture 3-P1 error requirements; X07 failure tests; X08 analysis |

### Example reasoning: unsafe balance functions

For `decrease(amount)` and `increase(amount)`, ask beyond the visible balance check:

- Can `amount` be negative, zero, too large, non-integer, or overflow the type?
- Can two requests race and both pass the balance check?
- Is authorization checked for the account and action?
- Is the update atomic and durable?
- Is the transaction logged without leaking sensitive data?
- What happens if the database or network fails midway?
- Can a replay duplicate the transaction?

This same reasoning reappears in Lecture 3-P1’s validation, velocity, transaction, and visibility controls.

### How it links to the labs

- [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements) converts each relevant OWASP risk into a requirement.
- [X06](CHEAT_SHEET.md#x06---software-composition-analysis) addresses supply-chain and known-component risk.
- [X08](CHEAT_SHEET.md#x08---static-code-analysis) detects selected code patterns before execution.
- [X09](CHEAT_SHEET.md#x09---sonarqube) aggregates code-quality and security findings.
- [X11a](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) tests the running application for common web weaknesses.
- [X11b](CHEAT_SHEET.md#x11b---fuzzing-with-burp-suite) explores input and authentication behavior.

Source basis: Lecture 1-P2, Secure Software Concepts x01, AY2025T3.

---

## Lecture 2-P1 - Vulnerability tracking and assurance maturity

### Vulnerability knowledge sources

| Resource | What it provides | Use it for |
|---|---|---|
| CVE | Identifier and public record for a disclosed vulnerability | Referencing the same vulnerability consistently |
| NVD | Enrichment such as severity, affected products, CPEs, references, and mitigation/fix information | Triage and version impact research |
| CVSS | Standardized severity characteristics and score | One input into prioritization, not the entire risk decision |
| CAPEC | Catalog of attack patterns and mitigations | Understanding how attackers exploit classes of weaknesses |
| MITRE ATT&CK | Real-world adversary tactics and techniques | Operational threat scenarios, detection, and defense coverage |
| CWE | Weakness classes in software/hardware | Root-cause classification and prevention guidance |

Do not confuse a **CVE** (a particular disclosed vulnerability) with a **CWE** (a type of weakness) or **CAPEC** (an attack pattern).

### CVSS versus project risk

CVSS estimates technical severity. Your prioritization must add context:

```text
Project priority = severity + reachability + exposure + asset value
                 + existing controls + active exploitation + business impact
```

A high score in an unused test-only dependency may be less urgent than a medium issue on an Internet-facing authentication path. Document the reasoning rather than silently ignoring either.

### Assurance maturity models

| Model | Purpose | Mental model |
|---|---|---|
| BSIMM | Observes software security activities used by real organizations | Compare the organization with observed practices and build an initiative |
| OWASP SAMM | Open framework for assessing and improving software assurance | Score practices, find gaps, choose a risk-based improvement roadmap |
| Microsoft SDL | Integrates security/privacy work across development phases | Apply defined security activities and gates throughout delivery |

BSIMM organizes practices into domains including governance, intelligence, development touchpoints, and deployment. SAMM expresses increasing maturity from an unfulfilled starting point through repeatable and scaled practice.

### Maturity lesson

Buying a scanner is not a mature security program. Maturity requires:

- ownership and governance;
- repeatable requirements and design review;
- trained developers and security champions;
- integrated tools with triage and remediation;
- measurement and feedback;
- deployment, monitoring, and incident response.

### How it links to the labs

- [X06](CHEAT_SHEET.md#x06---software-composition-analysis) connects dependency evidence to CVE/NVD data.
- [X08](CHEAT_SHEET.md#x08---static-code-analysis) and [X09](CHEAT_SHEET.md#x09---sonarqube) operationalize repeatable assurance activities.
- [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions) makes assurance repeatable in CI.
- [X11a](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) produces findings that must be classified, prioritized, fixed, and retested.
- The cross-lab pipeline represents a small project-level version of an SDL/SAMM improvement program.

Source basis: Lecture 2-P1, Secure Software Concepts x02, AY2025T3.

---

## Lecture 2-P2 - Secure software requirements I

### Requirement types

| Type | Question answered | Example |
|---|---|---|
| Functional requirement | What must the system do? | The LMS shall allow an instructor to upload a document. |
| Non-functional requirement | What measurable property/constraint must it have? | A login response shall complete within a defined threshold under a stated load. |
| Secure functional requirement | How must an ordinary function behave securely, including prohibited behavior? | An instructor may replace only documents belonging to modules they administer. |
| Functional security requirement | Which security service must exist? | The system shall authenticate administrators with MFA. |
| Non-functional security requirement | Which security quality/architecture constraint applies? | Sensitive traffic shall use the approved TLS configuration. |
| Secure development requirement | Which development activity must occur? | Every pull request shall pass dependency and static analysis before merge. |

### The requirements process

1. **Feasibility study** - identify stakeholders, assets, threats, attacker model, policies, standards, and regulations.
2. **Secure requirement gathering** - elicit CIA and AAA needs for features, data, and operations.
3. **Specification** - write unambiguous, measurable statements with acceptance criteria.
4. **Validation** - review necessity, consistency, feasibility, traceability, and testability with stakeholders.

### CIA + AAA

| Objective | Ask | Example requirement evidence |
|---|---|---|
| Confidentiality | Who may see which data, in which state? | Access-control test, encryption configuration, masking review |
| Integrity | Who may change it, and how is unauthorized change detected/prevented? | Validation test, authorization test, signature/hash verification |
| Availability | Which functions must remain available and recover how quickly? | Load/failover test, recovery exercise, SLA measurements |
| Authentication | How is an identity established and recovered? | MFA/login/recovery tests and session evidence |
| Authorization | What action may each identity perform on each resource? | Role/ownership matrix and negative tests |
| Accountability | Which actions must be attributable and retained? | Protected audit events and alert verification |

### Data confidentiality states

Classify and protect data:

- **at rest** - database, file, device storage, archives, backups;
- **in transit** - browser/API/service/network communication;
- **in use** - process memory, screen output, reports, logs, temporary files.

Side channels also matter. A timing difference, response length, status code, or error message may reveal information without directly returning the protected value.

### Availability terms

| Term | Meaning |
|---|---|
| SLA | Agreed measurable service commitment |
| MTD | Maximum tolerable downtime before impact becomes unacceptable |
| RTO | Target time to restore service after disruption |
| RPO | Maximum acceptable data-loss window measured backward from disruption |

“Five nines” availability is not a useful requirement unless scope, measurement interval, exclusions, dependencies, and recovery behavior are defined.

### Authentication requirement reminders

- Define user types, service identities, administrators, and recovery operators.
- Prefer long passwords/passphrases and screening against breached/common passwords over arbitrary composition rituals.
- Specify rate/velocity protection and secure recovery.
- Define MFA for sensitive roles/actions.
- Define maximum accepted length to prevent truncation or resource abuse while permitting password-manager-generated values.
- Never store recoverable plaintext passwords.

### How it links to Lab 2

[Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements) is the direct practical application:

1. Select 2-3 major functional and non-functional requirements.
2. Identify CIA/AAA assets and risks.
3. Write testable security requirements.
4. Use SecurityRAT to review applicable requirement families.
5. Persist follow-up work as owned tickets.

Use the [requirement record and traceability templates](CODE_SNIPPETS.md#lab-2---securityrat-and-requirement-templates).

Source basis: Lecture 2-P2, Secure Software Requirements x01, AY2025T3.

---

## Lecture 3-P1 - Secure software requirements II

### Authorization

Authentication answers “who are you?” Authorization answers “may this principal perform this action on this resource now?”

| Model | Decision basis | Caution |
|---|---|---|
| User-based access control | Individual identity | Becomes difficult to manage at scale |
| Role-based access control | Assigned role(s) | Roles can become overbroad; ownership checks may still be needed |
| Attribute-based/contextual control | Identity, resource, action, environment attributes | More expressive but easier to misconfigure |

Authorization must be enforced server-side at every protected action. A hidden button is not access control.

### Accountability and audit design requirements

Specify:

- what events are recorded;
- actor/service identity;
- action, target, result, time, and correlation identifier;
- protected storage and access;
- monitoring and alerting;
- retention, rotation, archive, and disposal;
- time synchronization;
- prohibited data such as passwords, secret tokens, or unnecessary personal data.

Logging is a **detective control**. It becomes useful only when protected, monitored, and connected to a response process.

### General application security requirements

**Session management**

- unpredictable and unique session identifiers;
- expiration, idle timeout, and maximum lifetime;
- renewal after authentication/privilege change;
- logout and server-side invalidation;
- concurrent-session policy;
- protection against reuse, theft, and fixation.

**Error management**

- safe user-facing message;
- no stack trace, secret, query, filesystem path, or internal architecture disclosure;
- detailed internal record tied to a correlation ID;
- fail closed for security decisions;
- rollback/release resources during partial failure.

**Configuration management**

- no production secret in source, images, or committed configuration;
- environment-specific, least-privilege values;
- protected initialization, rotation, and disposal;
- secure defaults and reviewed changes.

**Code management**

- authenticated source control;
- protected branches and review;
- secret scanning and dependency analysis;
- reproducible controlled deployment;
- no hard-coded credentials.

### Operational requirements

- deployment environment and user population;
- development/test/production separation;
- archive location, format, encryption, retrieval, retention, and deletion;
- business continuity and recovery;
- third-party/procurement requirements;
- applicable regulations such as PDPA/GDPR;
- anti-tampering/signing/licensing when relevant to COTS software.

### Anti-requirements and misuse cases

An anti-requirement states an unacceptable behavior:

```text
Ordinary requirement: The system shall create an identifier when given valid inputs.
Anti-requirement: The system shall not create duplicate, expired, or unauthorized identifiers.
```

A misuse case extends an ordinary use case with:

- a mis-actor;
- the attacker’s goal/action;
- the threatened legitimate use case;
- a mitigating security use case;
- the resulting testable requirement.

For a login flow, misuse cases may include credential capture, brute force, recovery abuse, session fixation, and user enumeration.

### ATM analysis mnemonic: IVTV

| Area | Questions |
|---|---|
| Input validation | Length, range/boundary, characters/encoding, syntax, and business semantics? |
| Velocity | How many attempts, transactions, bytes, or changes are permitted per actor/time/resource? |
| Transactions | What happens on interruption, replay, concurrency, rollback, and partial completion? |
| Visibility | What sensitive value remains on screen, in a receipt, response, log, cache, or side channel? |

### Framework links

- **PCI Software Security Framework**: payment software requirements and evidence-based assessment.
- **NIST SSDF**: high-level secure development practices, including security requirements, risk-informed design, threat modeling, training, and rigorous review of high-risk areas.

### How it links to the labs

- [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements): record authorization, audit, session, error, configuration, and operational requirements.
- [X03/X04](CHEAT_SHEET.md#x03---microsoft-threat-modeling-tool): translate misuse cases into threats on architecture elements and flows.
- [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions): automate negative, boundary, concurrency, and failure tests.
- [X08](CHEAT_SHEET.md#x08---static-code-analysis): detect selected error-handling, injection, and unsafe coding patterns.
- [X11b](CHEAT_SHEET.md#x11b---fuzzing-with-burp-suite): exercise input and velocity behavior in an authorized target.

Source basis: Lecture 3-P1, Secure Software Requirements x02, AY2025T3.

---

## Lecture 3-P2 - Threat modeling

### Threat versus risk

- A **threat** is a potential cause or scenario of harm.
- A **vulnerability** is the weakness that permits it.
- **Risk** considers the scenario in context, including likelihood and impact.
- A **control/mitigation** reduces likelihood, impact, or both.
- **Residual risk** remains after controls and must be accepted, reduced further, transferred, or avoided.

### Threat agents

Human threat agents may include:

- ignorant or error-prone users;
- accidental discoverers and curious users;
- script users;
- insiders and rogue administrators;
- organized cybercriminals;
- third parties/suppliers;
- nation-state or military actors.

Non-human threats include operating-system failure, malware/ransomware, network failure/attack, automation, and AI-enabled threats.

An attacker model must state capability, access, location, privilege, knowledge, motivation, and constraints.

### What to identify before STRIDE

1. Assets and security objectives
2. Physical and logical topology
3. External entities and human/non-human actors
4. Processes and privileged functionality
5. Data stores and classified data elements
6. Entry and exit points
7. Directional, labelled data flows
8. Trust boundaries where control or privilege changes
9. External dependencies and assumptions

### Data Flow Diagram elements

| Element | Meaning | Threat-model question |
|---|---|---|
| External entity | User or external system | How is it identified and constrained? |
| Process | Transforms data/performs work | Which privilege and validation does it use? |
| Data store | Persists data | Who can read/change/delete it, and how is it protected? |
| Data flow | Moves specific data | Which protocol, identity, direction, validation, and protection? |
| Trust boundary | Trust/privilege/ownership changes | Which checks occur when the flow crosses it? |

### Different attacker positions change controls

- **Web attacker**: can send parallel/malformed requests and observe responses.
- **Network attacker**: can read, intercept, replay, delay, or modify unprotected traffic.
- **Co-located attacker**: may access local files, memory, keyboard, or display.

Do not assume away a realistic attacker merely because the resulting controls are inconvenient.

### Threat-modeling process

1. Define security requirements and scope.
2. Create the application/architecture diagram.
3. Identify and prioritize applicable threats.
4. Select and implement controls.
5. Document assumptions, status, ownership, and residual risk.
6. Validate that the model is accurate and mitigations work.
7. Revisit the model when the system or threat landscape changes.

### Attack trees

An attack tree begins with the attacker’s goal at the root. Branches describe alternative or combined sub-goals; leaves are concrete ways to achieve them. Use trees to discover paths, prerequisites, shared controls, and where one mitigation blocks multiple branches.

### STRIDE memory table

| STRIDE | Attacker goal | Violated property | Typical controls |
|---|---|---|---|
| Spoofing | Pretend to be another identity | Authentication | MFA, certificates, secure sessions, service identity |
| Tampering | Change data/code/configuration | Integrity | Authorization, validation, signatures/MACs, protected deployment |
| Repudiation | Deny an action | Accountability/non-repudiation | Protected audit records, identity, timestamps, correlation |
| Information disclosure | Read protected information | Confidentiality | Encryption, masking, access control, minimization, safe errors |
| Denial of service | Prevent legitimate use | Availability | Limits, quotas, timeouts, redundancy, isolation, graceful degradation |
| Elevation of privilege | Gain unauthorized capability | Authorization | Least privilege, deny by default, isolation, server-side checks |

### How it links to X03 and X04

- [X03 Microsoft Threat Modeling Tool](CHEAT_SHEET.md#x03---microsoft-threat-modeling-tool) generates STRIDE candidates from a DFD and tracks threat status/mitigation.
- [X04 OWASP Threat Dragon](CHEAT_SHEET.md#x04---owasp-threat-dragon) provides a cross-platform diagram, rule engine, open-threat highlighting, and reporting.
- Use the [threat-model records](CODE_SNIPPETS.md#x03-and-x04---threat-model-records) to capture evidence beyond the diagram.

Source basis: Lecture 3-P2, Threat Modeling, AY2025T3.

---

## Lecture 4 - Secure software design I

### General design progression

```text
Architecture -> Detailed design -> Construction design
```

- Architecture defines views, patterns, components, topology, and major flows.
- Interface design defines the contract between components.
- Component design defines internal structure and behavior, often with class and sequence diagrams.

Security must be represented in all three. A design may be functionally correct yet insecure.

### Flaw versus bug

- **Flaw**: the intended design itself lacks or misplaces a control.
- **Bug**: the implementation fails to realize an otherwise sound design.

Scanners often find bugs more readily than flaws. Threat modeling, misuse cases, architecture review, and design principles are essential for flaws.

### Ten OWASP secure design principles

| Principle | Meaning | Quick review question |
|---|---|---|
| Minimize attack surface | Remove/restrict unnecessary features, interfaces, privileges, and exposure | Can this endpoint, port, parser, role, or feature be removed or narrowed? |
| Establish secure defaults | Initial behavior protects users without extra configuration | Is access denied, TLS enabled, and exposure minimized by default? |
| Least privilege | Grant only the capability needed, for only as long as needed | Does this user/service/database role need every granted permission? |
| Defense in depth | Use independent layers that fail differently | If the first control fails, what still prevents/detects the attack? |
| Fail securely | Exceptions do not grant access or leave unsafe partial state | Does every error path deny/rollback/release safely? |
| Do not trust services | Validate third-party input/output and plan for compromise/failure | What if the dependency is malicious, wrong, slow, or unavailable? |
| Separation of duties | Split conflicting powers to reduce fraud/abuse | Can one actor request, approve, execute, and conceal the same action? |
| Avoid security by obscurity | Do not depend on secrecy of design/source as the main control | Would the system remain secure if its architecture were known? |
| Keep security simple | Reduce unnecessary complexity and ambiguous logic | Can the control be smaller, centralized, explicit, and easier to verify? |
| Fix correctly | Address root cause, variants, tests, and regressions | Where else does the same pattern exist, and which test prevents recurrence? |

### Fail-secure example

Unsafe structure:

```text
isAdmin = true
try to check role
if check throws, isAdmin remains true
```

Secure structure:

```text
isAdmin = false
try to check role
only set true after explicit successful verification
on exception: deny, log safely, and return a controlled error
```

### Attack surface mapping

Inventory entry/exit points:

- forms, fields, headers, cookies, and APIs;
- files, databases, caches, and local storage;
- emails, messages, queues, and webhooks;
- runtime arguments and environment/configuration;
- login/recovery and administration;
- CRUD and business workflows;
- transactions and external integrations;
- operational and monitoring interfaces.

Group thousands of individual points into meaningful types, then review authentication, authorization, validation, data, exposure, and monitoring consistently.

### Security architecture

- **Physical architecture**: hosts, networks, zones, deployment locations, and physical components.
- **Logical architecture**: services, software components, protocols, identities, data, and interactions.

Security architecture is a deliberate set of controls, safer processes, and secure defaults. It is not a collection of security products without a threat/control rationale.

### How it links to the labs

- [Lab 1](CHEAT_SHEET.md#lab-1---docker-and-docker-compose): networks, published ports, service identities, volumes, and separation are architecture decisions.
- [X03/X04](CHEAT_SHEET.md#x03---microsoft-threat-modeling-tool): use DFDs and STRIDE to expose design flaws.
- [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions): Nginx adds a controlled edge layer; CI creates repeatable gates.
- [X06](CHEAT_SHEET.md#x06---software-composition-analysis): “do not trust services” includes third-party components and supply chains.
- [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions): “fix correctly” requires a regression test.
- [X08/X09](CHEAT_SHEET.md#x08---static-code-analysis): analysis validates selected implementation consequences but cannot prove the design is secure.

Source basis: Lecture 4, Secure Software Design x01, AY2025T3.

---

## Lecture 5 - Secure software design II

### CIA and AAA become architecture

| Objective | Design techniques |
|---|---|
| Confidentiality | Data classification, minimization, masking, TLS, encryption at rest, protected keys/secrets |
| Integrity | Validation, hashes/MACs/signatures, referential integrity, authorization, transactions, resource locking |
| Availability | Replication, failover, horizontal/vertical scaling, limits, timeouts, recovery, dependency isolation |
| Authentication | Knowledge/possession/inherence factors, MFA, SSO, secure enrollment and recovery |
| Authorization | RBAC/UBAC/context, least privilege, separation of duties, ownership and action checks |
| Accountability | Audit fields, protected collection/storage, correlation, monitoring, retention, no plaintext secrets |

### Symmetric versus asymmetric cryptography

| Property | Symmetric | Asymmetric |
|---|---|---|
| Keys | Same secret for encryption/decryption | Public/private key pair |
| Speed | Fast; suited to bulk data | Slower; suited to identity, key exchange, signatures |
| Key distribution | Shared secret must be exchanged/protected | Public key can be distributed; private key remains secret |
| Non-repudiation/signatures | Not provided by shared-key encryption alone | Digital signatures can provide origin/integrity evidence |
| Examples | AES | RSA/ECC families depending approved use |

Real protocols commonly combine both: asymmetric mechanisms establish identity/key agreement; symmetric authenticated encryption protects the session efficiently.

### Certificates

An X.509 certificate binds a public key to an identity through a certification chain. A secure client verifies:

- trusted issuer/chain;
- hostname/identity;
- validity period;
- allowed usage and algorithm;
- revocation/status according to the system design.

TLS protects data in transit only when certificate validation and protocol configuration are correct. It does not replace application authorization or secure storage.

### Hashing, salts, and password storage

A cryptographic hash maps variable input to a fixed-size digest and supports integrity use cases. A salt should be random and unique per password record to defeat precomputed tables and identical-hash comparison.

> For password storage, do not use plain MD5, SHA-1, or fast unsalted SHA-256. Use an approved, deliberately slow password-hashing function through a maintained library, with per-password salt and appropriate work parameters. A salt is not secret; a pepper, if used, is a separate protected secret.

Password records may also need metadata such as algorithm/version, work factor, account state, failed-attempt counters, password-change time, and session/recovery state. Do not log the password or resulting verifier.

### Referential integrity

Foreign-key and lifecycle rules keep references valid. Design deletion explicitly:

- restrict deletion while dependents exist;
- cascade only when the business/security meaning is safe;
- soft-delete with retention/access rules when required;
- archive/anonymize according to policy;
- audit high-impact changes.

### Resource locking and transactions

Concurrency can violate integrity even when each individual request looks valid. Consider:

- atomic database transactions;
- row/version locking or optimistic concurrency;
- idempotency keys for retried actions;
- replay protection;
- consistent ordering of multi-step operations;
- rollback and compensation on partial failure.

### Availability design

- **Replication** creates redundant copies/services but introduces consistency and failover questions.
- **Failover** switches from failed active capacity to standby capacity.
- **Vertical scaling** adds resources to an existing node.
- **Horizontal scaling** adds nodes and requires distribution/state design.

Redundancy is not a backup, and a backup is not proven until restore is tested.

### Authentication and passwordless design

Authentication factors:

- something known;
- something possessed;
- something inherent.

Passwordless systems still need secure enrollment, device binding, recovery, revocation, phishing resistance, session security, and audit. Removing a password does not remove authentication threats.

### Authorization and admin design

Avoid an unrestricted “super admin.” Separate operational administration from acting as a normal customer. Consider:

- who creates the initial administrator;
- MFA and step-up authentication;
- recovery without bypass/backdoor;
- dual approval for high-impact actions;
- just-in-time/time-limited privilege;
- complete audit and alerting;
- inability to read user secrets unnecessarily.

### Accountability design

Useful audit records answer who, what, where, when, target, result, and correlation. They must balance forensic value with minimization:

- do not store passwords, secret tokens, full payment data, or unnecessary personal data;
- protect logs against unauthorized reading and tampering;
- centralize/monitor important events;
- rotate, retain, archive, and dispose according to policy.

### Four-part secure design process

1. **Architecture** - physical/logical topologies, services, ports, protocols, identities, authentication/authorization, data, actors, audit.
2. **Identify threats** - boundaries, flows, entry/exit points, mis-actors, attack trees, STRIDE/OWASP/CWE lists.
3. **Prioritize and implement controls** - validation, safe errors, encryption/hashing, replication, access control, MFA, logging, and other contextual measures.
4. **Document and validate** - threat profile, residual risk, verification/validation report, and evidence.

### Data access control matrix

Build a matrix before implementation:

| Actor/service | Data/resource | Create | Read | Update | Delete | Special conditions/audit |
|---|---|---:|---:|---:|---:|---|
| Customer | Own profile | - | Allow | Allow | Limited | Re-authenticate sensitive changes |
| Customer | Other profile | - | Deny | Deny | Deny | Log repeated attempts |
| Support | Account status | - | Limited | Limited | Deny | Ticket and audit required |
| Administrator | Security configuration | Allow | Allow | Allow | Restricted | MFA, dual approval, full audit |
| Application service | Required tables | As needed | As needed | As needed | Usually restricted | Dedicated DB role |

### How it links to the labs

- [Lab 1](CHEAT_SHEET.md#lab-1---docker-and-docker-compose): implement topology, service separation, networks, persistence, and least exposure.
- [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements): express CIA/AAA and lifecycle needs before choosing controls.
- [X03/X04](CHEAT_SHEET.md#x03---microsoft-threat-modeling-tool): document architecture and systematically identify threats.
- [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions): implement TLS termination, reverse-proxy boundaries, and CI gates.
- [X06](CHEAT_SHEET.md#x06---software-composition-analysis): assess external component/dependency trust.
- [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions): verify authentication, authorization, concurrency, failure, and regression behavior.
- [X08/X09](CHEAT_SHEET.md#x08---static-code-analysis): inspect implementation against selected secure-design consequences.
- [X11a/X11b](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap): validate attacker-visible behavior on an authorized running target.

Source basis: Lecture 5, Secure Software Design x02, AY2025T3.

---

## Lecture 5.5 - Introduction to the second half

### What changes after secure design

The second half moves from deciding **what security should exist** to proving **what was implemented and verified**:

```text
Secure design -> Secure implementation -> Static analysis
              -> Secure verification/DAST -> Secure deployment
              -> Secure harness engineering
```

The key lesson is evidence. A claim such as “we used HTTPS” is incomplete without the relevant configuration, filenames, code, test result, and explanation of why the control satisfies a requirement or mitigates a threat.

### Deliverable 2 evidence map

| Area | Explain | Retain as evidence |
|---|---|---|
| Design review | Security refinements made after Deliverable 1 and why they improve the design | Updated diagram, requirement/threat IDs, design decision and rationale |
| CI/CD | Actual pipeline, tools, stages, triggers, and security gates | Workflow files and repeated successful run history from early implementation onward |
| Repository organization | How directories/files support later QA and source review | Repository tree, README, class/component-to-file mapping, contribution history |
| Deployment automation | How repository changes reach the target environment | Authorized deployment workflow, protected credentials, successful deployment evidence |
| Login | Server authentication, user authentication, TLS choices, key storage, password storage | Sequence diagram, relevant code/configuration, safe test evidence |
| Session management | Token/cookie structure, generation, transport, integrity and attack prevention | Cookie settings, rotation/invalidation logic, hijacking/replay tests |
| Access control | How each actor is restricted to permitted actions/resources | Access matrix, server-side enforcement code, positive and negative tests |
| Secure coding | Which recommended practices were followed | Filename-specific snippets, review checklist and analysis findings |
| Test automation | Dependency inventory, dependency checks, and automated tests | Lockfile/SBOM, SCA results, test code, CI results |

### Repository freeze and QA handoff

At the implementation submission point, the repository becomes the artifact reviewed by another QA/security team. Prepare it as though the reviewers cannot ask you to repair missing context:

- keep build and test instructions reproducible;
- remove real secrets and local-only generated data;
- identify the exact submitted commit;
- make diagrams and documentation match that commit;
- preserve CI, test and analysis evidence;
- make filenames and directory purpose understandable;
- record known limitations and residual risk honestly.

> Dates, page limits, practical-test arrangements and submission rules can change. Use the current LMS/lecturer announcement for administrative details; use this page for the technical content.

### How it links to the labs

- [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions) supplies TLS, reverse-proxy and CI/CD evidence.
- [X06](CHEAT_SHEET.md#x06---software-composition-analysis) supplies the dependency inventory and vulnerability analysis.
- [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions) supplies automated unit/integration/UI evidence.
- [X08/X09](CHEAT_SHEET.md#x08---static-code-analysis) provide source-analysis and quality-gate evidence.
- [X11a/X11b](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) reflect the later QA/security-team perspective.

Source basis: Lecture 5.5, Second Half Intro, AY2025T3.

---

## Lecture 6 - Secure implementation I

### DevOps, DevSecOps, and small changes

**DevOps** joins development and operations so software can be improved continuously through smaller, lower-risk changes. **DevSecOps** integrates security and compliance into that same delivery loop rather than placing them in a separate late review.

```text
Commit -> Build -> Test -> Security checks -> Delivery -> Deployment -> Monitor
           ^                                                        |
           +---------------- feedback and remediation --------------+
```

Security-relevant pipeline properties include:

- version-controlled workflow definitions;
- least-privilege workflow permissions;
- protected branches and reviewed changes;
- pinned/trusted actions and dependencies;
- secrets supplied through protected stores, not source code;
- failure stops promotion;
- retained logs, reports and artifacts;
- reproducible build and test stages.

### Continuous integration, delivery, and deployment

| Term | Meaning |
|---|---|
| Continuous integration | Frequently merge small changes and automatically build/test them |
| Continuous delivery | Keep a validated release ready; production deployment still has a deliberate/manual decision |
| Continuous deployment | Automatically deploy every change that passes all required gates |

For this module, the minimum emphasis is a CI pipeline with **build and test**. Security checks become stronger when SCA, SAST and other gates are added at the appropriate stages.

### TLS is necessary but not the whole security story

TLS can provide confidentiality and integrity in transit plus server authentication when configured and validated correctly. It does not automatically prevent:

- phishing on an attacker-controlled site;
- application-level broken access control;
- stolen endpoints or active sessions;
- insecure password recovery;
- unsafe storage or logging;
- a user ignoring certificate warnings.

Important implementation points:

- redirect HTTP to HTTPS and consider HSTS to resist downgrade/SSL-stripping paths;
- enable supported modern protocol versions and approved cipher suites;
- prefer key agreement that provides **perfect forward secrecy**, so later server-key compromise does not reveal captured past sessions;
- protect the server private key and restrict its filesystem/runtime access;
- automate certificate renewal and test expiry/failure behavior;
- verify certificate chain, hostname and validity rather than merely “using encryption.”

### Password storage and authentication reasoning

A conventional server receives a password through a protected TLS channel and compares a password-hash result with a stored verifier. Because hashing is one-way, the server cannot recover the original password from the verifier.

Do not confuse these layers:

| Layer | Protects | Does not by itself solve |
|---|---|---|
| TLS | Credentials and session traffic in transit | Phishing, authorization, insecure endpoint/server |
| Password hashing | Stored password verifiers after database exposure | Password interception, weak/reused passwords, account recovery |
| MFA | Adds another factor to authentication | Session theft, approval fatigue, recovery bypass |
| Rate limiting/lockout | Slows online guessing | Offline cracking of stolen verifiers |

The lecture also introduces Secure Remote Password (SRP) as an example of a password-authenticated key-exchange design and uses it to encourage deeper analysis rather than “security theatre.” The exam-worthy point is the reasoning: identify exactly what a protocol protects, its adoption/interoperability constraints, and the attacks that remain.

### Passwordless authentication

“Passwordless” may refer to a one-time email/SMS link or code, an authenticator approval, or a cryptographic passkey. These have different assurance properties.

| Method | Main strength | Key risks/limitations |
|---|---|---|
| Email link/code | Familiar and easy to deploy | Email account compromise, forwarding, phishing, replay, delayed delivery |
| SMS code | Broad device support | SIM swap, interception, phone-number recycling, phishing |
| Push approval | Convenient possession-based flow | Approval fatigue, wrong-session approval, device compromise |
| FIDO/WebAuthn passkey | Public-key challenge response; origin-bound and phishing-resistant | Enrollment/recovery, device ecosystem, synchronization, user confusion and account portability |

One-time-code controls from the lecture include:

- accept only the newest issued code/link;
- invalidate it after successful use;
- limit failed attempts;
- use a short expiry;
- rate-limit requests and prevent user enumeration;
- bind the result to the intended user, transaction and session;
- log security events without logging the secret.

### FIDO/passkey mental model

Registration creates a public/private key pair. The service stores the public key; the authenticator protects the private key. During login, the service sends a fresh challenge, the authenticator signs it after local user verification, and the service verifies the signature and origin-related data.

```text
Registration: authenticator creates key pair -> service stores public key
Login:        service challenge -> local user verification -> signed response
              -> service verifies signature, origin and challenge freshness
```

Passkeys reduce password and phishing exposure, but the design still requires secure enrollment, recovery, revocation, new-device setup, session management and audit.

### How it links to the labs

- [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements) defines authentication, recovery, session and cryptographic requirements.
- [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions) implements the reverse proxy, TLS and GitHub Actions pipeline.
- [X06](CHEAT_SHEET.md#x06---software-composition-analysis) adds dependency security to the pipeline.
- [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions) makes authentication and failure behavior repeatable tests.
- [X11a/X11b](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) can verify transport, authentication and session behavior on an authorized target.

Source basis: Lecture 6, Secure Implementation, AY2025T3.

---

## Lecture 7 - Secure implementation II: coding

### Secure-coding reference stack

The lecture treats secure coding as a disciplined use of maintained standards, framework controls and review checklists:

| Reference | Primary use |
|---|---|
| OWASP Top 10 Proactive Controls | Short defensive control priorities for developers |
| OWASP Cheat Sheet Series | Focused implementation guidance for a specific security topic |
| OWASP Security Knowledge Framework | Framework/language-oriented secure coding examples and knowledge |
| OWASP Secure Coding Practices Quick Reference Guide | Broad checklist covering validation, identity, sessions, access, crypto, errors, data, communications, databases and files |

Always match guidance to the project’s language/framework and current version. A copied snippet is not evidence until its assumptions, error handling, tests and surrounding authorization are understood.

### Proactive controls emphasized in the lecture

| Control | Main implementation lesson | Closest lab |
|---|---|---|
| C2 Use Cryptography to Protect Data | Use established protocols/libraries; protect data at rest and in transit; do not invent cryptography | [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions) |
| C3 Validate All Input and Handle Exceptions | Validate early and server-side, encode for the output context, and fail safely | [X08](CHEAT_SHEET.md#x08---static-code-analysis), [X11a](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) |
| C6 Keep Components Secure | Prefer secure framework features; inventory, monitor and update third-party code | [X06](CHEAT_SHEET.md#x06---software-composition-analysis) |
| C9 Implement Security Logging and Monitoring | Record security events safely, protect integrity and centralize monitoring | [X09](CHEAT_SHEET.md#x09---sonarqube), operational evidence |

### Input validation rules

Input validation rejects malformed or out-of-policy data before it reaches deeper components. It is one layer and does not replace parameterized SQL or contextual output encoding.

- validate as early as practical and repeat enforcement at the trusted server boundary;
- prefer native type/schema validators and strict type conversion;
- define numeric/date ranges and string/array size limits;
- use an allowlist for small enumerations and structured values;
- anchor regular expressions to the complete input where appropriate;
- normalize Unicode consistently before security comparisons;
- reject invalid input with controlled errors;
- validate syntax and business semantics separately.

**Allowlist vs blocklist:** an allowlist defines what is accepted. A blocklist tries to enumerate dangerous strings and is easily bypassed while also rejecting legitimate inputs such as names containing apostrophes.

### ReDoS and “evil regex”

Backtracking regex engines can explore exponentially many paths for a crafted non-matching input. Dangerous patterns often contain a repeated group whose contents also repeat or overlap, for example `(a+)+`.

Risk reduction:

- avoid nested ambiguous quantifiers and overlapping alternatives;
- bound input length before regex processing;
- use simple parsing or a non-backtracking engine where appropriate;
- test adversarial near-matches, not only valid examples;
- use time/resource limits as containment, not as the primary correction;
- never allow an untrusted user to supply an executable regex without strong controls.

### SQL injection prevention

Preferred order of defense:

1. Use prepared statements/parameterized queries.
2. Use safely constructed stored procedures where appropriate.
3. Apply allowlist validation for identifiers or values that cannot be parameterized.
4. Treat escaping as a fragile last resort specific to the exact interpreter/context.
5. Apply least privilege to the application’s database identity.

```text
SQL structure: fixed by developer
Untrusted value: bound as data through a parameter
Database account: allowed only the operations the application needs
```

Validation alone is not the primary SQL-injection defense because a value acceptable to the business may still contain characters meaningful to SQL.

### Secure logging

- encode or validate dangerous control characters to prevent log injection;
- never log passwords, session IDs, access tokens, full card data or unnecessary personal data;
- protect log access and integrity;
- forward important logs to a separate/central service so one compromised node cannot erase all evidence;
- define rotation, retention, monitoring and alerting;
- include useful correlation and outcome fields without exposing secrets.

### Docker implementation workflow

A `Dockerfile` assembles a repeatable application image. Compose describes and runs the application’s multiple services. Security review must include:

- trusted/minimal base images and pinned versions;
- non-root runtime users where possible;
- no secrets baked into image layers;
- small build context and `.dockerignore`;
- only required ports and capabilities;
- reproducible local build/test behavior matching CI.

### How it links to the labs

- [Lab 1](CHEAT_SHEET.md#lab-1---docker-and-docker-compose) applies the Dockerfile/Compose workflow and service isolation.
- [X05](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions) applies cryptographic transport and repeatable pipeline controls.
- [X06](CHEAT_SHEET.md#x06---software-composition-analysis) inventories and monitors components.
- [X08/X09](CHEAT_SHEET.md#x08---static-code-analysis) detect selected injection, validation, logging and quality patterns.
- [X11a/X11b](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) probe validation and injection behavior on an explicitly authorized target.

Source basis: Lecture 7, Secure Implementation 2 - Coding, AY2025T3.

---

## Lecture 8 - Secure implementation III: testing

### Test early and test often

Security testing belongs throughout development. Waiting for a completed system makes design and implementation faults more expensive to correct.

The testing pyramid suggests many fast unit tests, fewer integration/service tests, and a smaller number of slower UI/end-to-end tests:

| Test level | Scope | Security examples |
|---|---|---|
| Unit | Function/class/component in isolation | Validator boundaries, authorization predicate, token expiry calculation, safe exception result |
| Integration | Multiple components and real interfaces | API plus database ownership check, session rotation, transaction rollback, service permissions |
| UI/end-to-end | User-visible flow through the system | Login/logout, role-specific navigation, recovery, generic error message, browser security behavior |
| Manual security test | Contextual/adversarial exploration | Business logic abuse, chained weaknesses, usability/security tradeoffs |

The purpose is not merely code coverage. A security test verifies that an implemented control satisfies its security requirement under normal, boundary, negative, abuse and failure conditions.

### Requirements-to-tests chain

```text
Misuse case / risk
  -> security requirement
  -> ASVS verification requirement
  -> automated and/or manual test case
  -> expected secure result
  -> evidence, finding and remediation
```

Not every security test can be automated. Combine repeatable automation with focused manual review and testing.

### OWASP ASVS

The Application Security Verification Standard is a catalog of verifiable application-security requirements. The lecture groups them into 14 areas including architecture/threat modeling, authentication, sessions, access control, validation, cryptography, logging, data protection, communications, malicious code, business logic, files, APIs and configuration.

| Level | Intended assurance idea |
|---|---|
| Level 1 | Low-assurance baseline; requirements can be assessed mainly through penetration testing |
| Level 2 | Recommended for most applications handling data that needs protection |
| Level 3 | Highest assurance for critical/high-value or highly sensitive applications |

Choose the target level based on risk. Do not claim a complete level from a partial scanner run.

### OWASP WSTG

The Web Security Testing Guide turns security topics into test objectives and procedures. Categories covered in the lecture include:

- information gathering;
- configuration/deployment;
- identity, authentication and authorization;
- session management;
- input validation;
- error handling;
- weak cryptography;
- business logic;
- client-side behavior.

ASVS helps state **what must be verified**; WSTG helps design **how to test it**; a checklist/report preserves **what was actually done and found**.

### Authentication test examples

For a password or login requirement, test more than one “happy path”:

- accepted minimum/maximum lengths and characters;
- long passphrases, spaces and Unicode where required;
- commonly compromised password rejection;
- change/reset requiring the correct current proof;
- history/reuse behavior without creating a predictable bypass;
- encrypted credential transport;
- no default credentials;
- generic responses that do not reveal whether an account exists;
- consistent protection across web, mobile and alternate endpoints.

### Lockout, CAPTCHA, and recovery

Test lockout thresholds incrementally with an account you are authorized and able to lock. Verify when the lock occurs, how long it lasts, whether it applies consistently, whether it enables denial-of-service against victims, and whether unlock/recovery can be abused.

A CAPTCHA can slow automation but should not replace server-side rate control/lockout. Test whether it can be omitted, replayed, bypassed through another step/API, trusted only client-side, or treated as successful on errors.

An unlock link should be unpredictable, single-use, short-lived and bound to the intended account/process. Unlocking is distinct from changing/recovering a password even when some security practices overlap.

### Logging verification

| Area | Verify |
|---|---|
| Content | Required security events are recorded; secrets and unnecessary personal data are excluded |
| Location | Important evidence is sent away from the application host or otherwise protected from local compromise |
| Rotation/retention | Policy duration is met; permissions remain restrictive; an attacker cannot cheaply rotate away evidence |
| Access control | Raw logs and search tools are separated from ordinary user/admin roles |
| Review/alerting | Relevant patterns are detected, such as concentrated 40x probing or repeated 50x failures |

### Security-test report

A useful report communicates risk clearly:

1. scope, authorization, environment and limitations;
2. executive summary;
3. method and test coverage;
4. reproducible findings with evidence and impact;
5. recommended correction and references;
6. severity/context, status and residual risk;
7. appendices such as the completed checklist.

Lecture 8 also frames the later QA handoff: inspect the implementation explanation, perform manual source review, run static analysis and DAST, distinguish your own application from the assigned application, and recommend fixes supported by CWE/CVE or other reliable references where relevant.

### How it links to the labs

- [Lab 2](CHEAT_SHEET.md#lab-2---secure-software-requirements) provides the requirements and acceptance criteria to verify.
- [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions) implements the unit/integration/Selenium automation pyramid in CI.
- [X08/X09](CHEAT_SHEET.md#x08---static-code-analysis) provide automated source review and triage evidence.
- [X11a](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) applies DAST and WSTG-style thinking to an authorized running application.
- [X11b](CHEAT_SHEET.md#x11b---fuzzing-with-burp-suite) explores boundary, malformed and repeated inputs within explicit scope.

Source basis: Lecture 8, Secure Implementation 3 - Testing, AY2025T3.

---

## Lecture 9 - Static code analysis I

### Static versus dynamic analysis

**Static analysis** examines source code or another program representation without executing the program. **Dynamic analysis** observes a running program. Both are necessary because they see different evidence.

| Static analysis strengths | Static analysis limits |
|---|---|
| Can reason across many paths and incomplete code | Usually checks only selected properties |
| Finds recurring patterns early | Can miss real issues (false negatives) |
| Fits editor, commit and CI feedback loops | Can report non-issues (false positives) |
| Can trace data/control flow beyond one test input | Runtime configuration, environment and business context may be absent |
| Enforces coding/API rules consistently | Complex heaps, reflection, dynamic behavior and external services are difficult |

Static analysis complements manual code review, dependency analysis and DAST; it does not certify that an application is secure.

### What static analysis can detect

- buffer and memory errors in applicable languages;
- null dereferences and uninitialized data;
- unused/dead code and suspicious initialization;
- unvalidated or tainted input reaching a dangerous sink;
- API/framework rule violations;
- hard-coded secrets or suspicious constants;
- resource leaks and some race conditions;
- unsafe coding patterns and broken encapsulation.

### Spectrum of analysis

```text
Simple syntax/style checks
  -> pattern matching
  -> type-aware checks
  -> control/data-flow analysis
  -> abstract interpretation / model checking
  -> program verification / theorem proving
```

Power and precision usually cost more compute, setup, annotations, tuning and developer effort.

### Core techniques

| Technique | Mental model | Example use |
|---|---|---|
| Pattern matching | Search tokens/syntax for a known shape | Suspicious hard-coded credential or banned function |
| Data-flow analysis | Track where values/taint/definitions travel | Untrusted request input reaches SQL/command sink |
| Control-flow analysis | Model basic blocks and possible branches/jumps | Variable may remain uninitialized on one path |
| Abstract interpretation | Approximate many concrete runtime states with a manageable abstract state | Possible divide-by-zero or null state |
| Model checking | Check whether a model satisfies a stated property | Forbidden state or sequence is reachable |
| Constraint solving | Determine values/path conditions that satisfy program constraints | Produce a path demonstrating a possible failure |
| Program verification/theorem proving | Prove specified properties using formal assertions/logic | High-assurance safety/security-critical property |

A **basic block** is a straight-line sequence with entry at the beginning and control transfer at the end. A control-flow graph connects these blocks with directed edges. Data-flow analysis then reasons about values or facts along those possible paths.

### Soundness, approximation, and findings

Because programs can have loops, huge state spaces and dynamic behavior, analysis uses abstractions. An over-approximation considers behavior that may not occur and can create false positives; an under-approximation may omit behavior and create false negatives.

| Result | Meaning |
|---|---|
| True positive | Tool reports a real relevant issue |
| False positive | Tool reports an issue that is not exploitable/applicable in context |
| True negative | No issue exists for the checked property and none is reported |
| False negative | A real issue is missed |

Suppress a finding only with documented evidence and narrow scope. Broad suppression can hide future real vulnerabilities.

### Criteria for a successful static-analysis program

- acceptable false-positive and false-negative tradeoffs;
- warning volume that developers can realistically triage;
- clear location, context, data/control-flow trace and remediation help;
- findings that can be fixed and verified;
- rules tuned to the project without silencing useful categories;
- explicit design intent through annotations/assertions where supported;
- quality gates based on agreed severity and new-code policy;
- measured remediation and clean reruns.

### Triage workflow

```text
Finding -> locate source/sink and path -> reproduce/reason about reachability
        -> classify CWE and impact -> fix root cause
        -> add regression test -> rerun analysis -> document residual risk
```

Prioritize exposure, data sensitivity, reachability, privilege and exploitability rather than sorting only by the tool’s label.

### How it links to the labs

- [X08](CHEAT_SHEET.md#x08---static-code-analysis) introduces ESLint security rules, SARIF and CodeQL as practical points on the static-analysis spectrum.
- [X09](CHEAT_SHEET.md#x09---sonarqube) adds centralized findings, quality metrics, triage and quality gates.
- [X06](CHEAT_SHEET.md#x06---software-composition-analysis) is complementary: SCA identifies component/version risk while SAST reasons about application code.
- [X07](CHEAT_SHEET.md#x07---automated-testing-with-github-actions) provides dynamic evidence and regression tests for fixes.
- [X11a/X11b](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) test the running behavior static analysis cannot fully observe.

Source basis: Lecture 9, Static Code Analysis x01, AY2025T3.

---

## Detailed lecture-to-lab traceability matrix

| Lecture concept | Requirement/design artifact | Lab activity | Evidence to retain |
|---|---|---|---|
| SSDLC and quality gates | Security plan and definition of done | X05-X09 CI integration | Workflow policy and passing/failing runs |
| OWASP Broken Access Control | Authorization/ownership requirements | Lab 2, X07, X11a | Access matrix, negative tests, scan validation |
| Security Misconfiguration | Hardened baseline and safe-error requirement | Lab 1, X05, X09 | Compose/Nginx config and review results |
| Software Supply Chain Failures | Approved dependency/update process | X06 | SCA report, upgrade PR, exception record |
| Cryptographic Failures | Data classification and cryptographic requirements | Lab 2, X05 | TLS test/config and secret-handling evidence |
| Injection | Validation, parameterization, output-encoding requirements | X08, X11a/X11b | SARIF, requests/responses, regression tests |
| Insecure Design | Misuse cases and threat model | Lab 2, X03/X04 | Misuse diagram and threat report |
| Authentication Failures | Authentication/recovery/session requirements | Lab 2, X07, X11b | Automated tests and authorized fuzz results |
| Integrity Failures | Trusted build/update and data-integrity design | X05, X06, X08 | Protected workflow, pinned dependency, analysis |
| Logging/Alerting Failures | Accountability and monitoring requirements | Lab 2, X09, X11a | Audit events, alerts, retained reports |
| Exceptional Conditions | Fail-secure and rollback requirements | X07, X08 | Failure-path tests and static findings |
| CVE/NVD/CVSS | Vulnerability triage procedure | X06, X09 | Finding record with context and decision |
| CAPEC/ATT&CK | Attack scenarios and detection ideas | X03/X04 | Threat scenarios and monitoring controls |
| SAMM/BSIMM/SDL | Repeatable assurance roadmap | All labs | Tool/process coverage and improvement backlog |
| CIA/AAA | Security requirement catalogue | Lab 2 | Requirement IDs and acceptance tests |
| Anti-requirements/misuse cases | Adversarial requirements | Lab 2, X03/X04 | Misuse case linked to threat/control/test |
| DFD and trust boundaries | Threat-model diagram | X03/X04 | Labelled diagram and report |
| STRIDE | Categorized threats | X03/X04 | Status, mitigation, owner, validation |
| Minimize attack surface | Reduced ports/features/roles | Lab 1, X05 | Network/service inventory |
| Secure defaults | Hardened initial configuration | Lab 1, X05 | Configuration review |
| Least privilege/separation | Role and service permission design | Lab 1, Lab 2, X05 | Access matrix and deployment configuration |
| Defense in depth | Independent preventive/detective controls | X05-X11 | Layered architecture and evidence |
| Fix correctly | Root-cause remediation plus regression | X07-X11 | Fix commit, test, and clean rerun |
| Confidentiality/integrity design | Crypto, masking, validation, transaction controls | X05, X07 | TLS and behavior tests |
| Availability design | Limits, replication, failover, recovery | Lab 1, X07 | Health checks, recovery/failure evidence |
| Accountability design | Protected audit trail | Lab 2, X09, X11 | Audit/alert evidence |
| D2 implementation evidence | Traceable code/config/test documentation | X05-X09 | Exact commit, filenames, diagrams and repeated CI runs |
| DevSecOps pipeline | Security integrated into build/test delivery | X05-X09 | Versioned workflow, protected secrets and enforced gates |
| TLS/HSTS/PFS | Secure transport requirement and threat rationale | X05, X11a | Protocol/cipher config, certificate and redirect/HSTS checks |
| Passwordless/passkeys | Enrollment, recovery, challenge and session requirements | Lab 2, X07, X11 | Flow design, negative tests and audit evidence |
| OWASP Proactive Controls | Secure-coding checklist mapped to project controls | X06-X09 | Review record, code references and clean reruns |
| Input validation/ReDoS | Bounded server-side validation and resource requirements | X07, X08, X11b | Boundary/adversarial tests and static findings |
| Parameterized SQL | Injection-prevention and least-privilege requirements | X07, X08, X11 | Query code, tests, scan evidence and DB role config |
| ASVS/WSTG | Verification catalogue and test procedure | X07, X11a/X11b | Requirement-to-test map and completed checklist |
| Testing pyramid | Balanced unit/integration/UI/manual test plan | X07 | Test inventory, CI result and coverage rationale |
| Static-analysis theory | SAST rule selection, triage and quality-gate policy | X08/X09 | SARIF/issues, disposition, fix and clean rerun |

## Reverse lookup: start from the lab

### Lab 1 - Docker Compose infrastructure

Lecture connections:

- Lecture 1-P1: repeatable development environment and lifecycle foundation.
- Lecture 4: attack surface, secure defaults, least privilege, defense in depth.
- Lecture 5: physical/logical topology, availability, data storage, service accounts.
- Lecture 7: Dockerfile/Compose development workflow, minimal images, secret handling and least exposure.

Questions to answer while doing it:

- Which ports are published, and can any be removed?
- Which services need to communicate, and on which network?
- What data persists, and how is it protected/backed up?
- Which credentials/permissions does each service receive?

### Lab 2 - Secure requirements

Lecture connections:

- Lecture 2-P2: requirement types, process, CIA, authentication.
- Lecture 3-P1: authorization, accountability, session/error/configuration/operations, misuse cases.
- Lecture 3-P2: requirements become the objectives and scope of the threat model.
- Lecture 6: TLS, password storage, passwordless enrollment/recovery and session requirements.
- Lecture 8: requirements must map to ASVS/WSTG verification and expected results.

Deliverable logic:

```text
Asset + threat/risk -> requirement -> acceptance test -> owner/evidence
```

### X03/X04 - Threat-modeling tools

Lecture connections:

- Lecture 3-P2: DFDs, trust boundaries, actors, attack trees, STRIDE.
- Lecture 4: design principles guide mitigation selection.
- Lecture 5: four-part secure design process and data access matrix.

Do not stop after drawing. Triage threats, document mitigations, assign ownership, and validate implemented controls.

### X05 - Nginx, TLS, and GitHub Actions

Lecture connections:

- Lecture 1-P1: security gates in the SSDLC.
- Lecture 1-P2: cryptographic failure and integrity of delivery.
- Lecture 4: defense in depth, secure defaults, reduced attack surface.
- Lecture 5: confidentiality in transit and physical/logical architecture.
- Lecture 5.5: repeated CI/CD and implementation evidence required for the handoff.
- Lecture 6: DevSecOps pipeline, TLS, HSTS, cipher/key choices and perfect forward secrecy.
- Lecture 7: cryptography must use established protocols and secure configuration.

### X06 - Dependency analysis

Lecture connections:

- Lecture 1-P2: software supply chain failures.
- Lecture 2-P1: CVE/NVD/CVSS triage and assurance maturity.
- Lecture 4: do not trust external services/components.
- Lecture 5.5: dependency inventory and dependency-check evidence.
- Lecture 7: proactive control C6 and maintained component/framework security.
- Lecture 9: SCA complements static analysis but answers a different question.

### X07 - Automated tests and Selenium

Lecture connections:

- Lecture 1-P1: quality gates and sprint security.
- Lecture 3-P1: anti-requirements, input boundaries, velocity, transaction interruption.
- Lecture 4: fix correctly with regression tests and fail securely.
- Lecture 5: authentication, authorization, resource locking, and availability behavior.
- Lecture 5.5: automated-test evidence for the frozen implementation.
- Lecture 6: pipeline gates and authentication/passwordless behavior.
- Lecture 8: testing pyramid, control verification and requirements-to-tests traceability.
- Lecture 9: dynamic tests complement static reasoning and verify remediations.

### X08/X09 - Static analysis and SonarQube

Lecture connections:

- Lecture 1-P2: injection, exceptional conditions, and common weakness patterns.
- Lecture 2-P1: assurance activities and vulnerability prioritization.
- Lecture 4: scanners complement but do not replace design review.
- Lecture 5.5: static-analysis evidence forms part of the implementation/QA handoff.
- Lecture 7: secure coding patterns, ReDoS, validation, logging and parameterized queries.
- Lecture 8: automated source review plus requirement-based tests and reporting.
- Lecture 9: analysis spectrum, data/control flow, approximation, false positives/negatives and triage.

### X11a/X11b - ZAP and Burp

Lecture connections:

- Lecture 1-P2: attacker-visible OWASP vulnerability categories.
- Lecture 3-P1: misuse cases, input validation, velocity, and visibility.
- Lecture 3-P2: Web attacker model, entry/exit points, and threat validation.
- Lecture 5: verification of selected CIA/AAA controls.
- Lecture 6: TLS, authentication, passwordless, session and residual phishing risks.
- Lecture 7: adversarial validation, injection and logging behavior.
- Lecture 8: WSTG categories, authentication/lockout/CAPTCHA tests and security reporting.
- Lecture 9: dynamic testing exposes runtime behavior unavailable to static analysis.

Only test a target within explicit authorization and agreed impact limits.

## Quiz and exam quick recall

### One-line distinctions

- **Authentication vs authorization:** prove identity vs decide permitted action.
- **Confidentiality vs privacy:** preventing unauthorized disclosure vs appropriate handling of personal information and individual rights.
- **Threat vs vulnerability:** possible cause/scenario of harm vs weakness that enables it.
- **Risk vs severity:** contextual likelihood/impact vs standardized technical seriousness.
- **Flaw vs bug:** unsafe design vs incorrect implementation.
- **Hashing vs encryption:** one-way digest vs reversible transformation with a key.
- **Encoding vs encryption:** representation format vs confidentiality control.
- **RBAC vs ownership check:** role grants general capability; ownership constrains access to a particular object.
- **Replication vs backup:** live redundant copy/capacity vs recoverable historical copy.
- **RTO vs RPO:** restore-time target vs acceptable data-loss window.
- **SAST vs DAST:** analyze code/artifacts vs test a running system.
- **SCA vs SAST:** analyze third-party components vs analyze application code patterns.
- **CVE vs CWE:** specific public vulnerability vs weakness category.
- **CAPEC vs ATT&CK:** attack-pattern catalogue vs observed adversary tactics/techniques knowledge base.
- **Repudiation vs non-repudiation:** denying an action vs evidence that counters denial.
- **Continuous delivery vs deployment:** release remains ready for a deliberate deployment decision vs every passing change is deployed automatically.
- **TLS vs HSTS:** TLS protects an established connection vs HSTS tells a supporting browser to use HTTPS and resist HTTP downgrade paths.
- **Passwordless vs passkey:** passwordless is a broad login category; a passkey is a FIDO/WebAuthn public-key credential.
- **Validation vs parameterization:** validation enforces allowed input policy; parameterization keeps untrusted values separate from SQL structure.
- **ASVS vs WSTG:** verifiable security requirements vs practical web-security testing guidance.
- **Unit vs integration vs UI test:** one component vs cooperating components vs the user-visible end-to-end flow.
- **False positive vs false negative:** reported non-issue vs real issue the tool misses.
- **Data flow vs control flow:** how values/facts move vs which execution paths and blocks may be reached.

### Memory aids

```text
CIA    = Confidentiality, Integrity, Availability
AAA    = Authentication, Authorization, Accountability
STRIDE = Spoofing, Tampering, Repudiation,
         Information Disclosure, Denial of Service,
         Elevation of Privilege
IVTV   = Input validation, Velocity, Transactions, Visibility
CI/CD  = Continuous Integration / Continuous Delivery or Deployment
ASVS   = Application Security Verification Standard
WSTG   = Web Security Testing Guide
SAST   = Static Application Security Testing
DAST   = Dynamic Application Security Testing
```

### Five questions for any feature

1. Which asset and data does it touch?
2. Who may invoke it, on which resource, under which conditions?
3. How can a malicious or mistaken actor abuse it?
4. Which design control prevents, detects, or contains the abuse?
5. Which automated test or scan provides repeatable evidence?

## End-to-end project workflow

```text
1. Inventory stakeholders, actors, assets, data, dependencies, and constraints.
2. Write functional/non-functional and CIA/AAA security requirements.
3. Add anti-requirements and misuse cases.
4. Draw physical/logical architecture, DFDs, and trust boundaries.
5. Apply STRIDE/attack trees and prioritize risks.
6. Select controls using secure design principles.
7. Implement least-privilege infrastructure, TLS, secrets, and safe defaults.
8. Add unit, integration, UI, negative, and failure-path tests.
9. Automate CI, SCA, SAST, and quality gates.
10. Perform authorized DAST/fuzzing and validate findings.
11. Remediate root causes, add regression tests, and rerun checks.
12. Document evidence and residual risk; update models as the system changes.
```
