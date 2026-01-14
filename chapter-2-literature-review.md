# Chapter 2: Literature Review

## 2.1 Introduction and Scope

This chapter provides a comprehensive review of the literature on AI-enhanced software testing, with particular emphasis on fuzzing techniques applicable to automotive software development. The review synthesizes research from three converging fields: traditional AI approaches in software testing, large language models for code generation, and modern fuzzing methodologies.

The motivation for this review stems from the growing complexity of automotive software systems. Modern vehicles contain over 100 million lines of code distributed across dozens of electronic control units \cite{MLFuzzReview2020}. Testing such systems manually is impractical, creating demand for automated approaches that can identify vulnerabilities efficiently. Recent advances in large language models have opened new possibilities for test generation, but their applicability to safety-critical automotive systems remains underexplored.

The scope of this review is constrained to C-based software, reflecting the predominant language in embedded automotive systems. While research on LLM-based testing has focused largely on Python and JavaScript, the specific challenges of C, including pointer arithmetic, manual memory management, and undefined behavior, require dedicated investigation. This chapter examines how existing techniques might be adapted for automotive contexts and identifies gaps that this thesis aims to address.

The review is organized thematically rather than chronologically. Section 2.2 examines traditional AI approaches that predate large language models. Section 2.3 traces the evolution of LLMs and their application to code generation. Section 2.4 covers fuzzing fundamentals from early random testing to coverage-guided approaches. Section 2.5 surveys the current state of the art in AI-enhanced fuzzing, including LLM-based methods, neural network-guided approaches, and reinforcement learning applications. Section 2.6 addresses C programming standards relevant to automotive development. Section 2.7 identifies research gaps and positions this thesis within the literature. Section 2.8 concludes with an overview of automotive software development practices.


## 2.2 Traditional AI Approaches in Software Testing

Prior to the emergence of large language models, researchers explored various AI techniques for automating software testing. These approaches laid important groundwork for current methods and continue to influence the field.

Genetic algorithms represent one of the earliest applications of AI to software testing. These evolutionary approaches maintain populations of test cases that undergo selection, crossover, and mutation operations over successive generations. Wang et al. \cite{MLFuzzReview2020} provide a systematic review of this history, documenting how evolutionary testing dominated research from the late 1990s through the early 2010s. The key insight underlying these approaches was that test generation could be formulated as an optimization problem: define a fitness function based on coverage or fault detection, and allow evolution to discover effective test inputs.

The limitations of evolutionary approaches became apparent over time. Fitness function design proved challenging, as maximizing code coverage did not necessarily correlate with bug detection. Multi-objective optimization helped balance competing goals, but genetic algorithms fundamentally lacked the ability to reason about program semantics. They could explore the input space efficiently but could not understand what made certain inputs more meaningful than others.

Symbolic execution offered an alternative paradigm based on constraint solving rather than random search. The approach treats program inputs as symbolic values and accumulates path constraints during execution. Solving these constraints yields concrete inputs guaranteed to exercise specific program paths. Godefroid, Levin, and Tauth demonstrated the practical viability of this approach with SAGE \cite{SAGE2008}, which Microsoft deployed internally for security testing. SAGE processed billions of machine instructions and found vulnerabilities that conventional testing had missed.

Despite its elegance, symbolic execution faces fundamental scalability challenges. The number of paths through a program grows exponentially with the number of branch points, a phenomenon known as path explosion. Yavuz's recent work on DESTINA \cite{DESTINA2024} addresses this through targeted execution strategies, but complete path coverage remains infeasible for non-trivial programs. This limitation has motivated hybrid approaches that combine symbolic reasoning with other techniques.

Machine learning entered software testing research in supporting roles around 2015. A technical report from Sandia National Laboratories \cite{MLFuzzReviewSandia2019} surveyed early applications, including defect prediction, test case prioritization, and coverage estimation. These applications used ML models to inform human decision-making rather than to generate tests directly. While practical and useful, they did not fundamentally change how tests were created.

