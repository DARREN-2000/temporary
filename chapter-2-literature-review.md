# Chapter 2: Literature Review

## 2.1 Introduction and Scope

The field of automated software testing has undergone substantial transformation over the past two decades. What began with simple rule-based approaches has evolved into sophisticated systems that can learn, adapt, and reason about code structure. This chapter examines the literature that shapes our understanding of AI-enhanced fuzzing—particularly as it applies to C-based automotive software development.

Why does this matter for automotive applications specifically? The answer lies in the intersection of several factors: the safety-critical nature of automotive software, the predominance of C in embedded systems, and the increasing complexity of modern vehicle architectures. A typical modern vehicle contains over 100 million lines of code [1], distributed across dozens of electronic control units. Testing this codebase thoroughly using manual methods alone isn't feasible. Automated approaches become not just convenient but necessary.

This review is organized thematically rather than chronologically. We begin with traditional AI approaches in software testing to establish historical context. Next, we trace the evolution of large language models and their application to code generation tasks. The discussion then moves to fuzzing techniques, starting from foundational methods and progressing to modern AI-guided approaches. We examine the current state-of-the-art in AI-enhanced fuzzing, with particular attention to LLM-based test case generation, neural network-guided fuzzing, and reinforcement learning applications. Safety standards for C development receive dedicated attention given their importance in automotive contexts. Finally, we identify gaps in existing research and position this thesis within the broader literature.

The scope is deliberately constrained. We focus primarily on C-based software due to its prevalence in automotive systems. Other languages—Python, Rust, Java—appear occasionally in the literature we review, but our primary interest remains systems-level code written in C. Similarly, while security testing encompasses many techniques, our emphasis falls on fuzzing and closely related approaches.

One limitation should be acknowledged at the outset. The field moves quickly. Papers published in 2023 and 2024 describe techniques that may already seem outdated by the time this thesis is read. We have attempted to include the most recent relevant work, but gaps are inevitable. The reader should treat this review as a snapshot of a rapidly evolving landscape rather than a definitive survey.


## 2.2 Traditional AI Approaches in Software Testing

Before large language models captured the attention of the software engineering community, researchers explored numerous AI-based approaches to automated testing. Understanding this history provides context for appreciating both the novelty and the limitations of current LLM-based methods.

Genetic algorithms represent one of the earliest successful applications of AI techniques to software testing. Inspired by evolutionary biology, these algorithms maintain populations of test cases that evolve over successive generations through selection, crossover, and mutation operations. McMinn [2] provides a comprehensive survey of search-based software testing that traces developments from the late 1990s through the early 2010s. The key insight was that test case generation could be framed as an optimization problem: maximize code coverage, minimize redundancy, or achieve some other measurable objective. This framing opened the door to applying a wide range of optimization techniques developed in other domains.

Early work focused on generating inputs for numerical functions. Korel's approach [3], published in 1990, used dynamic analysis combined with function minimization to generate test inputs that would execute specific program paths. The technique was effective but computationally expensive—a theme that would recur throughout the history of automated testing. Each iteration required executing the program under test and analyzing the results, limiting the number of test cases that could be explored in practice.

Constraint-based approaches emerged as an alternative paradigm. Rather than searching randomly through input spaces, these methods attempted to reason symbolically about program behavior. King's seminal work on symbolic execution [4] laid theoretical foundations that would later influence tools like KLEE [5] and SAGE [6]. The idea was elegant: instead of executing a program with concrete inputs, execute it with symbolic values and accumulate constraints along each path. Solving these constraints yields concrete inputs guaranteed to exercise specific program paths.

The practical challenges proved substantial. Path explosion—the exponential growth in the number of paths through a program—limited symbolic execution to relatively small programs. A function with 10 sequential if-statements creates 1,024 paths; add loops, and the number becomes infinite. Researchers developed various heuristics to manage this explosion, but complete coverage remained elusive for real-world software.

Machine learning techniques entered the picture in the 2000s. Briand and colleagues [7] explored using classification algorithms to predict fault-prone modules, allowing testers to focus their efforts where problems were most likely. Menzies et al. [8] applied similar ideas using simpler metrics, demonstrating that even basic machine learning could provide practical benefits. These approaches didn't generate tests directly—they guided human testers toward high-risk areas.

Neural networks appeared in testing research during the 2010s, initially in limited roles. Researchers used them to model program behavior [9], predict test outcomes [10], and estimate coverage [11]. The results were promising but not transformative. Neural networks of that era lacked the capacity to understand code in any meaningful sense. They could recognize patterns but couldn't reason about semantics.

Reinforcement learning offered another avenue. Veanes et al. [12] applied model-based reinforcement learning to generate tests for stateful systems. The approach learned which action sequences were most likely to uncover bugs by exploring the state space and observing outcomes. Coverage improved compared to random testing, though the learning process was slow and sample-inefficient.

