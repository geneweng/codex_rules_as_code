# RuleProof: CI/CD and Audit Evidence for SNAP Eligibility Rules

**Product concept and market validation**  
**Research date:** 18 July 2026  
**Status:** Evidence-backed product thesis; customer discovery still required

## Executive summary

**RuleProof** would be a secure SaaS product, with an open-source local runner, that helps United States state agencies and their systems-integrator partners test changes to Supplemental Nutrition Assistance Program (SNAP) eligibility and benefit-calculation rules.

It is deliberately **not** a new eligibility engine and does not decide whether a person receives benefits. It is an independent assurance layer around existing engines:

1. ingest a policy change, implementation memo, decision table, or code diff;
2. use AI to propose source-linked test cases and affected rule paths;
3. run approved synthetic or de-identified household scenarios against the current and proposed engines;
4. show exactly which outcomes change, including boundary and temporal cases;
5. require policy, quality-control, and engineering review; and
6. produce a signed evidence package mapping sources → interpretations → tests → results → approvals.

The initial wedge is narrow: **regression and change-impact testing for SNAP financial eligibility and allotment calculations in one state**. Later extensions could cover Medicaid, Temporary Assistance for Needy Families, child-care subsidies, or cross-program interactions.

### Why this idea now

- USDA's FY2025 national SNAP payment error rate was **10.62%**, while states at or above 6% must prepare corrective action plans [1]. A payment error includes both underpayments and overpayments; it is not synonymous with fraud.
- Federal changes make states with error rates of 6% or more responsible for a share of benefit costs beginning in October 2027, creating unusually direct budget pressure [2].
- States are simultaneously absorbing frequent policy changes, operating complex legacy eligibility systems, and looking at AI. This increases the need for independent, reproducible assurance.
- A Georgetown/Digital Benefits Network challenge has already established interest in using generative AI to turn SNAP and Medicaid policy into questions, summaries, and code [3, 4]. RuleProof commercializes a safer adjacent task: AI proposes tests and impact analyses, while reviewed deterministic systems remain authoritative.
- Existing products concentrate on workflow, case management, rules execution, or developer-level testing. The product gap is a policy-owner-friendly, engine-neutral evidence layer connecting legal authority to regression results.

### Product thesis

> State benefit agencies will pay for an independent assurance platform if it measurably reduces rule-release defects and quality-control findings, shortens policy-to-production testing, integrates without replacing the incumbent eligibility system, and produces defensible evidence for agency and federal review.

This thesis is **plausible, not yet proven**. Desk research validates the pain, urgency, user population, technology feasibility, and adjacent spending. It does not prove procurement willingness or identify the exact budget owner. Those require interviews and a paid pilot.

## 1. Problem definition

SNAP combines federal rules, annual parameters, state options, waivers, operational guidance, and local implementation. A policy change can affect eligibility, deductions, household composition, reporting, work requirements, notices, and benefit amounts. The operational rule is then embedded in an integrated eligibility system, worker procedures, forms, and quality-control processes.

The failure mode is not simply “developers write the wrong code.” It is a broken chain of custody:

```text
Federal or state authority
        ↓
Policy interpretation and implementation memo
        ↓
Business requirements and decision tables
        ↓
Vendor or agency implementation
        ↓
Test cases and worker instructions
        ↓
Production determinations
```

Interpretation, requirements, code, tests, and operational guidance often live in different tools and are reviewed by different teams. A test may demonstrate that code matches a requirement without demonstrating that the requirement matches the governing source.

### Observable pain signals

1. **Material error rates.** USDA says state error rates measure the accuracy of eligibility and benefit determinations. Its FY2025 release reports a 10.62% national payment error rate and requires corrective action plans at 6% or above [1, 5].
2. **Direct state financial exposure.** Under the new cost-sharing schedule, states at 6–8% error face a 5% share, 8–10% a 10% share, and above 10% a 15% share, subject to timing and transition provisions [2]. Even a small improvement around a threshold can have a large fiscal value.
3. **Large operational scale.** SNAP serves tens of millions of people, and the USDA maintains monthly national program participation and benefit tables [6]. A rare edge-case defect can therefore affect many households.
4. **Large adjacent IT spending.** GAO found that CMS reimbursed states **$34.3 billion** for Medicaid claims and eligibility-related IT systems during FY2008–FY2018 [7]. This is not a SNAP testing-market estimate, but it proves that state eligibility technology is a substantial, established spending category.
5. **Modular procurement direction.** CMS now uses Streamlined Modular Certification for Medicaid enterprise systems, reinforcing a market direction toward measurable modules and outcomes rather than only monolithic replacements [8, 9]. SNAP is administered separately, but many states operate integrated eligibility platforms, making the modular pattern commercially relevant.
6. **Demonstrated AI interest.** The Policy2Code challenge explicitly tested generative AI on SNAP and Medicaid policy-to-question, policy-to-summary, and policy-to-code tasks [3].