Neural networks enabled more ambitious applications. Pei et al. \cite{DeepXplore2017} introduced DeepXplore for testing deep learning systems, defining neuron coverage as a testing adequacy criterion. Odena et al. \cite{TensorFuzz2019} extended coverage-guided fuzzing to neural networks with TensorFuzz, demonstrating that coverage-based feedback could guide test generation for ML models themselves. These works showed that neural networks could participate actively in the testing process, not merely support it.

The transition to generative approaches, using AI to create tests rather than select or prioritize them, marked a qualitative shift. This transition accelerated dramatically with the advent of large language models capable of producing syntactically valid code from natural language descriptions.


## 2.3 Large Language Models: Evolution and Code Generation

Large language models have transformed expectations for what automated systems can accomplish in software engineering. This section traces their evolution and examines their capabilities and limitations for code generation tasks.

The foundation for modern LLMs was established with the Transformer architecture, which enabled training on vastly larger datasets than previous approaches permitted. Subsequent scaling of Transformer-based models revealed emergent capabilities, including the ability to generate coherent text and, importantly for this thesis, syntactically valid source code. GPT-3, released in 2020 with 175 billion parameters, demonstrated that models trained primarily on next-token prediction could perform a wide range of tasks with minimal task-specific training.

Code-specific models followed rapidly. Codex, fine-tuned on publicly available source code, powered GitHub Copilot and achieved 28.8% success on the HumanEval benchmark in initial evaluations, rising to 70.2% with multiple sampling attempts. GPT-4 further improved code generation capabilities, with better contextual understanding and reduced occurrence of subtle errors. The expansion of context windows from 2K to 128K tokens enabled inclusion of multiple source files, header definitions, and detailed documentation in prompts.

Open-source alternatives have emerged as important counterparts to proprietary models. Meta's LLaMA demonstrated that competitive performance was achievable with smaller, publicly available models. Code-specific variants including StarCoder and CodeLLaMA targeted programming tasks specifically. These models enable local deployment without transmitting proprietary code to external services, a consideration relevant to automotive applications with intellectual property constraints.

Research has probed the boundaries of LLM code generation capabilities systematically. Deng et al. \cite{TitanFuzz2022} found that while models generated syntactically valid code at high rates, the generated code frequently contained semantic issues. Tests compiled successfully but exercised APIs in unintended ways. Wang et al. \cite{LLMMutationTesting2024} observed similar patterns in mutation testing experiments: high success rates on simple cases with degrading performance as complexity increased.

C presents particular challenges for LLM-based code generation. Training data skews heavily toward Python, JavaScript, and other high-level languages. C's distinctive features, such as pointer arithmetic, manual memory management, preprocessor macros, and undefined behavior, are less well represented. Empirical evaluations consistently show worse performance on C compared to Python. This gap has direct implications for automotive applications, where C remains the dominant language for embedded systems.

Context utilization presents an ongoing research question. While modern models accept 128K tokens or more, whether they effectively use all provided context remains unclear. Shrivastava et al. found that repository-level context improved generation quality but with diminishing returns as context length increased. For complex libraries with extensive APIs, even large context windows may prove insufficient.


## 2.4 Fuzzing Techniques: Foundations to AI-Guided Methods

Fuzzing, which involves testing software through automated generation of inputs, has evolved from simple random generation to sophisticated AI-guided approaches. This section traces that evolution and identifies the challenges that motivate integration with large language models.

The origins of fuzzing are traced to Barton Miller's 1988 experiments at the University of Wisconsin. Students subjected UNIX utilities to streams of random input and found that approximately 25% crashed or hung. This finding established fuzzing as a viable technique for uncovering reliability issues in production software.

