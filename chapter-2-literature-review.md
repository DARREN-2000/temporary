# Chapter 2: Literature Review

## 2.1 Introduction

This chapter reviews AI-enhanced software testing literature, focusing on fuzzing techniques for automotive C-based software. The review covers traditional AI approaches, large language models for code generation, fuzzing methodologies, and relevant safety standards.


## 2.2 Traditional AI Approaches in Software Testing

Genetic algorithms were among the earliest AI applications to software testing. These evolutionary approaches maintain populations of test cases undergoing selection, crossover, and mutation over successive generations. Wang et al. \cite{MLFuzzReview2020} documented how evolutionary testing dominated research from the late 1990s through the early 2010s. Test generation was formulated as an optimization problem: define a fitness function based on coverage or fault detection, and allow evolution to discover effective test inputs.

Evolutionary approaches showed limitations over time. Fitness function design proved challenging, as maximizing code coverage did not necessarily correlate with bug detection. Multi-objective optimization helped balance competing goals, but genetic algorithms lacked the ability to reason about program semantics.

Symbolic execution offered an alternative paradigm based on constraint solving. The approach treats program inputs as symbolic values and accumulates path constraints during execution. Solving these constraints yields concrete inputs guaranteed to exercise specific program paths. Godefroid, Levin, and Tauth demonstrated this with SAGE \cite{SAGE2008}, which Microsoft deployed for security testing, processing billions of machine instructions and finding vulnerabilities that conventional testing missed.

Symbolic execution faces scalability challenges. The number of paths grows exponentially with branch points, a phenomenon known as path explosion. Yavuz's DESTINA \cite{DESTINA2024} addresses this through targeted execution strategies, but complete path coverage remains infeasible for non-trivial programs.

Machine learning entered software testing around 2015. A Sandia National Laboratories report \cite{MLFuzzReviewSandia2019} surveyed early applications including defect prediction, test case prioritization, and coverage estimation. These applications informed decision-making rather than generating tests directly.

Neural networks enabled more ambitious applications. Pei et al. \cite{DeepXplore2017} introduced DeepXplore for testing deep learning systems. Odena et al. \cite{TensorFuzz2019} extended coverage-guided fuzzing to neural networks with TensorFuzz. These works showed neural networks could participate actively in testing, not merely support it.


## 2.3 Large Language Models and Code Generation

The Transformer architecture enabled training on vastly larger datasets than previous approaches permitted. Scaling revealed emergent capabilities including syntactically valid code generation. GPT-3, released in 2020 with 175 billion parameters, demonstrated that models trained on next-token prediction could perform diverse tasks with minimal task-specific training.

Code-specific models followed. Codex powered GitHub Copilot and achieved 28.8% success on HumanEval, rising to 70.2% with multiple sampling attempts. GPT-4 improved code generation with better contextual understanding. Context windows expanded from 2K to 128K tokens.

Open-source alternatives emerged. Meta's LLaMA showed competitive performance with smaller models. StarCoder and CodeLLaMA targeted programming tasks specifically, enabling local deployment without transmitting proprietary code externally.

Deng et al. \cite{TitanFuzz2022} found that models generated syntactically valid code at high rates but frequently contained semantic issues. Wang et al. \cite{LLMMutationTesting2024} observed similar patterns: high success on simple cases with degrading performance as complexity increased.

C presents particular challenges. Training data skews toward Python and JavaScript. C's pointer arithmetic, manual memory management, preprocessor macros, and undefined behavior are less well represented. Empirical evaluations show worse performance on C compared to Python.


## 2.4 Fuzzing Techniques

Fuzzing originated with Barton Miller's 1988 experiments at the University of Wisconsin. Students subjected UNIX utilities to random input and found approximately 25% crashed or hung, establishing fuzzing as a viable vulnerability discovery technique.

Early fuzzing was entirely random, limiting effectiveness. Random inputs rarely satisfy input validation checks, preventing exploration of deeper program logic.

Coverage-guided fuzzing represented a fundamental advance. AFL (American Fuzzy Lop), released in 2013, introduced instrumentation to track code paths. Inputs discovering new coverage were retained and mutated; others were discarded. This feedback loop improved efficiency by directing search toward unexplored behavior.

AFL spawned derivative tools. AFL++ \cite{AFLPlusPlus} added optimizations and mutation strategies. LibFuzzer integrated into LLVM. Syzkaller \cite{Syzkaller2020} adapted for kernel testing. OSS-Fuzz deployed these at scale. Ding and Le Goues \cite{OSSFuzzBugs2021} analyzed OSS-Fuzz bugs, documenting vulnerability classes other methods missed.

Grammar-based fuzzing addressed structural validity. Random mutations rarely produce syntactically valid inputs for complex formats. Grammar-based fuzzers generate inputs according to specified grammars, ensuring validity while exploring variations.

Challenges persist. Writing fuzz harnesses remains manual and labor-intensive. Coverage plateaus occur when random mutation cannot discover paths beyond certain checkpoints.


## 2.5 AI-Enhanced Fuzzing