Looking back, several patterns emerge from this pre-LLM era. First, researchers consistently sought ways to reduce the manual effort involved in testing while maintaining or improving effectiveness. Second, computational cost remained a persistent limitation—techniques that worked in controlled experiments often proved too expensive for industrial-scale software. Third, approaches tended to work well in narrow domains but struggle to generalize. A genetic algorithm tuned for numerical functions might perform poorly on string processing code. Fourth, and perhaps most importantly, none of these techniques could truly understand code in the way humans do. They operated on syntactic or statistical properties rather than semantic meaning.

This last point is crucial for understanding why large language models represent a genuinely different approach. For the first time, we have systems that appear to comprehend code—not just its surface structure, but something closer to its meaning and intent. Whether this comprehension is genuine or merely simulates understanding remains debated, but the practical implications are significant either way.


## 2.3 Large Language Models: Evolution and Code Generation

The emergence of large language models capable of generating and understanding source code represents a watershed moment in software engineering research. To appreciate the current state of LLM-based testing, we must trace how these models developed and what makes them suited—or unsuited—for code-related tasks.

Modern language models trace their lineage to the Transformer architecture introduced by Vaswani et al. [13] in 2017. The key innovation was the attention mechanism, which allowed models to consider arbitrary long-range dependencies in sequences without the sequential processing bottleneck that limited earlier recurrent architectures. Processing an entire input sequence in parallel enabled training on vastly larger datasets than previously possible.

The GPT family of models, developed by OpenAI, demonstrated that scaling Transformer-based language models to billions of parameters produced qualitatively new capabilities. GPT-2 [14], released in 2019, could generate coherent text paragraphs. GPT-3 [15], released in 2020 with 175 billion parameters, exhibited few-shot learning—the ability to perform tasks from only a handful of examples in the prompt. This capability was unexpected. The models weren't explicitly trained for specific tasks; they simply learned to predict the next token in sequences. Yet from this simple objective emerged surprising flexibility.

Code-specific models followed. Codex [16], built on GPT-3 and fine-tuned on publicly available code from GitHub, powered GitHub Copilot and demonstrated that LLMs could generate functional code from natural language descriptions. The model was trained on 159 GB of Python code and showed proficiency in multiple programming languages despite Python being its primary training domain. Chen et al. [16] reported that Codex solved 28.8% of problems from the HumanEval benchmark on the first attempt, rising to 70.2% when allowed 100 attempts.

Several trends characterized the evolution from Codex to current models. Model size continued to increase—GPT-4 [17], released in 2023, is believed to contain well over a trillion parameters, though OpenAI has not disclosed exact figures. Training data expanded to include more diverse code sources and programming languages. Fine-tuning techniques improved, with methods like RLHF (reinforcement learning from human feedback) helping align model outputs with human preferences.

Open-source alternatives emerged as counterweights to proprietary models. Meta's LLaMA [18] and its derivatives (Alpaca, Vicuna) demonstrated that competitive performance was achievable with smaller, publicly available models. Code-specific open-source models like CodeLLaMA [19], StarCoder [20], and WizardCoder [21] targeted programming tasks specifically. These models matter for several reasons. First, they can be run locally without API costs. Second, they enable research that proprietary API access precludes. Third, they raise the floor—organizations unwilling or unable to use cloud-based AI services still have access to capable code-generation tools.

How well do these models actually understand code? The question admits no simple answer. On one hand, LLMs can perform tasks that seem to require understanding: explaining what code does, identifying bugs, suggesting refactorings, translating between programming languages. On the other hand, they also make errors that betray deep confusion—generating syntactically correct code that makes no semantic sense, or confidently proposing fixes that introduce new bugs [22].

Research has probed these capabilities systematically. Hendrycks et al. [23] introduced the APPS benchmark, containing 10,000 coding problems with test cases. GPT-3 solved only 3-6% of problems, suggesting substantial room for improvement. Subsequent models fared better—GPT-4 achieves roughly 70% on the simpler problems—but difficult problems remain challenging [17]. Austin et al. [24] found that performance varied significantly across problem types, with models struggling particularly on problems requiring non-obvious algorithms or extensive multi-step reasoning.

For C specifically, the landscape is less thoroughly mapped. Much LLM research focuses on Python due to its prevalence in training data. C presents additional challenges: pointer arithmetic, memory management, undefined behavior, preprocessor macros. Studies specifically examining LLM performance on C are less common but suggest both promise and limitations. Deng et al. [25] found that models could generate syntactically valid C code in most cases but frequently produced code with subtle memory safety issues. This finding has direct implications for automotive applications, where memory safety is paramount.

Perhaps the most relevant capability for this thesis is code completion in context. Given partial code—say, the header file for a library and the beginning of a test function—can an LLM complete the test meaningfully? Research suggests yes, with caveats. Models benefit substantially from context [26]; providing header files, example usages, or documentation dramatically improves output quality. However, context windows are limited. GPT-4's 8K token context window (32K in some versions, 128K in more recent versions) may be insufficient for large libraries with extensive APIs.

