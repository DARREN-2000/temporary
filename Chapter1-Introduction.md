# Chapter 1: Introduction

The automobile industry has reached a turning point. What started as a mechanical engineering discipline, focused on combustion engines, transmissions, and chassis dynamics, has become something fundamentally different. Today, lines of code determine vehicle behavior as much as physical components do. A modern premium vehicle runs on over 100 million lines of source code, distributed across more than 100 Electronic Control Units (ECUs). These ECUs handle everything from powertrain management to advanced driver assistance. This shift did not happen gradually. It represents a complete restructuring of how vehicles are conceived, built, and maintained.

This thesis examines whether Large Language Models (LLMs) can automate fuzz driver generation within Continuous Integration, Continuous Deployment, and Continuous Testing pipelines. The research took place at CARIAD SE, the Volkswagen Group's automotive software company, where we encountered firsthand how development constraints at enterprise scale clash with the promises of security testing assisted by AI.



## 1.1 Background and Motivation

Three pressures are converging in the automotive industry: software complexity is exploding, cybersecurity requirements keep escalating, and development cycles must move faster than ever. Any one of these would be challenging on its own. Together, they make traditional security approaches insufficient.

### 1.1.1 Automotive Software Evolution and Complexity

Twenty years ago, vehicle electronics were straightforward. The engine control unit handled fuel injection and ignition timing. The ABS prevented wheel lockup during hard braking. The airbag controller monitored crash sensors and deployed restraints. These systems mostly operated in isolation, connected by wiring between individual points and proprietary protocols. A skilled engineer could understand the complete electronic architecture within weeks. That era is gone.

Today's automobile is better understood as a distributed computing platform that happens to have wheels. The concept of the Software Defined Vehicle has shifted competitive differentiation from mechanical systems to software capabilities. Tesla showed this most clearly: the same physical hardware receives new features, better performance, and improved safety through updates delivered over the air. Traditional automakers watched their customers receive Autopilot improvements, new entertainment apps, and even acceleration upgrades. All of this was delivered as software to vehicles sitting in driveways. The message was clear. Software now defines the vehicle experience.

The resulting complexity is staggering. The 100 million lines of code figure, often cited in industry reports, actually understates the problem because it treats all code as equivalent. Automotive software spans multiple programming paradigms, varying levels of safety criticality, and different execution environments. Code that is safety critical on AUTOSAR Classic platforms follows entirely different development practices than infotainment apps running on Linux. Control loops running in real time at microsecond intervals coexist with services connected to the cloud managing terabytes of driving data. The heterogeneity makes everything harder.

Consider a modern Level 2 or higher automated driving system. Perception algorithms process camera, radar, and lidar data, fusing these streams into a coherent model of the environment. Planning modules compute safe trajectories through traffic. Control systems translate trajectories into steering, throttle, and brake commands for actuators at lower levels. Each layer draws on different technical disciplines: computer vision, machine learning, control theory, systems that operate in real time, and functional safety engineering. Integration alone consumes enormous effort.

Legacy code adds to these difficulties. Much of the software in production vehicles was written years or decades ago. Code from previous vehicle generations gets carried forward, often with minimal documentation and no institutional knowledge about design decisions. The original developers have moved on. Specifications have been lost. The code works, mostly, but understanding why requires careful analysis through accumulated modifications. Writing security tests for such code demands comprehension of behavior that was never formally specified.

Automotive software also exhibits extreme longevity requirements. A vehicle sold today will remain on roads for 15 to 20 years. The software must remain functional, secure, and updatable throughout this period. Contrast this with smartphone applications, where a support window of three years is considered generous. Automotive software must anticipate attack vectors that do not yet exist, defend against exploitation techniques that have not been invented, and remain compatible with infrastructure that will evolve unpredictably. The temporal dimension of automotive software security extends far beyond typical enterprise software considerations.