## 2. Product design

### 2.1 Core workflow

#### Step 1: Create a change set

A policy analyst creates a RuleProof change set and attaches authoritative sources, implementation guidance, effective dates, state options, waiver documents, and the proposed requirements or engine version.

#### Step 2: Build the source map

AI extracts candidate definitions, conditions, exceptions, parameters, cross-references, and dates. Every candidate retains a page, section, and text-span citation. The analyst accepts, edits, rejects, or marks an item unresolved.

#### Step 3: Propose test scenarios

RuleProof proposes structured household scenarios covering:

- examples explicitly stated in guidance;
- values immediately below, at, and above thresholds;
- household-composition variants;
- combinations of deductions and income sources;
- exception and waiver paths;
- effective-date boundaries;
- missing, conflicting, or late evidence; and
- state options affected by the change.

AI never silently assigns the expected outcome. An expected result must be backed by an authoritative example, a reviewed calculation, or explicit policy-owner approval.

#### Step 4: Differential execution

Connectors run the scenario corpus against:

- the current production-equivalent engine;
- the candidate engine or rules package; and, optionally,
- an independent reference implementation.

RuleProof classifies changed results as intended, unexpected, unresolved, or non-comparable. It displays eligibility, allotment, intermediate calculations, notices or reason codes, and engine traces where available.

#### Step 5: Population impact sandbox

The product generates a synthetic household population or accepts an agency-controlled, de-identified dataset in a private environment. It estimates how many cases move between outcomes and highlights discontinuities, unexpected clusters, and cases near policy boundaries.

#### Step 6: Approval and evidence package

A release gate requires named approvals from policy, quality control, and technology owners. RuleProof produces a versioned package containing source hashes, interpretation decisions, tests, execution results, exceptions, reviewer actions, and engine identifiers.

### 2.2 What AI does—and does not do

| AI-assisted function | Required control |
|---|---|
| Extract candidate rules and dates | Source citations and analyst acceptance |
| Suggest affected rule paths | Dependency checks and owner review |
| Generate boundary and adversarial scenarios | Schema validation and deduplication |
| Draft plain-language explanations | Ground only in execution trace and source map |
| Cluster differential failures | Preserve underlying cases; no opaque suppression |
| Suggest missing coverage | Human approval of risk model and test additions |

AI does **not** issue eligibility decisions, publish unreviewed rules, invent expected results, train on applicant data by default, or replace appeal and quality-control processes.

### 2.3 Initial integrations

The MVP should support three adapter types:

1. REST or batch adapters for incumbent eligibility engines;
2. OpenFisca packages and tests; and
3. DMN decision services such as Camunda-compatible engines.

OpenFisca already supports YAML tests, calculation traces, reforms, and large simulations [10–12]. Camunda supports DMN modelling and process/task testing [13, 14]. RuleProof should complement these engines by adding cross-engine comparison, source provenance, policy-owner review, coverage analysis, and audit evidence—not duplicate their runtime.

### 2.4 Deployment model

- **Open-source runner:** Executes scenarios and normalizes outputs inside an agency or vendor network.
- **SaaS control plane:** Manages source maps, test design, review, evidence, and non-sensitive metadata.
- **Private-cloud option:** For agencies that cannot allow artifacts or telemetry outside their boundary.
- **No production applicant PII required:** Synthetic cases are the default. De-identified quality-control samples remain in the customer environment.

This architecture lowers procurement and security risk while creating an adoption path that does not require replacing a mission-critical system.

## 3. Target customer and jobs to be done

### Beachhead customer

A state human-services agency with:

- a SNAP payment error rate above or near 6%;
- an integrated eligibility system maintained by a major systems integrator;
- a pending set of federal or state rule changes;
- a policy and quality-control team able to define expected outcomes; and
- an executive mandate to reduce errors without a full system replacement.

### Economic buyer candidates

- SNAP program director;
- chief information officer or eligibility-systems director;
- quality-control or payment-accuracy director;
- systems-integrator delivery executive; or
- central digital-services office funding a cross-program capability.

The identity of the true budget owner is the most important unresolved commercial question.

### Users

- policy analysts mapping authority to operational rules;
- quality-control staff designing and reviewing cases;
- business analysts writing requirements;
- engineers and vendor teams implementing changes;
- test managers executing regression suites;
- auditors and agency counsel reviewing evidence.

### Jobs to be done

- “Before this policy change ships, show me every materially affected scenario.”
- “Prove that the implementation matches our approved interpretation.”
- “Tell me whether a changed outcome is intended or a regression.”
- “Give federal reviewers and auditors reproducible evidence.”
- “Let policy staff review rules without reading application source code.”
- “Reduce release risk without replacing the eligibility platform.”

## 4. Competitive landscape

### 4.1 Competitor and substitute map

| Category | Representative offering | Strength | Gap RuleProof targets |
|---|---|---|---|
| Integrated eligibility systems and integrators | Deloitte state eligibility services and other major integrators | Full implementation, policy expertise, incumbent relationships | Assurance is tied to the implementation vendor; provenance and reusable cross-engine evidence are not the primary product [15] |
| Government workflow/benefits platform | ServiceNow Social Benefits Playbook | Configurable benefits workflow and predefined eligibility policies | Primarily a delivery and execution platform, not an independent legal-to-test evidence layer [16] |
| Tax and benefit rule engine | OpenFisca | Open models, API, tests, reform simulation, traces | Requires model/code skills; does not itself supply a multi-party approval and engine-neutral audit product [10–12] |
| BPMN/DMN platform | Camunda | Mature process and decision modelling, execution, task testing | Generic developer platform; source-law provenance and benefits-specific test generation are outside its core [13, 14] |
| Government-built discovery portals | India's myScheme and similar services | Citizen-facing program discovery at national scale | Oriented to discovery and access, not release assurance for incumbent state engines [17] |
| Internal spreadsheets and test-management tools | Excel, Jira, test suites, policy memos | Familiar and already procured | Fragmented traceability, weak differential analysis, manual evidence assembly |
| General LLM copilots | Commercial chat and coding assistants | Fast extraction and code drafting | Probabilistic, weak authority model, no deterministic release gate |

### 4.2 Competitive position

RuleProof should be positioned as **independent release assurance**, analogous to CI/CD and observability for policy logic. It should integrate with rule engines and system integrators rather than compete to replace them.

The defensible asset is not a proprietary language model. It is the accumulated benefits-specific scenario grammar, source-to-rule ontology, connector ecosystem, reviewed test corpus, evidence schema, and measured ability to predict defects.

### 4.3 Why incumbents may still win

Large integrators could add similar functionality to existing contracts. Generic decision platforms could extend their testing products. Agencies may prefer internal tools to avoid another procurement. RuleProof therefore needs:

- fast deployment around an existing engine;
- visible independence from the implementation vendor;
- benefits-domain content that generic products lack;
- measurable quality or cycle-time improvements within one release; and
- an open evidence format that reduces lock-in.

## 5. Market validation

### 5.1 Evidence that supports the thesis

**Pain is validated.** USDA publishes state-level error rates specifically measuring eligibility and benefit-calculation accuracy [1, 5]. The national FY2025 rate exceeded the threshold that triggers corrective action.

**Urgency is validated.** State financial responsibility is now tied to error-rate bands, beginning in October 2027 under current law [2]. The commercial window is immediate because the relevant performance years and remediation work precede that date.

**Scale is validated.** SNAP operates nationally through state agencies, and Medicaid/CHIP alone enrolled 74.3 million people in March 2026 [18]. Many state systems determine eligibility across multiple programs, so a SNAP wedge can expand into a larger rules-assurance footprint.

**Technology spending is validated.** GAO's $34.3 billion figure for federally reimbursed Medicaid IT over FY2008–FY2018 shows that the surrounding market supports substantial technology procurement [7].