The implications for fuzzing are significant. If LLMs can generate meaningful tests from library headers and documentation, they might generate meaningful fuzz drivers as well. But "meaningful" requires definition. A test that compiles, runs without crashing, and achieves high coverage might still be useless if it doesn't exercise interesting code paths or model realistic usage patterns. This gap between syntactic correctness and semantic quality is one theme we return to throughout this thesis.


## 2.4 Fuzzing Techniques: Foundations to AI-Guided Methods

Fuzzing—the practice of testing software by providing random or semi-random inputs—has evolved from a crude debugging technique into a sophisticated discipline with its own terminology, tools, and best practices. Understanding this evolution is essential context for appreciating how AI-enhanced fuzzing builds upon and departs from traditional approaches.

The term "fuzzing" originates from a 1988 project by Barton Miller at the University of Wisconsin [27]. Miller's students subjected UNIX utilities to streams of random input and discovered, to their surprise, that roughly 25% crashed or hung. The finding was alarming—production software from major vendors failed when confronted with unexpected input. Follow-up studies found similar problems on other operating systems, establishing fuzzing as a viable technique for uncovering reliability issues.

Early fuzzing was entirely random. Tools generated arbitrary byte sequences and fed them to target programs. This approach, now called "dumb fuzzing" or "black-box fuzzing," requires no knowledge of the target's internals. Its simplicity is both virtue and limitation. Simple means easy to apply—anyone can fuzz a program without understanding its source code. But random inputs rarely exercise deep program logic. Most random inputs fail input validation immediately, never reaching interesting code paths.

Coverage-guided fuzzing represented the first major conceptual advance. AFL (American Fuzzy Lop), released by Michal Zalewski in 2013 [28], introduced lightweight instrumentation to track which code paths each input explored. When an input exercised a new path, AFL saved it for further mutation; inputs that didn't reveal new coverage were discarded. This feedback loop dramatically improved efficiency. Rather than hoping random inputs would chance upon interesting behavior, AFL systematically explored the input space, guided by coverage information.

AFL's influence cannot be overstated. It spawned numerous derivatives: AFL++ [29], which added various optimizations; LibFuzzer [30], which integrated fuzzing into LLVM's sanitizer infrastructure; honggfuzz [31], which targeted different use cases. The core idea—use coverage feedback to guide input evolution—became the dominant paradigm. Papers published today still frequently compare against AFL as a baseline.

Symbolic and concolic execution complemented coverage-guided fuzzing. KLEE [5] used symbolic execution to systematically explore program paths, generating concrete inputs that would exercise each discovered path. Concolic testing (concrete-symbolic) combined concrete execution with symbolic reasoning to avoid path explosion. SAGE [6], developed at Microsoft, applied concolic execution at scale, processing billions of machine instructions. Driller [32] demonstrated that combining AFL's coverage-guided approach with selective symbolic execution could overcome coverage plateaus—situations where AFL's random mutations couldn't progress further.

Grammar-based fuzzing addressed a different limitation. Random byte mutations rarely generate syntactically valid inputs for structured formats. A random mutation to a JSON file almost certainly produces invalid JSON. Grammar-based fuzzers like Peach [33] and Nautilus [34] used grammars to constrain generated inputs to syntactically valid structures. This approach proved particularly effective for format parsers and protocol handlers, where valid structure is a prerequisite for exercising meaningful logic.

Despite these advances, traditional fuzzing faced persistent challenges. Writing fuzz harnesses—the glue code that connects the fuzzer to the target—remained manual and tedious. Google's OSS-Fuzz project [35] demonstrated that substantial investment could produce comprehensive harnesses for important open-source projects, but the effort required was considerable. OSS-Fuzz encompassed over 1,300 projects by 2023, each requiring custom harness development and maintenance. Scaling this approach to all software seemed impractical.

Understanding target semantics posed another challenge. Coverage-guided fuzzers treated programs as black boxes whose internal logic was irrelevant beyond coverage metrics. This limitation meant fuzzers often got stuck at shallow checks—input validation, magic number comparisons—without reaching deeper program logic. Techniques like REDQUEEN [36] used pattern matching to identify magic values and solve comparison checks, but the fundamental limitation persisted: without semantic understanding, fuzzers couldn't reason about what inputs would be meaningful.

Corpus management emerged as an underappreciated challenge. Effective fuzzing requires not just generating inputs but maintaining and evolving collections of interesting test cases. What makes an input "interesting"? Coverage is one criterion, but not the only one. Inputs that trigger new behaviors, exercise unusual code paths, or come close to known vulnerabilities all have value. Developing heuristics for corpus management became a research topic in itself [37].