Early fuzzing was entirely random, requiring no knowledge of target program structure. This simplicity enabled broad applicability but limited effectiveness. Random inputs rarely satisfy input validation checks, preventing exploration of deeper program logic. The vast majority of randomly generated inputs fail immediately, making efficient bug finding largely a matter of luck.

Coverage-guided fuzzing represented a fundamental advance. AFL (American Fuzzy Lop), released in 2013, introduced lightweight instrumentation to track which code paths each input explored. Inputs that discovered new coverage were retained and mutated further; inputs that revealed no new coverage were discarded. This feedback loop dramatically improved efficiency by directing search toward unexplored program behavior.

AFL spawned an ecosystem of derivative tools. AFL++ \cite{AFLPlusPlus} added performance optimizations and additional mutation strategies. LibFuzzer integrated coverage-guided fuzzing into the LLVM toolchain. Syzkaller \cite{Syzkaller2020} adapted coverage-guided techniques for kernel testing. Google's OSS-Fuzz project deployed these tools at scale, continuously fuzzing hundreds of open-source projects. Ding and Le Goues \cite{OSSFuzzBugs2021} analyzed bugs found by OSS-Fuzz, documenting vulnerability classes that other testing methods had missed.

Grammar-based fuzzing addressed the structural validity problem. Random byte mutations rarely produce syntactically valid inputs for complex formats. A randomly mutated JSON file is almost certainly invalid JSON. Grammar-based fuzzers generate inputs according to specified grammars, ensuring structural validity while still exploring variations. This approach proved particularly effective for protocol parsers and file format handlers.

Despite these advances, several challenges persist. Writing fuzz harnesses, which are the glue code that connects fuzzers to target libraries, remains manual and labor-intensive. Coverage plateaus occur when random mutation cannot discover paths beyond certain checkpoints. These limitations create opportunities for AI-enhanced approaches that can reason about program structure and generate more targeted inputs.


## 2.5 AI-Enhanced Fuzzing: State of the Art

The convergence of machine learning and fuzzing has produced a rapidly growing body of research. This section surveys the current state of the art, organized by the primary AI technique employed.


### 2.5.1 LLM-Based Test Case Generation

Large language models offer a fundamentally different approach to test generation: rather than mutating existing inputs, they can synthesize tests from specifications, documentation, or source code. Research in this direction has accelerated since 2022.

Deng et al. introduced TitanFuzz \cite{TitanFuzz2022}, demonstrating that LLMs could generate effective fuzz targets for deep learning library APIs. By prompting models with API documentation, TitanFuzz produced test cases that achieved higher coverage than manually written tests for several libraries and discovered previously unknown bugs. The work established that LLM-generated tests could provide practical value, though the focus on well-documented Python libraries left open the question of applicability to less-documented C code.

FuzzGPT \cite{FuzzGPT2023} extended this approach with a focus on edge cases. The observation was that LLMs, having been trained on vast code repositories, possess implicit knowledge of typical API usage patterns. By prompting specifically for atypical or boundary-case inputs, FuzzGPT discovered bugs that conventional mutation-based fuzzers overlooked.

Fuzz4All \cite{Fuzz4All2023} pursued a more general approach, targeting multiple programming languages and application domains. The system used LLMs to generate both test inputs and the harness code to execute them. Coverage improvements over random generation were substantial, though still below expert-written harnesses.

Zhang et al. \cite{LLMDriverGenEffectiveness2023} directly evaluated the effectiveness of LLM-generated fuzz drivers compared to manually written alternatives. Results varied significantly across libraries: some LLM drivers achieved 80 to 90 percent of manual driver coverage, while others reached only 50 percent. Performance correlated with documentation quality and API complexity.

Recent work has explored combining LLM generation with traditional techniques. WhiteFox \cite{WhiteFox2023} uses LLMs for targeted compiler fuzzing. CKGFuzzer \cite{CKGFuzzer2024} augments LLM generation with code knowledge graphs. HyLLfuzz \cite{HyLLfuzz2024} combines LLM-generated seeds with mutation-based fuzzing. These hybrid approaches attempt to leverage LLM strengths while mitigating their limitations.