The supply chain structure further amplifies complexity. A single vehicle may incorporate software components from dozens of Tier 1 suppliers, each with their own development processes, quality standards, and security practices. The Original Equipment Manufacturer integrates these components into a coherent system, but visibility into supplier codebases is often limited. Security vulnerabilities may lurk in libraries from third parties, hidden behind proprietary interfaces and contractual barriers that impede thorough analysis. The attack surface extends across organizational boundaries that resist traditional security governance approaches.

### 1.1.2 Security Challenges in Modern Systems

Automotive cybersecurity differs fundamentally from traditional IT security. When a web server gets compromised, the consequences, such as data theft, service disruption, or financial loss, are serious but bounded. When a vehicle control system is compromised, people can be injured or killed. This is not exaggeration. Researchers have demonstrated remote exploitation of vehicle systems, including scenarios where attackers could disable brakes or seize steering control at highway speeds. The Miller and Valasek attack on a Jeep Cherokee in 2015 forced Chrysler to recall 1.4 million vehicles. An infotainment system connected via cellular networks had provided a pathway to vehicle networks that were critical for safety. Software vulnerability became physical danger.

Regulators have responded. UNECE Regulation No. 155, effective for new vehicle types since July 2022, requires cybersecurity management systems for manufacturers. ISO/SAE 21434:2021 sets cybersecurity engineering requirements throughout the vehicle lifecycle. These standards require systematic identification, assessment, and mitigation of cybersecurity risks. They require evidence that security was designed into systems, not added afterward. Compliance is mandatory. Vehicles failing these requirements cannot be sold in regulated markets.

The standards impose particular obligations regarding security testing. Organizations must demonstrate that they have verified the effectiveness of cybersecurity controls through appropriate testing methods. Fuzz testing, which involves the automated generation of malformed or unexpected inputs to discover software vulnerabilities, is explicitly recognized as a relevant technique. The challenge lies in scaling fuzz testing to match the scope of automotive software. Writing effective fuzz drivers requires deep understanding of API semantics, valid input ranges, and expected behaviors. A security expert might spend days developing a single fuzz driver for a complex library. Multiply this effort across thousands of components, and the impossibility of manual approaches becomes clear.

The economics of automotive cybersecurity create additional pressure. Security testing competes for resources with feature development, performance optimization, and regulatory compliance activities. Development teams face constant pressure to deliver new functionality on tight timelines. Security, often seen as a cost center rather than a value creator, frequently receives inadequate attention until problems become undeniable. Discovering vulnerabilities after release is expensive. Recalls cost millions, reputation damage lingers for years, and regulatory penalties can be substantial. Yet the organizational incentives often favor shipping quickly over shipping securely.

The threat landscape continues to evolve. Actors sponsored by nation states have shown interest in automotive systems as both espionage targets and potential weapons. Criminal organizations have developed vehicle theft techniques using keyless entry vulnerabilities. Researchers continuously find new attack vectors, from CAN bus injection to charging infrastructure compromise. Each connected feature, including telematics, updates delivered over the air, and communication between vehicles and everything around them, expands the attack surface. Security is not a problem that can be solved once; it requires continuous attention against an adaptive adversary.

The mandate for zero tolerance regarding safety that governs automotive development creates particular challenges for security testing. Systems that are critical for safety undergo exhaustive verification against precisely defined requirements. Every possible failure mode must be analyzed. Mitigations must be demonstrated through testing. This approach works well for safety engineering, where the hazards are known and the physics are deterministic. Security engineering faces a different problem: the adversary is intelligent, creative, and unconstrained by specifications. Attackers do not limit themselves to documented interfaces. They probe for implementation weaknesses, logic errors, and overlooked edge cases. Testing for security requires exploring the unexpected, not merely verifying the expected.

### 1.1.3 CI/CD/CT Pipeline Context

Modern software development has embraced Continuous Integration, Continuous Deployment, and Continuous Testing as foundational practices for managing complexity while maintaining development velocity. The philosophy is straightforward: integrate code changes frequently, test them automatically, and deploy validated changes rapidly. Small, incremental changes are easier to understand, review, and debug than large, infrequent releases. Automated testing catches regressions before they propagate. Fast feedback loops enable developers to address problems while context remains fresh.

