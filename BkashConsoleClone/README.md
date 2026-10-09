# 📱 BkashConsoleClone – Complete Project & Learning Reference

&gt; **Educational console simulation: নিজে code লিখে C# শেখা।**
&gt;
&gt; **Disclaimer:** real bKash account, real money, payment gateway, SMS/OTP বা financial transaction নেই। bKash-এর সঙ্গে affiliation নেই। Security, compliance, durability বা production-readiness দাবি নয়। সব fee/limit/identity/balance invented simulation data। Real credential ব্যবহার করবে না।

| বিষয় | সিদ্ধান্ত |
|---|---|
| Name | BkashConsoleClone |
| Platform | C# / .NET Console; .NET 10 LTS লক্ষ্য |
| Storage | শুধু in-memory; কোনো database/EF Core নয় |
| Tests | xUnit; Phase 2 থেকে ছোট tests, transaction-এর আগে integrity tests |
| Workflow | Design → ছোট snippet → নিজের implementation → review → refactor |
| Status | Revised documentation plan; application implementation/test execution হয়নি |
| Date | 9 October 2026 |

.NET 10 LTS নতুন setup-এর লক্ষ্য; .NET 8 support 10 November 2026-এ শেষ হবে, তাই সেটি long-term default নয়। Installed SDK ও supported SDK আলাদা; supported patch বেছে নেবে। [S1: official support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core)।

**এই একটিমাত্র README-তে পূর্ণ project plan + গভীর learning reference + UI preview আছে। Full application/feature/file source নেই।** Snippets ছোট isolated learning examples; project implementation নয়। Section 24-এর প্রত্যেক required topic Core feature, test অথবা mandatory learning lab-এ mapped। Lab integration optional হলেও lesson বাদ যাবে না।

## সূচিপত্র