**AI/RaC interest is validated.** Digital Benefits Network and Georgetown researchers have run explicit policy-to-code experiments for SNAP and Medicaid [3, 4]. The European GovTech4All initiative is also building a stack combining RaC guidelines, AI tools, APIs, and public-sector proofs of concept [19].

**Technical feasibility is validated in components.** OpenFisca demonstrates executable benefit modelling, traceable calculations, test cases, and population simulation [10–12]. General decision platforms demonstrate visual rule modelling and testing [13, 14]. The integration and governance product remains to be validated.

### 5.2 Evidence that is still missing

- How much of the SNAP payment error rate is addressable through better rule-change testing versus household reporting, caseworker procedure, source-data quality, staffing, or other causes?
- Will a state buy an independent tool, require it through its systems integrator, or expect the incumbent vendor to provide it?
- Does the target agency have accessible non-production interfaces and stable outputs for differential testing?
- Will policy staff maintain source mappings and approve expected outcomes?
- Which evidence format would USDA, state quality control, auditors, and counsel actually accept?
- Is deployment possible within a normal change order, or does it require a multiyear procurement?

RuleProof must not claim it can “reduce the payment error rate” until a pilot links prevented or detected defects to error categories used by SNAP Quality Control.

## 6. Market size and business model

### 6.1 Bottom-up initial market

The initial buyer universe is intentionally small: the 50 states, District of Columbia, territories, and their principal eligibility-system integrators. This is an enterprise government market, not a self-service SaaS market.

An illustrative—not externally validated—annual contract structure:

| Offer | Illustrative annual price |
|---|---:|
| One-program pilot, one major rule release | $100,000–$250,000 |
| State SNAP assurance subscription | $300,000–$750,000 |
| Integrated benefits suite | $750,000–$1.5 million |
| Integrator/OEM agreement | Negotiated platform fee plus deployments |

At an average $500,000 annual contract across 20 state customers, the focused recurring revenue opportunity would be approximately **$10 million annually**. Expansion to other benefits, integrators, and federal reference testing could increase it. This is a bottom-up planning scenario, not a market forecast.

The product can be attractive despite a modest customer count because the fiscal downside of crossing a SNAP error-rate threshold can reach tens or hundreds of millions of dollars for a state [2]. Pricing must nevertheless be tied to demonstrated avoided defects, reduced manual testing, or faster compliant releases—not to a speculative share of benefits spending.

### 6.2 Recommended commercial model

- paid 12–16 week diagnostic pilot;
- subscription based on programs, environments, and annual scenario volume;
- implementation and connector fees capped to avoid becoming a consultancy;
- private-cloud premium;
- free open-source runner and evidence schema;
- partner channel for systems integrators and public-sector digital consultancies.

## 7. MVP

### Scope

One state's SNAP gross income, net income, deductions, and allotment calculation for one policy release. Exclude non-financial categorical eligibility, fraud detection, identity verification, document adjudication, and final production decisions unless required for the selected calculation path.

### Required capabilities

1. source upload with stable paragraph citations and hashes;
2. human-reviewed rule and interpretation map;
3. AI-assisted generation of structured household scenarios;
4. scenario editor usable by a policy analyst;
5. adapter SDK plus one real incumbent-engine connector;
6. old-versus-new differential runner;
7. threshold, date, and combinatorial coverage reports;
8. review workflow for expected outcomes and intended changes;
9. exportable evidence bundle in JSON and human-readable PDF/HTML; and
10. complete audit log and role-based access control.

### Explicit non-goals

- replacing the state's integrated eligibility system;
- making live eligibility decisions;
- autonomously interpreting ambiguous law;
- using applicant data to train a general model;
- supporting every benefit program or rule language;
- promising a legally authoritative rule model.

### Success metrics for a pilot

- detect at least one previously unknown material defect or prove coverage of a high-risk release area;
- reduce manual scenario-design and evidence-assembly time by at least 30%;
- achieve 100% source linkage for approved expected outcomes in the pilot scope;
- reproduce current-engine outputs on an agreed baseline corpus;
- have policy staff independently create or approve scenarios without developer help;
- produce an evidence package accepted as useful by quality-control and audit stakeholders;
- introduce no applicant PII into the SaaS control plane.