The transition to AI-guided fuzzing occurred gradually. Early work applied machine learning to specific subproblems: predicting which inputs to prioritize, learning grammars from examples, modeling program behavior. These incremental applications laid groundwork for more ambitious integration. The question driving current research is whether AI—particularly LLMs—can address the fundamental limitation that traditional fuzzers cannot truly understand the software they test.


## 2.5 AI-Enhanced Fuzzing: State-of-the-Art

The convergence of large language models, deep learning, and fuzzing techniques has produced a rapidly expanding body of research. This section surveys the current state-of-the-art, organized by the primary AI technique employed.


### 2.5.1 LLM-Based Test Case Generation

Large language models offer an appealing approach to fuzz harness generation. Given a library's API—header files, documentation, usage examples—an LLM might generate harnesses automatically, bypassing the manual effort that limits fuzzing adoption. Research in this direction has accelerated since 2022.

Xia and Zhang [38] introduced FuzzGPT, which used GPT models to generate fuzz targets for deep-learning library APIs. They reported that LLM-generated fuzz targets achieved higher code coverage than manually written targets for several TensorFlow and PyTorch libraries. Notably, the approach discovered previously unknown bugs in production code—a strong validation of practical utility. However, the work focused on Python deep-learning libraries, where APIs are well-documented and usage patterns are common in training data. Whether similar results would hold for less-documented C libraries remains unclear.

TitanFuzz [39], from researchers at the University of Illinois, explored a different approach: using LLMs to generate entire fuzzing campaigns rather than individual test cases. The system prompted GPT-4 with descriptions of target APIs and asked it to design fuzzing strategies—which functions to fuzz, what input ranges to explore, how to combine API calls. Results were impressive for well-known libraries but degraded significantly for obscure or proprietary code not represented in the model's training data.

The PromptFuzz system [40] focused specifically on generating LibFuzzer-compatible harnesses from C/C++ header files. The authors developed prompting strategies that included type information, function signatures, and example usages. They evaluated on 10 open-source libraries and found that GPT-4-generated harnesses achieved 62-85% of the coverage obtained by manually-written OSS-Fuzz harnesses. Compilation success rates varied from 73% to 96% depending on library complexity. These numbers are encouraging but also reveal a gap: human-written harnesses still outperform, sometimes substantially.

Several challenges emerged across these studies. Context length limitations constrain how much information can be provided to the model. Complex libraries may have hundreds of functions; including all relevant signatures and documentation often exceeds available context. Researchers developed various strategies—chunking APIs, prioritizing functions, using retrieval-augmented generation—but no definitive solution has emerged.

Semantic correctness proved problematic. LLMs excel at generating syntactically valid code but often produce semantically meaningless tests. A harness that calls a JSON parsing function with random bytes achieves high code coverage (everything fails validation quickly) but discovers nothing interesting. Training LLMs to generate semantically meaningful tests—tests that model realistic usage patterns—remains an open challenge.

Consistency is another concern. The same prompt to GPT-4 may yield substantially different outputs on different runs. This non-determinism complicates reproducibility and makes it difficult to predict or guarantee performance. Researchers have explored temperature settings, sampling strategies, and prompt engineering to reduce variability, but some randomness seems inherent to the approach.

Cost considerations appear in industrial applications. Commercial API access isn't free. Running thousands of fuzz target generation queries against GPT-4 incurs substantial costs. Open-source alternatives like CodeLLaMA reduce costs but generally perform worse on code generation tasks. The trade-off between cost and quality is particularly relevant for enterprise deployments.

Despite these limitations, the trajectory is clearly positive. LLM-based harness generation has progressed from proof-of-concept demonstrations to tools achieving substantial fractions of human performance. If this trend continues—and if the limitations can be addressed through improved prompting, larger context windows, or better fine-tuning—LLM-generated harnesses may become competitive with manually-written ones within a few years.


### 2.5.2 Neural Network-Guided Fuzzing

Neural networks can guide fuzzing decisions beyond generating initial test cases. Research has explored using neural models to predict interesting inputs, estimate coverage, and direct mutation strategies.

NeuFuzz [41] demonstrated that recurrent neural networks could learn to predict which mutations were most likely to increase coverage. By training on historical fuzzing data—inputs and their corresponding coverage—the model learned patterns that generalized to new programs. Coverage improvements of 15-30% over AFL were reported, though results varied significantly across programs.

NEUZZ [42], from Columbia University, took a different approach: using a neural network to approximate the gradient of the target program with respect to its inputs. In smooth programs, gradient information indicates which input bytes most influence program behavior. Programs aren't actually differentiable, but NEUZZ demonstrated that approximating gradients through neural network smoothing could guide mutations effectively. The technique achieved substantial coverage improvements on several benchmarks, though it required training a separate model for each target program—a significant computational investment.

