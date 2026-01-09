# Chapter 1: Introduction

A modern car runs on code. Not just a little code—we're talking about 100 million lines of C++ spread across 100+ Electronic Control Units, all working together to keep the vehicle safe, connected, and responsive. That number stopped surprising me about six months into this research at CARIAD SE, the Volkswagen Group's software arm. What never stopped being surprising was how little of that code gets proper security testing.

This thesis examines whether Large Language Models can generate fuzz drivers automatically—and whether that automation can actually work inside the locked-down CI/CD pipelines that automotive companies operate. The short answer: yes, but not in the way we expected. Massive 34-billion parameter models failed completely. A tiny 1.5-billion parameter model, fine-tuned with LoRA on good examples, hit 45% code coverage. The infrastructure challenge—getting AI services to talk to air-gapped build runners—turned out to be harder than the AI problem itself.

---

## 1.1 Background and Motivation

Three forces are colliding in automotive software right now. The code is getting more complex every model year. The security requirements keep tightening. And everyone wants faster releases. Something has to give.

### 1.1.1 Automotive Software Evolution and Complexity

Twenty years ago, you could draw the complete electronics architecture of a car on a whiteboard. The engine controller handled fuel injection. The ABS module stopped wheel lockup. The airbag controller watched crash sensors. Each system did one thing, communicated over simple point-to-point wiring, and an experienced engineer could trace any signal path in an afternoon.

That world is gone.

Tesla changed everything. When customers started getting performance upgrades and new Autopilot features through overnight software updates—to cars already parked in their driveways—traditional automakers realized the game had shifted. The competition wasn't about horsepower anymore. It was about software capability. The industry now calls this the "Software-Defined Vehicle" concept, though the term undersells what's actually happening: vehicles have become distributed computing platforms that happen to have wheels.

The 100 million lines of code statistic gets thrown around a lot, but it obscures important details. That code isn't uniform. Safety-critical powertrain controllers run on AUTOSAR Classic with formal verification requirements and real-time deadlines measured in microseconds. The infotainment system runs Linux and talks to cloud services. The ADAS stack fuses camera, radar, and lidar data through neural networks that nobody fully understands. Getting all of this to work together—reliably, every time, for 15 to 20 years of vehicle lifetime—is an integration challenge unlike anything in traditional software engineering.

And then there's the legacy problem. A significant portion of the code in any production vehicle was written years or decades ago, by developers who have long since moved on, with documentation that may or may not exist. The code works. Usually. Understanding *why* it works often requires archaeological excavation through layers of patches and workarounds accumulated over multiple vehicle generations.

The supply chain makes everything worse. A single vehicle might contain software from forty different Tier 1 suppliers. The OEM integrates these components but rarely has full visibility into how they work internally. Security vulnerabilities can hide in third-party libraries, obscured by proprietary interfaces and NDA barriers. You can't test what you can't see.

### 1.1.2 Security Challenges in Modern Systems

When someone hacks a web server, they steal data or disrupt service. Serious, but bounded. When someone hacks a car, people can die.

This isn't theoretical. In 2015, Charlie Miller and Chris Valasek demonstrated remote exploitation of a Jeep Cherokee, gaining control over steering and brakes through the cellular-connected infotainment system. Chrysler recalled 1.4 million vehicles. The attack path ran from an entertainment feature to the CAN bus controlling safety-critical functions. A software vulnerability became a physical safety hazard.

Regulators noticed. UNECE Regulation No. 155, mandatory for new vehicle types since July 2022, requires formal cybersecurity management systems. ISO/SAE 21434:2021 specifies security engineering requirements throughout the vehicle lifecycle. The standards aren't optional—fail to comply, and you can't sell vehicles in regulated markets.

The standards demand security testing, and fuzz testing is explicitly recognized as a valid technique. The problem is scaling it. A security expert might spend three or four days writing a single fuzz driver for a complex C++ library. Do that math across thousands of components. It doesn't work.