A consistent finding across this literature is the gap between syntactic validity and semantic quality. LLMs produce code that compiles at high rates but may not exercise meaningful behavior. A test that calls a parsing function with random bytes achieves high coverage by failing validation immediately, contributing little to bug finding. Evaluating and improving semantic quality remains an open challenge.


### 2.5.2 Neural Network-Guided Fuzzing

Before LLMs dominated attention, researchers explored using smaller neural networks to guide fuzzing decisions. These approaches remain relevant due to their lower computational overhead.

Wang et al. developed NeuFuzz \cite{NeuFuzz2019}, training neural networks to predict which inputs would discover new coverage. Models learned from historical fuzzing data, observing which mutations proved productive. NeuFuzz achieved 15 to 30 percent coverage improvements over baseline AFL in experiments.

She, Woo, and Brumley introduced NEUZZ \cite{NEUZZ2018}, using neural networks to approximate program gradients. In smooth functions, gradient information indicates which input bytes most influence behavior. Programs contain discrete branches and are not inherently smooth, but neural approximation proved surprisingly effective. NEUZZ achieved substantial coverage improvements on several benchmarks.

CoCoFuzzing \cite{CoCoFuzzing2021} applied these techniques to neural code models, testing whether models like Codex were themselves vulnerable to adversarial inputs. The finding that code models could be fooled by carefully crafted inputs has implications for the reliability of AI coding assistants.

A limitation of neural-guided approaches is the training data requirement. Each target program requires a dedicated model trained on fuzzing data from that program. Transfer learning offers partial solutions, but models trained on one program do not always generalize to others.


### 2.5.3 Reinforcement Learning Applications

Reinforcement learning formulates fuzzing as a sequential decision problem: at each step, select a mutation strategy, observe the outcome, and adjust future decisions accordingly.

Böttinger, Godefroid, and Singh explored deep reinforcement learning for fuzzing \cite{DeepRLFuzz2018}, demonstrating that RL agents could learn mutation strategies superior to static heuristics. The learned policies were not human-interpretable but proved effective in practice.

BertRLFuzzer \cite{BertRLFuzzer2023} combined BERT language models with reinforcement learning, using the language model to understand input structure and RL to determine mutation locations. The combination outperformed either approach alone.

Reinforcement learning approaches face practical challenges including reward sparsity, as new coverage may occur only every thousands of trials, and substantial training costs. These factors limit applicability to settings where long training times are acceptable.


## 2.6 C and Safety Standards

Automotive software operates within a regulatory framework that shapes development and testing practices. This section examines the intersection of C programming standards, safety requirements, and fuzzing applicability.

MISRA C defines coding guidelines intended to eliminate dangerous C constructs from safety-critical software. The guidelines restrict dynamic memory allocation, recursion, and pointer operations, producing code that is more constrained but safer. MISRA-compliant code is less prone to certain vulnerability classes but may require different testing strategies than general C code.

CERT C Secure Coding Standards focus specifically on security, addressing vulnerability classes including buffer overflows, integer overflows, and format string vulnerabilities. While overlapping with MISRA, CERT C targets different concerns and may apply to different code components.

ISO 26262 specifies functional safety requirements for automotive systems. The standard defines Automotive Safety Integrity Levels (ASIL A through D) with increasingly stringent requirements for higher-risk systems. Testing is central to ISO 26262 compliance, with coverage metrics and verification activities specified according to ASIL level. Fuzzing is not explicitly mentioned but can contribute to required verification objectives.

ISO/SAE 21434 addresses automotive cybersecurity specifically. Unlike ISO 26262's focus on functional safety, ISO/SAE 21434 targets security throughout the vehicle lifecycle. The standard explicitly mentions fuzzing as a relevant testing technique, reflecting growing regulatory acceptance of the approach.

