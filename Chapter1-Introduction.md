# Chapter 1: Introduction

The automobile industry stands at an inflection point. What began as a mechanical engineering discipline—focused on combustion engines, transmissions, and chassis dynamics—has transformed into a software-intensive domain where lines of code now determine vehicle behavior as much as physical components. A modern premium vehicle contains over 100 million lines of source code distributed across more than 100 Electronic Control Units (ECUs), each responsible for functions ranging from powertrain management to advanced driver assistance systems. This is not a gradual evolution. It is a fundamental restructuring of how vehicles are conceived, developed, and maintained throughout their operational lifecycle.

This thesis investigates the intersection of artificial intelligence and automotive software security, specifically examining whether Large Language Models (LLMs) can automate the generation of fuzz drivers within Continuous Integration, Continuous Deployment, and Continuous Testing (CI/CD/CT) pipelines. The research was conducted at CARIAD SE, the automotive software company of the Volkswagen Group, where the practical constraints of enterprise-scale development collide with the theoretical promises of AI-assisted security testing.

---

## 1.1 Background and Motivation

The motivation for this research emerges from three converging pressures facing the automotive industry: exploding software complexity, escalating cybersecurity requirements, and the relentless demand for faster development cycles. Each pressure, considered independently, presents significant challenges. Together, they create a situation where traditional approaches to software security are fundamentally inadequate.

### 1.1.1 Automotive Software Evolution and Complexity

Twenty years ago, a vehicle's electronic systems were comparatively simple. The engine control unit managed fuel injection and ignition timing. The anti-lock braking system prevented wheel lockup during hard braking. The airbag controller monitored crash sensors and deployed restraints when necessary. These systems operated largely in isolation, connected by point-to-point wiring and proprietary communication protocols. A skilled automotive engineer could understand the complete electronic architecture of a vehicle within weeks. That era is gone.

The contemporary automobile is better understood as a distributed computing platform on wheels. The Software-Defined Vehicle (SDV) paradigm has shifted competitive differentiation from mechanical systems to software capabilities. Tesla demonstrated this transformation most visibly: the same physical hardware gains new features, improved performance, and enhanced safety through over-the-air software updates. Traditional automakers watched customers receive Autopilot improvements, new entertainment applications, and even acceleration upgrades—all delivered as software packages to vehicles already in customer driveways. The implications were clear. Software is no longer a supporting function for mechanical systems. Software defines the vehicle experience.

This transformation has produced staggering complexity. The 100 million lines of code figure, often cited in industry analyses, understates the challenge because it treats all code as equivalent. In reality, automotive software spans multiple programming paradigms, safety criticality levels, and execution environments. Safety-critical code running on AUTOSAR Classic platforms follows different development practices than infotainment applications built on Linux-based systems. Real-time control loops executing at microsecond intervals coexist with cloud-connected services managing terabytes of driving data. The heterogeneity is extreme.

Consider the software architecture of a modern Level 2+ automated driving system. Perception algorithms process data from cameras, radar, and lidar sensors, fusing multiple input streams into a coherent environmental model. Planning modules compute safe trajectories through complex traffic scenarios. Control systems translate these trajectories into steering, throttle, and brake commands executed by low-level actuators. Each layer involves distinct technical disciplines: computer vision, machine learning, control theory, real-time systems, and functional safety engineering. The integration challenges alone consume enormous engineering effort.

The codebase demographics compound the complexity problem. Much of the software running in production vehicles was written years or even decades ago. Legacy code from previous vehicle generations gets carried forward, often with minimal documentation and no surviving institutional knowledge about design decisions. Original developers have moved on. Specifications have been lost. The code works—mostly—but understanding why it works requires archaeological excavation through layers of accumulated modifications. Writing security tests for such code demands deep comprehension of behavior that may never have been formally specified.

Automotive software also exhibits extreme longevity requirements. A vehicle sold today will remain on roads for 15 to 20 years. The software must remain functional, secure, and updatable throughout this period. Contrast this with smartphone applications, where a three-year support window is considered generous. Automotive software must anticipate attack vectors that do not yet exist, defend against exploitation techniques that have not been invented, and remain compatible with infrastructure that will evolve unpredictably. The temporal dimension of automotive software security extends far beyond typical enterprise software considerations.