The automotive industry has adopted these principles, adapting them to constraints specific to the domain. The continuous deployment aspect requires careful interpretation. One does not casually push untested code to vehicles traveling at highway speeds. Automotive pipelines for continuous integration typically feed into more formal release processes where human review and additional validation occur before software reaches production vehicles. Nevertheless, the core principles apply: automate what can be automated, test early and often, maintain a continuously releasable codebase.

![Figure 1.1: The Automotive CI/CD Pipeline](bilder/cicd.png)
*Figure 1.1: The Automotive CI/CD Pipeline. The diagram illustrates the sequential stages from code commit through deployment, highlighting the testing bottleneck where security validation must occur within strict time constraints. Traditional manual security testing cannot scale to meet the velocity demands of modern development practices.*

Figure 1.1 shows a representative pipeline architecture used in automotive development. The pipeline begins with developers committing code changes to version control repositories. Automated build processes compile the code, generating executable artifacts for target platforms. Static analysis tools examine the source code for common defects, coding standard violations, and potential security weaknesses. Unit tests verify behavior at the component level. Integration tests confirm that components interact correctly. The pipeline may include testing with hardware in the loop, where software executes on actual ECU hardware connected to simulated vehicle systems. Each stage acts as a quality gate. Failures block progression until issues are resolved.

The temporal constraints on pipelines for continuous integration create tension with thorough security testing. A pipeline that is well optimized might complete build and basic testing within 15 to 30 minutes. Developers expect rapid feedback. Waiting hours for test results disrupts workflow and encourages practices that circumvent the pipeline entirely. Security testing, particularly dynamic analysis techniques like fuzzing, often requires extended execution to achieve meaningful coverage. A fuzzing campaign might run for hours or days to discover subtle vulnerabilities. This timescale is incompatible with developer expectations for fast feedback.

The "Shift Left" philosophy attempts to address this tension by moving security activities earlier in the development lifecycle. Rather than treating security as a final validation step before release, Shift Left integrates security considerations throughout development. Threat modeling occurs during design. Secure coding practices are enforced during implementation. Security tests run alongside functional tests in CI pipelines. The goal is catching security issues when they are cheapest to fix, during development rather than after deployment.

Implementing Shift Left for fuzz testing presents particular challenges. Traditional fuzzing requires substantial setup effort: writing harnesses that expose target functions, creating seed corpora of valid inputs, configuring fuzzing engines with appropriate parameters. This setup effort must be spread across many fuzzing executions to be effective in terms of cost. Pipelines for continuous integration run thousands of times daily. If each run required manual fuzzer configuration, the approach would be impractical. Automation is essential, but automating fuzz driver generation has historically required either sophisticated program analysis, which is expensive to implement and fragile in practice, or extensive manual effort, which does not scale.

The infrastructure complexity of enterprise environments for continuous integration adds to these challenges. Large organizations deploy systems for continuous integration across hybrid cloud architectures, balancing computational demands against security requirements. Build runners may execute in public cloud environments for scalability, while sensitive operations occur in private data centers or networks that are isolated from external connections. Network policies restrict communication between zones, preventing unauthorized data exfiltration but also making it harder to integrate AI services hosted in the cloud. A simple task like calling an LLM API from a build runner may encounter firewalls, proxy servers, and access control systems that block straightforward implementation approaches.

Security policies in automotive organizations reflect the sensitivity of the intellectual property involved. Source code for competitive vehicle features represents enormous investment. Exposure to unauthorized parties could enable competitors, compromise safety validation, or provide attackers with detailed vulnerability information. Networks isolated from external connections, while operationally burdensome, provide strong guarantees against unauthorized access. Integrating AI services hosted in the cloud into such environments requires careful architectural decisions that maintain security boundaries while enabling new capabilities.



## 1.2 Problem Statement

The automotive industry confronts a scalability crisis in security testing. Code production vastly outpaces security analysis capacity. A large automotive software organization might have hundreds of developers producing thousands of changes daily. Even security teams that are well staffed cannot review each change for vulnerabilities. Automated tools help, but static analysis generates false positives needing human triage, and dynamic analysis requires test harnesses that must be created manually.