## 8. Go-to-market plan

### Phase 1: discovery partner

Recruit one state with elevated payment-error exposure and a scheduled SNAP rule release. Offer a fixed-price diagnostic using synthetic cases and a read-only test interface. The ideal sponsor controls both policy and system testing or can convene them.

### Phase 2: integrator co-sell

After an independent pilot, offer RuleProof to systems integrators as a release-quality accelerator. Avoid presenting it as an audit weapon against the vendor; position shared, reproducible evidence as protection for both agency and integrator.

### Phase 3: shared test packs

Publish open, non-state-specific test schemas and federally grounded scenario packs. Sell state-option overlays, workflow, integrations, private deployment, and evidence management. Shared federal baselines create network effects while states retain control of their interpretations.

### Phase 4: adjacent programs

Expand first into Medicaid MAGI calculations, where federal/state variation, large enrollment, established modular certification, and integrated platforms create a natural adjacency. CMS reported 74,294,361 Medicaid and CHIP enrollees in March 2026 [18].

## 9. Customer discovery plan

Conduct at least 25 interviews before building beyond a clickable workflow and adapter prototype:

- 5 state SNAP policy leaders;
- 5 SNAP quality-control/payment-accuracy leaders;
- 5 eligibility-system product or test managers;
- 4 systems-integrator leaders;
- 3 federal or independent oversight specialists; and
- 3 legal-aid or beneficiary advocates.

### Interview questions

Avoid pitching initially. Ask for the last real rule change and reconstruct it.

1. “Walk me through the last SNAP policy change from receipt to production.”
2. “Where were interpretations recorded, and who approved them?”
3. “How were expected results for test cases established?”
4. “Which defects were found late, and what did they cost?”
5. “Which quality-control error categories could testing have prevented?”
6. “Can you run synthetic cases through production-equivalent logic today?”
7. “What evidence is assembled for release approval and audit?”
8. “Who owns the budget for reducing payment errors or improving rule releases?”
9. “What security, procurement, and vendor constraints would block an external assurance tool?”
10. “Would you pay for a 12-week pilot tied to an upcoming rule release? From which contract vehicle?”

### Strong validation signals

- three agencies provide a real past change package and test corpus;
- two agencies grant access to a non-production engine or batch runner;
- one agency signs a paid pilot or letter of intent with a named budget owner;
- an integrator agrees to build or support an adapter;
- quality-control staff can map discovered defects to official error categories.

### Kill or pivot criteria

Pivot away from state SNAP release assurance if, after 25 qualified interviews:

- fewer than five report material rule-release defects or evidence-assembly pain;
- no agency can expose a safe test interface within six months;
- payment errors are overwhelmingly unrelated to formal rule implementation;
- all likely buyers require functionality to be bundled exclusively with the incumbent system; or
- no buyer will fund a pilot above $100,000.

A likely pivot would be selling the same source-to-test evidence layer to systems integrators across Medicaid and SNAP, rather than selling directly to states.

## 10. Risks and mitigations

| Risk | Mitigation |
|---|---|
| AI generates a plausible but wrong test oracle | AI proposes scenarios only; reviewed sources or approved calculations establish expected results |
| Product is blamed for eligibility decisions | Keep it out of the production decision path; label evidence status and responsibility clearly |
| Sensitive data enters the SaaS | Default to synthetic households; run de-identified samples locally; prohibit model training on customer content |
| Integrations become bespoke consulting | Publish a narrow adapter contract; support batch files first; price and cap custom work |
| Policy staff reject developer tooling | Design scenario and approval workflows around policy concepts, not code |
| Incumbent vendor blocks access | Secure executive sponsorship; support black-box batch comparison; offer integrator partnership |
| Test coverage creates false confidence | Report uncovered paths, unresolved interpretations, and provenance gaps explicitly |
| Procurement takes too long | Start through existing integrator contracts, innovation vehicles, or fixed-scope diagnostic services |
| Evidence is not accepted by oversight bodies | Co-design export format with state QC and federal stakeholders before automating it |

## 11. Final assessment

### Recommendation: pursue discovery, not full build

RuleProof addresses a specific and urgent problem with a low-disruption product shape. The strongest features of the thesis are:

- a direct financial trigger tied to SNAP accuracy;
- a bounded initial domain with computable rules;
- a large installed base of expensive systems that are difficult to replace;
- demonstrated government interest in AI-assisted Rules as Code;
- component-level technical feasibility; and
- a safety posture that keeps AI away from final benefit decisions.

The weak point is commercial, not technical. It is not yet clear whether the agency, quality-control office, or incumbent integrator will own the purchase, nor how much of the measured error rate the product can address. A paid pilot must validate those points.

If discovery succeeds, the product has a credible expansion path from SNAP release testing to a general **assurance and evidence platform for executable public rules**. Its strategic category would be neither a legal chatbot nor a benefit calculator, but **continuous verification for policy implementation**.

## Sources

1. U.S. Department of Agriculture, Food and Nutrition Service, [“USDA Announces FY 2025 State Payment Error Rates in SNAP”](https://www.fns.usda.gov/newsroom/usda-0082.26), June 2026.
2. Associated Press, [“Dozens of States Could Face New Costs Because of High Error Rates in SNAP Food Aid”](https://apnews.com/article/4fa06549e3ec2bd1a1c665b1985cefea), June 2026. Used for a current explanation of the enacted cost-sharing schedule and state examples; implementation details should be verified against governing federal authorities during product discovery.
3. Digital Benefits Network, [“Policy2Code Prototyping Challenge”](https://digitalgovernmenthub.org/get-involved/policy2code/).
4. Digital Benefits Network and Massive Data Institute, [“AI-Powered Rules as Code: Experiments with Public Benefits Policy”](https://digitalgovernmenthub.org/publications/ai-powered-rules-as-code-experiments-with-public-benefits-policy/), 2024.
5. U.S. Department of Agriculture, Food and Nutrition Service, [“SNAP Payment Error Rates”](https://fns-prod.azureedge.us/snap/qc/per).
6. U.S. Department of Agriculture, Food and Nutrition Service, [“SNAP Data Tables”](https://www.fns.usda.gov/pd/supplemental-nutrition-assistance-program-snap).
7. U.S. Government Accountability Office, [*Medicaid Information Technology: Effective CMS Oversight and States' Sharing of Claims Processing and Information Retrieval Systems Can Reduce Costs*](https://www.gao.gov/products/gao-20-179), GAO-20-179, 2020.
8. Centers for Medicare & Medicaid Services, [“Streamlined Modular Certification”](https://www.medicaid.gov/medicaid/data-systems/streamlined-modular-certification).
9. Centers for Medicare & Medicaid Services, [MES Certification Repository](https://cmsgov.github.io/CMCS-DSG-DSS-Certification/SMC%20Process/).
10. OpenFisca, [“Tests”](https://openfisca.org/doc/contribute/tests.html).
11. OpenFisca, [“Using the /trace Endpoint”](https://openfisca.org/doc/openfisca-web-api/trace-simulation.html).
12. OpenFisca, [“Running a Simulation”](https://openfisca.org/doc/simulate/index.html).
13. Camunda, [“DMN in Modeler”](https://docs.camunda.io/docs/8.7/components/modeler/dmn/).
14. Camunda, [“Task Testing”](https://docs.camunda.io/docs/components/modeler/task-testing/).
15. Deloitte, [“State Integrated Eligibility Services”](https://www.deloitte.com/us/en/industries/government-public/about/health-and-human-services-eligibility-and-enrollment.html).
16. ServiceNow, [“Using the Eligibility Rules Engine”](https://www.servicenow.com/docs/r/government-industry/psds-using-sb-playbooks-pace.html).
17. Government of India, [“About myScheme”](https://www.myscheme.gov.in/about).
18. Centers for Medicare & Medicaid Services, [“March 2026 Medicaid & CHIP Enrollment”](https://www.medicaid.gov/medicaid/map-element-medicaid-and-chip-enrollment), updated June 2026.
19. Interoperable Europe, [“Citizen Centric Rules as Code”](https://interoperable-europe.ec.europa.eu/collection/eugovtech/citizen-centric-rules-code).

---

**Research limitations:** This is desk research, not primary customer research. The market-size figures are bottom-up scenarios rather than third-party forecasts. Current SNAP cost-sharing rules and implementation dates are time-sensitive and should be rechecked against federal primary sources before use in a sales proposal or investment decision.