The supply chain structure further amplifies complexity. A single vehicle may incorporate software components from dozens of Tier 1 suppliers, each with their own development processes, quality standards, and security practices. The Original Equipment Manufacturer (OEM) integrates these components into a coherent system, but visibility into supplier codebases is often limited. Security vulnerabilities may lurk in third-party libraries, hidden behind proprietary interfaces and contractual barriers that impede thorough analysis. The attack surface extends across organizational boundaries that resist traditional security governance approaches.

### 1.1.2 Security Challenges in Modern Systems

The cybersecurity landscape for automotive systems differs fundamentally from traditional IT security. When a web server is compromised, the consequences—data theft, service disruption, financial loss—are serious but bounded. When a vehicle control system is compromised, the consequences potentially include physical injury or death. This is not hyperbole. Researchers have demonstrated remote exploitation of vehicle systems, including scenarios where attackers could disable brakes or seize steering control at highway speeds. The Miller and Valasek Jeep Cherokee hack in 2015 forced Chrysler to recall 1.4 million vehicles. The attack surface was a cellular-connected infotainment system that provided a pathway to safety-critical vehicle networks. Software vulnerability became physical danger.

The regulatory environment has responded to these risks. UNECE Regulation No. 155, effective for new vehicle types from July 2022, mandates cybersecurity management systems for vehicle manufacturers. ISO/SAE 21434:2021 establishes requirements for cybersecurity engineering throughout the vehicle lifecycle, from concept through decommissioning. These standards demand systematic identification, assessment, and mitigation of cybersecurity risks. They require evidence that security has been designed into systems, not bolted on as an afterthought. Compliance is not optional—vehicles that fail to meet these requirements cannot be sold in regulated markets.

The standards impose particular obligations regarding security testing. Organizations must demonstrate that they have verified the effectiveness of cybersecurity controls through appropriate testing methods. Fuzz testing—the automated generation of malformed or unexpected inputs to discover software vulnerabilities—is explicitly recognized as a relevant technique. The challenge lies in scaling fuzz testing to match the scope of automotive software. Writing effective fuzz drivers requires deep understanding of API semantics, valid input ranges, and expected behaviors. A security expert might spend days developing a single fuzz driver for a complex library. Multiply this effort across thousands of components, and the impossibility of manual approaches becomes evident.

The economics of automotive cybersecurity create additional pressure. Security testing competes for resources with feature development, performance optimization, and regulatory compliance activities. Development teams face relentless pressure to deliver new functionality on aggressive timelines. Security, perceived as a cost center rather than a value creator, often receives inadequate attention until problems become undeniable. Post-release vulnerability discoveries are enormously expensive—recalls cost millions, reputation damage lingers for years, and regulatory penalties can be substantial. Yet the organizational incentives frequently favor shipping quickly over shipping securely.

The threat landscape continues to evolve. Nation-state actors have demonstrated interest in automotive systems as both espionage targets and potential weapons. Criminal organizations have developed vehicle theft techniques exploiting keyless entry vulnerabilities. Researchers continuously discover new attack vectors, from CAN bus injection to charging infrastructure compromise. Each connected feature—telematics, over-the-air updates, vehicle-to-everything communication—expands the attack surface. Security is not a problem that can be solved once; it demands continuous vigilance against an adaptive adversary.

The zero-tolerance safety mandate that governs automotive development creates particular challenges for security testing. Safety-critical systems undergo exhaustive verification against precisely defined requirements. Every possible failure mode must be analyzed. Mitigations must be demonstrated through testing. This approach works well for safety engineering, where the hazards are known and the physics are deterministic. Security engineering faces a different problem: the adversary is intelligent, creative, and unconstrained by specifications. Attackers do not limit themselves to documented interfaces. They probe for implementation weaknesses, logic errors, and overlooked edge cases. Testing for security requires exploring the unexpected, not merely verifying the expected.

### 1.1.3 CI/CD/CT Pipeline Context