Fuzz testing shows this scalability problem well. Fuzzing works. The OSS Fuzz project from Google has found over 10,000 vulnerabilities in software that is open source through continuous fuzzing. The technique is proven. The bottleneck is fuzz driver creation. A fuzz driver is a small program that takes inputs generated by the fuzzer and exercises target functionality in ways that might expose vulnerabilities. Writing effective drivers requires understanding API contracts, identifying interesting code paths, and constructing inputs that exercise edge cases. This is skilled work, requiring both security expertise and familiarity with the target codebase.

![Figure 1.2: The Fuzzing Process](bilder/fuzzing.png)
*Figure 1.2: The Fuzzing Process. The workflow diagram shows the fuzzing loop from input generation through execution monitoring and feedback. The stage labeled Driver Generation, highlighted in the center, represents the manual bottleneck that this thesis aims to automate through code generation based on LLMs. While the fuzzing engine operates autonomously, driver creation traditionally requires substantial human expertise.*

Figure 1.2 depicts the fuzzing process workflow, highlighting the driver generation stage as the critical manual intervention point. The fuzzer engine itself operates automatically, generating inputs, monitoring execution, and tracking coverage. Input selection and mutation follow algorithmic strategies that require no human involvement during execution. But before fuzzing can begin, someone must write the driver. This driver bridges the gap between raw byte streams, which is what the fuzzer produces, and structured API calls, which is what the target consumes. For complex APIs with multiple functions, intricate parameter relationships, and stateful interactions, driver development can consume days of expert effort.

The mismatch between testing capacity and code production creates a security gap. Code ships with inadequate fuzzing coverage. Vulnerabilities that could have been discovered through systematic fuzzing remain hidden until exploited by attackers or discovered through incident response. The security gap is not a theoretical concern. It shows up in real vulnerabilities affecting production systems.

Legacy code amplifies the problem. Millions of lines of existing code have never been systematically fuzzed. The original developers may have left the organization. Documentation may be sparse or outdated. Understanding the code sufficiently to write fuzz drivers requires reverse engineering effort that cannot be justified for every function in every library. Organizations face an uncomfortable choice: accept that large portions of their codebase will never receive thorough security testing, or find ways to dramatically reduce the cost of fuzz driver creation.

Large Language Models offer a potential solution. These models, trained on vast amounts of source code, can generate syntactically correct code that follows common patterns. Recent research has shown LLM capabilities in various software engineering tasks: code completion, bug detection, test generation, and documentation. The question is whether LLMs can generate fuzz drivers of sufficient quality to be useful. Specifically, can they generate drivers that compile, execute without crashing, and achieve meaningful code coverage of target functionality?

The optimistic view holds that LLMs can democratize fuzz driver creation. Rather than requiring security experts to manually craft each driver, engineers could prompt an LLM with target function signatures and receive working drivers in seconds. The pessimistic view notes that fuzz drivers require precise understanding of API semantics that may not be captured in training data, and that code generated by LLMs often contains subtle errors that undermine its utility.

This thesis investigates what happens in practice. The research examines not just whether LLMs can generate fuzz drivers in principle, but whether they can do so reliably enough, efficiently enough, and with sufficient quality to deploy in production pipelines for continuous integration. The enterprise context matters here. Solutions must work within real organizational constraints, including network security policies, cost management, and operational complexity.



## 1.3 Research Questions and Objectives

The main objective of this thesis is to design, implement, and evaluate an automated framework for fuzzing driven by AI that integrates with enterprise infrastructure for continuous integration at CARIAD SE. This objective breaks down into specific research questions covering different aspects of the challenge.

### 1.3.1 Primary Research Questions

**RQ1 (Effectiveness): Can Large Language Models generate fuzz drivers for C++ code that compile successfully, execute without errors, and achieve meaningful code coverage?**

This question looks at the fundamental viability of the approach. A fuzz driver that fails to compile is useless. One that compiles but crashes immediately provides no security value. A driver that runs but only exercises trivial code paths does not justify its existence. Meaningful coverage means exercising target code in ways that might expose vulnerabilities, such as reaching error handling paths, boundary conditions, and complex interaction sequences.