1. [Project Overview](#section-1)
2. [Complete Feature List](#section-2)
3. [Feature Tiers](#section-3)
4. [Functional Requirements](#section-4)
5. [Non-Functional Requirements](#section-5)
6. [Complete Folder Structure](#section-6)
7. [File Responsibilities](#section-7)
8. [Data Models and Properties](#section-8)
9. [In-memory Storage Design](#section-9)
10. [Data Relationships](#section-10)
11. [Architecture and Design Decisions](#section-11)
12. [Transaction Data Flow and Integrity](#section-12)
13. [Business Rules](#section-13)
14. [Error Handling Strategy](#section-14)
15. [Security Considerations](#section-15)
16. [Testing Strategy](#section-16)
17. [Development Roadmap](#section-17)
18. [Feature-wise Topic Mapping](#section-18)
19. [Foundation and OOP Coverage](#section-19)
20. [Additional Industry Topics](#section-20)
21. [Teaching and Review Workflow](#section-21)
22. [Feature Completion Checklist](#section-22)
23. [Known Limitations and Future Extensions](#section-23)
24. [Complete Topic Glossary](#section-24)
25. [Console UI Preview](#section-25)
26. [Git Workflow and References](#section-26)

<a id="section-1"></a>

# 1. Project Overview

## উদ্দেশ্য

User registration/login, wallet balance, cash in/out, send money, recharge simulation, history/receipt, admin controls/report তৈরি করতে করতে syntax, OOP, data integrity, architecture, clean code, testing, security সীমাবদ্ধতা ও Git শেখা। Application লক্ষ্য নয়, শেখার মাধ্যম।

## Scope

In scope: user/admin/session; personal/seeded-agent/system wallet; four transaction types; fees/limits; validation; immutable terminal history; retry/idempotency; audit; in-memory consistency; optional JSON checkpoint।

Out of scope: SQL Server/PostgreSQL/MySQL/SQLite/NoSQL/EF Core; real payments/OTP/SMS/telecom; real KYC/AML; web/mobile/API implementation; microservices; distributed processing; production availability। Future database শুধু conceptual discussion, explicit new scope ছাড়া যোগ নয়।

| সমস্যা | Simulation solution |
|---|---|
| half transfer | isolated candidate + one state publication |
| precision | decimal + explicit input/rounding policy |
| unauthorized action | service-level identity/role/ownership checks |
| duplicate retry | actor+key+payload+original result index |
| invalid input | parsing + format + business validation |
| audit/history drift | immutable snapshots + append-only API |

| দিক | এখানে | বাস্তব system-এ বিবেচ্য, এখানে নয় |
|---|---|---|
| Storage | memory, restart loss | durability/backup/recovery |
| Atomicity | single-process publication | durable transactions/concurrency |
| Accounting | signed postings/conservation | formal ledger/reconciliation |
| Authentication | short simulated PIN/hash/local lock | stronger auth/risk controls |
| Audit | append-only API, tamper-proof নয় | durable controlled audit |
| Operations | console process | monitoring/availability/incident response |

Skills: requirements→design→implementation→tests→review, type-safe data, focused classes, appropriate interfaces, invariants, deterministic tests, debugger, safe logging, dependency graph, design alternatives এবং refactoring।

<a id="section-2"></a>

# 2. Complete Feature List

## User Management

### F-01 · Registration [MVP]

- **Purpose / user action:** নাম/mobile/PIN/confirmation দিয়ে user+wallet।
- **Rules:** BR-01–04,15 (Section 13)।
- **Responsible:** UserService/InputValidator/hasher/store।
- **Topics:** [f-variables](#f-variables) · [f-collections](#f-collections) · [o-constructor](#o-constructor)।
- **Acceptance:** valid→one user+wallet balance 0; duplicate/invalid→neither inserted।

### F-02 · Login / Logout [MVP]

- **Purpose / user action:** credential যাচাই ও session শুরু/শেষ।
- **Rules:** BR-05–06,21 (Section 13)।
- **Responsible:** AuthService/SessionManager।
- **Topics:** [f-methods](#f-methods) · [f-loops](#f-loops) · [f-nullable](#f-nullable)।
- **Acceptance:** correct→session; wrong→generic message; threshold→lock; logout→protected calls denied।

### F-03 · User Profile [Core]

- **Purpose / user action:** নিজের safe তথ্য দেখা।
- **Rules:** BR-18,20 (Section 13)।
- **Responsible:** UserService/UserDetailsDto।
- **Topics:** [o-fields-properties](#o-fields-properties) · [r-extension-methods](#r-extension-methods)।
- **Acceptance:** own only; masked mobile; no PIN/hash/salt।

### F-04 · Account Status [MVP]

- **Purpose / user action:** Active/Suspended/Closed দেখা।
- **Rules:** BR-06 (Section 13)।
- **Responsible:** Account/AccountService।
- **Topics:** [f-record-struct-enum](#f-record-struct-enum) · [r-pattern-matching](#r-pattern-matching)।
- **Acceptance:** new Active; inactive money actions blocked।

### F-05 · Account Number [MVP]

- **Purpose / user action:** unique simulated mobile wallet number।
- **Rules:** BR-01,04 (Section 13)।
- **Responsible:** InputValidator/account repo।
- **Topics:** [f-string](#f-string) · [f-collections](#f-collections) · [o-immutability](#o-immutability)।
- **Acceptance:** Personal number: 11 ASCII digits 01[3-9]; unique/immutable; Agent/System seeded opaque IDs; no ownership verification claim।

### F-06 · PIN Change [Core]

- **Purpose / user action:** old/new/confirmation।
- **Rules:** BR-02,05,25 (Section 13)।
- **Responsible:** UserService/AuthService/hasher।
- **Topics:** [f-methods](#f-methods) · [o-encapsulation](#o-encapsulation)।
- **Acceptance:** old wrong/mismatch/same PIN reject; fresh salt/hash; session invalidate।

### F-07 · Session Management [Core]

- **Purpose / user action:** identity/idle timeout।
- **Rules:** BR-21 (Section 13)।
- **Responsible:** SessionManager/IClock।
- **Topics:** [f-nullable](#f-nullable) · [o-interface](#o-interface)।
- **Acceptance:** protected service checks expiry; fake-clock boundary; no credentials।

## Account Management

### F-08 · Account Creation [MVP]

- **Purpose / user action:** registration-এর সঙ্গে personal wallet।
- **Rules:** BR-04,15 (Section 13)।
- **Responsible:** UserService/Account/store।
- **Topics:** [o-constructor](#o-constructor) · [o-this](#o-this)।
- **Acceptance:** one wallet per User role; zero balance; user+wallet same commit।

### F-09 · Balance Check [MVP]

- **Purpose / user action:** PIN-confirmed own balance।
- **Rules:** BR-05,08,13 (Section 13)।
- **Responsible:** AccountService/AuthService।
- **Topics:** [f-variables](#f-variables) · [r-extension-methods](#r-extension-methods)।
- **Acceptance:** owner+PIN; two decimal display; no mutation।

### F-10 · Account Information [Core]

- **Purpose / user action:** type/status/date/daily usage।
- **Rules:** BR-11,18 (Section 13)।
- **Responsible:** AccountService/transaction repo।
- **Topics:** [f-linq](#f-linq) · [o-composition](#o-composition)।
- **Acceptance:** completed relevant principal amounts only; configured business day।

### F-11 · Status Validation [MVP]

- **Purpose / user action:** inactive participants আটকানো।
- **Rules:** BR-06 (Section 13)।
- **Responsible:** TransactionValidator।
- **Topics:** [f-conditions](#f-conditions) · [r-pattern-matching](#r-pattern-matching)।
- **Acceptance:** inactive sender/receiver/agent→Failed; no postings।

### F-12 · Balance Integrity [MVP]

- **Purpose / user action:** negative/invalid balance আটকানো।
- **Rules:** BR-07–08,15 (Section 13)।
- **Responsible:** Account/TransactionService।
- **Topics:** [o-encapsulation](#o-encapsulation) · [o-immutability](#o-immutability)।
- **Acceptance:** no public setter; positive movements; overflow no publication।

## Transaction Management

### F-13 · Cash In [MVP]

- **Purpose / user action:** seeded agent→নিজের wallet।
- **Rules:** BR-06–13,15,22 (Section 13)।
- **Responsible:** TransactionService/FeeCalculator।
- **Topics:** [f-methods](#f-methods) · [f-operators](#f-operators)।
- **Acceptance:** agent -amount, personal +amount; zero fee; agent balance check; role-play acknowledgement।

### F-14 · Cash Out [MVP]

- **Purpose / user action:** personal→agent।
- **Rules:** BR-06–13,15,22 (Section 13)।
- **Responsible:** TransactionService/FeeCalculator।
- **Topics:** [f-operators](#f-operators) · [f-exception-handling](#f-exception-handling)।
- **Acceptance:** personal -(amount+fee), agent +amount, fee wallet +fee; insufficient→unchanged।

### F-15 · Send Money [MVP]

- **Purpose / user action:** অন্য personal wallet-এ amount/reference/PIN।
- **Rules:** BR-06–15 (Section 13)।
- **Responsible:** TransactionService/validator।
- **Topics:** [f-collections](#f-collections) · [o-copy](#o-copy)।
- **Acceptance:** self-send reject; sender -(amount+fee), receiver +amount, fee wallet +fee; no partial update।

### F-16 · Recharge Simulation [Core]

- **Purpose / user action:** fake operator/amount top-up।
- **Rules:** BR-07–13,23 (Section 13)।
- **Responsible:** TransactionService/FakeRechargeGateway।
- **Topics:** [f-async](#f-async) · [f-task-thread](#f-task-thread) · [r-cancellation-token](#r-cancellation-token)।
- **Acceptance:** 20–1000 inclusive; fake prefix; settlement credit; failure/cancel before commit no debit।

### F-17 · Transaction History [MVP]

- **Purpose / user action:** নিজের submitted attempts।
- **Rules:** BR-18 (Section 13)।
- **Responsible:** TransactionService/transaction repo।
- **Topics:** [f-loops](#f-loops) · [f-linq](#f-linq) · [r-yield](#r-yield)।
- **Acceptance:** own participant/initiator only; newest+ID tie-break; empty safe।

### F-18 · Transaction Details [Core]

- **Purpose / user action:** ID দিয়ে own record।
- **Rules:** BR-18,20 (Section 13)।
- **Responsible:** TransactionService।
- **Topics:** [f-collections](#f-collections) · [f-nullable](#f-nullable)।
- **Acceptance:** unknown/other-user ID same not-found; no leakage।

### F-19 · Receipt [Core]

- **Purpose / user action:** completed immutable snapshot।
- **Rules:** BR-17 (Section 13)।
- **Responsible:** TransactionReceipt/ReceiptPrinter।
- **Topics:** [f-string](#f-string) · [f-record-struct-enum](#f-record-struct-enum)।
- **Acceptance:** Completed only; ID/amount/fee match; historical balance-after, no other wallet balance।

### F-20 · Transaction Status [MVP]

- **Purpose / user action:** attempt terminal result।
- **Rules:** BR-15,18 (Section 13)।
- **Responsible:** Transaction/TransactionService।
- **Topics:** [f-record-struct-enum](#f-record-struct-enum) · [o-encapsulation](#o-encapsulation)।
- **Acceptance:** Pending internal; stored Completed/Failed immutable; Failed safe code।

### F-21 · Transaction ID [MVP]

- **Purpose / user action:** opaque unique ID।
- **Rules:** BR-16 (Section 13)।
- **Responsible:** IIdGenerator/transaction repo।
- **Topics:** [o-interface](#o-interface) · [o-static-members](#o-static-members)।
- **Acceptance:** Guid candidate+insertion guard; 10k smoke test not uniqueness proof; fake collision no overwrite।

### F-22 · Filter / Sort [Core]

- **Purpose / user action:** type/status/date/amount/paging।
- **Rules:** BR-18 (Section 13)।
- **Responsible:** TransactionQuery/service।
- **Topics:** [f-lambda](#f-lambda) · [f-linq](#f-linq) · [r-deferred-execution](#r-deferred-execution)।
- **Acceptance:** combined filters; inclusive amounts; exclusive date end; stable paging।

### F-23 · Transaction Validation [MVP]

- **Purpose / user action:** rules before processing।
- **Rules:** BR-06–13,22–23 (Section 13)।
- **Responsible:** TransactionValidator।
- **Topics:** [f-conditions](#f-conditions) · [r-pattern-matching](#r-pattern-matching)।
- **Acceptance:** every rule pass/fail/boundary; validation no mutation।

### F-24 · Failure Handling [MVP]

- **Purpose / user action:** half update/false success আটকানো।
- **Rules:** BR-15,18 (Section 13)।
- **Responsible:** TransactionService/store।
- **Topics:** [f-exception-handling](#f-exception-handling) · [o-copy](#o-copy)।
- **Acceptance:** fault before publication old state intact; post-commit notification failure remains Completed।

### F-25 · Duplicate Prevention [Advanced]

- **Purpose / user action:** same retry once।
- **Rules:** BR-14 (Section 13)।
- **Responsible:** TransactionService/retry index।
- **Topics:** [f-collections](#f-collections) · [r-equality-hashing](#r-equality-hashing)।
- **Acceptance:** actor+key+same payload original result; changed payload conflict; one balance effect।

### F-26 · Limits / Fees [MVP]

- **Purpose / user action:** quote+confirm/configured limits।
- **Rules:** BR-11–12 (Section 13)।
- **Responsible:** FeeCalculator/FeePolicy/TransactionLimits।
- **Topics:** [f-operators](#f-operators) · [f-record-struct-enum](#f-record-struct-enum)।
- **Acceptance:** boundaries; fee 2 decimals; invalid config startup reject; completed principal daily usage।

### F-27 · Balance Update Rules [MVP]

- **Purpose / user action:** consistent signed postings।
- **Rules:** BR-08,15,22 (Section 13)।
- **Responsible:** Account/TransactionService/store।
- **Topics:** [o-copy](#o-copy) · [o-immutability](#o-immutability)।
- **Acceptance:** signed sum zero; all wallet total conserved including fee/agent/settlement।

## Admin Features

### F-28 · Admin Login [Core]

- **Purpose / user action:** admin authenticate/authorize।
- **Rules:** BR-05,19,24 (Section 13)।
- **Responsible:** AuthService/SeedData।
- **Topics:** [f-record-struct-enum](#f-record-struct-enum) · [o-interface](#o-interface)।
- **Acceptance:** registration cannot choose Admin; secret not source; service guard।

### F-29 · User List [Core]

- **Purpose / user action:** admin paged safe list।
- **Rules:** BR-19–20 (Section 13)।
- **Responsible:** AdminService।
- **Topics:** [f-linq](#f-linq) · [r-ienumerable-icollection](#r-ienumerable-icollection)।
- **Acceptance:** non-admin denied; no credential fields; deterministic pages।

### F-30 · User Search [Core]

- **Purpose / user action:** name/mobile keyword।
- **Rules:** BR-19–20 (Section 13)।
- **Responsible:** AdminService।
- **Topics:** [f-string](#f-string) · [f-lambda](#f-lambda)।
- **Acceptance:** case-insensitive name; empty policy; no match safe।

### F-31 · User Details [Core]

- **Purpose / user action:** admin safe user/account projection।
- **Rules:** BR-19–20 (Section 13)।
- **Responsible:** AdminService/UserDetailsDto।
- **Topics:** [o-immutability](#o-immutability) · [o-copy](#o-copy)।
- **Acceptance:** no hash/salt; DTO cannot mutate live state।

### F-32 · Activate / Deactivate [Core]

- **Purpose / user action:** admin Active↔Suspended+reason।
- **Rules:** BR-06,19 (Section 13)।
- **Responsible:** AdminService/AuditService/store।
- **Topics:** [o-encapsulation](#o-encapsulation) · [o-association](#o-association)।
- **Acceptance:** reason required; target session invalidated; system/agent protected; status+audit same publication।

### F-33 · Admin Transaction Search [Advanced]

- **Purpose / user action:** all attempts filters।
- **Rules:** BR-19 (Section 13)।
- **Responsible:** AdminService/TransactionQuery।
- **Topics:** [f-linq](#f-linq) · [r-deferred-execution](#r-deferred-execution)।
- **Acceptance:** authorized; 10k synthetic fixture timings documented, no universal SLA।

### F-34 · Transaction Report [Advanced]

- **Purpose / user action:** date/type volume/fee/failures।
- **Rules:** BR-19 (Section 13)।
- **Responsible:** ReportService।
- **Topics:** [f-linq](#f-linq) · [f-string](#f-string)।
- **Acceptance:** Completed principal/fee only; fixture totals match; no unnecessary async wrapper।

### F-35 · Summary Statistics [Advanced]

- **Purpose / user action:** counts/volume/fee/success ratio।
- **Rules:** BR-19 (Section 13)।
- **Responsible:** ReportService/SummaryStatisticsDto।
- **Topics:** [f-linq](#f-linq) · [f-operators](#f-operators)।
- **Acceptance:** zero attempts ratio N/A; empty Average safe।

### F-36 · Admin Audit [Core]

- **Purpose / user action:** append-only action/access outcome।
- **Rules:** BR-19–20 (Section 13)।
- **Responsible:** AuditService/audit repo।
- **Topics:** [o-immutability](#o-immutability) · [o-composition](#o-composition)।
- **Acceptance:** actor/action/target/time/outcome/reason; no edit/delete; no tamper-proof claim।

## System Features

### F-37 · Menus [MVP]

- **Purpose / user action:** main/user; admin Core-এ।
- **Rules:** BR-21 (Section 13)।
- **Responsible:** MainMenu/UserMenu/AdminMenu।
- **Topics:** [f-loops](#f-loops) · [f-conditions](#f-conditions)।
- **Acceptance:** invalid choice safe; EOF exit; back/logout; menu hiding not sole security।

### F-38 · Input Validation [MVP]

- **Purpose / user action:** parse/retry/masked PIN।
- **Rules:** BR-01–03,07 (Section 13)।
- **Responsible:** InputReader/InputValidator।
- **Topics:** [f-string](#f-string) · [r-params-ref-out-in](#r-params-ref-out-in)।
- **Acceptance:** empty/huge/negative safe; redirected input/EOF; culture explicit।

### F-39 · Exception Handling [MVP]

- **Purpose / user action:** expected vs unexpected।
- **Rules:** BR-15,20 (Section 13)।
- **Responsible:** OperationResult/UI/Program।
- **Topics:** [f-exception-handling](#f-exception-handling) · [r-exception-filters](#r-exception-filters)।
- **Acceptance:** safe expected result; recoverable log; corrupted-state suspicion safe stop।

### F-40 · Diagnostic Logging [Core]

- **Purpose / user action:** safe structured diagnostics।
- **Rules:** BR-20 (Section 13)।
- **Responsible:** IAppLogger/ConsoleAppLogger।
- **Topics:** [o-interface](#o-interface) · [r-idisposable](#r-idisposable)।
- **Acceptance:** levels/event/correlation; no PIN/hash/full mobile; logger failure no financial rollback।

### F-41 · Configuration [MVP]

- **Purpose / user action:** immutable settings।
- **Rules:** BR-11–12 (Section 13)।
- **Responsible:** AppConfiguration।
- **Topics:** [o-readonly-const](#o-readonly-const) · [o-immutability](#o-immutability)।
- **Acceptance:** invalid config rejected; version recorded; secrets separate।

### F-42 · In-memory Management [MVP]

- **Purpose / user action:** single source of truth।
- **Rules:** BR-04,15,18 (Section 13)।
- **Responsible:** InMemoryStateStore/repos।
- **Topics:** [f-collections](#f-collections) · [f-generics](#f-generics)।
- **Acceptance:** no mutable collection/object leak; registration/transfer one commit।

### F-43 · JSON Export / Import [Optional]

- **Purpose / user action:** explicit checkpoint।
- **Rules:** BR-20,26 (Section 13)।
- **Responsible:** JsonDataStore/DataSnapshot।
- **Topics:** [f-async](#f-async) · [r-idisposable](#r-idisposable)।
- **Acceptance:** roundtrip; corrupt/version-invalid old state intact; no session; credential policy।

### F-44 · Unit Testing [MVP]

- **Purpose / user action:** automated rule verification।
- **Rules:** BR-all (Section 13)।
- **Responsible:** tests/fakes।
- **Topics:** [o-interface](#o-interface) · [f-methods](#f-methods)।
- **Acceptance:** integrity tests before money milestone; actual test output, no assumed pass।

### F-45 · Debugging Support [Core]

- **Purpose / user action:** reproduce/inspect/fix/document।
- **Rules:** BR-20 (Section 13)।
- **Responsible:** debugger/learning-log।
- **Topics:** [f-dependency](#f-dependency) · [o-encapsulation](#o-encapsulation)।
- **Acceptance:** each milestone bug note+regression test; no secrets in output।


<a id="section-3"></a>

# 3. Feature Tiers

- **MVP:** F-01, F-02, F-04, F-05, F-08, F-09, F-11, F-12, F-13, F-14, F-15, F-17, F-20, F-21, F-23, F-24, F-26, F-27, F-37, F-38, F-39, F-41, F-42, F-44।
- **Core:** F-03, F-06, F-07, F-10, F-16, F-18, F-19, F-22, F-28, F-29, F-30, F-31, F-32, F-36, F-40, F-45।
- **Advanced:** F-25, F-33, F-34, F-35।
- **Optional:** F-43।

MVP gate: two users register/login → cash in → fee-confirmed send/cash out → history; invalid/faulted requests unchanged state; tests green। Fee/limit/config/tests MVP-তে, correctness পরে যোগ নয়। Core adds profile/PIN/timeout/recharge/receipt/admin/audit/logs। Advanced retry/reporting। Optional persistence/policies/CI/container। Tier complete হওয়ার আগে next tier feature implementation নয়; প্রয়োজনীয় concept/lab আগে শেখা যায়।

<a id="section-4"></a>

# 4. Functional Requirements

প্রতিটি F-ID functional requirement: তার action, rules ও acceptance পূরণ করতে হবে। Cross-cutting FR:

| ID | System অবশ্যই | Trace |
|---|---|---|
| FR-01 | user+wallet all-or-nothing | F-01/08/42 |
| FR-02 | credential/role/ownership service-এ | F-02/06/28 |
| FR-03 | protected action session/PIN | F-07/09/13–16 |
| FR-04 | quote/status/balance/limit before commit | F-11–16/23/26 |
| FR-05 | balances+postings+history one publication | F-20/24/27/42 |
| FR-06 | own history/details, completed receipt | F-17–19 |
| FR-07 | actor-scoped original retry result | F-25 |
| FR-08 | admin guard+required audit | F-28–36 |
| FR-09 | parse/EOF/cancel/errors predictable | F-37–40 |
| FR-10 | no database, import candidate validation | F-42/43 |
| FR-11 | rule-linked test/review evidence | F-44/45 |

<a id="section-5"></a>

# 5. Non-Functional Requirements

| Requirement | কেন | যাচাই |
|---|---|---|
| Maintainability/readability | future changes বোঝা | focused methods, names, review; line-count guideline hard rule নয় |
| Separation | UI/storage বদলে rules না ভাঙা | service-এ Console/File I/O নয়; UI-তে financial rules নয় |
| Testability | deterministic bugs | fake clock/ID/gateway; fresh store per test |
| Reliability/integrity | half update নয় | fault injection, signed sum zero, nonnegative balances |
| Validation/error handling | untrusted input | empty/huge/EOF/boundary/recovery tests |
| Performance basics | unnecessary scans | measured 10k fixture; average O(1) vs O(n), no universal SLA |
| Secure coding | leakage কমানো | ownership/lockout/redaction tests |
| Naming/style | predictability | PascalCase types/methods, camelCase locals, _camelCase fields, I interfaces; editorconfig |
| Predictability | repeatable results | IClock, explicit culture/timezone/rounding, stable sort |
| Extensibility | controlled change | contract tests, justified seams, no pattern count target |
| Console usability | navigation সহজ | numbered options, cancel/back, aligned tables, no-color fallback |
| Documentation | reasoning retained | README/ADR/test matrix/learning log sync |
| Resource safety | cleanup | using/disposal tests optional I/O |
| Memory awareness | unbounded growth বোঝা | history retention documented; paging does not reduce stored memory |

<a id="section-6"></a>

# 6. Complete Folder Structure

Target structure, একসঙ্গে সব তৈরি নয়। [L] learning lab, [O] optional integration। One main project, tests আলাদা। SDK-এর solution format .sln/.slnx দুটোই গ্রহণযোগ্য।

```text
BkashConsoleClone/
|-- README.md, .gitignore, .editorconfig, global.json, BkashConsoleClone.slnx
|-- docs/requirements.md, learning-log.md, test-matrix.md
|   `-- adr/0001-state-boundary.md, diagrams/architecture.txt
|-- src/BkashConsoleClone/
|   |-- Program.cs, BkashConsoleClone.csproj
|   |-- Models/User.cs, PinCredential.cs, Account.cs, Transaction.cs,
|   |   BalancePosting.cs, TransactionReceipt.cs, AuditLogEntry.cs,
|   |   UserSession.cs, DateRange.cs, TransactionQuery.cs,
|   |   TransactionRequest.cs, TransactionQuote.cs, OperationResult.cs,
|   |   IdempotencyEntry.cs, Dtos/UserDetailsDto.cs, SummaryStatisticsDto.cs
|   |-- Enums/UserRole.cs, AccountStatus.cs, AccountType.cs,
|   |   TransactionType.cs, TransactionStatus.cs, AuditAction.cs
|   |-- Configuration/AppConfiguration.cs, FeePolicy.cs, TransactionLimits.cs
|   |-- Services/UserService.cs, AuthService.cs, SessionManager.cs,
|   |   AccountService.cs, TransactionService.cs, TransactionValidator.cs,
|   |   FeeCalculator.cs, AdminService.cs, ReportService.cs, AuditService.cs,
|   |   IPinHasher.cs, Pbkdf2PinHasher.cs,
|   |   IRechargeGateway.cs, FakeRechargeGateway.cs
|   |   `-- Handlers/[O] ITransactionPolicy.cs, SendMoneyPolicy.cs,
|   |       CashInPolicy.cs, CashOutPolicy.cs, RechargePolicy.cs
|   |-- Repositories/IUserRepository.cs, InMemoryUserRepository.cs,
|   |   IAccountRepository.cs, InMemoryAccountRepository.cs,
|   |   ITransactionRepository.cs, InMemoryTransactionRepository.cs,
|   |   IAuditLogRepository.cs, InMemoryAuditLogRepository.cs
|   |-- Data/IStateStore.cs, InMemoryStateStore.cs, AppState.cs, SeedData.cs
|   |   `-- [O] DataSnapshot.cs, JsonDataStore.cs
|   |-- UI/MainMenu.cs, UserMenu.cs, AdminMenu.cs, ConsoleUi.cs,
|   |   InputReader.cs, ReceiptPrinter.cs, ConsoleNotificationSubscriber.cs
|   |-- Utilities/InputValidator.cs, MoneyExtensions.cs, StringExtensions.cs,
|   |   OperatorResolver.cs, IIdGenerator.cs, GuidIdGenerator.cs,
|   |   IClock.cs, SystemClock.cs
|   |-- Exceptions/BkashException.cs, StateCommitException.cs
|   `-- Logging/IAppLogger.cs, ConsoleAppLogger.cs, [O] FileAppLogger.cs
|-- tests/BkashConsoleClone.Tests/
|   |-- BkashConsoleClone.Tests.csproj
|   |-- Fakes/FakeClock.cs, FakeIdGenerator.cs, FakeRechargeGateway.cs,
|   |   CapturingLogger.cs, FaultInjectingStateStore.cs
|   |-- Models/AccountTests.cs
|   |-- Services/UserServiceTests.cs, AuthServiceTests.cs,
|   |   TransactionServiceTests.cs, AdminServiceTests.cs, ReportServiceTests.cs
|   |-- Utilities/InputValidatorTests.cs, FeeCalculatorTests.cs
|   |-- Data/StateStoreTests.cs, [O] JsonDataStoreTests.cs
|   `-- Integration/WorkflowTests.cs
|-- playground/BkashConsoleClone.Playground/
|   |-- Program.cs, BkashConsoleClone.Playground.csproj
|   `-- [L] FoundationLab.cs, AccountInheritanceLab.cs, HandlerTemplateLab.cs,
|       GenericRepositoryLab.cs, PartialConsoleLab.cs, QueryBuilderLab.cs,
|       CopyEqualityLab.cs, RuntimeLab.cs, AsyncLab.cs, ThreadRaceLab.cs
`-- [O] .github/workflows/ci.yml
```

Structure corrections: InputReader UI-তে; generic CRUD repository default নয়; account inheritance, partial UI, nested Builder lab-এ, domain-এ জোর নয়। Focused repositories storage seam; shared store write coordination। Every service interface নয়, actual substitution seams-এ interface।

<a id="section-7"></a>

# 7. File Responsibilities

প্রতিটি file-এর responsibility, collaborator, forbidden responsibility, first feature/phase ও topic নিচে। Interface contract, implementation actual adapter। একই row-তে pair থাকলেও দুই file-এর role আলাদা।

| File | Layer / কেন / কী থাকবে | যোগাযোগ | এখানে নয় | প্রথম feature/phase; topic |
|---|---|---|---|---|
| Program.cs | startup wiring/last boundary | menus/services/store | business/menu logic | F-37/P0; DI |
| User.cs | domain identity/role/credential | user/auth/repo | hashing/Console | F-01/P2; class |
| PinCredential.cs | immutable encoded hash/salt/parameters | hasher/User | plaintext PIN | F-01/P3; record |
| Account.cs | wallet identity/status/balance invariant | account/transaction/store | fee/I/O | F-08/P2; encapsulation |
| Transaction.cs | immutable terminal attempt | transaction/report/repo | UI/live balance | F-20/P5; enum |
| BalancePosting.cs | signed delta/before/after | transaction/store | independent mutation | F-27/P5; composition |
| TransactionReceipt.cs | safe completed snapshot | service/printer | other wallet balance | F-19/P7; record |
| AuditLogEntry.cs | append-only action/outcome | audit/store | edit/delete | F-36/P8; immutability |
| UserSession.cs | identity/role/activity | session/auth | credentials | F-02/P3; nullable |
| DateRange.cs | inclusive start/exclusive end | query/report | I/O | F-22/P7; struct |
| TransactionQuery.cs | filter/page/sort shape | service/repo | execute query | F-22/P7; nullable |
| TransactionRequest.cs | normalized transient request | transaction | retained PIN | F-15/P5; DTO |
| TransactionQuote.cs | fee/total/version | service/UI | commit | F-26/P5; record |
| OperationResult.cs | safe success/error/value | service/UI | raw exception | F-39/P3; generics |
| IdempotencyEntry.cs | actor/key/payload/result | transaction/store | PIN | F-25/P9; equality |
| UserDetailsDto.cs | safe user/account projection | user/admin/UI | hash/salt | F-03/P7; DTO |
| SummaryStatisticsDto.cs | aggregated numbers | report/UI | calculation | F-35/P8; record |
| AppConfiguration.cs | validated immutable settings | startup/services | secrets/execution | F-41/P3; readonly |
| FeePolicy.cs | rates/flat/rounding/version | fee | calculation | F-26/P5; decimal |
| TransactionLimits.cs | min/max/daily values | validator | balance update | F-26/P5; composition |
| UserService.cs | register/profile/PIN orchestration | repo/store/hasher/auth | Console | F-01/P3; DI |
| AuthService.cs | verify/lockout/guards | repo/hasher/session/clock | Console | F-02/P3; methods |
| SessionManager.cs | current session/timeout/invalidate | clock/auth/services | hashing | F-02/P3; state |
| AccountService.cs | authorized balance/info | repo/auth | transfer | F-09/P4; decimal/LINQ |
| TransactionService.cs | quote/validate/stage/commit/result | store/repos/fee/validator/clock/ID | UI/crypto formula | F-13/P5; orchestration |
| TransactionValidator.cs | pure business checks | policy/snapshot | mutation | F-23/P5; guards |
| FeeCalculator.cs | pure rounded fee | FeePolicy | wallet access | F-26/P5; switch |
| AdminService.cs | role-guarded admin workflow | repo/store/audit/session | Console | F-28/P8; authorization |
| ReportService.cs | authorized aggregates | repo/session | unneeded Task.Run | F-34/P8; LINQ |
| AuditService.cs | prepare safe audit record | clock/store | diagnostic sink | F-36/P8; record |
| IPinHasher.cs / Pbkdf2PinHasher.cs | hash contract/implementation | user/auth | lockout/session | F-01/P3; interface |
| IRechargeGateway.cs / FakeRechargeGateway.cs | fake outcome contract/adapter | transaction | real network | F-16/P7,P9; Task |
| IStateStore.cs / InMemoryStateStore.cs | read/stage/one publication | services/repo views | fee/auth rules | F-42/P3; abstraction |
| AppState.cs | owned canonical collections/version | store/repos | public mutable leak | F-42/P3; Dictionary |
| SeedData.cs | agent/system/admin seed | store/hasher/config | hard-coded secret | F-13/P4; initializer |
| DataSnapshot.cs [O] | versioned private checkpoint DTO | JSON/store | session/plain PIN | F-43/P11; serialization |
| JsonDataStore.cs [O] | save/load/candidate validation | snapshot/store | business fee rules | F-43/P11; async/using |
| MainMenu.cs | register/login/exit | user/auth | rules | F-37/P1; loops |
| UserMenu.cs | user prompts/results | services/input | balance mutation | F-37/P4; methods |
| AdminMenu.cs | admin navigation | admin/report | sole auth boundary | F-28/P8; switch |
| ConsoleUi.cs | format/print | menus/printer | repo access | F-37/P1; strings |
| InputReader.cs | parse/retry/EOF/masked PIN | UI/validator | money rules | F-38/P1; TryParse/out |
| ReceiptPrinter.cs | receipt rendering | receipt | transaction creation | F-19/P7; StringBuilder |
| ConsoleNotificationSubscriber.cs | optional post-commit notice | events/UI | required audit | P7; events |
| InputValidator.cs | pure format validation | UI/services | Console/state | F-38/P1; static |
| MoneyExtensions.cs | display/rounding helper | UI/fee | fee policy | F-09/P4; extension |
| StringExtensions.cs | safe masking | UI/log | auth | F-03/P7; extension |
| OperatorResolver.cs | simulated prefix map | recharge | real operator inference | F-16/P7; Dictionary |
| IIdGenerator.cs / GuidIdGenerator.cs | opaque candidate ID | services | PIN randomness | F-21/P5; interface |
| IClock.cs / SystemClock.cs | UTC clock seam | services | business day policy | F-02/P3; DI |
| BkashException.cs | exceptional failure base | boundary | normal menu flow | F-39/P3; inheritance |
| StateCommitException.cs | commit infrastructure error | store/boundary | rollback guarantee | F-24/P5; exception |
| IAppLogger.cs / ConsoleAppLogger.cs | safe diagnostic contract/sink | services/boundary | secrets | F-40/P8; interface |
| FileAppLogger.cs [O] | file sink/disposal | logger | audit authority | P11; IDisposable |


| Repository file | Contract / adapter responsibility | Not here | First use/topic |
|---|---|---|---|
| IUserRepository.cs | FindByMobile/FindById/safe user queries contract | PIN verify/Console | F-01/02/P3; interface |
| InMemoryUserRepository.cs | same shared store user view; normalized-key lookup | own second user collection | P3; Dictionary |
| IAccountRepository.cs | account snapshot by number/owner contract | transfer commit | F-08/P3; interface |
| InMemoryAccountRepository.cs | shared state account lookup/projection | mutable live Account exposure | P3; encapsulation |
| ITransactionRepository.cs | ID/participant/query terminal snapshot contract | independent balance write | F-17/P5; interface |
| InMemoryTransactionRepository.cs | shared dictionary lookup/filter; unique insertion store-owned | duplicate canonical List | P5; LINQ/Dictionary |
| IAuditLogRepository.cs | append/query contract, no edit/delete | business authorization | F-36/P8; interface |
| InMemoryAuditLogRepository.cs | store-owned append/query; required write participates in candidate | event-only required audit | P8; append-only |

All writes through shared candidate/publication boundary; repository contract method presence doesn't authorize caller. Generic IRepository/InMemoryRepository lab only; append-only history/audit cannot inherit arbitrary CRUD delete/update.

| Enum file | Values / collaborators | এখানে নয় | প্রথম use/topic |
|---|---|---|---|
| UserRole.cs | User/Admin; User/Auth/Admin | permission logic | P3; enum |
| AccountStatus.cs | Active/Suspended/Closed; Account/validator | transition logic | P2; enum |
| AccountType.cs | Personal/Agent/System; policy | behavior | P4; enum |
| TransactionType.cs | CashIn/CashOut/SendMoney/Recharge; fee/history | execute | P5; switch |
| TransactionStatus.cs | Pending/Completed/Failed; workflow | terminal editing | P5; state |
| AuditAction.cs | login/read/search/status/import/export; audit | sensitive text | P8; enum |

Optional policies: ITransactionPolicy contract; SendMoneyPolicy/CashInPolicy/CashOutPolicy/RechargePolicy prepare type-specific plan, never commit। First P6/P7; polymorphism/composition। Keep simple switch if readable।

Labs: FoundationLab (P1 types/control flow), AccountInheritanceLab (P6 protected/virtual/override/new), HandlerTemplateLab (P6 abstract/sealed override), GenericRepositoryLab (P6 constraints), PartialConsoleLab (P6 partial), QueryBuilderLab (P7 nested), CopyEqualityLab (P6 copy/hash), RuntimeLab (P6 boxing/conversions/GC), AsyncLab/ThreadRaceLab (P9 Task/cancel/race)। They reference generic examples, not live wallet store; no main app dependency।

Tests: AccountTests invariants P2; UserServiceTests atomic registration P3; AuthServiceTests lockout/session P3; TransactionServiceTests fees/postings/fault/retry P5/P9; AdminServiceTests authorization+audit P8; ReportServiceTests fixture totals P8; InputValidatorTests P2; FeeCalculatorTests P5; StateStoreTests snapshot/commit P3/P5; JsonDataStoreTests optional roundtrip/corruption P11; WorkflowTests actual in-memory collaborators P5+. No real money/network/shared global fixtures।

Fakes: FakeClock deterministic time; FakeIdGenerator fixed IDs/collision; FakeRechargeGateway success/fail/cancel; CapturingLogger redaction; FaultInjectingStateStore pre-publication faults। First relevant tests, no production logic।

Root/docs: README canonical plan; requirements change trace; learning-log mistakes/reviews; test-matrix rule→actual result; ADR alternatives/decision/consequences; architecture.txt ASCII map; gitignore build/secrets/checkpoints; editorconfig style; global.json actual supported SDK; solution project grouping; csproj framework/nullable/references; playground Program lab runner; optional ci.yml restore/build/test। No secrets/business rules in tooling files।

<a id="section-8"></a>

# 8. Data Models and Properties

R=required, O=optional। IDs/mobile string, money decimal, timestamps UTC DateTimeOffset। Credential encoded strings immutable; crypto boundary uses transient bytes। IReadOnlyList/record alone deep immutability নয়।

## User

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Id | string | R | opaque |
| Name | string | R | trimmed 2–80 |
| MobileNumber | string | R | unique normalized |
| Role | UserRole | R | registration User |
| Credential | PinCredential | R | no plaintext |
| CreatedAtUtc | DateTimeOffset | R | UTC |
| FailedPinAttempts | int | R | nonnegative |
| LockedUntilUtc | DateTimeOffset? | O | expiry |

## PinCredential

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| HashBase64 | string | R | derived key |
| SaltBase64 | string | R | fresh random |
| Algorithm | string | R | PBKDF2-SHA256 |
| Iterations | int | R | calibrated positive |
| KeyLengthBytes | int | R | configured |
| Version | int | R | format |

## Account

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Number | string | R | immutable; Personal mobile format, seeded Agent/System opaque ID |
| OwnerUserId | string? | O | required Personal/null seeded agent-system |
| Type | AccountType | R | kind |
| Status | AccountStatus | R | Active default |
| Balance | decimal | R | nonnegative two decimals |
| CreatedAtUtc | DateTimeOffset | R | UTC |

## Transaction

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Id | string | R | unique |
| InitiatorUserId | string | R | actor |
| Type | TransactionType | R | kind |
| SourceAccountNumber | string? | O | required Completed |
| DestinationAccountNumber | string? | O | required Completed |
| RequestedTarget | string? | O | failed unknown target |
| Amount | decimal | R | positive submitted |
| Fee | decimal | R | applied/Failed zero |
| QuotedFee | decimal? | O | quote evidence |
| Status | TransactionStatus | R | terminal |
| CreatedAtUtc | DateTimeOffset | R | accepted |
| CompletedAtUtc | DateTimeOffset? | O | Completed only |
| Reference | string? | O | safe &lt;=100 |
| FailureCode | string? | O | Failed only |
| PolicyVersion | string | R | version |
| RequestKey | string? | O | advanced |
| Postings | IReadOnlyList&lt;BalancePosting&gt; | R | defensive/Failed empty |

## BalancePosting

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| AccountNumber | string | R | existing |
| Delta | decimal | R | signed nonzero |
| BalanceBefore | decimal | R | nonnegative |
| BalanceAfter | decimal | R | before+delta |

## TransactionReceipt

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| TransactionId | string | R | Completed |
| Type | TransactionType | R | kind |
| TimestampUtc | DateTimeOffset | R | completion |
| SourceMasked | string | R | safe |
| DestinationMasked | string | R | safe |
| Amount | decimal | R | applied |
| Fee | decimal | R | applied |
| TotalDebit | decimal | R | source debit |
| ViewerBalanceAfter | decimal? | O | viewer only |
| Reference | string? | O | safe |

## AuditLogEntry

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Id | string | R | unique |
| ActorUserId | string | R | admin |
| Action | AuditAction | R | kind |
| TargetId | string? | O | safe |
| OccurredAtUtc | DateTimeOffset | R | UTC |
| Outcome | string | R | Succeeded/Denied/Failed |
| Reason | string? | O | required status change |
| CorrelationId | string | R | link |

## UserSession

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| UserId | string | R | identity |
| Role | UserRole | R | recheck calls |
| StartedAtUtc | DateTimeOffset | R | start |
| LastActivityAtUtc | DateTimeOffset | R | activity |
| SessionId | string | R | opaque |

## FeePolicy

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Version | string | R | ID |
| SendMoneyFlatFee | decimal | R | simulation |
| CashOutRate | decimal | R | fraction |
| CashInFee | decimal | R | 0 |
| RechargeFee | decimal | R | 0 |
| Rounding | MidpointRounding | R | AwayFromZero |

## TransactionLimits

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Type | TransactionType | R | kind |
| Minimum | decimal | R | positive |
| Maximum | decimal | R | &gt;=min |
| DailyMaximum | decimal | R | &gt;=max |

## AppConfiguration

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| FeePolicy | FeePolicy | R | immutable |
| Limits | IReadOnlyDictionary&lt;TransactionType, TransactionLimits&gt; | R | defensive |
| WalletCap | decimal | R | positive |
| MaxPinAttempts | int | R | 3 |
| LockDuration | TimeSpan | R | 5 minutes |
| IdleTimeout | TimeSpan | R | 5 minutes |
| BusinessTimeZoneId | string | R | supported UTC+06 |
| PolicyVersion | string | R | ID |
| HashIterations | int | R | calibrated |
| JsonEnabled | bool | R | false |

## TransactionRequest

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Type | TransactionType | R | kind |
| Target | string | R | normalized |
| Amount | decimal | R | positive |
| Reference | string? | O | safe |
| RequestKey | string? | O | retry |

## TransactionQuote

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Amount | decimal | R | requested |
| Fee | decimal | R | rounded |
| TotalDebit | decimal | R | amount+fee where applicable |
| PolicyVersion | string | R | recheck |

## DateRange

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| StartUtc | DateTimeOffset | R | inclusive |
| EndUtcExclusive | DateTimeOffset | R | &gt;start |

## TransactionQuery

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Type | TransactionType? | O | filter |
| Status | TransactionStatus? | O | filter |
| Range | DateRange? | O | filter |
| MinAmount | decimal? | O | inclusive |
| MaxAmount | decimal? | O | inclusive |
| Page | int | R | &gt;=1 |
| PageSize | int | R | 1–100 |
| Descending | bool | R | true |

## OperationResult&lt;T&gt;

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| Success | bool | R | outcome |
| Code | string | R | safe |
| Message | string | R | safe |
| Value | T? | O | success value |

## IdempotencyEntry

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| ActorUserId | string | R | scope |
| Key | string | R | retry key |
| RequestFingerprint | string | R | canonical payload |
| TransactionId | string | R | terminal |
| OriginalResult | OperationResult&lt;TransactionReceipt&gt; | R | safe result |

## UserDetailsDto

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| UserId | string | R | identity |
| Name | string | R | safe |
| MaskedMobile | string | R | safe |
| Role | UserRole | R | role |
| AccountStatus | AccountStatus? | O | wallet |
| CreatedAtUtc | DateTimeOffset | R | UTC |

## SummaryStatisticsDto

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| UserCount | int | R | User-role personal users only; Admin excluded |
| CompletedCount | int | R | count |
| FailedCount | int | R | count |
| CompletedVolume | decimal | R | sum |
| CollectedFee | decimal | R | sum |
| SuccessRate | decimal? | O | zero attempts null |

## AppState

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| UsersByMobile | Dictionary&lt;string, User&gt; | R | canonical |
| AccountsByNumber | Dictionary&lt;string, Account&gt; | R | canonical |
| TransactionsById | Dictionary&lt;string, Transaction&gt; | R | canonical |
| AuditEntries | List&lt;AuditLogEntry&gt; | R | append |
| IdempotencyIndex | Dictionary&lt;(string ActorId, string Key), IdempotencyEntry&gt; | R | advanced |
| BaselineTotal | decimal | R | seed total |
| Version | long | R | publication count |

## DataSnapshot [Optional]

| Property | Type | Required | Purpose/rule |
|---|---|---|---|
| SchemaVersion | int | R | supported |
| ExportedAtUtc | DateTimeOffset | R | time |
| Users | IReadOnlyList&lt;UserSnapshot&gt; | R | private DTO |
| Accounts | IReadOnlyList&lt;AccountSnapshot&gt; | R | balances |
| Transactions | IReadOnlyList&lt;TransactionSnapshot&gt; | R | terminal/postings |
| AuditEntries | IReadOnlyList&lt;AuditSnapshot&gt; | R | audit |
| IdempotencyEntries | IReadOnlyList&lt;IdempotencySnapshot&gt; | R | results |
| BaselineTotal | decimal | R | conservation |

## Model ownership/lookup/topic

User owns Credential; User role has one Personal Account, Admin default no wallet; seeded Agent/System owner null। Account has many postings; Transaction owns immutable postings and references actor/source/destination IDs। Receipt/DTO transient safe projection, not stored source of truth। Session references identity only। Config composed immutable policies।

UserService/AuthService replace User/credential/lockout through store; TransactionService prepares account replacements+terminal records; AdminService status+audit; AuditService prepares entries; SessionManager owns one session; startup owns config/seed; JsonDataStore validates optional checkpoint। Collection/lookup Section 9। Topics: class/constructor/encapsulation for User/Account; composition/immutability for credential/postings; record/struct for receipt/range; Dictionary/generics for store; async/serialization for checkpoint। Snapshot child DTOs mirror listed shapes, excluding session/plain PIN; version validation prevents silent invalid reconstruction।

<a id="section-9"></a>

# 9. In-memory Storage Design

| Collection | সুবিধা / limitation | Final use |
|---|---|---|
| List&lt;User&gt; | ordered/dynamic; mobile scan | first lab, canonical নয় |
| Dictionary&lt;string,User&gt; | average O(1) mobile; ordering assumed নয় | usersByMobile |
| Dictionary&lt;string,Account&gt; | key lookup; controlled replacement | accountsByNumber |
| List&lt;Transaction&gt; | append easy; ID scan | history lab, duplicate canonical নয় |
| Dictionary&lt;string,Transaction&gt; | ID uniqueness/lookup; queries scan | transactionsById |
| List&lt;AuditLogEntry&gt; | append/order; grows | owned audit list |
| HashSet&lt;string&gt; | membership; no payload/result | learning lab only |
| Dictionary&lt;(ActorId,Key),IdempotencyEntry&gt; | actor retry/result; extra memory | Advanced retry index |

একই entity List+Dictionary-তে দুটো canonical copy নয়। ID user lookup initially scan; measured need হলে secondary index same publication-এ। Repository safe DTO/immutable snapshot দেয়; read-only interface cast/nested mutation থেকে ownership রক্ষা একা করে না। IEnumerable শুধু enumeration contract; ICollection Count/mutation members দেয় কিন্তু read-only implementation mutation reject করতে পারে। [S4: IEnumerable](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1?view=net-10.0), [S5: ICollection](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.icollection-1?view=net-10.0)।

AppState store-owned; services prepare independent candidate values। Account replacement বা private copied candidate-এ controlled mutation; live Account mutate-then-rollback default নয়। Mutable nested reference share হলে candidate isolation ভাঙে। IReadOnlyList-এর elements-ও immutable/defensive copies হবে। Restart-এ data/history/lockout/retry index হারায়।

<a id="section-10"></a>

# 10. Data Relationships

```text
User [User role] 1 -------- 1 Personal Account
User [Admin role] 1 ------- 0 Personal Account [default]
User 1 ------------------- 1 owned PinCredential
Agent/System Account ----- OwnerUserId null [seeded]

Account 1 ------ * BalancePosting * ------ 1 Transaction
Transaction ---- InitiatorUserId ---- User
Transaction ---- SourceAccountNumber ---- Account
Transaction ---- DestinationAccountNumber ---- Account
Transaction ---- owns immutable postings
Transaction ---- derives Receipt [not another source of truth]

Service ---> Repository contract ---> In-memory view
Service ---> IStateStore -----------> Shared state publication
```

Completed transactions have valid source+destination including agent/settlement। Failed unknown target may have null destination + safe RequestedTarget, not dangling Account object। IDs avoid mutable bidirectional graphs। Composition means lifecycle ownership, not merely constructor injection।

<a id="section-11"></a>

# 11. Architecture and Design Decisions

```text
              Program.cs [composition root]
                        | wires
                        v
+--------------------------------------------------+
| UI: menus / input / receipt / notification        |
+------------------------+-------------------------+
                         | use cases
                         v
+--------------------------------------------------+
| Services: user/auth/account/transaction/admin     |
| pure validation / fee / safe result              |
+------------+-------------------+-----------------+
             | uses              | contracts
             v                   v
+-----------------------+  +-----------------------+
| Models / invariants   |  | Repo / State / Clock   |
| no Console/File I/O   |  | ID / Hasher / Gateway  |
+-----------------------+  | Logger                |
                          +-----------+-----------+
                                      ^ implements
                          +-----------+-----------+
                          | In-memory/fake adapters|
                          | optional JSON/file sink|
                          +------------------------+
```

UI→services; services→domain/contracts; infrastructure implements contracts; domain no UI/storage dependency। Runtime flow ও source dependency direction আলাদা। Folder boundary compiler-enforced নয়; review/tests enforce।

| Decision | সমস্যা / need | সহজ বিকল্প | Industry context / avoid |
|---|---|---|---|
| Service layer/SRP | UI rules ছড়িয়ে যায় | focused pure methods | use cases; one-line helper service নয় |
| Domain invariants | invalid state | pure validation+immutable shape | rule-rich models; DTO behavior নয় |
| Focused repository | storage substitution/test seam | owned Dictionary | generic CRUD force নয় |
| State commit boundary | multi-wallet/history consistency | candidate+one swap | unit-of-work ধারণা; ACID claim নয় |
| Manual DI | hidden globals/new | constructor args | testable seams; container এখন নয় |
| Policy/Strategy optional | growing switch | simple switch | actual variation হলে; topic পূরণে নয় |
| Abstract Template Method lab | fixed skeleton+variable steps | composition | genuine family behavior; duplicate engine নয় |
| DTO | safe projection | dedicated record | boundary; every model duplicate নয় |
| Immutability | shared drift | defensive copy | snapshots; nested mutable record যথেষ্ট নয় |
| Events | optional notification | callback | required audit/commit event-only নয় |
| Generic repo/Builder/partial lab | language/design trade-off | focused repo/query class | main project-এ unnecessary patterns নয় |

SOLID: S focused reasons-to-change; O policy only when variation warrants; L substitutes preserve contracts/invariants; I narrow query/append contracts; D clock/store/gateway seams। DRY shared business knowledge, identical-looking lines সব abstract নয়। KISS simplest correct design। Composition default preference, inheritance ban নয়।

Future DB requires durable commit/concurrency adapter too; repository বদলালেই business orchestration কখনো বদলাবে না দাবি নয়। Domain rules পুনর্লিখন কমানো লক্ষ্য। Database এখন নয়।

<a id="section-12"></a>

# 12. Transaction Data Flow and Integrity

```text
UI input -> parse/format -> session/ownership/PIN
 -> quote fee/total -> confirm -> current rules recheck
 -> prepare candidate balances/postings/terminal record
 -> publish ONE AppState
 -> committed result/receipt/history
 -> optional notification/log
```

UI parse/EOF/cancel; Auth actor নির্ধারণ, caller sender ID বিশ্বাস নয়; quote confirm; validator current status/balance/limits; service candidate plan; store one publication; receipt committed snapshot; notification after commit।

| Operation | Debit | Credit |
|---|---|---|
| Cash In | agent amount | personal amount |
| Cash Out | personal amount+fee | agent amount + fee wallet fee |
| Send Money | sender amount+fee | receiver amount + fee wallet fee |
| Recharge | personal amount | settlement amount |

**All accounts counted: total debit=total credit; signed sum=0।** Fee already credited; credit+fee আবার যোগ নয়। All wallet total including agent/fee/settlement conserved after seed, Cash In/Out-সহ। Real cash movement নেই।

Pseudocode only:

```text
read owned current snapshot
validate identity/participants/amount/policy/limits
prepare independent replacements and terminal record
prepare retry entry if enabled
validate candidate invariants and uniqueness
publish ONE state reference
return committed result
notify outside commit
```

Registration user+wallet; admin status+required audit same principle। Candidate copy O(n) acceptable learning trade-off; immutable unchanged elements share safely, mutable elements copy/replace। No await/I/O/callback between final validation and publication। Single-threaded console, thread-safe claim নয়; concurrent version needs coordination/version checks, atomic reference assignment alone নয়।

Failure contract:
- malformed/unauthenticated/cancelled quote: no financial attempt record; safe validation/auth diagnostics।
- authenticated, syntactically valid, PIN-confirmed submission: business rejection→Failed record, empty postings, applied Fee=0।
- staging/commit infrastructure fault: old live state intact; healthy store হলে separate Failed append চেষ্টা; append-ও fail হলে safe error/log, guaranteed history নয়।
- post-commit notification/print/log fault: Completed remains; no rollback/false Failed।
- fatal resource/corruption: safe stop; log/recovery guarantee নেই।

Pending internal; terminal stored records immutable। Optional reversal linked compensating transaction, original edit নয়।

Idempotency: actor+key scope; canonical type/target/amount/reference fingerprint (PIN নয়); same payload original terminal result, changed payload conflict; Failed business result cached too; new attempt new key। UI key once confirm, retry reuse। Replay needs authenticated authorized actor; no new financial effect। Result index+history+balances same commit। Restart index lost।

<a id="section-13"></a>

# 13. Business Rules

সব নিচের default invented learning policy, real bKash rules নয়।

| Rule | Contract |
|---|---|
| BR-01 | 11 ASCII digits, 01[3-9], trim only, unique; no silent +880 conversion |
| BR-02 | simulated PIN 5 ASCII digits as string; confirmation; all-same/adjacent ascending or descending digits reject (no wrap); no real PIN |
| BR-03 | name 2–80 trimmed; reference &lt;=100; control chars reject |
| BR-04 | registration User role; exactly one Personal wallet; zero balance; immutable number |
| BR-05 | 3 consecutive failed PIN checks→5-min lock; all PIN entry points share counter; lock invalidates session; success/expiry resets; generic unknown identity |
| BR-06 | Active participants only; suspended/closed no transactions; suspended login denied; Closed transition UI out of scope |
| BR-07 | amount positive, &lt;=2 fractional digits; reject extra precision, not silently round; parse overflow safe |
| BR-08 | nonnegative balance, source amount+fee sufficient; personal cap; decimal overflow checked |
| BR-09 | target exists/type valid; self-send reject; ownership from session |
| BR-10 | Cash In/Out Active seeded Agent; role-play acknowledgement; agent funds check |
| BR-11 | per-type min/max/daily; completed principal only, fee excluded; UTC+06 business day |
| BR-12 | fee 2 decimals AwayFromZero; quote/confirm; immutable policy version recheck |
| BR-13 | balance/new money submission PIN; no retained PIN; cash-in not real agent auth |
| BR-14 | actor+key+same payload one effect; changed payload conflict; original result index |
| BR-15 | balances/postings/history candidate publication; failed no money mutation |
| BR-16 | Guid candidate ID+unique insertion guard; collisions tested; no uniqueness proof from 10k sample |
| BR-17 | Completed receipt; historical snapshot; only viewer balance |
| BR-18 | own history/details; terminal immutable; Failed safe code |
| BR-19 | admin guard every service; reason status change; system/agent protected; status+audit atomic |
| BR-20 | no PIN/hash/salt/full mobile in logs/DTO/receipt; safe text; append-only not tamper-proof |
| BR-21 | one session; 5-min idle check protected calls; logout/PIN change/lock/suspension invalidate |
| BR-22 | signed postings zero; correct participant types; fee/settlement included |
| BR-23 | fake operator map; recharge 20–1000; no real portability inference |
| BR-24 | admin local supplied secret; no admin registration; seed agent funds startup only; no menu mint |
| BR-25 | old PIN verify; new different; fresh salt; logout |
| BR-26 | import version/IDs/relations/postings/index/baseline validation; one publication; no sessions |

| Type | Min | Max | Daily principal | Fee |
|---|---:|---:|---:|---|
| Cash In | 10 | 25000 | 50000 | 0 |
| Cash Out | 10 | 25000 | 50000 | amount×0.015 |
| Send Money | 10 | 25000 | 50000 | flat 5 |
| Recharge | 20 | 1000 | 5000 | 0 |

Personal cap 100000; seeded agent 1000000; fee/settlement 0; admin no wallet default। Agent/system no personal cap, nonnegative/range still applies। Cash In daily personal receiver incoming; other types personal initiator outgoing; receiver incoming Send not their outgoing usage। Seed baseline all wallets।

Input: ASCII digits/dot decimal/no grouping; display BDT/৳ two decimals। UTC timestamps; business local-day start/end convert UTC; DateTime.Now.Date নয়।

<a id="section-14"></a>

# 14. Error Handling Strategy

Expected validation/auth/business errors safe OperationResult (InvalidInput, Unauthorized, InactiveAccount, InsufficientBalance, LimitExceeded, DuplicateConflict)। Exceptions exceptional infrastructure/programming failures; custom exception lesson lab/boundary, invalid menu choice throw নয়।

UI safe messages; Program last boundary safe diagnostic+exit policy। Catch specific recoverable exceptions; no empty catch; rethrow throw preserves stack, throw ex alters it। Finally cleanup ordinary control flow/unwinding, fatal termination guarantee নয়। [S6: exception statements](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements)।

No blanket catch-and-continue after suspected state corruption। Post-commit result distinguish from notification failure। Exception filter side-effect-free।

<a id="section-15"></a>

# 15. Security Considerations

Simulation controls, real financial safety guarantee নয়। Short PIN space ছোট; hash+local lock alone offline guessing/local attacker ঠেকায় না। No custom crypto/plaintext fallback/reversible PIN store।

Hasher fresh random salt, explicit PBKDF2-SHA256 work factor/version, constant-time equal-length derived-byte comparison। [S19: PBKDF2](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.rfc2898derivebytes.pbkdf2?view=net-10.0), [S20: FixedTimeEquals](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicoperations.fixedtimeequals?view=net-10.0)। Work factor Phase 3-এ current guidance+device benchmark দেখে document; universal secure iteration claim নয়। Hash/salt sensitive; transient PIN .NET string reliably erase দাবি নয়। Auth verifies identity; authorization checks role/ownership every service।

Secret masked local prompt/environment, no Git; environment/user-secrets encrypted vault নয়। Least privilege: own wallet/history; admin no credentials/no arbitrary balance edit। Masked input not security boundary। Lockout local/restart-reset; service guards even direct call।

| Threat | Mitigation | Residual |
|---|---|---|
| guesses | shared counter/lock | restart/offline |
| role/ID spoof | service identity/ownership | local code tampering |
| duplicate retry | actor+key/result | restart loss |
| sensitive logs | safe DTO/redaction | debugger/memory |
| checkpoint tampering | whole candidate validation | authenticity not proven |
| half update | candidate publication | crash memory loss |

Diagnostic logging: UTC/level/event/correlation/actor ID/transaction ID/safe code; no PIN/hash/full mobile; safe free text। Audit separate business record; required status audit candidate-এ, event-only নয়। Admin read/search audit append failure→do not release requested result, safe error; denied action logging best effort, authorization never becomes success। Logger failure cannot undo committed money।

Config startup validates fee/limit ordering/cap/timeouts/hash settings/timezone; immutable snapshot+version; change restart/revalidated snapshot, quote mismatch requote।

<a id="section-16"></a>

# 16. Testing Strategy

Unit: pure model/fee/validator/services with fakes। Integration: actual in-memory collaborators together, no database/network। Mock verifies interaction; fake controlled working substitute; framework optional। Fresh store/test, no real sleeps/shared statics।

| Scenario | Expected |
|---|---|
| duplicate/registration fault | no extra user/wallet |
| wrong PIN 1/2/3, expiry | counter/lock/session correct |
| inactive participant | Failed, no postings |
| 0,-1,9.99,10,25000,25000.01 | type boundary outcomes |
| recharge 19.99,20,1000,1000.01 | reject/pass/pass/reject |
| 1.001/huge/empty/EOF | reject safely/exit; no silent precision |
| exact amount+fee / cent short | success/failure |
| cap exact / cent above | success/failure |
| daily remaining/midnight | correct day/limit |
| self-send/unknown | Failed no money |
| candidate debit/credit/history faults | old state intact |
| commit then notification/log fault | Completed retained |
| ID collision | no overwrite/effect |
| retry same/changed/different actor | original/conflict/independent |
| other user's history/detail | denied/not-found |
| direct non-admin service call | denied |
| required audit failure | status unchanged |
| empty report | counts 0, ratio N/A |
| DTO/nested-copy mutation | live state unaffected |
| corrupt/version/dangling JSON [O] | old state intact |

Invented fixture: sender 1000, receiver 100, fee 0; Send 200 fee 5→795/300/5; total 1100 unchanged। Failure before publication→1000/100/0। Cash Out 100 fee1.50→source debit101.50, agent+100, fee+1.50।

Gate: rule pass/fail/boundary; conservation/nonnegative; failed no applied fee/postings; actual build/test evidence; no Console in services; no mutable leak; manual happy/failure flow; docs updated। Coverage number not correctness proof। TDD one red→green→refactor exercise; every UI line mandatory TDD নয়।

<a id="section-17"></a>

# 17. Development Roadmap

| Phase | কাজ / new files | Topics | নিজের task / exit gate |
|---|---|---|---|
| P0 | SDK/Git/docs/Program/nullable | tooling/dependencies | empty app builds; explain scope; first commit |
| P1 | menu/input/format | types/operators/conditions/loops/methods/arrays/string/TryParse | own menu+invalid+EOF; no money code |
| P2 | User/Account/enums/xUnit | class/properties/access/constructor/this/encapsulation/value-reference | models+invariant tests; no public balance setter |
| P3 | store/repos/auth/credential/session/config/result | collections/generics/interfaces/manual DI/nullable/exceptions/time | atomic registration/login/logout+lockout tests |
| P4 | account service/seed/user menu | decimal/format/readonly/const/initializers | balance/status; explicit seed baseline |
| P5 | transaction/postings/fee/limits/validator/ID/history | atomicity/failure/rounding/basic LINQ | cash in/send/out/history; fault tests; MVP gate |
| P6 | OOP/runtime labs, optional policies | inheritance/abstract/override/new/sealed/partial/copy/equality/boxing | explain alternatives; no forced hierarchy |
| P7 | profile/PIN/timeout/recharge/receipt/query/notice | record/struct/StringBuilder/lambda/LINQ/delegate/event/yield | safe user Core; fake recharge initially synchronous |
| P8 | admin/audit/logger | authorization/DTO/GroupBy/structured logging | admin+atomic audit; Core gate |
| P9 | async fake gateway/retry/report finish/thread lab | async/Task/Thread/cancel/lock/Interlocked/idempotency | cancel/failure/retry tests; race lab; Advanced gate |
| P10 | analyzer/format/profile/CI concepts | quality/refactor/observability/deployment | 10k synthetic timing; topic evidence audit |
| P11 [O] | checkpoint/JSON/file logger | async I/O/serialization/disposal | valid roundtrip/corrupt import/old state preservation |

MVP session identity/login/logout আছে; Core-এ idle timeout ও PIN-change invalidation complete হবে।

Tests earlier than draft Phase4: correctness before money engine। Interfaces/clock when auth needs them, deeper theory P6। Fee/config MVP। Events after reliable commit। Async only fake delay/I/O, not report wrapper; Thread separate lab। Required async/disposal topics still lab even JSON skipped।

Await fake gateway before final commit, then recheck session/status/balance/limits। No external side effect; cancel before commit no movement, after commit Completed। No lock across await; Task not necessarily thread-pool/new thread; async doesn't automatically create responsive menu. [S3: async](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)।

<a id="section-18"></a>

# 18. Feature-wise Topic Mapping

Every F-ID topics linked Section2; dependency grouping:

| Features | First lesson | Reuse |
|---|---|---|
| 37/38 | P1 control flow/input | throughout |
| 01/04/05/08/42 | P2–3 models/collections | all use cases |
| 02/07 | P3 nullable/time/DI | P7 timeout |
| 09/11/12/41 | P3–4 decimal/invariant/config | money engine |
| 13–15/20/21/23/24/26/27 | P5 posting/fee/atomicity | P6 policy lab/P9 retry |
| 17/18/22 | P5 basic LINQ/P7 query | reports |
| 03/06/10/19 | P7 DTO/PIN/receipt | auth/decimal/time |
| 16 | P7 fake/P9 async | cancellation/errors |
| 28–36 | P8–9 admin/audit/report | LINQ/immutability |
| 25 | P9 retry | dictionary/equality |
| 39/40/44/45 | P2–3 tests/errors/P8 logging | quality |
| 43 | P11 optional | async/disposal/serialization |

<a id="section-19"></a>

# 19. Foundation and OOP Coverage

Section24 has dedicated **19 Foundation + 30 OOP + 19 Runtime entries**, not merely mentions। Static Members and Static Class separate; Association/Aggregation/Composition separate; record/struct/enum one Foundation entry with three subexamples। Each entry supplies phase/file/feature, prerequisite, class, purpose, industry context, task/test/review।

Core = application correctness; Supporting = quality/literacy/lab; Optional = app integration optional, required topic lesson still mandatory। Learning evidence: own explanation→predict output→small task→test→review→transfer to feature। Planned coverage ≠ implemented/mastered। Section24 index is clickable coverage matrix; all 68 entries must reach Reviewed or Needs revision explicitly, no auto completion।

<a id="section-20"></a>

# 20. Additional Industry Topics

Beyond language topics, these are additional learning modules, not forced features:

| Topic(s) | Class/phase | Project context / task |
|---|---|---|
| Clean code/SOLID/DRY/KISS | Core/P1–10 | focused methods; Section11 decision review |
| Composition over inheritance | Core/P6 | compare policy vs subclass; preserve invariants |
| Coupling/cohesion/dependency graph | Core/P3,P6 | draw object/project/package dependencies; no cycles |
| Refactoring/appropriate abstraction | Core/P6,P10 | green tests; explain simpler alternative |
| Code review/debugging | Core/every phase | breakpoint/watch/call stack/conditional breakpoint; regression |
| Unit/boundary/integration tests | Core/P2–9 | Section16; fake vs actual in-memory collaborators |
| TDD/mocking | Supporting/P2,P5 | one red-green-refactor; interaction verification only if meaningful |
| Git/GitHub/documentation/SemVer | Core+Supporting/P0,P10 | small commits/ADR/milestone tags |
| Parsing/validation/business enforcement | Core/P1–5 | format vs auth vs domain; UI-only guards forbidden |
| State/service responsibility/consistency | Core/P3–5 | store ownership/candidate commit |
| Transaction/idempotency/audit | Core/P5,P8,P9 | postings/result index/required audit same publication |
| Config/error reporting/structured logging | Core/P3–8 | immutable policy/version/safe codes/correlation |
| Authentication vs authorization | Core/P3,P8 | direct service ownership/role tests |
| PIN/hash/secrets/least privilege | Core/P3,P8 | no real PIN; hasher seam; safe DTOs |
| Sensitive logging/session/auth state | Core/P3,P7 | redaction; logout/lock/timeout/PIN invalidation |
| Rate limiting/threat model | Supporting/P3,P8 | local attempts; residual risks, not distributed protection |
| CI/CD/static analysis/code quality | Supporting+Optional/P10 | restore/build/test; analyzer review; CD concept only |
| Profiling/observability | Supporting/P8,P10 | timings/allocation/logs/metrics/traces concept; no SLA |
| Config management/deployment | Supporting/P10 | version/restart; publish runtime-dependent vs self-contained concept |
| Future API/database | Optional discussion/P10 | DTO adapter/durable commit/concurrency; no DB now |
| Culture/timezone/rounding/overflow | Core/P1,P5 | money/day boundaries; candidate checked before publish |
| Property-style invariant tests | Supporting/P10 | generated simulated sequences; conservation |
| Serialization/schema/version | Optional/P11 | reject unsupported checkpoint; explicit migration plan |
| Package hygiene | Supporting/P0,P10 | minimal compatible dependencies; vulnerability review |

Industry modules first explain purpose/prerequisites/alternatives, then one own task+verification; Section21 teaching contract applies।

<a id="section-21"></a>

# 21. Teaching and Review Workflow

Every first lesson: (1) Topic Name (2) সহজ Definition (3) Purpose (4) Mental Model (5) Syntax explained (6) Small outside example (7) Project Context (8) Code Idea/hint (9) Data Flow (10) Industry Usage (11) Best Practices (12) Alternatives (13) My Task (14) Testing expected results (15) Review। Complexity অনুযায়ী length। Section24 gives reference baseline; live teaching adds line-by-line explanation/debugging when needed।

No full project/full feature/full file source। Small isolated snippets allowed; task then wait for own code। Hint-first, not reveal solution। Review correctness→invariants/auth→errors/tests→design/readability; explain why, not only correction। Reused topic “Topics used: List&lt;T&gt;, foreach, LINQ”, no repeated full lecture unless asked/forgotten। Pattern before problem/need/simple alternative/industry/avoidance।

Deep-learning ladder: predict without running → run lab → explain difference → write own variation → boundary test → feature application or justify no application → review। Mastery requires evidence, README length নয়।

<a id="section-22"></a>

# 22. Feature Completion Checklist

- [ ] action/requirements and Section2 acceptance
- [ ] happy/invalid/EOF/cancel
- [ ] rules/auth/session/ownership
- [ ] money/time/fee/limit boundaries
- [ ] pre/post-commit failure semantics
- [ ] no mutable leak/sensitive output
- [ ] focused readable code
- [ ] actual tests/manual workflow
- [ ] new/reused topic evidence
- [ ] next dependency ready
- [ ] docs/learning-log/commit updated

```text
Feature: F-ID
Completed: verified behaviors + actual evidence
Remaining: unmet criteria
Topics Learned: new
Topics Reused: previous
Problems Found: bug/cause/hint/fix
Next Task: one self-implementation task
```

No automatic Completed/100%-bug-free claim।

<a id="section-23"></a>

# 23. Known Limitations and Future Extensions

## Known limitations

Restart/crash loses memory/lockout/retry; no durable ACID/concurrent multi-user/real financial integration। Candidate publication single-process consistency, not crash recovery। Short PIN/offline hash/local tampering risks; append-only not tamper-proof। Seeded agent role-play not real authorization। Fake prefix not real operator/portability। Signed postings educational, not certified ledger। Fatal resource failure no log/history guarantee। Unbounded history/candidate O(n) copy; paging not retention। Terminal Unicode/color/input limitations need fallback। Tests reduce defects, no production proof।

## Optional checkpoint

Admin explicit private Export/Import; no default auto-save। Public report export masked DTO/no credentials, not login-restorable checkpoint। Private checkpoint may include hash/salt/parameters, never PIN; short PIN offline risk; restricted local file, no Git/share। Sessions excluded; import logs out।

Deserialize dedicated versioned DTO→validate duplicate IDs/one personal wallet/owner/status/nonnegative/terminal records/posting chains/fees/index/baseline→candidate→one publication। Total sum alone insufficient; per-account before/after consistency and references too। Current admin identity/role must remain valid in candidate; append import audit with publication। Authenticity not proven by shape validation।

Save temp file same directory→flush/close→supported replace/rename/backup; platform crash guarantees limited, not durable DB। Corrupt/version/permission failure preserves old live state; tests temporary directory। Last checkpoint after restart, subsequent changes lost।

| Extension | Prerequisite / acceptance |
|---|---|
| DI container/appsettings | manual DI/config understood; no service locator/secrets; same tests |
| CSV statement | safe DTO; formula-like fields escaped; no credentials |
| Reversal | linked compensating postings, original immutable, funds/fee policy |
| Agent creation | admin/audit/reallocation policy; no arbitrary mint |
| Multi-language | UI resources; rules unchanged |
| CI/profiling/indexes | local green tests; measured need; index same commit |
| Future API/DB | explicit new scope; auth/concurrency/durable commit review; no DB now |

No microservices/CQRS/event sourcing/message broker just to look industry-level।

<a id="section-24"></a>

# 24. Complete Topic Glossary

**Required coverage: 19 Foundation + 30 OOP + 19 Runtime = 68 dedicated entries।** প্রথমে prerequisite, তারপর example output predict, তারপর own task। Core/Supporting/Optional application classification lesson বাদ দেওয়ার অনুমতি নয়। সব 68 lessons mandatory; lab feature integration optional হতে পারে।

Snippets isolated fragments, same project/file-এ সব paste করবে না। Type declarations আর top-level statements আলাদা context লাগে; comments indicate member fragments। Standard namespaces System/Collections.Generic/Linq/Threading/Tasks/IO যেখানে দরকার add করবে। Nullable enabled। Expected compile failures lab-এ isolate, main build-এ নয়। **এই README তৈরি করতে application বা C# snippets execute করা হয়নি; expected results teaching targets।**

## 24.0 Master Index / Coverage Matrix

### 24.A – C# Foundation

| # | Topic | Phase / file / feature | Prerequisite / classification |
|---|---|---|---|
| 1 | [Variables & Data Types](#f-variables) | P1/P4; F-38/09; InputReader/Account | none; Core |
| 2 | [Operators](#f-operators) | P1/P5; F-26/23; FeeCalculator/Validator | types; Core |
| 3 | [Conditions](#f-conditions) | P1/P5; F-37/23; MainMenu/Validator | operators; Core |
| 4 | [Loops](#f-loops) | P1/P5; F-37/17; MainMenu/history | conditions; Core |
| 5 | [Methods](#f-methods) | P1/P3; F-38/01; InputReader/UserService | types/control flow; Core |
| 6 | [Arrays & Collections](#f-arrays) | P1/P3; menu/boundary fixtures; FoundationLab | loops/methods; Core |
| 7 | [List, Dictionary, HashSet](#f-collections) | P3/P9; F-42/25; AppState/GenericRepositoryLab | arrays/generics intro; Core |
| 8 | [String & StringBuilder](#f-string) | P1/P7; F-38/19; InputReader/ReceiptPrinter | types/loops; Core |
| 9 | [Exception Handling (General)](#f-exception-handling) | P3/P5; F-39/24; boundary/custom exception lab | methods/control flow; Core |
| 10 | [Generics](#f-generics) | P3/P6; F-42/39; OperationResult/GenericRepositoryLab | methods/class; Core |
| 11 | [Delegates](#f-delegates) | P7; F-22; query/FoundationLab | methods/generics; Supporting |
| 12 | [Events](#f-events) | P7; optional notice; ConsoleNotificationSubscriber | delegates; Supporting |
| 13 | [Lambda Expressions](#f-lambda) | P7; F-22/30; query/AdminService | delegates/methods; Core |
| 14 | [LINQ](#f-linq) | P5 basic/P7–8 deep; F-17/22/34; services | collections/predicates; Core |
| 15 | [async / await](#f-async) | P9/P11; F-16/43; fake gateway/AsyncLab | methods/Task/errors; Supporting |
| 16 | [Task & Thread Basics](#f-task-thread) | P9; AsyncLab/ThreadRaceLab; F-16 support | delegates/methods/async intro; Supporting |
| 17 | [record, struct, enum](#f-record-struct-enum) | P2/P7; F-04/19/22; enums/receipt/DateRange | class/value-reference; Core |
| 18 | [Nullable Reference Types](#f-nullable) | P0 enabled/P3 explained; F-07/18; session/repo | reference/conditions; Core |
| 19 | [Dependency Concepts](#f-dependency) | P0/P3/P10; Program/csproj/contracts | class/methods; Core |

### 24.B – OOP

| # | Topic | Phase / file / feature | Prerequisite / classification |
|---|---|---|---|
| 1 | [Class & Object](#o-class-object) | P2; F-01/08; User/Account | methods/types; Core |
| 2 | [Fields vs Properties](#o-fields-properties) | P2; F-12; Account | class; Core |
| 3 | [Access Modifiers](#o-access-modifiers) | P2/P6; Account/AccountInheritanceLab | class/properties; Core+Supporting protected |
| 4 | [Constructor](#o-constructor) | P2; F-08; Account | class/properties; Core |
| 5 | [Constructor Overloading](#o-constructor-overload) | P2/P6; model lab/seed | constructor; Supporting |
| 6 | [this Keyword](#o-this) | P2; Account/model lab | constructor/fields; Core |
| 7 | [Static Members](#o-static-members) | P4/P6; SeedData/lab; F-21 supporting | class; Supporting |
| 8 | [Static Class / Static Methods](#o-static-class) | P1/P4; InputValidator/extensions | methods/class; Core |
| 9 | [Readonly & Const](#o-readonly-const) | P3/P4; config/service fields | fields/types; Core |
| 10 | [Object Initializers](#o-object-initializer) | P4; seed/config/test DTO | constructor/properties; Core |
| 11 | [Encapsulation](#o-encapsulation) | P2/P5; F-12/27; Account/store | access/properties; Core |
| 12 | [Inheritance](#o-inheritance) | P6; AccountInheritanceLab | class/access/constructor; Supporting |
| 13 | [Polymorphism](#o-polymorphism) | P3/P6; clock/hasher/gateway/optional policy | interface or inheritance; Core |
| 14 | [Method Overloading](#o-method-overload) | P4/P6; formatting/helper lab | methods/types; Supporting |
| 15 | [Method Overriding](#o-method-override) | P6; AccountInheritanceLab | inheritance/virtual; Supporting |
| 16 | [virtual, override, new](#o-virtual-override-new) | P6; AccountInheritanceLab | inheritance/override; Supporting |
| 17 | [Abstraction](#o-abstraction) | P3/P6; IClock/IStateStore | class/methods; Core |
| 18 | [Interface](#o-interface) | P3; repo/clock/hasher/gateway | abstraction; Core |
| 19 | [Abstract Class](#o-abstract-class) | P6; HandlerTemplateLab | inheritance/abstraction; Supporting |
| 20 | [Association](#o-association) | P2/P5; User/Account/Transaction IDs | class; Core |
| 21 | [Aggregation](#o-aggregation) | P6; report groups existing transactions; lab | association/collections; Supporting |
| 22 | [Composition](#o-composition) | P3/P5; User credential/Transaction postings | association/immutability; Core |
| 23 | [Sealed Class](#o-sealed-class) | P6; lab/adapter choice | inheritance; Supporting |
| 24 | [Sealed Method](#o-sealed-method) | P6; HandlerTemplateLab | override; Supporting |
| 25 | [Partial Class](#o-partial-class) | P6; PartialConsoleLab | class; Optional app/mandatory lab |
| 26 | [Nested Class](#o-nested-class) | P7; QueryBuilderLab | class/access; Optional app/mandatory lab |
| 27 | [ToString(), Equals(), GetHashCode()](#o-object-methods) | P6; CopyEqualityLab/DateRange | class/value-reference; Core+Supporting custom overrides |
| 28 | [Reference Type vs Value Type](#o-ref-vs-value) | P2/P6; Account/decimal/DateRange/lab | types/class; Core |
| 29 | [Shallow Copy vs Deep Copy](#o-copy) | P5/P6; AppState/CopyEqualityLab | reference/value/collections; Core |
| 30 | [Immutability](#o-immutability) | P3/P5/P7; credential/history/receipt | encapsulation/copy; Core |

### 24.C – C# Language & Runtime

| # | Topic | Phase / file / feature | Prerequisite / classification |
|---|---|---|---|
| 1 | [Value Types vs Reference Types (Deep)](#r-value-reference-deep) | P6; RuntimeLab/CopyEqualityLab; F-42/27 | o-ref-vs-value/copy; Supporting deep lesson |
| 2 | [Stack ও Heap + Misconceptions](#r-stack-heap) | P6; RuntimeLab; DateRange design | value/reference; Supporting |
| 3 | [Boxing ও Unboxing](#r-boxing) | P6; RuntimeLab; generic collection rationale | types/casts; Supporting |
| 4 | [Type Conversion](#r-type-conversion) | P1/P6; InputReader/RuntimeLab | types/operators; Core+Supporting |
| 5 | [Pattern Matching](#r-pattern-matching) | P5/P7; validator/status/query | conditions/types; Core |
| 6 | [Extension Methods](#r-extension-methods) | P4/P7; MoneyExtensions/StringExtensions | static/methods/this; Supporting |
| 7 | [Optional ও Named Parameters](#r-optional-named-params) | P6; RuntimeLab/query/helper | methods/overloading; Supporting |
| 8 | [params, ref, out, in](#r-params-ref-out-in) | P1 out/P6 deeper; InputReader/RuntimeLab | methods/value-reference; Supporting |
| 9 | [IEnumerable&lt;T&gt; vs ICollection&lt;T&gt;](#r-ienumerable-icollection) | P3/P7; repository safe results | collections/interfaces; Core |
| 10 | [Iterators ও yield](#r-yield) | P7; history paging lab; FoundationLab | loops/IEnumerable; Supporting |
| 11 | [Deferred Execution](#r-deferred-execution) | P7/P8; query/ReportService | LINQ/iterators; Core |
| 12 | [Equality ও Hashing](#r-equality-hashing) | P6/P9; CopyEqualityLab/retry | object methods/collections; Core |
| 13 | [IDisposable ও using](#r-idisposable) | P6 lab/P11 optional; FileAppLogger/JSON | exceptions/resources; Supporting |
| 14 | [Garbage Collection](#r-gc) | P6/P10; RuntimeLab/history memory | reference/lifecycle; Supporting |
| 15 | [is, as, typeof, nameof](#r-is-as-typeof-nameof) | P6; RuntimeLab/validator | types/patterns; Supporting |
| 16 | [Generics Constraints](#r-generics-constraints) | P6; GenericRepositoryLab | generics/interfaces; Supporting |
| 17 | [Exception Filters](#r-exception-filters) | P5/P11; boundary/lab | exception handling/conditions; Supporting |
| 18 | [Async Exception Handling](#r-async-exception) | P9; fake gateway/AsyncLab; F-16 | async/errors; Supporting |
| 19 | [CancellationToken](#r-cancellation-token) | P9/P11; AsyncLab/fake gateway/JSON | Task/async/errors/disposal; Supporting |

## 24.A – C# Foundation

<a id="f-variables"></a>

### Variables & Data Types

**1. Topic Name:** Variables & Data Types।

**2. Definition:** typed variable value/reference রাখে; var inferred static type, dynamic নয়।

**3. Purpose:** typed domain/API/config-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1/P4; F-38/09; InputReader/Account।

**4. Mental Model:** সঠিক বাক্সে সঠিক data: mobile/PIN text, money decimal।

**5. Syntax Explained:** string text, decimal m suffix, bool condition; var RHS থেকে type infer করে।

**6. Small Example (isolated, project implementation নয়):**

```csharp
string code = "00125";
decimal price = 19.95m;
```

**7. Project Context:** P1/P4; F-38/09; InputReader/Account। **Prerequisite/classification:** none; Core।

**8. Code Idea / Hint:** mobile/PIN/amount/date/status-এর type বেছে কারণ লিখবে। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** input text → typed parse/value → valid domain field → formatted output।

**10. Industry Usage:** typed domain/API/config। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** decimal finite precision/range; rounding প্রয়োজন।

**12. Alternatives:** integer minor units discussion, এখন decimal।

**13. My Task:** mobile/PIN/amount/date/status-এর type বেছে কারণ লিখবে। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** 00125 leading zero থাকে; decimal amount exact intended decimal; invalid type operation compile failure। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="f-operators"></a>

### Operators

**1. Topic Name:** Operators।

**2. Definition:** arithmetic/comparison/logical/assignment operators; && short-circuit।

**3. Purpose:** pricing/eligibility-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1/P5; F-26/23; FeeCalculator/Validator।

**4. Mental Model:** হিসাব ও অনুমতি: amount+fee, active&&enough।

**5. Syntax Explained:** + arithmetic, &gt;= comparison, && short-circuit; = assignment vs == equality।

**6. Small Example (isolated, project implementation নয়):**

```csharp
decimal total = 100m + 1.5m;
bool valid = total > 0m && total <= 200m;
```

**7. Project Context:** P1/P5; F-26/23; FeeCalculator/Validator। **Prerequisite/classification:** types; Core।

**8. Code Idea / Hint:** fee expression, precedence, short-circuit নিজে পরীক্ষা। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** amount/rate/status → arithmetic/comparison → fee/permission result → validation।

**10. Industry Usage:** pricing/eligibility। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** complex expression named helper; decimal overflow before commit।

**12. Alternatives:** simple if।

**13. My Task:** fee expression, precedence, short-circuit নিজে পরীক্ষা। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** total101.5; equality vs assignment; null guard short-circuit। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="f-conditions"></a>

### Conditions

**1. Topic Name:** Conditions।

**2. Definition:** if/switch branches; switch expression produces value, statement workflow চালায়।

**3. Purpose:** navigation/rules-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1/P5; F-37/23; MainMenu/Validator।

**4. Mental Model:** রাস্তার sign দেখে এক পথ; menu বনাম label selection।

**5. Syntax Explained:** switch arms pattern =&gt; result; _ fallback; statement switch void workflow।

**6. Small Example (isolated, project implementation নয়):**

```csharp
int choice = 2;
string label = choice switch { 1 => "Start", 2 => "Help", _ => "Unknown" };
```

**7. Project Context:** P1/P5; F-37/23; MainMenu/Validator। **Prerequisite/classification:** operators; Core।

**8. Code Idea / Hint:** menu switch statement, pure label expression; void call expression-এ জোর নয়। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** choice/status → matched branch → use case or safe fallback।

**10. Industry Usage:** navigation/rules। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** guard clauses reduce nesting; if simpler for few conditions।

**12. Alternatives:** সরল focused method/type; প্রয়োজন ছাড়া abstraction নয়।

**13. My Task:** menu switch statement, pure label expression; void call expression-এ জোর নয়। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** 2 Help;99 Unknown; invalid choice safe loop। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="f-loops"></a>

### Loops

**1. Topic Name:** Loops।

**2. Definition:** for/while/do-while/foreach repetition; break/continue/return আলাদা।

**3. Purpose:** batch/input/retry-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1/P5; F-37/17; MainMenu/history।

**4. Mental Model:** menu repeat বনাম history traversal; exit condition আবশ্যক।

**5. Syntax Explained:** foreach variable gets each element; while condition before, do after; break exits loop।

**6. Small Example (isolated, project implementation নয়):**

```csharp
foreach (string item in new[] { "A", "B" })
    Console.WriteLine(item);
```

**7. Project Context:** P1/P5; F-37/17; MainMenu/history। **Prerequisite/classification:** conditions; Core।

**8. Code Idea / Hint:** menu exit/EOF; foreach empty collection; for/while compare। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** initial menu state → prompt → action → exit/next iteration; history sequence → each item।

**10. Industry Usage:** batch/input/retry। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** enumeration-এর সময় collection mutate নয়।

**12. Alternatives:** LINQ pure query।

**13. My Task:** menu exit/EOF; foreach empty collection; for/while compare। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** A then B; empty zero iterations; EOF exits। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="f-methods"></a>

### Methods

**1. Topic Name:** Methods।

**2. Definition:** named behavior parameters/return নেয়; by-value default copies argument value, reference valueও।

**3. Purpose:** services/calculations-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1/P3; F-38/01; InputReader/UserService।

**4. Mental Model:** এক station: input→focused work→output।

**5. Syntax Explained:** return type/name/parameters/body; =&gt; expression-bodied method, not always lambda।

**6. Small Example (isolated, project implementation নয়):**

```csharp
static int Square(int value) => value * value;
```

**7. Project Context:** P1/P3; F-38/01; InputReader/UserService। **Prerequisite/classification:** types/control flow; Core।

**8. Code Idea / Hint:** pure validation ও UI method আলাদা করবে; parameter/local scope explain। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** caller arguments → parameter values → focused body → return/result or explicit side effect।

**10. Industry Usage:** services/calculations। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** small verb names; no god method।

**12. Alternatives:** local function for local helper।

**13. My Task:** pure validation ও UI method আলাদা করবে; parameter/local scope explain। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** Square3=9; return ends method; side effects identified। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="f-arrays"></a>

### Arrays & Collections

**1. Topic Name:** Arrays & Collections।

**2. Definition:** array fixed length indexed reference type; elements mutable; collection broader family।

**3. Purpose:** buffers/fixed fixtures-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1/P3; menu/boundary fixtures; FoundationLab।

**4. Mental Model:** fixed locker বনাম expandable shelf।

**5. Syntax Explained:** T[] array; index starts0; Length fixed; Array.Sort mutates sequence।

**6. Small Example (isolated, project implementation নয়):**

```csharp
int[] values = { 3, 1, 2 };
Array.Sort(values);
```

**7. Project Context:** P1/P3; menu/boundary fixtures; FoundationLab। **Prerequisite/classification:** loops/methods; Core।

**8. Code Idea / Hint:** array Length/index bounds এবং dynamic List compare। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** fixed array → index/foreach → element; Sort → same array reordered।

**10. Industry Usage:** buffers/fixed fixtures। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** fixed size≠immutable; assignment aliases।

**12. Alternatives:** List dynamic size।

**13. My Task:** array Length/index bounds এবং dynamic List compare। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** sorted1,2,3; empty0; invalid index throws in lab। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="f-collections"></a>

### List, Dictionary, HashSet

**1. Topic Name:** List, Dictionary, HashSet।

**2. Definition:** List ordered dynamic; Dictionary average O(1) key lookup; HashSet membership/uniqueness, result cache নয়।

**3. Purpose:** indexes/caches/sets-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3/P9; F-42/25; AppState/GenericRepositoryLab।

**4. Mental Model:** তালিকা/সূচি/unique stamp।

**5. Syntax Explained:** &lt;K,V&gt; key/value types; Add duplicate behavior; TryGetValue bool+out value।

**6. Small Example (isolated, project implementation নয়):**

```csharp
var map = new Dictionary<string, int> { ["A"] = 2 };
var seen = new HashSet<string>();
bool first = seen.Add("A");
```

**7. Project Context:** P3/P9; F-42/25; AppState/GenericRepositoryLab। **Prerequisite/classification:** arrays/generics intro; Core।

**8. Code Idea / Hint:** same data lookup/order/duplicates compare; retry original result dictionary design। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** normalized key → Dictionary lookup; ordered items → List traversal; key → HashSet membership।

**10. Industry Usage:** indexes/caches/sets। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** ordering business contract নয়; HashSet alone no payload/result।

**12. Alternatives:** scan tiny non-key data।

**13. My Task:** same data lookup/order/duplicates compare; retry original result dictionary design। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** second Add false; missing TryGetValue false; duplicate dictionary Add rejected। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="f-string"></a>

### String & StringBuilder

**1. Topic Name:** String & StringBuilder।

**2. Definition:** string immutable reference; StringBuilder mutable text builder; culture formatting explicit।

**3. Purpose:** parsing/display/reports-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1/P7; F-38/19; InputReader/ReceiptPrinter।

**4. Mental Model:** new লেখা বনাম scratchpad draft।

**5. Syntax Explained:** $ interpolation; F2 two decimals; AppendLine adds newline; ToString materializes।

**6. Small Example (isolated, project implementation নয়):**

```csharp
var builder = new System.Text.StringBuilder();
builder.AppendLine("Note");
builder.AppendLine($"Value: {12.5m:F2}");
```

**7. Project Context:** P1/P7; F-38/19; InputReader/ReceiptPrinter। **Prerequisite/classification:** types/loops; Core।

**8. Code Idea / Hint:** Trim/Contains/StringComparison/masking ও receipt formatting tasks। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** raw text → trim/validate → safe text; receipt fields → builder → formatted string।

**10. Industry Usage:** parsing/display/reports। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** builder সব ছোট concat-এ faster দাবি নয়।

**12. Alternatives:** interpolation few lines।

**13. My Task:** Trim/Contains/StringComparison/masking ও receipt formatting tasks। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** 12.50 display; whitespace reject; short mobile safe; no secret output। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="f-exception-handling"></a>

### Exception Handling (General)

**1. Topic Name:** Exception Handling (General)।

**2. Definition:** try/catch/finally exceptional flow; specific catches আগে; throw rethrow preserves stack।

**3. Purpose:** I/O/error propagation-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3/P5; F-39/24; boundary/custom exception lab।

**4. Mental Model:** normal invalid input result, emergency exception।

**5. Syntax Explained:** try protected block, catch selected handler, finally cleanup; throw; rethrows।

**6. Small Example (isolated, project implementation নয়):**

```csharp
try { int.Parse("bad"); }
catch (FormatException) { Console.WriteLine("Invalid"); }
finally { Console.WriteLine("Cleanup"); }
```

**7. Project Context:** P3/P5; F-39/24; boundary/custom exception lab। **Prerequisite/classification:** methods/control flow; Core।

**8. Code Idea / Hint:** recoverable/fatal policy; custom exception ছোট lab; throw vs throw ex। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** exceptional operation → thrown exception → selected handler → cleanup → safe boundary result।

**10. Industry Usage:** I/O/error propagation। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** finally fatal termination guarantee নয়; no empty catch/blanket continue।

**12. Alternatives:** result for expected failures।

**13. My Task:** recoverable/fatal policy; custom exception ছোট lab; throw vs throw ex। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** bad→catch+finally ordinary execution; precommit fault no effect। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S6: Exception handling](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements)।

[↑ Master Index](#section-24)

---

<a id="f-generics"></a>

### Generics

**1. Topic Name:** Generics।

**2. Definition:** type parameter typed reuse; caller concrete type supplies।

**3. Purpose:** collections/results/libraries-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3/P6; F-42/39; OperationResult/GenericRepositoryLab।

**4. Mental Model:** এক template, compiler-controlled contents।

**5. Syntax Explained:** &lt;T&gt; placeholder; method call type inference; where constraints next runtime topic।

**6. Small Example (isolated, project implementation নয়):**

```csharp
static T Echo<T>(T value) => value;
int number = Echo(7);
```

**7. Project Context:** P3/P6; F-42/39; OperationResult/GenericRepositoryLab। **Prerequisite/classification:** methods/class; Core।

**8. Code Idea / Hint:** generic result/constraint lab; audit/user same CRUD কেন নয় explain। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** concrete type argument → typed generic member → compile-checked value/result।

**10. Industry Usage:** collections/results/libraries। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** no generic base pattern-count; nullable T deliberate।

**12. Alternatives:** concrete type without variation।

**13. My Task:** generic result/constraint lab; audit/user same CRUD কেন নয় explain। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** Echo7=7; incompatible assignment compile failure। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S17: Constraints](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)।

[↑ Master Index](#section-24)

---

<a id="f-delegates"></a>

### Delegates

**1. Topic Name:** Delegates।

**2. Definition:** type-safe callable reference; Func returns, Action void; custom delegate signature।

**3. Purpose:** callbacks/predicates-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P7; F-22; query/FoundationLab।

**4. Mental Model:** method-এর কাজের ঠিকানা।

**5. Syntax Explained:** Func&lt;input,return&gt;; Action&lt;input&gt; void; delegate variable invoke with ().

**6. Small Example (isolated, project implementation নয়):**

```csharp
Func<int, bool> isEven = n => n % 2 == 0;
bool answer = isEven(4);
```

**7. Project Context:** P7; F-22; query/FoundationLab। **Prerequisite/classification:** methods/generics; Supporting।

**8. Code Idea / Hint:** named/custom/Func compare; predicate parameter helper নিজে। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** compatible method/lambda → delegate value → caller invokes → predicate/result।

**10. Industry Usage:** callbacks/predicates। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** callback wallet secretly mutate নয়।

**12. Alternatives:** interface for stateful strategy।

**13. My Task:** named/custom/Func compare; predicate parameter helper নিজে। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** 4 true;3 false; multicast return last result discussion। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S23: Delegates/lambdas/events](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/delegates-lambdas)।

[↑ Master Index](#section-24)

---

<a id="f-events"></a>

### Events

**1. Topic Name:** Events।

**2. Definition:** publisher-controlled delegate notification; outsiders subscribe/unsubscribe, publisher raises।

**3. Purpose:** UI/optional observers-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P7; optional notice; ConsoleNotificationSubscriber।

**4. Mental Model:** ঘণ্টা শুনে listeners; commit owner নয়।

**5. Syntax Explained:** event restricts external invocation; += subscribe, -= unsubscribe, ?.Invoke null-safe।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Inside a lab publisher:
public event EventHandler? Changed;
private void RaiseChanged() => Changed?.Invoke(this, EventArgs.Empty);
```

**7. Project Context:** P7; optional notice; ConsoleNotificationSubscriber। **Prerequisite/classification:** delegates; Supporting।

**8. Code Idea / Hint:** two subscribers/unsubscribe/throwing handler; required audit event-only নয়। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** committed fact → publisher raises → subscribed handlers → optional notices; failure isolated।

**10. Industry Usage:** UI/optional observers। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** ?.Invoke only null safety; handler exception still propagates; lifetime leaks।

**12. Alternatives:** callback।

**13. My Task:** two subscribers/unsubscribe/throwing handler; required audit event-only নয়। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** zero subscribers safe; unsubscribed not called; committed Completed retained after notice failure। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S23: Delegates/lambdas/events](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/delegates-lambdas)।

[↑ Master Index](#section-24)

---

<a id="f-lambda"></a>

### Lambda Expressions

**1. Topic Name:** Lambda Expressions।

**2. Definition:** anonymous function compatible delegate/expression tree; closure captures variables।

**3. Purpose:** LINQ/callbacks-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P7; F-22/30; query/AdminService।

**4. Mental Model:** inline rule, captured threshold সময়ের সঙ্গে বদলায়।

**5. Syntax Explained:** parameters =&gt; body; expression returns value, block uses explicit return where needed।

**6. Small Example (isolated, project implementation নয়):**

```csharp
Func<int, int> twice = value => value * 2;
Console.WriteLine(twice(3));
```

**7. Project Context:** P7; F-22/30; query/AdminService। **Prerequisite/classification:** delegates/methods; Core।

**8. Code Idea / Hint:** lambda→named method; closure threshold mutate then predict। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** arguments/captured variables → anonymous body → delegate result → query decision।

**10. Industry Usage:** LINQ/callbacks। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** long rules lambda-এ নয়; loop capture।

**12. Alternatives:** named method।

**13. My Task:** lambda→named method; closure threshold mutate then predict। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** twice3=6; closure sees changed variable at invocation। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S23: Delegates/lambdas/events](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/delegates-lambdas)।

[↑ Master Index](#section-24)

---

<a id="f-linq"></a>

### LINQ

**1. Topic Name:** LINQ।

**2. Definition:** typed query operators; Where deferred, Sum/Count/ToList immediate; SQL নয়।

**3. Purpose:** reports/projections-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P5 basic/P7–8 deep; F-17/22/34; services।

**4. Mental Model:** filter→project→sort→aggregate pipeline।

**5. Syntax Explained:** Where predicate, Select projection, OrderBy key, Sum aggregate; query syntax alternative।

**6. Small Example (isolated, project implementation নয়):**

```csharp
var numbers = new[] { 1, 2, 3, 4 };
int total = numbers.Where(n => n % 2 == 0).Sum();
```

**7. Project Context:** P5 basic/P7–8 deep; F-17/22/34; services। **Prerequisite/classification:** collections/predicates; Core।

**8. Code Idea / Hint:** Where/Select/OrderBy/ThenBy/Skip/Take/GroupBy/Sum; Completed principal UTC-range usage। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** safe source → filter/project/sort → enumerate/materialize/aggregate → safe DTO।

**10. Industry Usage:** reports/projections। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** server DateTime.Today নয়; failed not revenue; repeated enumeration।

**12. Alternatives:** foreach।

**13. My Task:** Where/Select/OrderBy/ThenBy/Skip/Take/GroupBy/Sum; Completed principal UTC-range usage। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** sum6; empty Sum0; Average empty handled; stable paging। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S15: LINQ](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/statements/linq)।

[↑ Master Index](#section-24)

---

<a id="f-async"></a>

### async / await

**1. Topic Name:** async / await।

**2. Definition:** await suspends continuation until completion without blocking wait; new thread automatic নয়।

**3. Purpose:** I/O orchestration-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P9/P11; F-16/43; fake gateway/AsyncLab।

**4. Mental Model:** waiting≠parallel workers; চুলা চলাকালে অন্য কাজ সম্ভব।

**5. Syntax Explained:** async marks method transformation; Task&lt;T&gt; eventual T; await unwraps result/failure।

**6. Small Example (isolated, project implementation নয়):**

```csharp
static async Task<int> ReadLaterAsync()
{ await Task.Delay(10); return 7; }
```

**7. Project Context:** P9/P11; F-16/43; fake gateway/AsyncLab। **Prerequisite/classification:** methods/Task/errors; Supporting।

**8. Code Idea / Hint:** await success/fail/cancel; commit after final revalidation। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** call → synchronous prefix → incomplete await → continuation → result/fault/cancel।

**10. Industry Usage:** I/O orchestration। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** no async void except required event signature; no Result/Wait habit; no auto-responsive menu claim।

**12. Alternatives:** sync pure calculation।

**13. My Task:** await success/fail/cancel; commit after final revalidation। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** result7; awaited fault observed; precommit cancel no movement। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S3: Asynchronous programming](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)।

[↑ Master Index](#section-24)

---

<a id="f-task-thread"></a>

### Task & Thread Basics

**1. Topic Name:** Task & Thread Basics।

**2. Definition:** Task eventual completion/result; may have no dedicated thread. Thread execution path; Task.Run typically pool CPU scheduling।

**3. Purpose:** CPU/I/O/concurrency literacy-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P9; AsyncLab/ThreadRaceLab; F-16 support।

**4. Mental Model:** ticket versus worker; waiting ticket worker continuously busy নয়।

**5. Syntax Explained:** Task.Run delegate schedules CPU work; await observes; Thread.Start/Join lab comparison।

**6. Small Example (isolated, project implementation নয়):**

```csharp
Task<int> work = Task.Run(() => 2 + 3);
int result = await work;
```

**7. Project Context:** P9; AsyncLab/ThreadRaceLab; F-16 support। **Prerequisite/classification:** delegates/methods/async intro; Supporting।

**8. Code Idea / Hint:** Delay vs Run vs Thread; counter race then lock/Interlocked, no wallet parallel writes। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** CPU delegate → Task.Run scheduler → worker execution → Task completion → await result।

**10. Industry Usage:** CPU/I/O/concurrency literacy। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** Task not always pool; concurrency≠parallel; no lock across await।

**12. Alternatives:** single-thread wallet।

**13. My Task:** Delay vs Run vs Thread; counter race then lock/Interlocked, no wallet parallel writes। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** result5; race may lose increments, not every run; coordinated count exact। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S3: Asynchronous programming](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)।

[↑ Master Index](#section-24)

---

<a id="f-record-struct-enum"></a>

### record, struct, enum

**1. Topic Name:** record, struct, enum।

**2. Definition:** enum named values; struct value type; record equality helpers, can be mutable; record class reference/record struct value।

**3. Purpose:** DTO/value/state-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2/P7; F-04/19/22; enums/receipt/DateRange।

**4. Mental Model:** state label/small value/snapshot তিন tool।

**5. Syntax Explained:** enum named integral values; readonly record struct value; record default class।

**6. Small Example (isolated, project implementation নয়):**

```csharp
enum Mode { Basic, Advanced }
readonly record struct Pair(int X, int Y);
record Label(string Text);
```

**7. Project Context:** P2/P7; F-04/19/22; enums/receipt/DateRange। **Prerequisite/classification:** class/value-reference; Core।

**8. Code Idea / Hint:** three compare; nested-list record with alias; invalid/default struct test। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** validated data → enum/value/snapshot construction → equality/copy/display।

**10. Industry Usage:** DTO/value/state। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** record≠deep immutable; enum undefined casts; default struct bypass ctor।

**12. Alternatives:** class identity।

**13. My Task:** three compare; nested-list record with alias; invalid/default struct test। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** Pair copies; Label value equality; nested mutable with shared। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S2: Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)।

[↑ Master Index](#section-24)

---

<a id="f-nullable"></a>

### Nullable Reference Types

**1. Topic Name:** Nullable Reference Types।

**2. Definition:** compiler annotations/flow warnings; reference runtime unchanged; runtime null prevention guarantee নয়।

**3. Purpose:** absence contracts-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P0 enabled/P3 explained; F-07/18; session/repo।

**4. Mental Model:** absence label, magic protection নয়।

**5. Syntax Explained:** ? nullable annotation/value type; ?. null conditional; ?? fallback; ! suppresses warning only।

**6. Small Example (isolated, project implementation নয়):**

```csharp
string? label = null;
int length = label?.Length ?? 0;
```

**7. Project Context:** P0 enabled/P3 explained; F-07/18; session/repo। **Prerequisite/classification:** reference/conditions; Core।

**8. Code Idea / Hint:** lookup null handle; nullable value vs reference; no unjustified !। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** optional lookup/session → null-flow check → guarded access or safe absence result।

**10. Industry Usage:** absence contracts। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** Completed participants required; CashIn agent source not always null।

**12. Alternatives:** explicit result।

**13. My Task:** lookup null handle; nullable value vs reference; no unjustified !। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** null length0; missing safe result; nonnull length। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="f-dependency"></a>

### Dependency Concepts

**1. Topic Name:** Dependency Concepts।

**2. Definition:** object/project/package dependency আলাদা; constructor injection supplies dependency externally।

**3. Purpose:** composition/testability-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P0/P3/P10; Program/csproj/contracts।

**4. Mental Model:** Program wiring board, service hidden storage বানায় না।

**5. Syntax Explained:** constructor receives capability; readonly stores dependency; project reference/package not same concept।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Inside a small lab consumer:
private readonly Func<int> _read;
public Reader(Func<int> read) { _read = read; }
```

**7. Project Context:** P0/P3/P10; Program/csproj/contracts। **Prerequisite/classification:** class/methods; Core।

**8. Code Idea / Hint:** object+package/project graph; fake clock; wiring order। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** startup dependencies → constructor injection → consumer contract call → adapter/fake result।

**10. Industry Usage:** composition/testability। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** DI clock/hasher/gateway/logger too; container optional।

**12. Alternatives:** direct new stable pure helper।

**13. My Task:** object+package/project graph; fake clock; wiring order। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** deterministic fake; no cycle; null dependency rejected। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

## 24.B – OOP

<a id="o-class-object"></a>

### Class & Object

**1. Topic Name:** Class & Object।

**2. Definition:** class blueprint/type, object instance with identity/state; new invokes creation।

**3. Purpose:** domain entities/services-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2; F-01/08; User/Account।

**4. Mental Model:** ফর্ম template বনাম filled form।

**5. Syntax Explained:** class defines type; new constructs instance; members state/behavior।

**6. Small Example (isolated, project implementation নয়):**

```csharp
class Note { public string Text { get; init; } = ""; }
```

**7. Project Context:** P2; F-01/08; User/Account। **Prerequisite/classification:** methods/types; Core।

**8. Code Idea / Hint:** two independent objects, identity vs data compare। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** validated constructor inputs → new instance → state/behavior → safe projection।

**10. Industry Usage:** domain entities/services। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** model valid state; DTO/entity distinction।

**12. Alternatives:** record value data।

**13. My Task:** two independent objects, identity vs data compare। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** two new objects ReferenceEquals false; independent text। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-fields-properties"></a>

### Fields vs Properties

**1. Topic Name:** Fields vs Properties।

**2. Definition:** field storage; property accessors contract; auto-property backing storage compiler creates।

**3. Purpose:** encapsulated APIs-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2; F-12; Account।

**4. Mental Model:** ভেতরের shelf বনাম controlled window।

**5. Syntax Explained:** field direct storage; get/set/init accessors; =&gt; getter expression; private setter limits writes।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Inside a lab type:
private int _count;
public int Count => _count;
```

**7. Project Context:** P2; F-12; Account। **Prerequisite/classification:** class; Core।

**8. Code Idea / Hint:** private field/read-only property/auto-property compare। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** private storage → getter/validated method → controlled visible value।

**10. Industry Usage:** encapsulated APIs। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** getter no surprising mutation; private setter alone all invariants নয়।

**12. Alternatives:** immutable replacement।

**13. My Task:** private field/read-only property/auto-property compare। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** external assignment read-only compile fail; method-controlled change visible। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-access-modifiers"></a>

### Access Modifiers

**1. Topic Name:** Access Modifiers।

**2. Definition:** public accessible subject to containing type; private containing type; protected derived; internal same assembly, project shorthand নয়।

**3. Purpose:** library boundaries-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2/P6; Account/AccountInheritanceLab।

**4. Mental Model:** দরজার permission levels।

**5. Syntax Explained:** visibility before declaration; protected internal OR, private protected AND; file scope extra concept।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Members in a lab base type:
public int Visible { get; }
private int _hidden;
protected int Shared;
internal int Local;
```

**7. Project Context:** P2/P6; Account/AccountInheritanceLab। **Prerequisite/classification:** class/properties; Core+Supporting protected।

**8. Code Idea / Hint:** same class/derived/other assembly access matrix নিজে test। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** caller location → compiler accessibility check → allowed member or compile error।

**10. Industry Usage:** library boundaries। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** least exposed API; protected mutable balance invariant bypass নয়।

**12. Alternatives:** protected validated method।

**13. My Task:** same class/derived/other assembly access matrix নিজে test। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** private external denied; protected derived allowed; internal other assembly denied। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S21: Access modifiers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)।

[↑ Master Index](#section-24)

---

<a id="o-constructor"></a>

### Constructor

**1. Topic Name:** Constructor।

**2. Definition:** instance creation initialization; no return type; enforce required state; constructor method-এর মতো ordinary return নয়।

**3. Purpose:** entity initialization-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2; F-08; Account।

**4. Mental Model:** valid object-এর entry gate।

**5. Syntax Explained:** type name(parameters), no return type; constructor body validates/assigns; base chain before derived body।

**6. Small Example (isolated, project implementation নয়):**

```csharp
class Tag
{ public string Name { get; }
  public Tag(string name) { Name = name; } }
```

**7. Project Context:** P2; F-08; Account। **Prerequisite/classification:** class/properties; Core।

**8. Code Idea / Hint:** required identity/valid amount initial state guards নিজে design। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** creation arguments → initialization/constructor checks → valid instance or failure।

**10. Industry Usage:** entity initialization। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** no plaintext PIN model constructor; hash value passed after hasher।

**12. Alternatives:** validated factory।

**13. My Task:** required identity/valid amount initial state guards নিজে design। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** missing required args compile fail; invalid domain input reject; no partial store insert। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S22: Constructors](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors)।

[↑ Master Index](#section-24)

---

<a id="o-constructor-overload"></a>

### Constructor Overloading

**1. Topic Name:** Constructor Overloading।

**2. Definition:** same type multiple constructors different signatures; this chaining centralizes initialization।

**3. Purpose:** creation ergonomics-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2/P6; model lab/seed।

**4. Mental Model:** এক gate-এর কয়েক entry path, একই checks।

**5. Syntax Explained:** : this(...) delegates to another constructor; signatures differ; central validation once।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Inside a lab Tag type:
public Tag() : this("untitled") { }
public Tag(string name) { Name = name; }
```

**7. Project Context:** P2/P6; model lab/seed। **Prerequisite/classification:** constructor; Supporting।

**8. Code Idea / Hint:** default/custom creation chain; role default User cannot escalate via UI। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** selected signature → this chain → central initialization → instance।

**10. Industry Usage:** creation ergonomics। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** only meaningful alternatives; no constructor overload just topic।

**12. Alternatives:** named factory।

**13. My Task:** default/custom creation chain; role default User cannot escalate via UI। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** both valid; validation once; same invariant। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S22: Constructors](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors)।

[↑ Master Index](#section-24)

---

<a id="o-this"></a>

### this Keyword

**1. Topic Name:** this Keyword।

**2. Definition:** current instance; this(...) constructor chaining; static method has no instance this।

**3. Purpose:** instance initialization-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2; Account/model lab।

**4. Mental Model:** নিজের badge বনাম incoming parameter।

**5. Syntax Explained:** this.member current instance; this(...) initializer chaining; static has no this।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Inside a lab constructor:
this._name = name;
```

**7. Project Context:** P2; Account/model lab। **Prerequisite/classification:** constructor/fields; Core।

**8. Code Idea / Hint:** field/parameter same name resolution; chaining vs instance reference। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** incoming parameter → current instance member assignment → own state।

**10. Industry Usage:** instance initialization। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** extension this parameter different syntactic role।

**12. Alternatives:** _field naming avoids ambiguity।

**13. My Task:** field/parameter same name resolution; chaining vs instance reference। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** correct field set; static this compile fail in lab। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-static-members"></a>

### Static Members

**1. Topic Name:** Static Members।

**2. Definition:** type-level field/property/method, not per instance; mutable static shared across tests/lifetimes।

**3. Purpose:** constants/caches with lifecycle care-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P4/P6; SeedData/lab; F-21 supporting।

**4. Mental Model:** সব instance-এর shared noticeboard।

**5. Syntax Explained:** static member belongs to type; type.member access; shared mutable field lifetime।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Inside a lab type:
public static int CreatedCount { get; private set; }
```

**7. Project Context:** P4/P6; SeedData/lab; F-21 supporting। **Prerequisite/classification:** class; Supporting।

**8. Code Idea / Hint:** two instances vs static counter; reset/lifetime risk; actual ID uses injectable Guid। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** type lifetime → shared static member → all callers observe same type-level state।

**10. Industry Usage:** constants/caches with lifecycle care। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** single-thread no race মানে restart/collision/test pollution নেই নয়।

**12. Alternatives:** instance dependency।

**13. My Task:** two instances vs static counter; reset/lifetime risk; actual ID uses injectable Guid। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** counter shared; instance fields independent; no static wallet store। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-static-class"></a>

### Static Class / Static Methods

**1. Topic Name:** Static Class / Static Methods।

**2. Definition:** static class instantiate/inherit করা যায় না; stateless methods type দিয়ে call।

**3. Purpose:** utility/extension APIs-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1/P4; InputValidator/extensions।

**4. Mental Model:** utility desk, per-user state নয়।

**5. Syntax Explained:** static class only static members, no new; static method no instance receiver।

**6. Small Example (isolated, project implementation নয়):**

```csharp
static class TextTools
{ public static bool IsBlank(string? s) => string.IsNullOrWhiteSpace(s); }
```

**7. Project Context:** P1/P4; InputValidator/extensions। **Prerequisite/classification:** methods/class; Core।

**8. Code Idea / Hint:** pure static helper vs session manager compare। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** caller values → type-level pure helper → returned result, no session state।

**10. Industry Usage:** utility/extension APIs। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** no hidden global config/session।

**12. Alternatives:** instance when substitutable/stateful।

**13. My Task:** pure static helper vs session manager compare। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** null/space true; text false; new TextTools compile fail। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-readonly-const"></a>

### Readonly & Const

**1. Topic Name:** Readonly & Const।

**2. Definition:** const compile-time value; readonly field declaration/constructor assign; readonly reference nested object mutable হতে পারে।

**3. Purpose:** dependency references/constants-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3/P4; config/service fields।

**4. Mental Model:** fixed address বনাম fixed contents আলাদা।

**5. Syntax Explained:** const literal compile-time; readonly field constructor assignment; reference readonly not nested freeze।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Inside a lab type:
public const int DaysPerWeek = 7;
private readonly List<int> _values = new();
```

**7. Project Context:** P3/P4; config/service fields। **Prerequisite/classification:** fields/types; Core।

**8. Code Idea / Hint:** const/readonly config decision; readonly List Add experiment। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** compile-time literal or constructor value → fixed binding → read; nested mutable content still possible।

**10. Industry Usage:** dependency references/constants। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** fees mutable policy values const নয়; const version inlining caveat।

**12. Alternatives:** immutable config record।

**13. My Task:** const/readonly config decision; readonly List Add experiment। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** field reassignment later denied; list Add still allowed। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-object-initializer"></a>

### Object Initializers

**1. Topic Name:** Object Initializers।

**2. Definition:** constructor runs then accessible fields/set/init members assigned; only public set নয়; init works।

**3. Purpose:** settings/DTO/test fixtures-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P4; seed/config/test DTO।

**4. Mental Model:** object তৈরি তারপর allowed labels বসানো।

**5. Syntax Explained:** new Type(args) { AccessibleMember = value }; init works at initialization; constructor first।

**6. Small Example (isolated, project implementation নয়):**

```csharp
class Label { public string Text { get; init; } = ""; }
// Caller fragment: new Label { Text = "A" }
```

**7. Project Context:** P4; seed/config/test DTO। **Prerequisite/classification:** constructor/properties; Core।

**8. Code Idea / Hint:** constructor+initializer order; init/private-set access compare। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** constructor → accessible init/set assignments → completed object।

**10. Industry Usage:** settings/DTO/test fixtures। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** initializer strong invariants bypass নয়; required doesn't ensure nonnull runtime।

**12. Alternatives:** constructor for mandatory state।

**13. My Task:** constructor+initializer order; init/private-set access compare। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** init allowed at creation; private set outside denied; later init assignment denied। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S8: Object initializers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers); [S22: Constructors](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors)।

[↑ Master Index](#section-24)

---

<a id="o-encapsulation"></a>

### Encapsulation

**1. Topic Name:** Encapsulation।

**2. Definition:** state+behavior boundary, valid transitions through controlled API; private set alone insufficient if methods allow invalid changes।

**3. Purpose:** domain integrity-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2/P5; F-12/27; Account/store।

**4. Mental Model:** vault নয়, controlled service window।

**5. Syntax Explained:** private state + public validated operation; API controls invariant, not visibility alone।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Inside a lab counter:
public int Count { get; private set; }
public void Increment() => Count++;
```

**7. Project Context:** P2/P5; F-12/27; Account/store। **Prerequisite/classification:** access/properties; Core।

**8. Code Idea / Hint:** Account invariants and store no mutable leak নিজে enforce। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** requested transition → invariant checks → controlled candidate state → publication।

**10. Industry Usage:** domain integrity। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** protected public fields not invariant-safe।

**12. Alternatives:** immutable replacements।

**13. My Task:** Account invariants and store no mutable leak নিজে enforce। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** external setter denied; negative/overflow candidate rejected; DTO mutation no live effect। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-inheritance"></a>

### Inheritance

**1. Topic Name:** Inheritance।

**2. Definition:** derived type inherits eligible members; genuine is-a contract; base constructors not inherited।

**3. Purpose:** framework families-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; AccountInheritanceLab।

**4. Mental Model:** এক family-এর common contract, শুধু similar fields নয়।

**5. Syntax Explained:** Derived : Base; base(args) constructor chain; protected extension points deliberate।

**6. Small Example (isolated, project implementation নয়):**

```csharp
class Shape { }
class Circle : Shape { }
```

**7. Project Context:** P6; AccountInheritanceLab। **Prerequisite/classification:** class/access/constructor; Supporting।

**8. Code Idea / Hint:** base/derived constructor order; Personal/Agent lab; compare AccountType+policy। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** derived creation → base constructor body → derived body → inherited contract।

**10. Industry Usage:** framework families। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** main app default composition; inheritance only natural variation।

**12. Alternatives:** interface/policy।

**13. My Task:** base/derived constructor order; Personal/Agent lab; compare AccountType+policy। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** derived usable as base; invariants/preconditions preserved। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-polymorphism"></a>

### Polymorphism

**1. Topic Name:** Polymorphism।

**2. Definition:** same contract call selects compatible implementation; runtime dispatch via interface/virtual; overloading compile-time selection আলাদা।

**3. Purpose:** adapters/test doubles-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3/P6; clock/hasher/gateway/optional policy।

**4. Mental Model:** এক button, device অনুযায়ী কাজ।

**5. Syntax Explained:** contract reference assigned concrete implementation; interface/virtual call runtime dispatch।

**6. Small Example (isolated, project implementation নয়):**

```csharp
IEnumerable<int> values = new List<int> { 1, 2 };
```

**7. Project Context:** P3/P6; clock/hasher/gateway/optional policy। **Prerequisite/classification:** interface or inheritance; Core।

**8. Code Idea / Hint:** fake+actual adapter same contract; no concrete type branching required। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** contract reference → actual implementation dispatch → contract-compatible result।

**10. Industry Usage:** adapters/test doubles। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** Liskov contract; interface doesn't automatically guarantee semantics।

**12. Alternatives:** simple switch few cases।

**13. My Task:** fake+actual adapter same contract; no concrete type branching required। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** same consumer accepts both; behavior-specific fixture results। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-method-overload"></a>

### Method Overloading

**1. Topic Name:** Method Overloading।

**2. Definition:** same name different parameter signatures; return type alone not enough; compiler chooses based on call types।

**3. Purpose:** ergonomic libraries-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P4/P6; formatting/helper lab।

**4. Mental Model:** এক label-এর distinct input slots।

**5. Syntax Explained:** same name, distinct parameter signature; return type alone not signature distinction।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Inside a lab helper:
public int Size(string text) => text.Length;
public int Size(int[] values) => values.Length;
```

**7. Project Context:** P4/P6; formatting/helper lab। **Prerequisite/classification:** methods/types; Supporting।

**8. Code Idea / Hint:** two overloads; null ambiguity; optional parameter trade-off। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** call compile-time argument types → overload resolution → selected signature execution।

**10. Industry Usage:** ergonomic libraries। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** same semantic purpose, confusing overload নয়।

**12. Alternatives:** named methods।

**13. My Task:** two overloads; null ambiguity; optional parameter trade-off। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** string length/array length correct; return-only overload compile failure। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-method-override"></a>

### Method Overriding

**1. Topic Name:** Method Overriding।

**2. Definition:** derived replaces virtual/abstract inherited implementation; runtime type dispatch।

**3. Purpose:** framework extensibility-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; AccountInheritanceLab।

**4. Mental Model:** base promise, derived implementation।

**5. Syntax Explained:** override member must inherit virtual/abstract/override slot; same contract/covariant return rules।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Derived member fragment; base declares virtual Describe:
public override string Describe() => "Circle";
```

**7. Project Context:** P6; AccountInheritanceLab। **Prerequisite/classification:** inheritance/virtual; Supporting।

**8. Code Idea / Hint:** base reference derived instance call predict; ToString safe override। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** base virtual slot call → runtime derived override → result।

**10. Industry Usage:** framework extensibility। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** preserve contract, no secrets ToString।

**12. Alternatives:** interface composition।

**13. My Task:** base reference derived instance call predict; ToString safe override। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** override derived result; no override nonvirtual compile fail। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-virtual-override-new"></a>

### virtual, override, new

**1. Topic Name:** virtual, override, new।

**2. Definition:** virtual permits override; override dispatch chain; new hides member, call depends on reference compile-time type for hidden member।

**3. Purpose:** legacy/API dispatch literacy-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; AccountInheritanceLab।

**4. Mental Model:** same sign hidden vs genuine replace।

**5. Syntax Explained:** virtual opens slot, override replaces slot, new hides member; declared reference matters for hidden call।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// In separate lab derived types:
public override string Describe() => "Derived";
// Alternative hiding experiment:
public new string Describe() => "Hidden";
```

**7. Project Context:** P6; AccountInheritanceLab। **Prerequisite/classification:** inheritance/override; Supporting।

**8. Code Idea / Hint:** base/derived references compare for override vs hiding। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** declared reference/member selection → virtual dispatch or hidden separate member → result।

**10. Industry Usage:** legacy/API dispatch literacy। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** new not override; avoid hiding unless deliberate compatibility reason।

**12. Alternatives:** composition।

**13. My Task:** base/derived references compare for override vs hiding। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** base ref override→derived; base ref hidden→base; derived ref hidden→hidden। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-abstraction"></a>

### Abstraction

**1. Topic Name:** Abstraction।

**2. Definition:** essential contract exposes needed behavior, hides irrelevant implementation; design principle, interface একমাত্র tool নয়।

**3. Purpose:** module boundaries-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3/P6; IClock/IStateStore।

**4. Mental Model:** steering জানলেই drive, engine internals নয়।

**5. Syntax Explained:** narrow method/interface capability; consumer doesn't access implementation fields।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Lab caller depends on capability:
Func<DateTimeOffset> now = () => DateTimeOffset.UtcNow;
```

**7. Project Context:** P3/P6; IClock/IStateStore। **Prerequisite/classification:** class/methods; Core।

**8. Code Idea / Hint:** service কী জানবে/জানবে না লিখবে; no dictionary exposure। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** consumer request → narrow contract → hidden implementation → required result।

**10. Industry Usage:** module boundaries। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** not everything interface; useful seam only।

**12. Alternatives:** focused method/concrete stable helper।

**13. My Task:** service কী জানবে/জানবে না লিখবে; no dictionary exposure। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** fake time works without service change; contract same। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-interface"></a>

### Interface

**1. Topic Name:** Interface।

**2. Definition:** contract implementable by class/record/struct; modern interfaces default/static members থাকতে পারে, তাই কখনো implementation নেই বলা ভুল।

**3. Purpose:** DI/adapters-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3; repo/clock/hasher/gateway।

**4. Mental Model:** capability promise; consumers need small contract।

**5. Syntax Explained:** interface contract; implementation : Interface; implicit public or explicit interface member implementation।

**6. Small Example (isolated, project implementation নয়):**

```csharp
interface ITextSource { string Read(); }
```

**7. Project Context:** P3; repo/clock/hasher/gateway। **Prerequisite/classification:** abstraction; Core।

**8. Code Idea / Hint:** focused query/append contract; two fake implementations; service interface need justify। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** consumer contract → concrete/fake implementation → tested contract result।

**10. Industry Usage:** DI/adapters। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** no interface-per-class ritual; no forced generic CRUD।

**12. Alternatives:** abstract class if shared instance state needed।

**13. My Task:** focused query/append contract; two fake implementations; service interface need justify। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** missing member compile failure; substitutes pass semantic tests। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S11: Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface)।

[↑ Master Index](#section-24)

---

<a id="o-abstract-class"></a>

### Abstract Class

**1. Topic Name:** Abstract Class।

**2. Definition:** direct instantiate নয়; can hold state/constructors/concrete/virtual/abstract members; abstract member optional, abstract type can have none।

**3. Purpose:** genuine shared families-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; HandlerTemplateLab।

**4. Mental Model:** common skeleton with variable step।

**5. Syntax Explained:** abstract type no direct new; abstract member no body; concrete subclass implements; shared fields possible।

**6. Small Example (isolated, project implementation নয়):**

```csharp
abstract class Formatter
{ public abstract string Format(int value); }
```

**7. Project Context:** P6; HandlerTemplateLab। **Prerequisite/classification:** inheritance/abstraction; Supporting।

**8. Code Idea / Hint:** template skeleton small lab vs policy composition; not main duplicate engine। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** concrete subclass instance → common skeleton → abstract step implementation → result।

**10. Industry Usage:** genuine shared families। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** one class base, multiple interfaces; modern interface may share default code but no instance fields।

**12. Alternatives:** interface+composition।

**13. My Task:** template skeleton small lab vs policy composition; not main duplicate engine। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** new Formatter denied; concrete subclass implements abstract member। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S11: Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface)।

[↑ Master Index](#section-24)

---

<a id="o-association"></a>

### Association

**1. Topic Name:** Association।

**2. Definition:** independent objects related/used; reference or ID can express relation; no lifecycle ownership implied।

**3. Purpose:** domain relationships-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2/P5; User/Account/Transaction IDs।

**4. Mental Model:** দুই বন্ধু স্বাধীন, যোগাযোগ আছে।

**5. Syntax Explained:** no dedicated keyword; ID/reference relationship modeled with ownership rules।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Conceptual ID relation:
string relatedItemId = "item-1";
```

**7. Project Context:** P2/P5; User/Account/Transaction IDs। **Prerequisite/classification:** class; Core।

**8. Code Idea / Hint:** actor/source/destination association diagram; no mutable bidirectional graph। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** actor/account ID → relation lookup → independent related entity।

**10. Industry Usage:** domain relationships। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** DI itself composition proof নয়।

**12. Alternatives:** IDs vs object refs trade-off।

**13. My Task:** actor/source/destination association diagram; no mutable bidirectional graph। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** known IDs resolve; failed unknown target safe nullable, not dangling object। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-aggregation"></a>

### Aggregation

**1. Topic Name:** Aggregation।

**2. Definition:** whole groups independent parts; parts can outlive group; ownership/lifecycle semantic, syntax আলাদা নয়।

**3. Purpose:** catalogs/reports/grouping-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; report groups existing transactions; lab।

**4. Mental Model:** team ভাঙলেও player থাকে।

**5. Syntax Explained:** no dedicated keyword; independent part lifecycle plus grouping contract।

**6. Small Example (isolated, project implementation নয়):**

```csharp
var catalog = new List<string> { "item-1", "item-2" };
```

**7. Project Context:** P6; report groups existing transactions; lab। **Prerequisite/classification:** association/collections; Supporting।

**8. Code Idea / Hint:** report group remove করলে transactions remain; shared references discuss। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** existing independent parts → report/group → group discarded, original parts retained।

**10. Industry Usage:** catalogs/reports/grouping। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** any List automatically aggregation নয়; lifecycle explain।

**12. Alternatives:** projection snapshot if no sharing needed।

**13. My Task:** report group remove করলে transactions remain; shared references discuss। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** discard group leaves source history intact। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-composition"></a>

### Composition

**1. Topic Name:** Composition।

**2. Definition:** whole owns parts and their lifecycle; references alone/constructor DI alone যথেষ্ট নয়।

**3. Purpose:** owned value parts-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3/P5; User credential/Transaction postings।

**4. Mental Model:** document owns its lines; external service not owned part automatically।

**5. Syntax Explained:** no dedicated keyword; ownership+defensive construction enforce part lifetime/immutability।

**6. Small Example (isolated, project implementation নয়):**

```csharp
record Line(string Text);
record Document(IReadOnlyList<Line> Lines);
```

**7. Project Context:** P3/P5; User credential/Transaction postings। **Prerequisite/classification:** association/immutability; Core।

**8. Code Idea / Hint:** defensive owned postings; injected FeeCalculator association not forced composition। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** validated owned values → defensive construction → whole-owned immutable parts।

**10. Industry Usage:** owned value parts। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** record list shallow, copy ownership required।

**12. Alternatives:** independent referenced entities for aggregation।

**13. My Task:** defensive owned postings; injected FeeCalculator association not forced composition। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** external mutable list edits cannot change stored transaction; ownership tests। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-sealed-class"></a>

### Sealed Class

**1. Topic Name:** Sealed Class।

**2. Definition:** sealed prevents subclassing; intent restriction, universal performance guarantee নয়।

**3. Purpose:** controlled APIs-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; lab/adapter choice।

**4. Mental Model:** closed extension boundary।

**5. Syntax Explained:** sealed class blocks derivation; can implement interfaces।

**6. Small Example (isolated, project implementation নয়):**

```csharp
sealed class FixedLabel { }
```

**7. Project Context:** P6; lab/adapter choice। **Prerequisite/classification:** inheritance; Supporting।

**8. Code Idea / Hint:** attempt derived class in isolated compile-error lab; explain adapter sealing reason। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** attempted derived declaration → compiler rejects; direct/interface use still allowed।

**10. Industry Usage:** controlled APIs। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** not every class sealed blindly; optimize after measurement।

**12. Alternatives:** virtual documented extension point।

**13. My Task:** attempt derived class in isolated compile-error lab; explain adapter sealing reason। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** subclass compile denied; interface implementation still possible। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S24: Sealed](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed)।

[↑ Master Index](#section-24)

---

<a id="o-sealed-method"></a>

### Sealed Method

**1. Topic Name:** Sealed Method।

**2. Definition:** sealed override prevents further overriding inherited virtual method; not arbitrary normal method sealing।

**3. Purpose:** framework invariants-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; HandlerTemplateLab।

**4. Mental Model:** override chain এখানে বন্ধ।

**5. Syntax Explained:** sealed override closes inherited virtual slot; normal nonvirtual already non-overridable।

**6. Small Example (isolated, project implementation নয়):**

```csharp
// Intermediate derived member:
public sealed override string Describe() => "Fixed";
```

**7. Project Context:** P6; HandlerTemplateLab। **Prerequisite/classification:** override; Supporting।

**8. Code Idea / Hint:** base→middle→leaf chain; leaf override denied, other members can extend। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** base slot → intermediate sealed override → leaf inherits fixed slot।

**10. Industry Usage:** framework invariants। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** sealed override requires virtual/abstract ancestor; lab only if no natural app use।

**12. Alternatives:** sealed whole class।

**13. My Task:** base→middle→leaf chain; leaf override denied, other members can extend। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** leaf override compile fail; base ref gets middle implementation। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S24: Sealed](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed)।

[↑ Master Index](#section-24)

---

<a id="o-partial-class"></a>

### Partial Class

**1. Topic Name:** Partial Class।

**2. Definition:** same type split across source parts, compile merges; namespace/type identity must match।

**3. Purpose:** generated code/tooling-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; PartialConsoleLab।

**4. Mental Model:** এক বইয়ের chapters আলাদা files, বই এক।

**5. Syntax Explained:** same partial type declarations compile together; parts can access each other's private members।

**6. Small Example (isolated, project implementation নয়):**

```csharp
partial class LabView { public string Title => "Lab"; }
partial class LabView { public int Width => 40; }
```

**7. Project Context:** P6; PartialConsoleLab। **Prerequisite/classification:** class; Optional app/mandatory lab।

**8. Code Idea / Hint:** two files one type; compare separate InputReader/ConsoleUi responsibilities। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** same type parts → compiler merges → one type with combined members।

**10. Industry Usage:** generated code/tooling। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** partial doesn't fix god class; generated code common use।

**12. Alternatives:** focused separate classes।

**13. My Task:** two files one type; compare separate InputReader/ConsoleUi responsibilities। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** instance has both members; mismatch namespace separate type। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S25: Partial classes](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/partial-classes-and-methods)।

[↑ Master Index](#section-24)

---

<a id="o-nested-class"></a>

### Nested Class

**1. Topic Name:** Nested Class।

**2. Definition:** type inside type; scope/encapsulation tool, Builder বাধ্যতামূলক নয়।

**3. Purpose:** scoped helpers/builders-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P7; QueryBuilderLab।

**4. Mental Model:** parent context-এর ছোট helper।

**5. Syntax Explained:** class inside class; accessibility governs external use; parent type doesn't require parent instance for nested creation।

**6. Small Example (isolated, project implementation নয়):**

```csharp
class LabQuery { private class State { } }
```

**7. Project Context:** P7; QueryBuilderLab। **Prerequisite/classification:** class/access; Optional app/mandatory lab।

**8. Code Idea / Hint:** nested Builder vs simple query record; justify public/private access। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** parent scope declaration → accessibility check → scoped helper instance।

**10. Industry Usage:** scoped helpers/builders। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** not all helper nested; no forced fluent framework।

**12. Alternatives:** top-level focused type।

**13. My Task:** nested Builder vs simple query record; justify public/private access। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** private nested external inaccessible; query constraints still validated। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="o-object-methods"></a>

### ToString(), Equals(), GetHashCode()

**1. Topic Name:** ToString(), Equals(), GetHashCode()।

**2. Definition:** representation/equality/hash contract; equal values must same hash, same hash not necessarily equal; object default class identity।

**3. Purpose:** collections/value objects-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; CopyEqualityLab/DateRange।

**4. Mental Model:** label vs identity vs index bucket।

**5. Syntax Explained:** override ToString/Equals/GetHashCode; equal→same hash; record synthesizes value behavior।

**6. Small Example (isolated, project implementation নয়):**

```csharp
record Pair(int X, int Y);
// Compare two new Pair(1, 2) values in lab.
```

**7. Project Context:** P6; CopyEqualityLab/DateRange। **Prerequisite/classification:** class/value-reference; Core+Supporting custom overrides।

**8. Code Idea / Hint:** safe ToString; equality symmetry/transitivity; hash-set duplicate values। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** object → safe representation or semantic equality → hash bucket plus equality check।

**10. Industry Usage:** collections/value objects। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** hash not durable ID/security fingerprint; mutable keys dangerous।

**12. Alternatives:** record for value semantics, ID key for entity।

**13. My Task:** safe ToString; equality symmetry/transitivity; hash-set duplicate values। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** equal pairs same hash; different values may collide; no secret ToString। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S2: Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)।

[↑ Master Index](#section-24)

---

<a id="o-ref-vs-value"></a>

### Reference Type vs Value Type

**1. Topic Name:** Reference Type vs Value Type।

**2. Definition:** assignment copies variable value: value-type instance or reference to object; reference by-value parameter still reference copy।

**3. Purpose:** API semantics-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P2/P6; Account/decimal/DateRange/lab।

**4. Mental Model:** photo copy বনাম link copy।

**5. Syntax Explained:** assignment copies variable value; reference type object mutation differs from reference reassignment।

**6. Small Example (isolated, project implementation নয়):**

```csharp
int a = 10; int b = a; b = 20;
var first = new List<int>(); var second = first; second.Add(1);
```

**7. Project Context:** P2/P6; Account/decimal/DateRange/lab। **Prerequisite/classification:** types/class; Core।

**8. Code Idea / Hint:** value copy vs shared object; method reassign reference vs mutate object। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** assignment/parameter → instance fields or reference value copied → independent scalar/shared object effect।

**10. Industry Usage:** API semantics। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** struct reference fields still shared; memory placement separate question।

**12. Alternatives:** immutable values/defensive copy।

**13. My Task:** value copy vs shared object; method reassign reference vs mutate object। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** a10/b20; first count1; reassign parameter doesn't replace caller ref। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S7: Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)।

[↑ Master Index](#section-24)

---

<a id="o-copy"></a>

### Shallow Copy vs Deep Copy

**1. Topic Name:** Shallow Copy vs Deep Copy।

**2. Definition:** reference assignment no object clone; shallow clone new outer object shares nested refs; deep copy chosen owned mutable graph copies।

**3. Purpose:** snapshots/staging-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P5/P6; AppState/CopyEqualityLab।

**4. Mental Model:** new folder same linked files vs independent file copies।

**5. Syntax Explained:** assignment aliases; ToList new outer list; MemberwiseClone shallow; owned nested values copy deliberately।

**6. Small Example (isolated, project implementation নয়):**

```csharp
var source = new List<List<int>> { new() { 1 } };
var shallow = source.ToList();
shallow[0].Add(2);
```

**7. Project Context:** P5/P6; AppState/CopyEqualityLab। **Prerequisite/classification:** reference/value/collections; Core।

**8. Code Idea / Hint:** candidate isolation nested-list failure; immutable element sharing safe; deep owned copy task। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** live snapshot → independent outer/owned nested replacements → isolated candidate → publish।

**10. Industry Usage:** snapshots/staging। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** ToList only outer copy; cycles/shared identity make generic deep copy nontrivial।

**12. Alternatives:** immutable replacements।

**13. My Task:** candidate isolation nested-list failure; immutable element sharing safe; deep owned copy task। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** source inner count2 after shallow; independent deep candidate leaves source unchanged। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S7: Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)।

[↑ Master Index](#section-24)

---

<a id="o-immutability"></a>

### Immutability

**1. Topic Name:** Immutability।

**2. Definition:** state cannot change after construction; private set permits class mutation, init shallow; record may mutable; nested ownership matters।

**3. Purpose:** events/value data/cache safety-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3/P5/P7; credential/history/receipt।

**4. Mental Model:** historical photo not editable live view।

**5. Syntax Explained:** get-only/init/readonly help; with new outer record; nested references still shared unless owned immutable।

**6. Small Example (isolated, project implementation নয়):**

```csharp
record Label(string Text);
var first = new Label("A");
var second = first with { Text = "B" };
```

**7. Project Context:** P3/P5/P7; credential/history/receipt। **Prerequisite/classification:** encapsulation/copy; Core।

**8. Code Idea / Hint:** record with mutable list experiment; immutable receipt snapshot; no historic drift। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** construction → fixed owned state → safe read; with → new outer snapshot।

**10. Industry Usage:** events/value data/cache safety। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** immutable object helps reads, whole system not automatically thread-safe; private set not absolute immutable।

**12. Alternatives:** defensive snapshot।

**13. My Task:** record with mutable list experiment; immutable receipt snapshot; no historic drift। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** firstA/secondB; nested list may alias; history remains fixed after wallet updates। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S2: Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)।

[↑ Master Index](#section-24)

---

## 24.C – C# Language & Runtime

<a id="r-value-reference-deep"></a>

### Value Types vs Reference Types (Deep)

**1. Topic Name:** Value Types vs Reference Types (Deep)।

**2. Definition:** value assignment copies instance fields; reference fields inside struct still shared; reference assignment copies managed reference, not stable raw address।

**3. Purpose:** API/copy correctness-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; RuntimeLab/CopyEqualityLab; F-42/27।

**4. Mental Model:** photo copy with shared link inside photo; semantic copy not deep graph clone।

**5. Syntax Explained:** ref aliases variable slot; by-value class parameter copies reference; struct fields copied, refs still shared।

**6. Small Example (isolated, project implementation নয়):**

```csharp
var first = new List<int> { 1 };
var second = first;
second.Add(2);
```

**7. Project Context:** P6; RuntimeLab/CopyEqualityLab; F-42/27। **Prerequisite/classification:** o-ref-vs-value/copy; Supporting deep lesson।

**8. Code Idea / Hint:** four cases: value by value/ref, reference by value/ref; struct containing List alias test। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** argument value/reference slot → by-value copy or ref alias → mutation/reassignment effect।

**10. Industry Usage:** API/copy correctness। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** value type always deep copy ভুল; location separate।

**12. Alternatives:** immutable value fields only।

**13. My Task:** four cases: value by value/ref, reference by value/ref; struct containing List alias test। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** first count2; struct scalar independent but nested List shared; ref reassign affects caller। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S7: Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)।

[↑ Master Index](#section-24)

---

<a id="r-stack-heap"></a>

### Stack ও Heap + Misconceptions

**1. Topic Name:** Stack ও Heap + Misconceptions।

**2. Definition:** call frames/local storage vs managed object storage; type semantics does not guarantee placement; JIT/registers/closures/async state affect storage।

**3. Purpose:** runtime/performance literacy-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; RuntimeLab; DateRange design।

**4. Mental Model:** desk/work frame vs warehouse; analogy implementation guarantee নয়।

**5. Syntax Explained:** class field value lives inline in containing object; semantic type alone not placement rule।

**6. Small Example (isolated, project implementation নয়):**

```csharp
class Holder { public int Number; }
// Number is a value-type field within its containing object.
```

**7. Project Context:** P6; RuntimeLab; DateRange design। **Prerequisite/classification:** value/reference; Supporting।

**8. Code Idea / Hint:** class struct field/array element/boxed/local/closure placement discuss; don't infer from type alone। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** source constructs → runtime/JIT storage choices → managed references/roots → lifetime।

**10. Industry Usage:** runtime/performance literacy। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** GC doesn't prevent reachable-object retention; large structs copy expensive।

**12. Alternatives:** choose semantics first, profile later।

**13. My Task:** class struct field/array element/boxed/local/closure placement discuss; don't infer from type alone। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** value field can be within heap object; local may register; DateRange not chosen because always stack। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S9: GC fundamentals](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals); [S7: Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)।

[↑ Master Index](#section-24)

---

<a id="r-boxing"></a>

### Boxing ও Unboxing

**1. Topic Name:** Boxing ও Unboxing।

**2. Definition:** value→object/interface box copies value; unbox exact underlying value type; allocation costs subject to optimization।

**3. Purpose:** allocation-aware libraries-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; RuntimeLab; generic collection rationale।

**4. Mental Model:** value in wrapper, then extract exact kind।

**5. Syntax Explained:** object boxed=value; (ExactValueType)boxed unboxes; numeric conversion separate step।

**6. Small Example (isolated, project implementation নয়):**

```csharp
int number = 42;
object boxed = number;
int result = (int)boxed;
```

**7. Project Context:** P6; RuntimeLab; generic collection rationale। **Prerequisite/classification:** types/casts; Supporting।

**8. Code Idea / Hint:** List&lt;int&gt; vs List&lt;object&gt;; boxed int→long cast failure vs unbox then convert। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** value instance → boxed copied value → exact unbox → optional numeric conversion।

**10. Industry Usage:** allocation-aware libraries। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** generics reduce boxing, don't eliminate all; object args may box।

**12. Alternatives:** typed generic APIs।

**13. My Task:** List&lt;int&gt; vs List&lt;object&gt;; boxed int→long cast failure vs unbox then convert। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** result42; (long)boxed InvalidCastException; (long)(int)boxed42। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S13: Boxing/unboxing](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing)।

[↑ Master Index](#section-24)

---

<a id="r-type-conversion"></a>

### Type Conversion

**1. Topic Name:** Type Conversion।

**2. Definition:** implicit/explicit numeric/reference conversion; implicit can lose precision (long→double), so always no loss বলা ভুল।

**3. Purpose:** input/interoperability-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1/P6; InputReader/RuntimeLab।

**4. Mental Model:** wide container doesn't mean exact representation।

**5. Syntax Explained:** (T) explicit cast; Convert behavior differs; TryParse returns bool/out; checked integral overflow।

**6. Small Example (isolated, project implementation নয়):**

```csharp
double value = 9.99;
int truncated = (int)value;
bool ok = decimal.TryParse("123.45", out decimal amount);
```

**7. Project Context:** P1/P6; InputReader/RuntimeLab। **Prerequisite/classification:** types/operators; Core+Supporting।

**8. Code Idea / Hint:** cast vs round; checked integral overflow; explicit culture TryParse। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** source value/text → implicit/cast/parse → target or safe failure।

**10. Industry Usage:** input/interoperability। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** TryParse input safe but NumberStyles/culture must explicit; decimal-double mixing avoided।

**12. Alternatives:** validated typed input।

**13. My Task:** cast vs round; checked integral overflow; explicit culture TryParse। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** truncated9; parse123.45 with defined culture; invalid false; large long double precision example। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S12: Numeric conversions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/numeric-conversions)।

[↑ Master Index](#section-24)

---

<a id="r-pattern-matching"></a>

### Pattern Matching

**1. Topic Name:** Pattern Matching।

**2. Definition:** test type/constant/property/relational shape and bind variables; patterns no need forced inheritance।

**3. Purpose:** validation/state branching-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P5/P7; validator/status/query।

**4. Mental Model:** data shape অনুযায়ী gate।

**5. Syntax Explained:** is Type variable binds; relational/property patterns; switch when additional guard।

**6. Small Example (isolated, project implementation নয়):**

```csharp
object item = 42;
if (item is int number && number > 10)
    Console.WriteLine(number);
```

**7. Project Context:** P5/P7; validator/status/query। **Prerequisite/classification:** conditions/types; Core।

**8. Code Idea / Hint:** enum switch/relational amount patterns; null/exhaustive default। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** value shape/type → matching pattern → bound variable/branch।

**10. Industry Usage:** validation/state branching। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** pattern concise but business rules still clear; no rich-agent real classification।

**12. Alternatives:** if/switch।

**13. My Task:** enum switch/relational amount patterns; null/exhaustive default। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** 42 printed; null no match; unknown enum handled/rejected। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S10: Type testing/casts](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast)।

[↑ Master Index](#section-24)

---

<a id="r-extension-methods"></a>

### Extension Methods

**1. Topic Name:** Extension Methods।

**2. Definition:** static helper called instance-style; classic syntax first parameter this; doesn't modify original type or gain private access।

**3. Purpose:** formatting/fluent helpers-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P4/P7; MoneyExtensions/StringExtensions।

**4. Mental Model:** বাইরের helper, দেখতে built-in call।

**5. Syntax Explained:** non-nested static class, static method, first this T receiver in classic syntax।

**6. Small Example (isolated, project implementation নয়):**

```csharp
static class TextTools
{ public static bool IsBlank(this string? s) => string.IsNullOrWhiteSpace(s); }
```

**7. Project Context:** P4/P7; MoneyExtensions/StringExtensions। **Prerequisite/classification:** static/methods/this; Supporting।

**8. Code Idea / Hint:** mask/format helper; null/invalid-length policy; static call equivalent। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** receiver value → static extension parameter → helper result; original type unchanged।

**10. Industry Usage:** formatting/fluent helpers। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** business fee/auth logic hidden extension নয়; null receiver possible।

**12. Alternatives:** utility method।

**13. My Task:** mask/format helper; null/invalid-length policy; static call equivalent। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** blank true; normal false; mobile short safe; original string unchanged। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="r-optional-named-params"></a>

### Optional ও Named Parameters

**1. Topic Name:** Optional ও Named Parameters।

**2. Definition:** optional default argument; named identifies parameter; compiler/call-site default versioning matters।

**3. Purpose:** readable API calls-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; RuntimeLab/query/helper।

**4. Mental Model:** labelled slots avoid many indistinguishable booleans।

**5. Syntax Explained:** parameter = default; name: argument; required before optional; named avoids bool ambiguity।

**6. Small Example (isolated, project implementation নয়):**

```csharp
static string Describe(string text, bool upper = false)
    => upper ? text.ToUpperInvariant() : text;
// Caller: Describe(text: "a", upper: true)
```

**7. Project Context:** P6; RuntimeLab/query/helper। **Prerequisite/classification:** methods/overloading; Supporting।

**8. Code Idea / Hint:** named call order/default change implications; avoid custom fee rate bypass। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** call arguments/defaults → named/positional binding → method body।

**10. Industry Usage:** readable API calls। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** defaults may embedded at call site; too many options use query object।

**12. Alternatives:** overload/options record।

**13. My Task:** named call order/default change implications; avoid custom fee rate bypass। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** default a; upper A; named labels clear; invalid names compile fail। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="r-params-ref-out-in"></a>

### params, ref, out, in

**1. Topic Name:** params, ref, out, in।

**2. Definition:** params variable arguments; ref caller initialized read/write alias; out callee assigns; in readonly reference, not deep immutable object।

**3. Purpose:** parsing/performance-aware APIs-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P1 out/P6 deeper; InputReader/RuntimeLab।

**4. Mental Model:** many tickets / shared slot / output slot / read-only slot।

**5. Syntax Explained:** params last parameter; ref initialized, out callee assigned, in readonly reference; modifiers at call as required।

**6. Small Example (isolated, project implementation নয়):**

```csharp
static int Sum(params int[] values) => values.Sum();
int.TryParse("12", out int parsed);
// Lab signatures: void Change(ref int x); int Read(in Pair value);
```

**7. Project Context:** P1 out/P6 deeper; InputReader/RuntimeLab। **Prerequisite/classification:** methods/value-reference; Supporting।

**8. Code Idea / Hint:** four separate labs; ref caller mutation; out assignment; in nested refs caveat। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** caller values/slots → array or alias/output/read-only parameter → result/caller effect।

**10. Industry Usage:** parsing/performance-aware APIs। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** in not automatic faster; params object boxes; no ref live wallet balance bypass।

**12. Alternatives:** return value/result record।

**13. My Task:** four separate labs; ref caller mutation; out assignment; in nested refs caveat। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** Sum1,2=3; parsed12; unassigned out compile failure; ref changes caller। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

[↑ Master Index](#section-24)

---

<a id="r-ienumerable-icollection"></a>

### IEnumerable&lt;T&gt; vs ICollection&lt;T&gt;

**1. Topic Name:** IEnumerable&lt;T&gt; vs ICollection&lt;T&gt;।

**2. Definition:** enumeration capability vs Count/Add/Remove/Contains/IsReadOnly; IEnumerable not inherently single-use/immutable; ICollection may read-only।

**3. Purpose:** collection API design-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P3/P7; repository safe results।

**4. Mental Model:** playlist playback capability vs collection operations, underlying object can vary।

**5. Syntax Explained:** IEnumerable GetEnumerator; ICollection Count/IsReadOnly/mutation methods; concrete capability may reject mutation।

**6. Small Example (isolated, project implementation নয়):**

```csharp
IEnumerable<int> sequence = new List<int> { 1, 2 };
ICollection<int> collection = new List<int> { 1, 2 };
int count = collection.Count;
```

**7. Project Context:** P3/P7; repository safe results। **Prerequisite/classification:** collections/interfaces; Core।

**8. Code Idea / Hint:** same object interface views; cast-back mutation demo; defensive snapshot and immutable elements। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** concrete source → chosen interface capability → enumeration/count/mutation or rejection।

**10. Industry Usage:** collection API design। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** read-only interface not security/deep immutable boundary।

**12. Alternatives:** IReadOnlyList snapshot with owned immutable elements।

**13. My Task:** same object interface views; cast-back mutation demo; defensive snapshot and immutable elements। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** count2; read-only collection Add may throw; IEnumerable can enumerate repeatedly if source supports। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S4: IEnumerable&lt;T&gt;](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1?view=net-10.0); [S5: ICollection&lt;T&gt;](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.icollection-1?view=net-10.0)।

[↑ Master Index](#section-24)

---

<a id="r-yield"></a>

### Iterators ও yield

**1. Topic Name:** Iterators ও yield।

**2. Definition:** yield return produces next item, state machine resumes; yield break ends; body generally starts enumeration not call।

**3. Purpose:** streaming/pipelines-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P7; history paging lab; FoundationLab।

**4. Mental Model:** one dish per request, not full batch upfront।

**5. Syntax Explained:** IEnumerable&lt;T&gt; method yield return next; yield break end; foreach drives MoveNext/disposal।

**6. Small Example (isolated, project implementation নয়):**

```csharp
static IEnumerable<int> Two()
{ yield return 1; yield return 2; }
```

**7. Project Context:** P7; history paging lab; FoundationLab। **Prerequisite/classification:** loops/IEnumerable; Supporting।

**8. Code Idea / Hint:** iterator execution timing, early stop/disposal; page over snapshot। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** consumer MoveNext → iterator resumes → next value → pause/end/dispose।

**10. Industry Usage:** streaming/pipelines। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** stored in-memory history already occupies memory; yield doesn't remove it; sorted query may buffer।

**12. Alternatives:** Skip/Take materialized snapshot।

**13. My Task:** iterator execution timing, early stop/disposal; page over snapshot। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** Two sequence1,2; no body execution until enumerate; Take1 stops early। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S16: Iterators/yield](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/yield)।

[↑ Master Index](#section-24)

---

<a id="r-deferred-execution"></a>

### Deferred Execution

**1. Topic Name:** Deferred Execution।

**2. Definition:** query description vs evaluation; Where/Select deferred, aggregates/materializers immediate; deferred may streaming or buffering।

**3. Purpose:** query composition/reports-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P7/P8; query/ReportService।

**4. Mental Model:** recipe লেখা রান্না নয়; OrderBy needs all ingredients before first output।

**5. Syntax Explained:** Where/Select builds pipeline; foreach/ToList evaluates; OrderBy buffers before first output।

**6. Small Example (isolated, project implementation নয়):**

```csharp
var values = new List<int> { 1, 2 };
var query = values.Where(n => n > 1);
values.Add(3);
var result = query.ToList();
```

**7. Project Context:** P7/P8; query/ReportService। **Prerequisite/classification:** LINQ/iterators; Core।

**8. Code Idea / Hint:** enumerate twice/source change vs snapshot; ToList outer copy not deep। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** query description → source/captured state at enumeration → streamed/buffered evaluation → result।

**10. Industry Usage:** query composition/reports। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** not all LINQ deferred; OrderBy deferred but buffers; no IQueryable DB needed।

**12. Alternatives:** explicit materialize owned snapshot।

**13. My Task:** enumerate twice/source change vs snapshot; ToList outer copy not deep। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** result2,3; immediate snapshot before add excludes3; repeated query can change। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S15: LINQ](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/statements/linq)।

[↑ Master Index](#section-24)

---

<a id="r-equality-hashing"></a>

### Equality ও Hashing

**1. Topic Name:** Equality ও Hashing।

**2. Definition:** ReferenceEquals identity; Equals semantic equality; equal→same hash, hash collision allowed; comparer choice controls dictionary behavior।

**3. Purpose:** sets/caches/idempotency-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6/P9; CopyEqualityLab/retry।

**4. Mental Model:** same person vs same form data vs index bucket।

**5. Syntax Explained:** Equals semantics+GetHashCode contract; StringComparer config; no hash-only equality check।

**6. Small Example (isolated, project implementation নয়):**

```csharp
var ids = new HashSet<string>(StringComparer.Ordinal);
bool first = ids.Add("A");
bool second = ids.Add("A");
```

**7. Project Context:** P6/P9; CopyEqualityLab/retry। **Prerequisite/classification:** object methods/collections; Core।

**8. Code Idea / Hint:** reflexive/symmetric/transitive equality; immutable keys; record nested collection equality caveat। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** key → hash bucket → comparer equality → found/new membership।

**10. Industry Usage:** sets/caches/idempotency। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** GetHashCode unstable across runs, not transaction ID/secure fingerprint; record List uses list equality not element deep equality automatically।

**12. Alternatives:** ID keys/canonical payload comparison।

**13. My Task:** reflexive/symmetric/transitive equality; immutable keys; record nested collection equality caveat। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** first true/second false; equal values same hash; collision not equality proof। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S2: Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)।

[↑ Master Index](#section-24)

---

<a id="r-idisposable"></a>

### IDisposable ও using

**1. Topic Name:** IDisposable ও using।

**2. Definition:** deterministic cleanup by resource owner; using disposes on ordinary scope exit; GC doesn't call Dispose automatically।

**3. Purpose:** file/network resource lifetimes-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6 lab/P11 optional; FileAppLogger/JSON।

**4. Mental Model:** borrowed tool ফেরত owner দেয়, arbitrary borrower নয়।

**5. Syntax Explained:** using declaration disposes at containing scope end; using block narrower; await using async cleanup।

**6. Small Example (isolated, project implementation নয়):**

```csharp
using var stream = new MemoryStream();
stream.WriteByte(1);
```

**7. Project Context:** P6 lab/P11 optional; FileAppLogger/JSON। **Prerequisite/classification:** exceptions/resources; Supporting।

**8. Code Idea / Hint:** disposal on exception; borrowed vs owned stream; await using concept। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** owner acquires → uses resource → scope exit/exception → Dispose; borrowed owner unchanged।

**10. Industry Usage:** file/network resource lifetimes। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** not every injected IDisposable wrapped using per call; lifecycle owner handles।

**12. Alternatives:** try/finally/owning container।

**13. My Task:** disposal on exception; borrowed vs owned stream; await using concept। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** owned stream disposed at scope end; borrowed shared logger not prematurely disposed। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S18: using/disposal](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using)।

[↑ Master Index](#section-24)

---

<a id="r-gc"></a>

### Garbage Collection

**1. Topic Name:** Garbage Collection।

**2. Definition:** managed memory reclaimed when unreachable from roots; cycles alone not leak, static/event references can retain; Gen0/1/2 concept।

**3. Purpose:** memory/performance diagnostics-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6/P10; RuntimeLab/history memory।

**4. Mental Model:** reachable map, not reference-count-only cleaner।

**5. Syntax Explained:** roots→reachable graph; generations0/1/2; no forced GC for normal business flow।

**6. Small Example (isolated, project implementation নয়):**

```csharp
var temporary = new byte[128];
// Drop ownership when no longer needed; no forced GC in app.
```

**7. Project Context:** P6/P10; RuntimeLab/history memory। **Prerequisite/classification:** reference/lifecycle; Supporting।

**8. Code Idea / Hint:** draw roots/static/event retention; managed vs resource cleanup; no forced GC habit। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** roots → reachable graph → unreachable managed objects eligible → runtime collection।

**10. Industry Usage:** memory/performance diagnostics। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** GC not memory-growth cure; history intentionally reachable; unmanaged cleanup Dispose।

**12. Alternatives:** bounded retention/profiling।

**13. My Task:** draw roots/static/event retention; managed vs resource cleanup; no forced GC habit। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** unreachable eligible not immediate guarantee; retained static remains; exact collection timing not asserted। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S9: GC fundamentals](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals); [S7: Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)।

[↑ Master Index](#section-24)

---

<a id="r-is-as-typeof-nameof"></a>

### is, as, typeof, nameof

**1. Topic Name:** is, as, typeof, nameof।

**2. Definition:** is type/pattern; as compatible reference or nullable value conversion else null; typeof Type metadata; nameof compile-time symbol name।

**3. Purpose:** validation/metadata/errors-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; RuntimeLab/validator।

**4. Mental Model:** type check / safe cast / type card / name label।

**5. Syntax Explained:** is pattern bool/binding; as nullable/reference; typeof(T) metadata; nameof(symbol) compile-time text।

**6. Small Example (isolated, project implementation নয়):**

```csharp
object value = 42;
int? number = value as int?;
Type type = typeof(int);
string name = nameof(value);
```

**7. Project Context:** P6; RuntimeLab/validator। **Prerequisite/classification:** types/patterns; Supporting।

**8. Code Idea / Hint:** as int? vs invalid as int; is binding; nameof rename refactor। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** object/symbol/type → test/cast/metadata/name operation → bool/value/Type/string।

**10. Industry Usage:** validation/metadata/errors। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** as not only reference types; nullable values too; nameof doesn't rename text automatically without refactor।

**12. Alternatives:** is pattern often clearer।

**13. My Task:** as int? vs invalid as int; is binding; nameof rename refactor। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** number42; incompatible null; typeof int; name value। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S10: Type testing/casts](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast)।

[↑ Master Index](#section-24)

---

<a id="r-generics-constraints"></a>

### Generics Constraints

**1. Topic Name:** Generics Constraints।

**2. Definition:** where expresses type capability compiler can rely on; class/struct/interface/base/new/notnull; constraints must valid combinations/order।

**3. Purpose:** type-safe reusable algorithms-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P6; GenericRepositoryLab।

**4. Mental Model:** generic machine accepts only compatible tools।

**5. Syntax Explained:** where T : class/interface/new(); new last; struct implies nonnullable value capability।

**6. Small Example (isolated, project implementation নয়):**

```csharp
static T Create<T>() where T : new() => new T();
```

**7. Project Context:** P6; GenericRepositoryLab। **Prerequisite/classification:** generics/interfaces; Supporting।

**8. Code Idea / Hint:** class/struct/interface/new constraints labs; no IEntity forced main app। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** supplied type → compiler capability check → allowed generic operations or compile error।

**10. Industry Usage:** type-safe reusable algorithms। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** new() last; struct nonnullable and implies constructor capability; class? distinct nullable intent।

**12. Alternatives:** supplied factory Func&lt;T&gt;।

**13. My Task:** class/struct/interface/new constraints labs; no IEntity forced main app। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** public parameterless type accepted; missing ctor denied; struct+new constraint invalid combination। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S17: Constraints](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)।

[↑ Master Index](#section-24)

---

<a id="r-exception-filters"></a>

### Exception Filters

**1. Topic Name:** Exception Filters।

**2. Definition:** catch when selects handler before unwinding; false filter leaves exception to another handler; predicate no side effects।

**3. Purpose:** selective infrastructure recovery-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P5/P11; boundary/lab।

**4. Mental Model:** শর্তযুক্ত emergency door।

**5. Syntax Explained:** catch(Type ex) when(predicate); false continues handler search; filter before unwind।

**6. Small Example (isolated, project implementation নয়):**

```csharp
try { throw new InvalidOperationException("lab"); }
catch (InvalidOperationException) when (true)
{ Console.WriteLine("Handled"); }
```

**7. Project Context:** P5/P11; boundary/lab। **Prerequisite/classification:** exception handling/conditions; Supporting।

**8. Code Idea / Hint:** specific code filter vs catch-inside-if; false filter propagation; no duplicate control-flow exceptions needed। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** thrown exception → handler type/filter check before unwind → selected handler/upstream।

**10. Industry Usage:** selective infrastructure recovery। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** filter not guarantee recovery; don't mutate state inside filter।

**12. Alternatives:** separate specific exception/result code।

**13. My Task:** specific code filter vs catch-inside-if; false filter propagation; no duplicate control-flow exceptions needed। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** true handled; false next catch/upstream; original stack retained। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S6: Exception handling](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements)।

[↑ Master Index](#section-24)

---

<a id="r-async-exception"></a>

### Async Exception Handling

**1. Topic Name:** Async Exception Handling।

**2. Definition:** async Task failures stored in task and observed at await; non-async task-returning method may throw synchronously; async void different caller semantics।

**3. Purpose:** async orchestration-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P9; fake gateway/AsyncLab; F-16।

**4. Mental Model:** ticket carries failure, await opens result envelope।

**5. Syntax Explained:** await inside try; faulted Task rethrows at await; cancellation OperationCanceledException separate।

**6. Small Example (isolated, project implementation নয়):**

```csharp
Task<int> task = Task.FromException<int>(new InvalidOperationException("lab"));
try { await task; }
catch (InvalidOperationException) { Console.WriteLine("Observed"); }
```

**7. Project Context:** P9; fake gateway/AsyncLab; F-16। **Prerequisite/classification:** async/errors; Supporting।

**8. Code Idea / Hint:** await fault/cancel; WhenAll failures inspect tasks; no fire-and-forget money operation। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** asynchronous failure → faulted Task → await rethrows → specific handler।

**10. Industry Usage:** async orchestration। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** no async void except required events; all task failures observed; synchronous invocation may need same try boundary।

**12. Alternatives:** sync pure service।

**13. My Task:** await fault/cancel; WhenAll failures inspect tasks; no fire-and-forget money operation। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** await catches fault; cancel separate; postcommit subscriber fault not Failed transfer। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S3: Asynchronous programming](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)।

[↑ Master Index](#section-24)

---

<a id="r-cancellation-token"></a>

### CancellationToken

**1. Topic Name:** CancellationToken।

**2. Definition:** cooperative stop request; source requests, listener observes; not force-kill/rollback; source disposable।

**3. Purpose:** request/timeouts/I/O-এর কাজে এই capability ব্যবহার বা সচেতনভাবে এড়িয়ে সঠিক design বেছে নেওয়া। Project-এ লক্ষ্য: P9/P11; AsyncLab/fake gateway/JSON।

**4. Mental Model:** stop signal, worker safely responds।

**5. Syntax Explained:** source.Token passed down; Cancel request; ThrowIfCancellationRequested/Delay observes; source using cleanup।

**6. Small Example (isolated, project implementation নয়):**

```csharp
using var source = new CancellationTokenSource();
source.Cancel();
await Task.Delay(100, source.Token);
```

**7. Project Context:** P9/P11; AsyncLab/fake gateway/JSON। **Prerequisite/classification:** Task/async/errors/disposal; Supporting।

**8. Code Idea / Hint:** catch OperationCanceledException; before/after commit semantics; timeout vs user cancellation distinguish। প্রথমে smallest outside example, তারপর listed feature/lab; main wallet state-এ experiment নয়।

**9. Data Flow:** source request → token propagated → listener observes → precommit safe cancellation or committed result।

**10. Industry Usage:** request/timeouts/I/O। Main app-এ natural use না থাকলে lab sufficient; pattern count লক্ষ্য নয়।

**11. Best Practices / Pitfalls:** cancel not undo real external side effect; source disposed; don't pass cancelled token to committed-result lookup blindly।

**12. Alternatives:** sync cancel/back before submission।

**13. My Task:** catch OperationCanceledException; before/after commit semantics; timeout vs user cancellation distinguish। নিজের variation ও learning-log explanation জমা দেবে; পূর্ণ solution চাইবে না।

**14. Testing / Expected Result:** pre-cancel Delay throws; before commit no movement; after commit Completed; token ignored operation may continue। Predict first; actual output/test evidence পরে সংরক্ষণ।

**15. Review:** definition নিজের ভাষায়, syntax line-by-line, expected result-এর কারণ, একটি failure/boundary, listed alternative কখন ভালো এবং project/lab decision explain করবে। Test evidence ছাড়া mastered নয়।

**Reference:** [S14: Cancellation](https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads)।

[↑ Master Index](#section-24)

---

## 24.D – Project Supporting Reference

এই helper concepts মূল 68-এর বাইরে; প্রথম lesson Section21-এর 15-field format-এ হবে। এখানে purpose/context/task/test reference:

| Topic | Purpose / phase / context | Own task / verification |
|---|---|---|
| decimal + rounding | P4–5 money/fee; finite precision | reject1.001; Round1.005 AwayFromZero→1.01; fee boundary |
| TryParse + culture | P1 input, out bool result | invalid/huge/EOF; explicit invariant dot/no grouping |
| DateTimeOffset/TimeSpan/IClock | P3–7 UTC/timeouts/day ranges | fake-clock lock/idle exact boundary; UTC+06 midnight |
| Guid + uniqueness | P5 ID candidate not proof | fake collision rejected; no overwrite |
| Guard clause/invariant | P2–5 valid state | negative/overflow rejected; validation no mutation |
| Atomic candidate publication | P5 state consistency | debit/credit/history faults before publication old state intact |
| Idempotency | P9 actor+key+payload+result | same original/changed conflict; one effect |
| DTO | P7 safe projection | no credential/nested mutable leak; ownership checks |
| Hashing/salt/byte[]/Convert | P3 credential | fresh salt, same PIN verifies; Base64 roundtrip; no plaintext/log |
| Authentication/authorization | P3/P8 identity vs permission | direct non-admin/other-owner calls denied |
| Strategy/Template Method | P6 optional/lab | simple switch alternative; preserve contract; no forced engine |
| Serialization/JSON | P11 optional checkpoint | DTO/version/corruption/relations/index/baseline; candidate import |
| xUnit/AAA/Theory/Fakes | P2–9 rule tests | arrange/action/assert; boundary cases; no real sleeps |
| Debugger/Watch/Call Stack | P2+ reproduce bugs | find first incorrect state; regression test; no secrets |
| lock/Interlocked/thread safety | P9 ThreadRaceLab only | counter exact with coordination; not multi-wallet atomicity |
| Structured logging/audit | P8 safe diagnostics vs business evidence | redaction; status+required audit atomic; notice failure separate |
| Configuration/secrets | P3–8 policy vs credentials | invalid config startup reject; no secret Git; versioned quote |
| Append-only/state transitions | P5/P8 immutable terminal facts | no edit/delete; Failed postings empty; Completed receipt |
| Regex | P1 supporting format | anchored ASCII-digit pattern, simple length/prefix checks alternative |
| Fluent Builder/IQueryable | P7 lab/concept only | nested builder vs simple query; IEnumerable main; no DB provider |

## 24.E – Deep Understanding Checkpoints

### Copy semantics vs parameter passing

```text
Value type by value     -> scalar fields copied; nested refs still shared
Reference type by value -> reference copied; object mutation shared
Value type by ref       -> caller variable slot aliased
Reference type by ref   -> caller reference slot aliased; reassignment visible
```

নিজে চারটি ছোট lab লিখে mutation/reassignment আলাদা predict করবে। Class→reference এবং ref parameter এক জিনিস নয়। Stack/heap দিয়ে semantics explain করবে না। [S7](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)।

### LINQ evaluation matrix

| Operator | When | Memory / trap |
|---|---|---|
| Where/Select | deferred | per-element; closure/source may change |
| OrderBy | deferred | buffers for sorting before first item |
| Sum/Count/Any | immediate scalar | Any may short-circuit; Count may use collection count |
| ToList/ToArray | immediate materialization | new outer container, nested objects not deep copied |
| GroupBy | deferred sequence | grouping requires buffering for LINQ-to-Objects |

Own task: query before source change; snapshot before source change; nested object mutation after snapshot; compare all three। IEnumerable in-memory main app; IQueryable provider concepts only। [S15](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/statements/linq)।

### OOP dispatch matrix

| Case | Base reference | Derived reference |
|---|---|---|
| virtual+override | actual derived implementation | derived implementation |
| nonvirtual/virtual hidden with new | base selected member | derived hidden member |
| interface implementation | implementation contract dispatch | concrete/explicit access rules |
| overload | chosen compile-time signature | chosen compile-time signature |

Own task: generic Shape/Formatter lab, not wallet engine rewrite। Explain base contract/precondition/invariant; sealed override freezes slot, not all behavior। [S24](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed)।

### Async/failure/commit

```text
request -> await FAKE operation -> revalidate -> stage -> publish -> notice
cancel/fault before publish: no money change
cancel/notice fault after publish: committed Completed retained
```

Await does not make multi-wallet updates atomic or menu automatically interactive। Cancellation cooperative, not rollback। Own task: fault/cancel at each boundary, prove state/result; WhenAll lab no live wallets। [S3](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/), [S14](https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads)।

### Events vs required consistency

Optional notification may fail after commit; required audit/history must be staged with state, not best-effort event subscriber। Publisher lifetime can retain subscribers; unsubscribe owned subscription। Own task: zero/two/throwing listeners and disposed subscriber। [S23](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/delegates-lambdas)।

### GC vs disposal

GC handles unreachable managed memory, not deterministic Dispose। Reachable history grows by design; event/static roots may retain otherwise-unused objects। Borrowed injected logger not disposed per operation; startup owns lifetime। Own task: owned/borrowed stream lab; no GC.Collect in main app। [S9](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals), [S18](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using)।

### Learning evidence tracker

```text
Topic ID:
Status: Not started / Practised / Reviewed / Needs revision
Own definition:
Prediction vs actual:
Own task/test evidence:
Project feature or lab and why:
Bug/misconception corrected:
Review feedback:
Next revision:
```

<a id="section-25"></a>

# 25. Console UI Preview

এটি **planned terminal UI**, web/mobile নয়; screenshot বা implemented app নয়। ASCII boxes/numbered menus। নিচের identity/IDs/data synthetic; different error screens alternatives, এক sequential trace নয়। Real credential দেবে না। Actual full Guid details-এ, table-এ short display+collision-safe selection।

Pink heading/green success/red error optional; text alone সব বোঝা যাবে। Terminal width narrow হলে plain list fallback; Unicode ৳ না চললে BDT। Dynamic names/references wrap/truncate safe; border overflow নয়। Console color reset finally; redirected input-এ masked ReadKey assumption নয়; safe input/EOF policy।

## UI Navigation Index

- [Main Menu](#ui-main)
- [Registration](#ui-register)
- [Login / Admin Routing](#ui-login)
- [User Dashboard](#ui-user)
- [Balance Check](#ui-balance)
- [Cash In](#ui-cash-in)
- [Cash Out](#ui-cash-out)
- [Send Money & Confirmation](#ui-send)
- [Recharge Simulation](#ui-recharge)
- [Successful Receipt](#ui-receipt)
- [History / Filters / Details](#ui-history)
- [Profile / Account Info](#ui-profile)
- [Change PIN](#ui-pin)
- [Validation / Failure / Timeout](#ui-error)
- [Admin Dashboard](#ui-admin)
- [Admin Users / Status Change](#ui-admin-users)
- [Reports / Audit](#ui-report)
- [Optional Data Tools](#ui-data-tools)

<a id="ui-main"></a>

### Main Menu

```text
+--------------------------------------------------------------+
|                          MAIN MENU                           |
+--------------------------------------------------------------+
| BKASH CONSOLE CLONE                                          |
| Educational Simulation | In-memory only                      |
| [1] Register                                                 |
| [2] Login (User / Admin)                                     |
| [0] Exit                                                     |
|                                                              |
| Select an option: _                                          |
+--------------------------------------------------------------+
```

<a id="ui-register"></a>

### Registration

```text
+--------------------------------------------------------------+
|                         REGISTRATION                         |
+--------------------------------------------------------------+
| Full Name       : Joy [example]                              |
| Mobile Number   : 01712345678 [synthetic]                    |
| New PIN         : *****                                      |
| Confirm PIN     : *****                                      |
|                                                              |
| [1] Submit  [0] Cancel                                       |
| Success: user + personal wallet created. Balance: BDT 0.00   |
| No real identity verification or real account creation.      |
+--------------------------------------------------------------+
```

<a id="ui-login"></a>

### Login / Admin Routing

```text
+--------------------------------------------------------------+
|                    LOGIN / ADMIN ROUTING                     |
+--------------------------------------------------------------+
| Mobile Number : 01712345678 [synthetic]                      |
| PIN           : *****                                        |
| [1] Login  [0] Back                                          |
|                                                              |
| Success: route using stored role, not selected role.         |
| Failure: Invalid credentials or account unavailable.         |
| Lock policy: 3 failed PIN checks -> 5-minute local lock.     |
+--------------------------------------------------------------+
```

Admin same Login entry; role verified from stored identity। Registration cannot select Admin; admin seed secret local supplied, not README/source।

<a id="ui-user"></a>

### User Dashboard

```text
+--------------------------------------------------------------+
|                        USER DASHBOARD                        |
+--------------------------------------------------------------+
| Welcome, Joy                                                 |
| Account: 017****5678 | Status: Active                        |
| Balance hidden until PIN-confirmed balance check.            |
|                                                              |
| [1] Check Balance        [2] Cash In                         |
| [3] Cash Out             [4] Send Money                      |
| [5] Mobile Recharge      [6] Transaction History             |
| [7] My Profile           [8] Change PIN                      |
| [9] Account Information  [0] Logout                          |
|                                                              |
| Select an option: _                                          |
+--------------------------------------------------------------+
```

MVP-তে unimplemented Core option hide বা “Not available in this milestone”; broken placeholder workflow নয়। Menu hide security নয়; services guard।

<a id="ui-balance"></a>

### Balance Check

```text
+--------------------------------------------------------------+
|                        BALANCE CHECK                         |
+--------------------------------------------------------------+
| Enter PIN: *****                                             |
| Account : 017****5678                                        |
| Balance : BDT 795.00 [example snapshot]                      |
|                                                              |
| [0] Back                                                     |
| Wrong PIN: balance stays hidden; shared attempt policy.      |
+--------------------------------------------------------------+
```

<a id="ui-cash-in"></a>

### Cash In

```text
+--------------------------------------------------------------+
|                           CASH IN                            |
+--------------------------------------------------------------+
| SIMULATED AGENT ROLE-PLAY ONLY                               |
| Agent Account : AGENT-DEMO-001                               |
| Amount        : BDT 1000.00                                  |
| Fee           : BDT 0.00                                     |
| Agent debit   : BDT 1000.00                                  |
| Wallet credit : BDT 1000.00                                  |
|                                                              |
| [1] Acknowledge simulation and confirm  [0] Cancel           |
| Enter your PIN: *****                                        |
| Agent balance/status and wallet cap checked at submit.       |
+--------------------------------------------------------------+
```

<a id="ui-cash-out"></a>

### Cash Out

```text
+--------------------------------------------------------------+
|                           CASH OUT                           |
+--------------------------------------------------------------+
| Agent Account : AGENT-DEMO-001                               |
| Amount        : BDT 100.00                                   |
| Fee (1.5%)    : BDT 1.50                                     |
| Total Debit   : BDT 101.50                                   |
|                                                              |
| [1] Confirm  [0] Cancel                                      |
| Enter PIN: *****                                             |
| Personal -101.50 | Agent +100.00 | Fee wallet +1.50          |
+--------------------------------------------------------------+
```

<a id="ui-send"></a>

### Send Money & Confirmation

```text
+--------------------------------------------------------------+
|                  SEND MONEY & CONFIRMATION                   |
+--------------------------------------------------------------+
| Receiver Number : 01812341234 [synthetic input]              |
| Amount          : BDT 200.00                                 |
| Reference       : Lunch                                      |
|                                                              |
| Receiver preview: 018****1234                                |
| Amount          : BDT 200.00                                 |
| Fee             : BDT 5.00                                   |
| Total Debit     : BDT 205.00                                 |
| Policy Version  : SIM-1                                      |
|                                                              |
| [1] Confirm  [0] Cancel                                      |
| Enter PIN: *****                                             |
| Advanced: retry key generated once, reused on retry.         |
+--------------------------------------------------------------+
```

Quote fee calculation confirmation নয় authorization/commit; target/status/funds/limits/session current submit-এ recheck। Full input only prompt-এ; persistent output/logs masked। Cancel before submit no financial record/effect।

<a id="ui-recharge"></a>

### Recharge Simulation

```text
+--------------------------------------------------------------+
|                     RECHARGE SIMULATION                      |
+--------------------------------------------------------------+
| Mobile       : 01912345678 [synthetic]                       |
| Operator     : SIM-OPERATOR-C [fake prefix map]              |
| Amount       : BDT 100.00                                    |
| Fee          : BDT 0.00                                      |
| Total Debit  : BDT 100.00                                    |
|                                                              |
| [1] Confirm  [0] Cancel                                      |
| Enter PIN: *****                                             |
| Waiting for FAKE gateway...                                  |
| No telecom/network/payment request is sent.                  |
+--------------------------------------------------------------+
```

Fake mapping example: 017→SIM-A, 018→SIM-B, 019→SIM-C; other allowed-format prefixes may be unsupported by simulation; never infer real operator/MNP। P9 async lab timeout/cancel; interactive cancel key needs explicit input design, await alone নয়।

<a id="ui-receipt"></a>

### Successful Receipt

```text
+--------------------------------------------------------------+
|                      SUCCESSFUL RECEIPT                      |
+--------------------------------------------------------------+
| TRANSACTION SUCCESSFUL [SIMULATION]                          |
| Transaction ID : TXN-DEMO-002 [short display only]           |
| Type           : Send Money                                  |
| Receiver       : 018****1234                                 |
| Amount         : BDT 200.00                                  |
| Fee            : BDT 5.00                                    |
| Total Debit    : BDT 205.00                                  |
| Status         : Completed                                   |
| Time           : 2026-10-09 18:05:00 +06:00 [example]        |
|                                                              |
| Press Enter to return...                                     |
+--------------------------------------------------------------+
```

Actual receipt full ID on details; table short-ID selection resolves full ID without ambiguity; no receiver balance/hash/salt/PIN। Balance-after only own historical snapshot and explicit reveal policy; default receipt above hides balance। Failed receipt unavailable।

<a id="ui-history"></a>

### History / Filters / Details

```text
+--------------------------------------------------------------+
|                 HISTORY / FILTERS / DETAILS                  |
+--------------------------------------------------------------+
| ID (short)    Type       Amount      Fee       Status        |
| DEMO-004      Recharge   1001.00     0.00      Failed        |
| DEMO-002      SendMoney   200.00     5.00      Completed     |
| DEMO-001      CashIn     1000.00     0.00      Completed     |
|                                                              |
| [1] Filter  [2] Sort  [3] Details  [4] Next page             |
| [0] Back                                                     |
| Empty: No transactions found.                                |
+--------------------------------------------------------------+
```

```text
+--------------------------------------------------------------+
|                       FILTER / DETAILS                       |
+--------------------------------------------------------------+
| Type/status/date/amount filters optional; page >=1.          |
| Start inclusive, end exclusive UTC range.                    |
| Select own transaction ID: _                                 |
| Completed -> receipt; Failed -> safe failure code.           |
| Unknown or other-user ID -> Transaction not found.           |
+--------------------------------------------------------------+
```

<a id="ui-profile"></a>

### Profile / Account Info

```text
+--------------------------------------------------------------+
|                    PROFILE / ACCOUNT INFO                    |
+--------------------------------------------------------------+
| Name          : Joy                                          |
| Mobile        : 017****5678                                  |
| Role          : User                                         |
| Account Type  : Personal                                     |
| Status        : Active                                       |
| Created       : example UTC timestamp                        |
| Send usage    : BDT 200.00 / 50000.00 today                  |
| Recharge usage: BDT 0.00 / 5000.00 today                     |
|                                                              |
| PIN/hash/salt never shown. [0] Back                          |
+--------------------------------------------------------------+
```

<a id="ui-pin"></a>

### Change PIN

```text
+--------------------------------------------------------------+
|                          CHANGE PIN                          |
+--------------------------------------------------------------+
| Old PIN       : *****                                        |
| New PIN       : *****                                        |
| Confirm PIN   : *****                                        |
|                                                              |
| [1] Submit  [0] Cancel                                       |
| Wrong old / same new / mismatch -> safe error.               |
| Success -> new salt/hash saved; session logged out.          |
+--------------------------------------------------------------+
```

<a id="ui-error"></a>

### Validation / Failure / Timeout

```text
+--------------------------------------------------------------+
|                VALIDATION / FAILURE / TIMEOUT                |
+--------------------------------------------------------------+
| INPUT ERROR: Enter a positive amount, max 2 decimals.        |
| BUSINESS ERROR: Insufficient balance for amount + fee.       |
| Submitted Failed attempt: no balance/fee/postings applied.   |
| CANCELLED: No money moved before commit.                     |
| TIMEOUT: Session expired. Please log in again.               |
| NOTICE ERROR: Transaction completed; display failed.         |
| UNEXPECTED ERROR: Safe message + correlation ID.             |
|                                                              |
| [0] Back / Login as applicable                               |
| Fatal/corrupt-state suspicion -> safe exit, not continue.    |
+--------------------------------------------------------------+
```

<a id="ui-admin"></a>

### Admin Dashboard

```text
+--------------------------------------------------------------+
|                       ADMIN DASHBOARD                        |
+--------------------------------------------------------------+
| Welcome, Admin | Authorized role: Admin                      |
| [1] User List                                                |
| [2] Search User                                              |
| [3] User Details                                             |
| [4] Activate / Suspend Account                               |
| [5] Search Transactions                                      |
| [6] Transaction Reports                                      |
| [7] Summary Statistics                                       |
| [8] Audit Logs                                               |
| [9] Data Tools [only if JSON enabled]                        |
| [0] Logout                                                   |
+--------------------------------------------------------------+
```

<a id="ui-admin-users"></a>

### Admin Users / Status Change

```text
+--------------------------------------------------------------+
|                 ADMIN USERS / STATUS CHANGE                  |
+--------------------------------------------------------------+
| User ID    Name    Mobile         Account Status             |
| U-DEMO-1   Joy     017****5678    Active                     |
| U-DEMO-2   Rahim   018****1234    Active                     |
|                                                              |
| Select target ID: U-DEMO-2                                   |
| Current: Active -> Requested: Suspended                      |
| Reason [required]: Training exercise                         |
| [1] Confirm  [0] Cancel                                      |
| Status + audit same publication; target session invalidated. |
| Agent/system wallets not editable through this menu.         |
+--------------------------------------------------------------+
```

<a id="ui-report"></a>

### Reports / Audit

```text
+--------------------------------------------------------------+
|                       REPORTS / AUDIT                        |
+--------------------------------------------------------------+
| EXAMPLE FIXTURE ONLY, NOT MEASURED APP OUTPUT                |
| Users: 2 | Completed: 3 | Failed: 1                          |
| Completed principal volume: BDT 1300.00                      |
| Collected fee: BDT 5.00 | Success rate: 75.00%               |
| Seed agent: 1000000.00; Cash In: 1000.00 and 100.00          |
| Send: 200.00, fee5.00; Recharge1001.00 failed                |
| Final: Joy795.00 / Rahim300.00 / Agent998900.00              |
| Fee5.00 / Settlement0.00; total1000000.00 conserved.         |
|                                                              |
| Audit: time / actor / action / target / outcome / reason     |
| [1] Date/type filters  [0] Back                              |
+--------------------------------------------------------------+
```

Report fixture fully synthetic and separate from alternative error previews; Completed principal counts cash in1000+100+send200=1300, failed excluded। Audit append-only API, not tamper-proof file।

<a id="ui-data-tools"></a>

### Optional Data Tools

```text
+--------------------------------------------------------------+
|                     OPTIONAL DATA TOOLS                      |
+--------------------------------------------------------------+
| [1] Export PRIVATE checkpoint                                |
| [2] Import checkpoint                                        |
| [0] Back                                                     |
|                                                              |
| Sensitive hash/salt may be included, never plaintext PIN.    |
| Do not share/commit checkpoint. No sessions restored.        |
| Import -> full candidate validation -> one publication.      |
| Failure -> old live state retained; success -> logout.       |
+--------------------------------------------------------------+
```

## UI Acceptance

- [ ] all navigation options valid; Back/Cancel/Logout/Exit distinct
- [ ] invalid input/EOF/narrow terminal/no-color safe
- [ ] amount/fee/total policy matches Section13
- [ ] PIN masked, no secret persistent output
- [ ] balance hidden dashboard; own details only
- [ ] history/status/receipt same committed snapshot
- [ ] Completed vs post-commit notice failure distinguished
- [ ] session/admin/ownership guards even direct service call
- [ ] Core/Advanced/Optional options enabled only when implemented

## UI Responsibility

MainMenu registration/login; UserMenu user workflows; AdminMenu authorized navigation; InputReader input/parse/EOF; ConsoleUi rendering/color; ReceiptPrinter safe receipt; services all auth/business/commit rules। No service Console calls, no UI balance mutation।

<a id="section-26"></a>

# 26. Git Workflow and References

## Git / documentation

Small focused commits; milestone branch/PR review; main build/test green। Ignore bin/obj/.vs/secrets/.env/private checkpoint/logs; no hard-coded admin credential। learning-log own explanation/prediction/test/bug/review; ADR context/options/decision/consequences।

```text
docs: define wallet simulation invariants
feat: add validated input menu
test: cover duplicate registration atomicity
refactor: isolate transaction candidate publication
```

Milestone tags: v0.1.0-mvp / v0.2.0-core / v0.3.0-advanced। Semantic versioning pre-1.0 conventions documented, tags not production certification। Optional CI restore/build/test/style checks, no real deployment/payment pipeline।

Setup lessons explain each step why/what: supported SDK→console skeleton→nullable→Git→tests→project references→format/analyzers। Choose compatible stable xUnit runner/template/package; current getting-started page may demonstrate preview versions, don't copy preview numbers blindly. [S27](https://xunit.net/docs/getting-started/v3/cmdline)। No setup/implementation automatically started by this README delivery।

## Revision summary / draft corrections

- Section24 coverage linked to real feature/test/lab, not merely mentions; 19+30+19 dedicated entries.
- Foundation array/exception/Task and all runtime gaps explicit; static and relationship concepts separated.
- Broken topic anchors replaced with explicit unique IDs; UI added to TOC.
- record/private-set/init/read-only interface/Task/GC/as/implicit-conversion misconceptions corrected.
- validate-first alone not atomicity; isolated candidate+single publication+fault tests defined.
- fee/config/limits/tests promoted to correctness MVP; no late testing dependency gap.
- seeded agent/fee/settlement makes all-wallet conservation explicit; real finance claims removed.
- HashSet-only retry replaced with actor/key/payload/original-result index.
- required audit not optional event subscriber; notification failure doesn't reverse Completed.
- repository-only future DB swap guarantee removed; commit/concurrency migration also needs review.
- inheritance/partial/Builder/generic CRUD/threading lab-first when no natural app need.
- Optional JSON credentials/session/schema/integrity/privacy limitations defined.

## Official references

Reference facts vs design: C#/.NET semantics below verified against documentation; fee/limit/roles/seed/UI/architecture are invented project decisions, not official bKash policies. Sources checked 9 October 2026; package/SDK support recheck at setup. Snippets own educational examples, not copied full tutorials.

- [S1: Support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core)
- [S2: Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
- [S3: Asynchronous programming](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [S4: IEnumerable&lt;T&gt;](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1?view=net-10.0)
- [S5: ICollection&lt;T&gt;](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.icollection-1?view=net-10.0)
- [S6: Exception handling](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements)
- [S7: Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)
- [S8: Object initializers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers)
- [S9: GC fundamentals](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals)
- [S10: Type testing/casts](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast)
- [S11: Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface)
- [S12: Numeric conversions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/numeric-conversions)
- [S13: Boxing/unboxing](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing)
- [S14: Cancellation](https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads)
- [S15: LINQ](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/statements/linq)
- [S16: Iterators/yield](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/yield)
- [S17: Constraints](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)
- [S18: using/disposal](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using)
- [S19: PBKDF2 API](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.rfc2898derivebytes.pbkdf2?view=net-10.0)
- [S20: FixedTimeEquals](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicoperations.fixedtimeequals?view=net-10.0)
- [S21: Access modifiers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)
- [S22: Constructors](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors)
- [S23: Delegates/lambdas/events](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/delegates-lambdas)
- [S24: Sealed](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed)
- [S25: Partial classes](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/partial-classes-and-methods)
- [S26: System.Text.Json](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/overview)
- [S27: xUnit getting started; select stable packages](https://xunit.net/docs/getting-started/v3/cmdline)
- [S28: override vs new](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/knowing-when-to-use-override-and-new-keywords)

## Documentation verification and scope of evidence

This deliverable is README only, not implemented application or project ZIP. Topic counts/explicit anchors/internal links/fences/feature coverage/ASCII screen widths are programmatically checked. C# compiler/unit/integration tests have not been executed here; expected test outcomes are acceptance targets, not reported pass results. Deep learning requires own implementation/lab/test/review evidence. No 100%-error-free or production-ready claim.