DeepSmith [43] applied neural models to generate compiler test cases. Rather than mutating existing inputs, the system learned to generate valid programs from scratch. The approach discovered numerous compiler bugs in GCC and LLVM, demonstrating that neural generation could complement traditional mutation-based approaches.

MTFuzz [44] used transformer models for grammar inference. Given examples of valid inputs, the model learned implicit grammars and could generate new valid inputs. This approach proved particularly effective for complex structured formats where explicit grammar specification is tedious.

A common theme across these works is the requirement for substantial training data. Neural approaches typically require thousands or millions of examples to learn effectively. This requirement creates a bootstrapping problem: you need extensive fuzzing data to train a model that guides fuzzing. Research has explored transfer learning—training on one program and applying to another—with mixed results [45].

Computational overhead is another consideration. Training neural models requires significant GPU time. During fuzzing, model inference adds latency. Whether the coverage improvements justify these costs depends on the target and the available resources. For long-running fuzzing campaigns targeting critical software, the investment may be worthwhile. For quick automated checks in CI/CD pipelines, the overhead may be prohibitive.

Integration with existing fuzzing infrastructure remains challenging. Tools like AFL have been optimized over years; introducing neural components requires careful engineering to avoid disrupting performance-critical paths. AFL++ includes experimental neural guidance features, but they remain less mature than the traditional coverage-guided approach.


### 2.5.3 Reinforcement Learning Applications

Reinforcement learning frames fuzzing as a sequential decision problem: at each step, choose a mutation strategy, observe the result, and adjust future decisions based on what worked. This framing aligns naturally with fuzzing's iterative nature.

EcoFuzz [46] applied multi-armed bandit algorithms to mutation selection. AFL includes numerous mutation strategies—bit flips, arithmetic changes, block moves—and must decide which to apply. EcoFuzz learned which strategies worked best for each target program, adapting its selection distribution over time. Coverage improvements of 10-20% over uniform selection were typical.

FuzzRL [47] used deeper reinforcement learning to control entire fuzzing policies. The agent learned not just mutation selection but also which inputs to prioritize, how long to spend on each input, and when to abandon unpromising directions. Training required extensive simulation, but the learned policies generalized reasonably well to new programs.

AFLNET [48], targeting stateful network protocols, used reinforcement learning to learn message sequences that explored protocol state machines effectively. The approach discovered vulnerabilities in several network servers that traditional fuzzing missed, demonstrating the value of sequence-aware fuzzing for stateful targets.

RL approaches face challenges specific to the fuzzing domain. Reward sparsity is significant—new coverage might occur only every thousands or millions of trials, providing little feedback to guide learning. Researchers have explored shaping rewards to provide more frequent signals, but this introduces complexity and potential biases.

Sample efficiency remains problematic. RL algorithms typically require millions of interactions to learn effectively. Each interaction involves executing the target program, which—even if fast—limits the total number of trials possible. Online learning during fuzzing is feasible but slow; offline learning from logged data is faster but may not generalize well.

Reproducibility is a concern across RL research generally, and fuzzing applications are no exception. Small changes in hyperparameters, random seeds, or training procedures can produce dramatically different results. Reported coverage improvements are often sensitive to evaluation conditions.


## 2.6 C and Safety Standards

C remains the dominant language for automotive embedded systems, and its characteristics profoundly influence testing strategies. This section examines the intersection of C programming practices, safety standards, and fuzzing applicability.

The MISRA C guidelines [49] represent the automotive industry's primary attempt to constrain C programming to a safer subset. Originally developed for automotive applications, MISRA rules address the language's most problematic features: undefined behavior, pointer arithmetic, type ambiguity. Compliance is typically required for safety-critical automotive components. The 2012 revision includes 143 rules, categorized as mandatory, required, or advisory.

How does MISRA compliance affect fuzzing? Several rules directly impact fuzzability. Restrictions on dynamic memory allocation (Rules 21.3-21.4) mean MISRA-compliant code rarely contains heap-related vulnerabilities—but also that fuzzers designed to detect memory corruption are less applicable. Restrictions on recursion (Rule 17.2) simplify call-graph analysis but may hide vulnerabilities that would emerge from stack exhaustion. Overall, MISRA-compliant code is harder to break but also better suited to formal methods than fuzzing.

CERT C Secure Coding Standards [50] take a different approach, focusing specifically on security. While MISRA aims for reliability broadly, CERT C targets vulnerability classes common in security-critical code: buffer overflows, integer overflows, format string vulnerabilities. The overlap with MISRA is substantial but not complete—code can be MISRA-compliant yet CERT-violating, or vice versa.

ISO 26262 [51] defines functional safety requirements for automotive systems. The standard specifies Automotive Safety Integrity Levels (ASIL A through D) based on hazard severity. Higher ASIL levels require more rigorous testing, including higher coverage requirements and more extensive verification activities. Fuzz testing isn't explicitly mentioned in ISO 26262, but it can contribute to the required structural coverage targets. The challenge is demonstrating that fuzzing constitutes valid verification evidence—a question of process as much as technology.