Consistency matters too. An LLM that sometimes produces excellent drivers but usually generates garbage is less useful than one that reliably produces adequate drivers. Variance matters for production: operators need predictable behavior to plan testing campaigns and interpret results.

**RQ2 (Optimization): Does LLM scale, meaning parameter count, determine fuzz driver quality, or can smaller models with tuning specific to the domain match or exceed larger models that are general purpose?**

The assumption that "bigger is better" is common in LLM discussions. Models with more parameters consistently outperform smaller ones on standard benchmarks. The largest models, such as GPT 4, Claude 3, and Gemini Ultra, show impressive capabilities across diverse tasks. They are also expensive, require substantial compute, and are often available only through APIs with rate limits.

Tuning for specific tasks offers an alternative path. Rather than relying on the general knowledge encoded in massive models, this approach adapts smaller models to specific domains or tasks. Low Rank Adaptation enables efficient tuning with reduced computational requirements. A small model, tuned on examples of high quality fuzz drivers, might outperform a generic large model that has never seen specialized fuzzing code. This hypothesis, if validated, has significant implications for deployment cost and operational feasibility.

**RQ3 (Feasibility): Can fuzz driver generation assisted by AI be integrated into secure enterprise pipelines for continuous integration that are isolated from external networks while meeting performance, cost, and security requirements?**

Academic research often evaluates techniques in isolation, abstracting away deployment concerns. Production deployment demands practical answers. Can the solution run within time constraints of continuous integration? Does the cost fit budget limits? Can the system operate under network security policies that restrict external communication? Is the operational complexity acceptable?

This question acknowledges that a technically superior solution that cannot be deployed is less valuable than a modest solution that actually works. The research examines not just algorithmic performance but feasibility at the system level.

### 1.3.2 Secondary Research Questions

Beyond the primary questions, several secondary inquiries guide the research:

**SQ1: What characteristics of target code influence LLM fuzz driver generation success?**

Some code may be easier to fuzz than other code. Simple APIs with interfaces that are well documented might yield better drivers generated by LLMs than complex APIs with implicit state requirements. Understanding these characteristics helps identify where fuzzing assisted by AI delivers the most value and where human expertise remains essential.

**SQ2: How do different prompt engineering strategies affect generation quality?**

LLM outputs depend heavily on input prompts. The same model given different prompts produces dramatically different results. Systematic investigation of prompt structures, context provision, and instruction formatting may reveal strategies that consistently improve driver quality.

**SQ3: What are the economic tradeoffs between different LLM deployment options?**

Models hosted in the cloud offer convenience but incur costs per request. Models that are self hosted require infrastructure investment but avoid pricing based on usage. Models that have been tuned demand training investment but may reduce inference costs. Understanding these tradeoffs enables informed decisions about deployment architecture.

**SQ4: What are the failure modes of fuzz drivers generated by LLMs, and how can they be detected or mitigated?**

LLMs fail in characteristic ways. Understanding these failure patterns enables development of validation mechanisms that catch problematic drivers before they waste fuzzing resources. Filtering and refinement after generation may improve the effective success rate.



## 1.4 Scope and Contributions

This thesis focuses specifically on C++ code running on automotive ECU platforms. C++ dominates automotive software that is critical for safety due to its combination of performance, determinism, and mature tooling. While the techniques investigated may apply to other languages, the experimental evaluation concentrates on C++ targets representative of automotive workloads.

The fuzzing approach examined is coverage guided greybox fuzzing, specifically using libFuzzer as the fuzzing engine. LibFuzzer integrates with toolchains based on LLVM that are common in automotive development and provides coverage feedback based on instrumentation that guides input generation toward unexplored code paths. Other fuzzing approaches, such as blackbox fuzzing, symbolic execution, and concolic testing, fall outside the scope of this investigation.

The enterprise environment is the development infrastructure at CARIAD SE, including Azure DevOps for orchestration of continuous integration, Azure cloud services for compute resources, and corporate network policies governing data flows and system access. Results may not generalize directly to organizations with different infrastructure, though the architectural patterns identified should transfer.