The economics don't help either. Security testing competes with feature development for engineering resources. Development teams face constant pressure to ship functionality on aggressive schedules. Security is a cost center; new features drive sales. When deadlines get tight, guess what gets cut?

Meanwhile, the threat landscape keeps evolving. Nation-state actors probe automotive systems. Criminal organizations exploit keyless entry vulnerabilities for theft. Researchers discover new attack vectors—CAN bus injection, charging infrastructure compromise, over-the-air update manipulation. Every connected feature expands the attack surface.

The safety engineering mindset that works well for functional safety—exhaustively verify against known requirements—doesn't transfer directly to security. Safety hazards are predictable; physics doesn't change. Attackers are creative. They don't limit themselves to documented interfaces. They look for implementation mistakes, logic errors, edge cases nobody thought to test.

### 1.1.3 CI/CD/CT Pipeline Context

Modern software development runs on Continuous Integration. Write code, commit it, watch automated tests run, get feedback in minutes. Small changes, fast feedback, catch problems early. The automotive industry has adapted these practices, though "continuous deployment" means something different when you're shipping to vehicles traveling at highway speeds. There are still formal release gates, human reviews, and extensive validation before code reaches production. But the underlying philosophy—automate testing, integrate frequently, keep the codebase releasable—has taken hold.

Figure 1.1 shows what a typical automotive CI/CD pipeline looks like. Developers commit code. Automated builds compile it. Static analyzers check for common mistakes. Unit tests run. Integration tests verify component interactions. Hardware-in-the-loop testing exercises the software on actual ECU hardware. Each stage is a quality gate—failures block progress until someone fixes them.