Modern software development has embraced Continuous Integration, Continuous Deployment, and Continuous Testing as foundational practices for managing complexity while maintaining development velocity. The philosophy is straightforward: integrate code changes frequently, test them automatically, and deploy validated changes rapidly. Small, incremental changes are easier to understand, review, and debug than large, infrequent releases. Automated testing catches regressions before they propagate. Fast feedback loops enable developers to address problems while context remains fresh.

The automotive industry has adopted CI/CD principles, adapting them to domain-specific constraints. The continuous deployment aspect requires careful interpretation—one does not casually push untested code to vehicles traveling at highway speeds. Automotive CI/CD pipelines typically feed into more formal release processes where human review and additional validation occur before software reaches production vehicles. Nevertheless, the core principles apply: automate what can be automated, test early and often, maintain a continuously releasable codebase.

Figure 1.1 illustrates a representative automotive CI/CD pipeline architecture. The pipeline begins with developers committing code changes to version control repositories. Automated build processes compile the code, generating executable artifacts for target platforms. Static analysis tools examine the source code for common defects, coding standard violations, and potential security weaknesses. Unit tests verify component-level behavior. Integration tests confirm that components interact correctly. The pipeline may include hardware-in-the-loop testing, where software executes on actual ECU hardware connected to simulated vehicle systems. Each stage acts as a quality gate—failures block progression until issues are resolved.

![Figure 1.1: The Automotive CI/CD Pipeline](bilder/cicd.png)
*Figure 1.1: The Automotive CI/CD Pipeline. The diagram illustrates the sequential stages from code commit through deployment, highlighting the testing bottleneck where security validation must occur within strict time constraints. Traditional manual security testing cannot scale to meet the velocity demands of modern development practices.*

The temporal constraints on CI/CD pipelines create tension with thorough security testing. A well-optimized pipeline might complete build and basic testing within 15 to 30 minutes. Developers expect rapid feedback—waiting hours for test results disrupts workflow and encourages practices that circumvent the pipeline entirely. Security testing, particularly dynamic analysis techniques like fuzzing, often requires extended execution to achieve meaningful coverage. A fuzzing campaign might run for hours or days to discover subtle vulnerabilities. This timescale is incompatible with developer expectations for fast feedback.

The "Shift Left" philosophy attempts to address this tension by moving security activities earlier in the development lifecycle. Rather than treating security as a final validation step before release, Shift Left integrates security considerations throughout development. Threat modeling occurs during design. Secure coding practices are enforced during implementation. Security tests run alongside functional tests in CI pipelines. The goal is catching security issues when they are cheapest to fix—during development rather than after deployment.

Implementing Shift Left for fuzz testing presents particular challenges. Traditional fuzzing requires substantial setup effort: writing harnesses that expose target functions, creating seed corpora of valid inputs, configuring fuzzing engines with appropriate parameters. This setup effort must be amortized across many fuzzing executions to be cost-effective. CI pipelines run thousands of times daily; if each run required manual fuzzer configuration, the approach would be infeasible. Automation is essential, but automating fuzz driver generation has historically required either sophisticated program analysis (expensive to implement, fragile in practice) or extensive manual effort (does not scale).

The infrastructure complexity of enterprise CI/CD environments compounds these challenges. Large organizations deploy CI systems across hybrid cloud architectures, balancing computational demands against security requirements. Build runners may execute in public cloud environments for scalability, while sensitive operations occur in private data centers or air-gapped networks. Network policies restrict communication between zones, preventing unauthorized data exfiltration but also complicating integration of cloud-based AI services. A seemingly simple task—calling an LLM API from a build runner—may encounter firewalls, proxy servers, and access control systems that defeat naive implementation approaches.

Security policies in automotive organizations reflect the sensitivity of the intellectual property involved. Source code for competitive vehicle features represents enormous investment. Exposure to unauthorized parties could enable competitors, compromise safety validation, or provide attackers with detailed vulnerability information. Air-gapped networks, while operationally burdensome, provide strong guarantees against unauthorized access. Integrating cloud AI services into such environments requires careful architectural decisions that maintain security boundaries while enabling new capabilities.

---

## 1.2 Problem Statement

The automotive industry faces a scalability crisis in security testing. The rate at which new code is produced vastly exceeds the rate at which security experts can analyze it. A large automotive software organization might employ hundreds of developers producing thousands of code changes daily. The security team, even if generously staffed, cannot review each change for potential vulnerabilities. Automated tools help, but static analysis produces false positives requiring human triage, and dynamic analysis requires test harnesses that must be manually created.