### 2.5.1 LLM-Based Test Case Generation

LLMs can synthesize tests from specifications, documentation, or source code rather than mutating existing inputs.

Deng et al. introduced TitanFuzz \cite{TitanFuzz2022}, generating fuzz targets for deep learning library APIs. TitanFuzz achieved higher coverage than manually written tests for several libraries and discovered previously unknown bugs.

FuzzGPT \cite{FuzzGPT2023} focused on edge cases. LLMs possess implicit knowledge of typical API usage patterns; prompting for atypical inputs discovered bugs that mutation-based fuzzers overlooked.

Fuzz4All \cite{Fuzz4All2023} targeted multiple programming languages and domains, generating both test inputs and harness code. Coverage improvements over random generation were substantial but below expert-written harnesses.

Zhang et al. \cite{LLMDriverGenEffectiveness2023} evaluated LLM-generated fuzz drivers against manual alternatives. Some achieved 80 to 90 percent of manual driver coverage; others reached only 50 percent. Performance correlated with documentation quality and API complexity.

WhiteFox \cite{WhiteFox2023} uses LLMs for compiler fuzzing. CKGFuzzer \cite{CKGFuzzer2024} augments generation with code knowledge graphs. HyLLfuzz \cite{HyLLfuzz2024} combines LLM-generated seeds with mutation-based fuzzing.

A consistent finding is the gap between syntactic validity and semantic quality. LLMs produce compiling code but may not exercise meaningful behavior.


### 2.5.2 Neural Network-Guided Fuzzing

Smaller neural networks can guide fuzzing with lower computational overhead than LLMs.

Wang et al. developed NeuFuzz \cite{NeuFuzz2019}, training networks to predict inputs discovering new coverage. NeuFuzz achieved 15 to 30 percent coverage improvements over AFL.

She, Woo, and Brumley introduced NEUZZ \cite{NEUZZ2018}, using networks to approximate program gradients. Neural approximation proved effective despite programs containing discrete branches.

CoCoFuzzing \cite{CoCoFuzzing2021} tested whether code models like Codex were vulnerable to adversarial inputs. Code models could be fooled by carefully crafted inputs.

Neural-guided approaches require training data for each target program. Transfer learning offers partial solutions, but models do not always generalize.


### 2.5.3 Reinforcement Learning Applications

Reinforcement learning formulates fuzzing as a sequential decision problem: select mutation strategy, observe outcome, adjust future decisions.

Böttinger, Godefroid, and Singh \cite{DeepRLFuzz2018} demonstrated RL agents could learn mutation strategies superior to static heuristics.

BertRLFuzzer \cite{BertRLFuzzer2023} combined BERT with RL, using the language model to understand input structure and RL to determine mutation locations. The combination outperformed either approach alone.

RL approaches face reward sparsity and substantial training costs, limiting applicability.


## 2.6 C and Safety Standards

MISRA C defines coding guidelines eliminating dangerous C constructs from safety-critical software. The guidelines restrict dynamic memory allocation, recursion, and pointer operations.

CERT C Secure Coding Standards address buffer overflows, integer overflows, and format string vulnerabilities.

ISO 26262 specifies functional safety requirements for automotive systems, defining Automotive Safety Integrity Levels (ASIL A through D). Testing is central to compliance.

ISO/SAE 21434 addresses automotive cybersecurity, explicitly mentioning fuzzing as a relevant testing technique.

AUTOSAR standardizes software architecture for automotive applications. Standardized interfaces facilitate automated testing.


## 2.7 Research Gaps and Thesis Positioning

Analysis reveals several gaps:

Integration with development workflows is underexplored. Klooster et al. \cite{CiCdFuzzing2022} identified challenges including speed, reliability, and determinism that benchmark evaluations do not capture.

C-specific evaluation is limited. Most LLM fuzzing research targets Python or JavaScript. Works such as ECG \cite{ECG2024} and GDBFuzz \cite{GDBFuzz2023} address embedded systems but remain exceptions.

Semantic quality lacks established metrics. Whether generated tests exercise meaningful behavior is rarely assessed systematically.

Automotive-specific evaluation is rare. SAFLITE \cite{SAFLITE2024} and CAN bus testing research \cite{CANBusFuzzAI2021} demonstrate feasibility but do not provide comprehensive frameworks.

This thesis investigates LLM-based fuzz harness generation for C libraries in automotive contexts.


## 2.8 Automotive Software Development

Development cycles in automotive require 3 to 5 years from conception to production. Software freezes months before launch.

Supply chain complexity means vehicles incorporate software from numerous suppliers. OEMs may receive binaries without source access.

The V-model remains dominant. Requirements flow down through design phases; verification flows up through testing levels.

Software-defined vehicles represent change. Companies like CARIAD bring development in-house. Over-the-air updates enable post-production modifications.

Cybersecurity requirements have increased. UNECE WP.29 R155 mandates capabilities throughout the vehicle lifecycle. Fuzzing supports compliance by identifying vulnerabilities.

CI Spark \cite{CISpark2023} indicates industry interest in LLM-assisted testing for CI/CD integration.