\begin{figure}[htbp]
\centering
\includegraphics[width=\textwidth]{bilder/cicd.png}
\caption{The Automotive CI/CD Pipeline. Developers commit code changes that flow through automated build, analysis, and test stages. The bottleneck for security testing occurs at the integration test stage, where fuzz testing would ideally run but traditionally requires manual driver creation that doesn't fit the 15-30 minute feedback window developers expect.}
\label{fig:cicd-pipeline}
\end{figure}

The time constraints create real tension. A well-optimized pipeline finishes in 15 to 30 minutes. Developers expect fast feedback. But meaningful fuzzing campaigns run for hours, sometimes days, to discover subtle vulnerabilities. You can't wait that long in a CI loop.

The "Shift Left" philosophy tries to address this by moving security earlier in development. Test during implementation, not after. Catch issues when they're cheap to fix. The idea is sound. The execution is hard. Fuzz testing requires writing harnesses that expose target functions, creating input corpora, configuring fuzzer parameters. That setup work doesn't amortize well across the thousands of daily pipeline runs in a large organization.

Enterprise infrastructure adds another layer of complexity. CARIAD's CI systems span Azure cloud services and on-premises data centers. Network policies restrict what can talk to what. Build runners in certain zones can't reach external endpoints—by design, to prevent data exfiltration. A seemingly simple task—calling an LLM API during a build—runs into firewalls, proxies, and access control systems that block it cold.

Air-gapped networks exist for good reasons. Automotive source code represents billions of euros in R&D investment. Leaking it to competitors or exposing it to attackers would be catastrophic. But those same security policies that protect intellectual property also prevent straightforward integration of cloud-hosted AI services. We hit this wall hard during the research, and solving it consumed more effort than the actual AI work.

---

## 1.2 Problem Statement

Here's the core problem: security experts cannot write fuzz drivers fast enough. Manual driver creation is a bottleneck.

The rate of code production in a large automotive software organization vastly exceeds any realistic security testing capacity. Hundreds of developers producing thousands of commits daily. The security team, however well-staffed, cannot manually review everything. Static analysis helps but generates false positives requiring human triage. Dynamic analysis—including fuzzing—requires test harnesses that must be written by hand.

Fuzzing works. Google's OSS-Fuzz project has found over 10,000 vulnerabilities in open-source software through continuous fuzzing. The technique scales beautifully once set up. But setup is the problem. A fuzz driver is a small program that takes random byte streams from the fuzzer and turns them into structured API calls against the target code. Writing one requires understanding the API's contracts, identifying interesting code paths, and constructing inputs that exercise edge cases. For a complex library with stateful interactions and intricate parameter relationships, driver development takes days of expert effort.

Figure 1.2 illustrates the fuzzing workflow. The fuzzer engine runs autonomously—mutating inputs, monitoring execution, tracking coverage, adjusting strategy. No human intervention needed during operation. But before any of that starts, someone must write the driver that bridges fuzzer output to target input. That's the manual step. That's what we want to automate.

\begin{figure}[htbp]
\centering
\includegraphics[width=\textwidth]{bilder/fuzzing.png}
\caption{The Fuzzing Process. The workflow shows input generation, target execution, and coverage-guided feedback operating autonomously. The "Driver Generation" stage (center) represents the manual bottleneck—traditionally requiring security expertise to create the bridge between raw fuzzer output and structured API calls.}
\label{fig:fuzzing-process}
\end{figure}

The mismatch between testing capacity and code production creates a security gap. Code ships without adequate fuzzing coverage. Vulnerabilities that systematic testing could have caught remain latent until attackers find them or incident response digs them up. This gap isn't theoretical—it shows up in real vulnerabilities affecting production systems.

Legacy code makes everything harder. Millions of lines that have never been systematically fuzzed. Original developers gone. Documentation incomplete. The reverse engineering effort needed to write proper fuzz drivers can't be justified for every function in every library. Organizations face an uncomfortable tradeoff: accept that large portions of the codebase will never get thorough security testing, or find ways to dramatically reduce the cost of driver creation.

Large Language Models trained on source code can generate syntactically correct programs following common patterns. Recent work shows LLMs performing well on code completion, test generation, bug detection. The question is whether they can generate fuzz drivers good enough to be useful—drivers that compile, run without crashing, and achieve meaningful coverage.

This thesis investigates that question in a production context. Not just "can LLMs generate fuzz drivers?" but "can they do it reliably enough, cheaply enough, and within the constraints of a real automotive CI/CD environment?"

---

## 1.3 Research Questions and Objectives

The goal is straightforward: design, build, and evaluate an AI-driven fuzzing framework that actually works inside CARIAD's enterprise infrastructure.

That goal breaks down into specific questions.

### 1.3.1 Primary Research Questions

**RQ1 (Effectiveness): Can LLMs generate fuzz drivers for C++ that compile, execute, and achieve meaningful code coverage?**

A fuzz driver that doesn't compile is worthless. A driver that compiles but crashes immediately provides nothing. A driver that runs but only exercises trivial paths—calling one function once and exiting—fails to justify its existence. "Meaningful coverage" means reaching the code paths where vulnerabilities hide: error handlers, boundary conditions, complex state interactions.

Consistency matters too. An LLM that sometimes produces excellent drivers but usually outputs garbage is less useful than one that reliably produces adequate drivers. Production deployment needs predictability.

**RQ2 (Optimization): Does model size determine driver quality, or can small fine-tuned models beat large general-purpose ones?**

The default assumption is that bigger models perform better. GPT-4 beats GPT-3.5. The 70B parameter variant outperforms the 7B version. This pattern holds across most benchmarks.

But big models are expensive. They require significant compute, often only available through rate-limited APIs with per-token pricing. If a small model, fine-tuned specifically on fuzz driver examples using Low-Rank Adaptation (LoRA), could match or exceed a general-purpose giant—that changes the economics dramatically.

**RQ3 (Feasibility): Can AI-assisted fuzzing run in a secure, air-gapped CI/CD pipeline while meeting performance and cost constraints?**

Academic papers often evaluate techniques in isolation, ignoring deployment realities. Production deployment demands practical answers. Can the solution run within CI time constraints? Does the cost fit budget limits? Can the system work within network policies that block external communication?

A brilliant solution that can't actually be deployed is worth less than a modest one that works.

### 1.3.2 Secondary Research Questions

**SQ1: What makes target code easier or harder for LLM-driven fuzzing?**

Simple APIs with clear documentation probably yield better LLM-generated drivers than complex stateful interfaces with implicit requirements. Understanding which targets benefit most from automation helps prioritize where to apply it.

**SQ2: How do prompt engineering strategies affect output quality?**

LLM behavior depends heavily on how you ask. Same model, different prompt, different result. Investigating what works—few-shot examples, structured formats, explicit constraints—might reveal consistently effective strategies.

**SQ3: What are the cost tradeoffs between deployment options?**

Cloud APIs charge per token. Self-hosted models need infrastructure. Fine-tuning demands training compute. Which approach makes economic sense depends on usage patterns and organizational constraints.

**SQ4: How do LLM-generated drivers fail, and can we detect bad ones before running them?**

LLMs make characteristic mistakes. If we understand the failure patterns, we can build filters that catch problematic drivers before they waste fuzzing resources.

---

## 1.4 Scope and Contributions

This work focuses on C++ code for automotive ECUs. C++ dominates safety-critical automotive software—performance, determinism, mature tooling. The techniques might apply elsewhere, but the experiments target C++ workloads typical of the domain.

We used libFuzzer for fuzzing—coverage-guided, integrated with LLVM, widely used. Other approaches (blackbox fuzzing, symbolic execution, concolic testing) fall outside scope.

The infrastructure context is CARIAD's environment: Azure DevOps for CI/CD orchestration, Azure cloud for compute, corporate network policies restricting data flows. Results may not transfer directly to organizations with different setups, though the architectural patterns should generalize.

**Contribution 1: LLM Benchmark for Fuzz Driver Generation**

We evaluated 14 models from 1.5B to 46B parameters. The results challenged assumptions. Codellama-34b and Codestral-22b—big, well-regarded code models—achieved 0% compilation success on our targets. A fine-tuned Qwen-1.5b reached 45% coverage. "Bigger is better" isn't always true for specialized tasks.

**Contribution 2: LoRA Fine-Tuning Strategy**

We developed an approach using Low-Rank Adaptation to specialize small models on curated fuzz driver examples. The fine-tuned Qwen-1.5b ran 33% faster with 55% lower cost than larger alternatives—while producing better drivers. This provides a practical path for organizations that can't afford frontier model API costs.

**Contribution 3: Enterprise Integration Architecture**

We solved the "Azure impasse"—getting cloud AI services to work with air-gapped CI runners. The solution uses self-hosted runners, Azure Private Link, and careful network configuration. This contribution addresses a real deployment barrier that would otherwise block production use in security-conscious organizations.

---

## 1.5 Thesis Organization

**Chapter 2: Literature Review** examines existing work—traditional AI in testing, LLM evolution and code generation capabilities, fuzzing techniques from foundations through AI-guided approaches, CI/CD security integration. The chapter identifies gaps this thesis addresses.

**Chapter 3: Methodology and Framework** describes the research approach: the conceptual framework for AI-driven white-box fuzzing, technical architecture, and the three-phase strategy—local LLM evaluation, LoRA optimization, enterprise integration. Evaluation metrics and validation approaches are specified.

**Chapter 4: Implementation** covers concrete realization—toolchain selection, experimental setup for each phase, and the architectural challenges we encountered during enterprise integration. We document what didn't work as well as what did.

**Chapter 5: Experimental Results** presents findings: model performance comparisons, coverage data across target repositories, optimization outcomes, economic analysis.

**Chapter 6: Discussion** interprets results against research questions, examines unexpected findings about model architecture, acknowledges limitations, and explores implications for automotive industry practice.

**Chapter 7: Conclusion and Future Work** synthesizes contributions, assesses objective achievement, identifies future directions, and offers recommendations for practitioners.

---

The following chapters build the evidence needed to answer whether LLMs can make security testing scale in automotive software. The complexity is real. The constraints are real. Whether AI can actually help—that's what the experiments reveal.