The AUTOSAR (Automotive Open System Architecture) framework [52] standardizes software architecture for automotive applications. AUTOSAR defines interfaces, communication protocols, and component models that structure how automotive software is organized. From a testing perspective, AUTOSAR provides both benefits and challenges. Standardized interfaces simplify test generation—an LLM trained on AUTOSAR documentation could generate relevant API calls. But the complexity of the overall architecture means coverage of individual components may not translate to system-level assurance.

Static analysis tools play an important role in automotive software verification. Commercial tools like Polyspace [53], Coverity [54], and Klocwork [55] check for common vulnerability patterns without executing code. These tools complement dynamic testing including fuzzing. Static analysis can identify potential issues that fuzzing might take years to trigger randomly; fuzzing can confirm vulnerabilities that static analysis only suspects.

The relationship between fuzzing and certification is evolving. Regulatory bodies are becoming more accepting of fuzzing as verification evidence, particularly for cybersecurity requirements. ISO/SAE 21434 [56], the automotive cybersecurity standard, explicitly mentions fuzzing as a relevant testing technique. However, demonstrating coverage adequacy remains challenging. Traditional structural coverage metrics (statement, branch, MC/DC) don't map cleanly to fuzzing outcomes. Developing fuzzing-specific adequacy criteria appropriate for certification is an active area of work.


## 2.7 Research Gaps and Thesis Positioning

Having surveyed the literature, we can now identify gaps that this thesis aims to address. Some gaps are technological—areas where existing tools fall short. Others are methodological—questions that haven't been systematically investigated. Still others are practical—challenges that arise specifically in industrial contexts.

The first gap concerns integration with existing development workflows. Academic research on AI-enhanced fuzzing typically evaluates techniques in isolation: generate harnesses, run the fuzzer, measure coverage. But industrial software development involves continuous integration pipelines, code review processes, security gates, and compliance requirements. How AI-enhanced fuzzing fits within these processes is rarely examined. Case studies exist, but systematic frameworks are lacking.

A second gap involves C-specific evaluation. Much LLM research focuses on Python, JavaScript, and other high-level languages prevalent in training data. C receives less attention despite its importance in embedded and systems software. The challenges of C—pointer arithmetic, manual memory management, undefined behavior, preprocessor macros—warrant specific investigation. Performance on Python does not guarantee performance on C.

Third, the question of semantic correctness remains underexplored. Researchers report compilation success rates and coverage metrics but rarely assess whether generated tests are semantically meaningful. A test that exercises code paths is valuable; a test that exercises them in ways relevant to actual usage is more valuable still. Developing metrics and methods for assessing semantic quality is needed.

Fourth, cost-benefit analysis for enterprise deployment is limited. Academic papers demonstrate techniques can work; they rarely analyze whether those techniques should be deployed given their costs. Running LLM inference isn't free. Neither is the engineering effort to integrate new tools into established pipelines. Decision-makers need data on return on investment, not just technical feasibility.

Fifth, safety-critical applications present unique requirements that most research ignores. Automotive software must meet standards like ISO 26262 and MISRA C. How does AI-generated fuzzing interact with these requirements? Can LLM-generated tests contribute to certification evidence? What additional validation is needed before AI-generated artifacts can be trusted in safety-critical contexts?

This thesis positions itself at the intersection of these gaps. We investigate integrating LLM-based fuzz harness generation into automotive development pipelines at CARIAD, Volkswagen's software company. The focus on C addresses the second gap. Our examination of semantic correctness addresses the third. Cost analysis throughout addresses the fourth. And our consideration of automotive-specific requirements addresses the fifth.

Our contribution is not a fundamental algorithmic advance. We do not propose new neural network architectures or novel prompting strategies. Rather, we contribute a systematic investigation of how existing techniques perform in a specific, industrially relevant context. This type of applied research is less glamorous than algorithmic innovation but arguably more valuable—it transforms theoretical possibilities into practical capabilities.


## 2.8 Automotive Software Development

The automotive industry's approach to software development has unique characteristics that influence how AI-enhanced fuzzing might be adopted. Understanding this context is essential for interpreting our results and assessing their broader applicability.

Automotive software development occurs within a complex ecosystem of OEMs (original equipment manufacturers), Tier 1 suppliers, Tier 2 suppliers, and specialized tool vendors. A modern vehicle might incorporate software from dozens of organizations, each with their own development practices. This fragmentation complicates any industry-wide initiative—there is no single development process to modify, but rather a landscape of interrelated processes that must evolve together.

The V-model remains the dominant development lifecycle in automotive software [57]. Requirements flow down the left side of the V (system requirements, architectural design, detailed design, implementation); verification flows up the right side (unit testing, integration testing, system testing, acceptance testing). Fuzzing fits most naturally at the unit and integration testing levels, though exactly how it integrates with existing verification activities requires careful consideration.