The thesis makes three primary contributions:

**Contribution 1: Comprehensive LLM Benchmark for Fuzz Driver Generation**

We evaluated 14 Large Language Models with parameter counts ranging from 1.5 billion to 46 billion, including both models for code generation that are general purpose and specialized variants. This benchmark provides data that challenges assumptions about model scale and capability. The finding that massive models like Codellama 34b and Codestral 22b achieved 0% compilation success while smaller models that had been tuned reached 45% coverage goes against the "bigger is better" assumption common in LLM discussions. The benchmark methodology, evaluation metrics, and detailed results contribute to ongoing research on security testing assisted by AI.

**Contribution 2: Strategy for Optimization through Low Rank Adaptation**

We developed and validated an approach for tuning using Low Rank Adaptation on curated fuzz driver examples. The tuned Qwen 1.5b model achieved 33% faster inference and 55% lower operational cost compared to larger alternatives that are general purpose, while delivering superior coverage results. This provides a practical path for organizations wanting to deploy fuzzing assisted by AI without the computational demands and API costs of frontier models.

**Contribution 3: Enterprise Integration Architecture**

We designed and implemented a solution for connecting AI services hosted in the cloud with runners for continuous integration that are isolated from external networks, overcoming network security constraints that initially seemed impossible to solve. The architecture uses runners that are self hosted, Azure Private Link, and careful network configuration to maintain security boundaries while enabling AI service access. This contribution addresses a practical barrier, which we refer to as the "Azure impasse," that would otherwise block production deployment in organizations that are security conscious.

These contributions together advance both scientific understanding of LLM capabilities for security testing and practical engineering of systems for continuous integration that are integrated with AI.



## 1.5 Thesis Organization

The rest of this thesis is structured as follows.

**Chapter 2: Literature Review** surveys relevant research. It examines traditional AI approaches in software testing, traces the evolution of Large Language Models and their application to code generation, reviews fuzzing techniques from foundational methods through approaches guided by AI, and looks at prior work on security integration in pipelines for continuous integration. The chapter identifies research gaps that this thesis addresses and positions the work within the broader academic context.

**Chapter 3: Methodology and Framework** presents the research philosophy and approach. It introduces the conceptual framework for fuzzing driven by AI that uses white box techniques, describes the technical architecture design, and outlines the research strategy in three phases: local LLM evaluation, model optimization with Low Rank Adaptation, and enterprise integration of continuous integration. Evaluation methodology, including metrics and validation approaches, is specified in detail.

**Chapter 4: Implementation** covers the concrete realization of the framework. It addresses toolchain selection and rationale, describes the experimental setup for each research phase, and documents architectural challenges encountered during enterprise integration along with their solutions. Implementation decisions are justified with reference to requirements and constraints.

**Chapter 5: Experimental Results** presents the empirical findings: results of fuzz driver generation by LLMs across evaluated models, performance variations across different target repositories, model optimization outcomes, and economic analysis of resource utilization and costs.

**Chapter 6: Discussion** interprets results in relation to the research questions. It examines what the data shows about LLM capabilities, addresses unexpected findings regarding model architecture and performance, acknowledges limitations affecting conclusions, and explores implications for automotive industry practice and enterprise integration of continuous integration.

**Chapter 7: Conclusion and Future Work** synthesizes contributions, assesses achievement of research objectives, and identifies directions for future investigation. It offers recommendations for practitioners considering security testing assisted by AI and reflects on the broader significance of the findings.



This introduction has established the context, motivation, and objectives for investigating fuzz driver generation assisted by AI in automotive pipelines for continuous integration. The combination of software complexity, security requirements, and development velocity in the automotive industry creates conditions where traditional manual security testing approaches fall short. Large Language Models offer a potential path to automating the fuzz driver creation bottleneck, but practical deployment requires answering questions about effectiveness, optimization, and enterprise integration. The following chapters develop the theoretical background, methodology, implementation details, and empirical evidence to address these questions.