Fuzz testing exemplifies this scalability problem. Fuzzing is effective—Google's OSS-Fuzz project has discovered over 10,000 vulnerabilities in open-source software through continuous fuzzing. The technique works. The bottleneck is fuzz driver creation. A fuzz driver is a small program that takes fuzzer-generated inputs and exercises target functionality in ways that might reveal vulnerabilities. Writing effective fuzz drivers requires understanding API contracts, identifying interesting code paths, and constructing inputs that exercise edge cases. This is skilled work, demanding both security expertise and familiarity with the target codebase.

Figure 1.2 depicts the fuzzing process workflow, highlighting the driver generation stage as the critical manual intervention point. The fuzzer engine itself operates automatically, generating inputs, monitoring execution, and tracking coverage. Input selection and mutation follow algorithmic strategies that require no human involvement during execution. But before fuzzing can begin, someone must write the driver. This driver bridges the gap between raw byte streams (what the fuzzer produces) and structured API calls (what the target consumes). For complex APIs with multiple functions, intricate parameter relationships, and stateful interactions, driver development can consume days of expert effort.

![Figure 1.2: The Fuzzing Process](bilder/fuzzing.png)
*Figure 1.2: The Fuzzing Process. The workflow diagram shows the fuzzing loop from input generation through execution monitoring and feedback. The "Driver Generation" stage, highlighted in the center, represents the manual bottleneck that this thesis aims to automate through LLM-based code generation. While the fuzzing engine operates autonomously, driver creation traditionally requires substantial human expertise.*

The mismatch between testing capacity and code production creates a security gap. Code ships with inadequate fuzzing coverage. Vulnerabilities that could have been discovered through systematic fuzzing remain latent until exploited by attackers or discovered through incident response. The security gap is not a hypothetical concern—it manifests in real vulnerabilities affecting production systems.

Legacy code amplifies the problem. Millions of lines of existing code have never been systematically fuzzed. The original developers may have left the organization. Documentation may be sparse or outdated. Understanding the code sufficiently to write fuzz drivers requires reverse engineering effort that cannot be justified for every function in every library. Organizations face an uncomfortable choice: accept that large portions of their codebase will never receive thorough security testing, or find ways to dramatically reduce the cost of fuzz driver creation.

Large Language Models offer a potential solution. These models, trained on vast corpora of source code, can generate syntactically correct code that follows common patterns. Recent research has demonstrated LLM capabilities in various software engineering tasks: code completion, bug detection, test generation, and documentation. The question is whether LLMs can generate fuzz drivers of sufficient quality to be useful—drivers that compile, execute without crashing, and achieve meaningful code coverage of target functionality.

The optimistic view holds that LLMs can democratize fuzz driver creation. Rather than requiring security experts to manually craft each driver, engineers could prompt an LLM with target function signatures and receive working drivers in seconds. The pessimistic view notes that fuzz drivers require precise understanding of API semantics that may not be captured in training data, and that LLM-generated code often contains subtle errors that undermine its utility.

This thesis investigates the practical reality between these extremes. The research addresses not merely whether LLMs can generate fuzz drivers in principle, but whether they can do so with sufficient reliability, efficiency, and quality to be deployed in production CI/CD pipelines. The enterprise context matters: solutions must work within organizational constraints including network security policies, cost management requirements, and operational complexity limitations.

---

## 1.3 Research Questions and Objectives

The overarching objective of this thesis is to design, implement, and evaluate an automated AI-driven fuzzing framework that integrates with enterprise CI/CD infrastructure at CARIAD SE. This objective decomposes into specific research questions addressing different aspects of the challenge.

### 1.3.1 Primary Research Questions

**RQ1 (Effectiveness): Can Large Language Models generate fuzz drivers for C++ code that compile successfully, execute without errors, and achieve meaningful code coverage?**

This question addresses the fundamental viability of the approach. A fuzz driver that fails to compile is useless. A driver that compiles but crashes immediately provides no security value. A driver that runs but exercises only trivial code paths fails to justify its existence. Meaningful coverage requires exercising the target code in ways that might reveal vulnerabilities—reaching error handling paths, boundary conditions, and complex interaction sequences.