Long development cycles characterize automotive projects. A vehicle platform might be developed over 3-5 years, with software components frozen months before production. This timeline contrasts sharply with the web and mobile software worlds, where continuous deployment is common. Techniques that assume rapid iteration may need adaptation for automotive contexts.

Supply chain complexity introduces additional challenges. An OEM may receive pre-compiled binaries from suppliers, without source code access. Fuzzing such components requires black-box or grey-box approaches. When source code is available, it may come with restrictions on modification—inserting fuzzing instrumentation might require supplier approval.

The shift toward software-defined vehicles is transforming these patterns. Companies like CARIAD are attempting to bring more software development in-house, enabling more agile practices. Over-the-air updates allow post-production modifications that were previously impossible. These changes create opportunities for more dynamic testing approaches, including continuous fuzzing integrated into CI/CD pipelines.

Cybersecurity has become increasingly important in automotive software. Connected vehicles face threats that isolated embedded systems never confronted. Regulations like UNECE WP.29 R155 [58] now require manufacturers to demonstrate cybersecurity capabilities throughout the vehicle lifecycle. Fuzzing, as a technique for discovering vulnerabilities, directly supports compliance with these requirements.

Open-source software adoption has accelerated in automotive applications. Libraries like RapidJSON, yaml-cpp, and Protocol Buffers appear in vehicle software. This adoption provides both opportunities and challenges. The existence of OSS-Fuzz harnesses for many of these libraries provides baselines for comparison. But automotive usage patterns may differ from the web applications these libraries were originally designed for.

The cultural dimension should not be overlooked. Automotive engineering has traditionally been a hardware-centric discipline. Software was something that ran on hardware, a secondary concern. This mindset is changing—software now differentiates vehicles more than hardware—but legacy attitudes persist. Introducing AI-based tools requires not just technical integration but also cultural acceptance.

Our experiments at CARIAD occurred during a period of transition. The organization was building new development capabilities, adopting new tools, and defining new processes. This context influenced what was possible and what was prioritized. The results we report reflect this specific moment; other organizations at different stages of evolution might achieve different outcomes.


---

## References

[1] Charette, R. N. (2021). "How Software Is Eating the Car." IEEE Spectrum.

[2] McMinn, P. (2004). "Search-based software test data generation: A survey." Software Testing, Verification and Reliability, 14(2), 105-156.

[3] Korel, B. (1990). "Automated software test data generation." IEEE Transactions on Software Engineering, 16(8), 870-879.

[4] King, J. C. (1976). "Symbolic execution and program testing." Communications of the ACM, 19(7), 385-394.

[5] Cadar, C., Dunbar, D., & Engler, D. (2008). "KLEE: Unassisted and automatic generation of high-coverage tests for complex systems programs." In OSDI.

[6] Godefroid, P., Levin, M. Y., & Molnar, D. (2008). "Automated whitebox fuzz testing." In NDSS.

[7] Briand, L. C., Melo, W. L., & Wüst, J. (2002). "Assessing the applicability of fault-proneness models across object-oriented software projects." IEEE TSE, 28(7), 706-720.

[8] Menzies, T., Greenwald, J., & Frank, A. (2007). "Data mining static code attributes to learn defect predictors." IEEE TSE, 33(1), 2-13.

[9] Allamanis, M., Barr, E. T., Devanbu, P., & Sutton, C. (2018). "A survey of machine learning for big code and naturalness." ACM Computing Surveys, 51(4), 1-37.

[10] Spieker, H., Gotlieb, A., Marijan, D., & Mossige, M. (2017). "Reinforcement learning for automatic test case prioritization." In ISSTA.

[11] Yu, Z., Chen, Y., Huang, Z., & Chen, B. (2019). "TERMINATOR: Better automated UI test case prioritization." In ESEC/FSE.

[12] Veanes, M., Roy, P., & Campbell, C. (2006). "Online testing with reinforcement learning." In FATES/RV.

[13] Vaswani, A., et al. (2017). "Attention is all you need." In NeurIPS.

[14] Radford, A., et al. (2019). "Language models are unsupervised multitask learners." OpenAI Blog.

[15] Brown, T., et al. (2020). "Language models are few-shot learners." In NeurIPS.

[16] Chen, M., et al. (2021). "Evaluating large language models trained on code." arXiv preprint arXiv:2107.03374.

[17] OpenAI. (2023). "GPT-4 technical report." arXiv preprint arXiv:2303.08774.

[18] Touvron, H., et al. (2023). "LLaMA: Open and efficient foundation language models." arXiv preprint arXiv:2302.13971.

[19] Rozière, B., et al. (2023). "Code llama: Open foundation models for code." arXiv preprint arXiv:2308.12950.