The AUTOSAR framework standardizes software architecture for automotive applications. Standardized interfaces facilitate automated testing, as tests can target well-defined APIs rather than project-specific implementations. This standardization potentially enables broader applicability of LLM-generated tests, assuming models can learn AUTOSAR conventions.


## 2.7 Research Gaps and Thesis Positioning

Analysis of the literature reveals several gaps that this thesis aims to address.

First, integration with development workflows is underexplored. Academic research typically evaluates techniques in isolation, but practical deployment requires integration with CI/CD pipelines, code review processes, and compliance workflows. Klooster et al. \cite{CiCdFuzzing2022} examined fuzzing in CI/CD environments and identified practical challenges including speed, reliability, and determinism that benchmark evaluations do not capture.

Second, C-specific evaluation is limited. Most LLM fuzzing research targets Python or JavaScript, with C receiving comparatively little attention. The distinctive challenges of C, specifically memory safety, undefined behavior, and preprocessor complexity, merit dedicated investigation. Works such as ECG \cite{ECG2024} and GDBFuzz \cite{GDBFuzz2023} address embedded systems but remain exceptions.

Third, semantic quality lacks established metrics. Existing evaluations focus on compilation success and coverage achievement. Whether generated tests exercise meaningful behavior, rather than simply reaching code through trivial paths, is rarely assessed systematically.

Fourth, automotive-specific evaluation is rare. Claims of applicability to safety-critical systems are common, but actual evaluation on automotive software is uncommon. SAFLITE \cite{SAFLITE2024} and research on CAN bus testing \cite{CANBusFuzzAI2021} demonstrate feasibility but do not provide comprehensive evaluation frameworks.

This thesis positions itself at the intersection of these gaps. We investigate LLM-based fuzz harness generation for C libraries in automotive contexts, evaluating not only coverage metrics but also integration considerations and practical deployment factors. The contribution is applied validation: demonstrating what works, what does not, and what remains to be addressed.


## 2.8 Automotive Software Development

Automotive software development has characteristics that distinguish it from other software domains and influence the applicability of AI-enhanced testing approaches.

Development cycles in automotive are substantially longer than in consumer software. A vehicle platform may require 3 to 5 years from conception to production, with software frozen months before launch. This timeline limits the applicability of techniques that assume rapid iteration.

Supply chain complexity introduces additional considerations. Modern vehicles incorporate software from numerous suppliers, each with independent development processes. OEMs may receive compiled binaries without source code access, limiting testing options. When source code is available, modification rights may be restricted.

The V-model remains the dominant development lifecycle in automotive. Requirements flow down through system, architectural, and detailed design phases; verification flows up through unit, integration, and system testing. Fuzzing aligns naturally with unit and integration testing but requires consideration of how results feed into broader verification activities.

Software-defined vehicles represent an emerging paradigm shift. Companies like CARIAD aim to bring more software development in-house, enabling more agile practices. Over-the-air updates allow post-production modifications previously impossible. These changes create opportunities for more dynamic testing approaches, including continuous fuzzing integrated into development pipelines.

Cybersecurity requirements have increased significantly. Connected vehicles face threats that isolated systems never encountered. Regulations including UNECE WP.29 R155 mandate cybersecurity capabilities throughout the vehicle lifecycle. Fuzzing directly supports compliance with these requirements by identifying vulnerabilities before deployment.

CI Spark \cite{CISpark2023} and similar industry initiatives indicate growing interest in LLM-assisted testing for CI/CD integration. The practical deployment of such tools in automotive contexts, with their specific constraints and requirements, is an area where this thesis aims to contribute.

---

This chapter has reviewed the evolution from traditional AI approaches through large language models to current AI-enhanced fuzzing techniques. The analysis has identified gaps in existing research, particularly regarding C-specific evaluation, automotive deployment, and semantic quality assessment. The following chapter describes our methodology for investigating LLM-based fuzz harness generation within this context.