The question also implicitly asks about consistency. An LLM that occasionally generates excellent drivers but usually produces garbage is less valuable than one that reliably generates adequate drivers. Variance matters for production deployment: operators need predictable behavior to plan testing campaigns and interpret results.

**RQ2 (Optimization): Does LLM scale (parameter count) determine fuzz driver quality, or can smaller models with domain-specific fine-tuning match or exceed larger general-purpose models?**

The "bigger is better" assumption pervades discussions of LLM capabilities. Models with more parameters consistently outperform smaller models on standard benchmarks. The largest models—GPT-4, Claude 3, Gemini Ultra—demonstrate impressive capabilities across diverse tasks. These models are also expensive to run, require substantial computational resources, and often available only through rate-limited APIs.

Fine-tuning offers an alternative path. Rather than relying on the general knowledge encoded in massive models, fine-tuning adapts smaller models to specific domains or tasks. Low-Rank Adaptation (LoRA) enables efficient fine-tuning with reduced computational requirements. A small model, fine-tuned on high-quality fuzz driver examples, might outperform a generic large model that has never seen specialized fuzzing code. This hypothesis, if validated, has significant implications for deployment cost and operational feasibility.

**RQ3 (Feasibility): Can AI-assisted fuzz driver generation be integrated into secure, air-gapped enterprise CI/CD pipelines while meeting performance, cost, and security requirements?**

Academic research often evaluates techniques in isolation, abstracting away deployment concerns. Production deployment demands answers to practical questions. Can the solution run within the time constraints of CI/CD pipelines? Does the cost fit within budget constraints? Can the system operate within network security policies that restrict external communication? Are the operational complexity and maintenance burden acceptable?

This question recognizes that a technically superior solution that cannot be deployed is less valuable than a modest solution that works in practice. The research investigates not merely algorithmic performance but system-level feasibility.

### 1.3.2 Secondary Research Questions

Beyond the primary questions, several secondary inquiries inform the research:

**SQ1: What characteristics of target code influence LLM fuzz driver generation success?**

Some code may be easier to fuzz than other code. Simple APIs with well-documented interfaces might yield better LLM-generated drivers than complex APIs with implicit state requirements. Understanding these characteristics helps identify where AI-assisted fuzzing delivers the most value and where human expertise remains essential.

**SQ2: How do different prompt engineering strategies affect generation quality?**

LLM outputs depend heavily on input prompts. The same model given different prompts produces dramatically different results. Systematic investigation of prompt structures, context provision, and instruction formatting may reveal strategies that consistently improve driver quality.

**SQ3: What are the economic trade-offs between different LLM deployment options?**

Cloud-hosted models offer convenience but incur per-request costs. Self-hosted models require infrastructure investment but avoid usage-based pricing. Fine-tuned models demand training investment but may reduce inference costs. Understanding these trade-offs enables informed decisions about deployment architecture.

**SQ4: What are the failure modes of LLM-generated fuzz drivers, and how can they be detected or mitigated?**

LLMs fail in characteristic ways. Understanding these failure patterns enables development of validation mechanisms that catch problematic drivers before they waste fuzzing resources. Post-generation filtering and refinement may improve the effective success rate.

---

## 1.4 Scope and Contributions

This thesis focuses specifically on C++ code running on automotive ECU platforms. C++ dominates safety-critical automotive software due to its combination of performance, determinism, and mature tooling. While the techniques investigated may apply to other languages, the experimental evaluation concentrates on C++ targets representative of automotive workloads.

The fuzzing approach examined is coverage-guided greybox fuzzing, specifically using libFuzzer as the fuzzing engine. LibFuzzer integrates with LLVM-based toolchains common in automotive development and provides instrumentation-based coverage feedback that guides input generation toward unexplored code paths. Other fuzzing approaches—blackbox fuzzing, symbolic execution, concolic testing—fall outside the scope of this investigation.

The enterprise environment is CARIAD SE's development infrastructure, including Azure DevOps for CI/CD orchestration, Azure cloud services for compute resources, and corporate network policies governing data flows and system access. Results may not generalize directly to organizations with different infrastructure, though the architectural patterns identified should transfer.