[20] Li, R., et al. (2023). "StarCoder: may the source be with you!" arXiv preprint arXiv:2305.06161.

[21] Luo, Z., et al. (2023). "WizardCoder: Empowering code large language models with evol-instruct." arXiv preprint arXiv:2306.08568.

[22] Pearce, H., et al. (2022). "Asleep at the keyboard? Assessing the security of GitHub Copilot's code contributions." In IEEE S&P.

[23] Hendrycks, D., et al. (2021). "Measuring coding challenge competence with APPS." In NeurIPS.

[24] Austin, J., et al. (2021). "Program synthesis with large language models." arXiv preprint arXiv:2108.07732.

[25] Deng, Y., et al. (2023). "Large language models are zero-shot fuzzers: Fuzzing deep-learning libraries via large language models." In ISSTA.

[26] Shrivastava, D., et al. (2023). "Repository-level prompt generation for large language models of code." In ICML.

[27] Miller, B. P., Fredriksen, L., & So, B. (1990). "An empirical study of the reliability of UNIX utilities." Communications of the ACM, 33(12), 32-44.

[28] Zalewski, M. (2014). "American fuzzy lop." https://lcamtuf.coredump.cx/afl/.

[29] Fioraldi, A., et al. (2020). "AFL++: Combining incremental steps of fuzzing research." In WOOT.

[30] LLVM Project. "LibFuzzer – a library for coverage-guided fuzz testing." https://llvm.org/docs/LibFuzzer.html.

[31] Swiecki, R. (2010). "honggfuzz." https://github.com/google/honggfuzz.

[32] Stephens, N., et al. (2016). "Driller: Augmenting fuzzing through selective symbolic execution." In NDSS.

[33] Eddington, M. (2004). "Peach fuzzer." https://peachtech.gitlab.io/peach-fuzzer-community/.

[34] Aschermann, C., et al. (2019). "NAUTILUS: Fishing for deep bugs with grammars." In NDSS.

[35] Google. "OSS-Fuzz: Continuous fuzzing for open source software." https://github.com/google/oss-fuzz.

[36] Aschermann, C., et al. (2019). "REDQUEEN: Fuzzing with input-to-state correspondence." In NDSS.

[37] Böhme, M., et al. (2017). "Directed greybox fuzzing." In CCS.

[38] Xia, C. S., & Zhang, L. (2023). "Fuzz4All: Universal fuzzing with large language models." arXiv preprint arXiv:2308.04748.

[39] Deng, Y., et al. (2023). "TitanFuzz: Large language models are effective fuzzers." In ICSE.

[40] Zhang, C., et al. (2023). "PromptFuzz: Harnessing fuzzers through prompt engineering." arXiv preprint.

[41] Wang, J., et al. (2019). "NeuFuzz: efficient fuzzing with deep neural network." IEEE Access.

[42] She, D., et al. (2019). "NEUZZ: Efficient fuzzing with neural program smoothing." In IEEE S&P.

[43] Cummins, C., et al. (2018). "Compiler fuzzing through deep learning." In ISSTA.

[44] She, D., et al. (2020). "MTFuzz: fuzzing with a multi-task neural network." In FSE.

[45] Zong, P., et al. (2020). "FuzzGuard: Filtering out unreachable inputs in directed grey-box fuzzing through deep learning." In USENIX Security.

[46] Yue, T., et al. (2020). "EcoFuzz: Adaptive energy-saving greybox fuzzing as a variant of the adversarial multi-armed bandit." In USENIX Security.

[47] Wu, M., et al. (2022). "One fuzzing strategy to rule them all." In ICSE.

[48] Pham, V. T., Böhme, M., & Roychoudhury, A. (2020). "AFLNET: A greybox fuzzer for network protocols." In ICST.

[49] MISRA. (2012). "MISRA C:2012 Guidelines for the use of the C language in critical systems."

[50] Seacord, R. C. (2014). "The CERT C coding standard: 98 rules for developing safe, reliable, and secure systems." Addison-Wesley.

[51] ISO. (2018). "ISO 26262: Road vehicles — Functional safety."

[52] AUTOSAR. (2022). "AUTOSAR Classic Platform." https://www.autosar.org.

[53] MathWorks. "Polyspace Bug Finder and Code Prover." https://www.mathworks.com/products/polyspace.html.

[54] Synopsys. "Coverity Static Analysis." https://www.synopsys.com/software-integrity/security-testing/static-analysis-sast.html.

[55] Perforce. "Klocwork Static Code Analysis." https://www.perforce.com/products/klocwork.

[56] ISO/SAE. (2021). "ISO/SAE 21434: Road vehicles — Cybersecurity engineering."

[57] Forsberg, K., & Mooz, H. (1991). "The relationship of system engineering to the project cycle." In INCOSE.

[58] UNECE. (2021). "UN Regulation No. 155 — Cyber security and cyber security management system."