The thesis makes three primary contributions:

**Contribution 1: Comprehensive LLM Benchmark for Fuzz Driver Generation**

We evaluated 14 Large Language Models spanning parameter counts from 1.5 billion to 46 billion, including both general-purpose code generation models and specialized variants. This benchmark provides empirical data challenging conventional assumptions about model scale and capability. The finding that massive models (Codellama-34b, Codestral-22b) achieved 0% compilation success while smaller fine-tuned models reached 45% coverage contradicts the "bigger is better" narrative prevalent in LLM discourse. The benchmark methodology, evaluation metrics, and detailed results constitute a contribution to the research community investigating AI-assisted security testing.

**Contribution 2: LoRA Fine-Tuning Optimization Strategy**

We developed and validated an approach using Low-Rank Adaptation to fine-tune smaller models on curated fuzz driver examples. The fine-tuned Qwen-1.5b model demonstrated 33% faster inference and 55% lower operational cost compared to larger general-purpose alternatives while achieving superior coverage results. This contribution provides a practical path for organizations seeking to deploy AI-assisted fuzzing without the computational demands and API costs of frontier models.

**Contribution 3: Enterprise Integration Architecture**

We designed and implemented a solution for integrating cloud-hosted AI services with air-gapped CI/CD runners, overcoming network security constraints that initially appeared insurmountable. The architecture employs self-hosted runners, Azure Private Link, and careful network configuration to maintain security boundaries while enabling AI service access. This contribution addresses a practical barrier—the "Azure impasse"—that would otherwise prevent production deployment in security-conscious organizations.

Together, these contributions advance both the scientific understanding of LLM capabilities for security testing and the practical engineering of AI-integrated CI/CD systems.

---

## 1.5 Thesis Organization

The remainder of this thesis is organized as follows.

**Chapter 2: Literature Review** surveys the relevant research landscape. The chapter examines traditional AI approaches in software testing, traces the evolution of Large Language Models and their application to code generation, reviews fuzzing techniques from foundational methods through contemporary AI-guided approaches, and analyzes prior work on CI/CD pipeline security integration. The chapter identifies research gaps that this thesis addresses and positions the work within the broader academic context.

**Chapter 3: Methodology and Framework** presents the research philosophy and approach guiding this investigation. The chapter introduces the conceptual framework for AI-driven white-box fuzzing, describes the technical architecture design, and outlines the three-phase research strategy: local LLM evaluation, model optimization with LoRA, and enterprise CI/CD integration. The evaluation methodology, including metrics and validation approaches, is specified in detail.

**Chapter 4: Implementation** details the concrete realization of the framework. The chapter covers toolchain selection and rationale, describes the experimental setup for each research phase, and documents the architectural challenges encountered during enterprise integration along with their solutions. Implementation decisions are justified with reference to requirements and constraints.

**Chapter 5: Experimental Results** presents the empirical findings. The chapter reports LLM fuzz driver generation results across the evaluated models, analyzes performance variations across different target repositories, documents model optimization outcomes, and provides economic analysis of resource utilization and cost implications.

**Chapter 6: Discussion** interprets the results in relation to the research questions. The chapter examines what the data reveals about LLM capabilities, addresses unexpected findings regarding model architecture and performance, acknowledges limitations and constraints affecting the conclusions, and explores implications for automotive industry practice and enterprise CI/CD integration.

**Chapter 7: Conclusion and Future Work** synthesizes the contributions, assesses the degree to which research objectives were achieved, and identifies directions for future investigation. The chapter offers recommendations for practitioners considering AI-assisted security testing and reflects on the broader significance of the findings.

---

*This introduction has established the context, motivation, and objectives for investigating AI-assisted fuzz driver generation in automotive CI/CD pipelines. The automotive industry's software complexity, security requirements, and development velocity demands create conditions where traditional manual approaches to security testing are inadequate. Large Language Models offer potential for automating the bottleneck of fuzz driver creation, but practical deployment requires addressing questions of effectiveness, optimization, and enterprise integration. The following chapters develop the theoretical grounding, methodological approach, implementation details, and empirical evidence necessary to answer these questions.*
