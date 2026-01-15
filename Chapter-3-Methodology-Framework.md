# 3. METHODOLOGY AND FRAMEWORK

This chapter describes the research methodology used to investigate how large language models can be integrated into automotive software fuzzing workflows. The approach presented here was developed through practical work at CARIAD SE from May to September 2025. We explain the conceptual foundations of our framework, the technical design that supports it, and the three phase research strategy that guided our work from initial experiments to enterprise deployment.

The methodology was shaped by several factors. It needed to balance academic goals with industrial needs, since the research took place within a real automotive software development environment. It also needed to adapt to the fast changing field of large language models, where new models appear regularly. Most importantly, it needed to produce results that could guide both immediate deployment decisions and future research directions.


## 3.1 Conceptual Framework: AI Driven White Box Fuzzing

The conceptual framework for this research combines established ideas from software security testing with new possibilities created by large language models. To explain our approach clearly, we first review existing fuzzing methods and then describe how LLM integration changes the picture.

### Background on Fuzzing Approaches

Fuzzing has been used to find software bugs since the late 1980s when researchers at the University of Wisconsin showed that random input could crash many Unix programs. Since then, the field has developed two main approaches that differ in how much they know about the program being tested.

Black box fuzzing treats the target program as a closed box. The fuzzer knows nothing about the source code or internal workings of the program. It generates inputs based only on what it can observe from outputs and any available format specifications. This approach is simple to set up and works with any program, but it explores the input space slowly because it cannot tell which inputs are likely to find new program behaviors. Random changes proceed without guidance, and reaching deep code paths often takes very long fuzzing runs.

White box fuzzing takes the opposite approach by using detailed analysis of the program. Methods like symbolic execution build mathematical models of program paths and use solvers to generate inputs that reach specific code locations. This can explore program behavior systematically, but it faces scaling problems. The number of possible paths grows very fast as programs get larger, limiting how well these methods work on real software. The computational cost of the analysis adds further overhead.

Grey box fuzzing sits between these two extremes and has proven very effective in practice. Tools like AFL instrument the target program to collect simple coverage feedback during execution. This feedback guides a genetic algorithm that evolves the input set toward greater coverage without needing expensive analysis. The coverage information provides enough guidance to explore efficiently while keeping overhead low enough for practical use. AFL and similar tools have found thousands of bugs in production software and form the basis for many industrial fuzzing systems.

### The Fuzz Driver Problem

Despite the success of grey box fuzzing, one major limitation remains that motivated our research. This is the need for test harnesses, commonly called fuzz drivers. A fuzz driver is a small program that reads input from the fuzzer and uses it to exercise target code in a controlled way. Writing good fuzz drivers requires understanding both the target API and how the fuzzer works.

Consider a typical C++ library with hundreds of public functions across many classes. A security engineer who wants to fuzz this library must identify which functions are good targets, understand the types and valid values for parameters, construct sequences of API calls that test meaningful functionality, and handle memory and resource cleanup properly. This work can take hours to days for each target function, and the drivers need updates as the library changes.

The literature review in Chapter 2 showed that this driver generation problem is a key barrier to fuzzing adoption, especially in environments like automotive software development where codebases are large and security testing resources are limited. Previous attempts to automate driver generation used static analysis rules or templates, but these struggle with the understanding needed to build meaningful test scenarios.

### Using LLMs for Driver Generation

Our research tested whether large language models could automate fuzz driver generation. This idea rests on several observations about what LLMs can do.

Modern LLMs show strong ability to understand code meaning. Models like GPT 4 and specialized code models can explain what functions do, find bugs, suggest fixes, and write code that compiles and runs correctly. This understanding goes beyond simple pattern matching to include relationships between functions, common usage patterns, and standard code structures.

LLMs are also good at tasks that require combining information from multiple sources. Creating a good fuzz driver requires putting together information from function signatures, documentation, example code, and general knowledge about the programming language. This is exactly the kind of task where LLMs perform well.

The fuzzing context also provides clear success measures that support improvement over time. Unlike open ended code generation where quality is subjective, fuzz drivers either compile or they do not. They either achieve coverage or they do not. This creates a tight feedback loop that can guide prompt engineering and model training.

### Our Approach

Based on these observations, we developed an approach that positions the LLM as an intelligent front end to standard fuzzing tools. The LLM handles the cognitive work of understanding target code and building test harnesses, while proven fuzzing engines handle the execution and mutation that need speed and reliability.

Figure 3.1 shows our LLM fuzzing workflow. The process starts when source code for the target library is given to the LLM along with context like documentation and examples. The LLM analyzes this information and produces two outputs. The first output path goes through Define Target to Fuzz Tests Generation, producing the actual fuzz driver code. The second output path produces Seed Inputs that serve as starting points for mutation.

Both outputs feed into the Fuzzing Engine. Inside the engine, tests are executed and the system checks whether a crash was detected. If yes, reports are generated for analysis. If no crash occurs, the inputs are mutated and the cycle continues with new test execution. The LLM is not involved in this execution loop, which avoids the latency and cost that would come from calling the model for every mutation.

This design keeps what works well in existing fuzzing practice while addressing the driver generation problem. It also maintains clear separation of concerns. The LLM provides intelligence during setup, while the fuzzing engine provides speed during execution. This separation proved important for practical deployment.

### Assumptions and Scope

Our framework rests on three assumptions that we tested through experiments. First, we assumed that current LLMs have enough code understanding to work with C and C++ library interfaces. Second, we assumed that generated fuzz drivers would compile successfully often enough to be practically useful. Third, we assumed that any coverage gaps could be improved through model training rather than fundamental changes to the approach.

One important difference from some prior work is our focus on the setup phase only. Some approaches use the LLM as a mutation operator within the fuzzing loop, calling it to generate each new input. We use the LLM only for driver generation and initial seed creation. This reflects practical limits. Calling an LLM for every mutation would be too slow and expensive for CI/CD use. By limiting LLM involvement to the setup phase, we keep fuzzing fast while still gaining from AI assisted automation.


## 3.2 Technical Architecture Design

Moving from concept to working system required careful design decisions. The technical architecture needed to meet several requirements that reflected both research goals and industrial deployment needs.

### Requirements

We identified five main requirements through discussions with security engineers and CI/CD teams at CARIAD.

First, the architecture needed to support experiments with different LLM setups. During research, we needed to test different models, prompt strategies, and training approaches. The design could not be tied to one specific model or API.

Second, the architecture needed to allow local development and testing. Security engineers should be able to run the complete workflow on their own machines without needing network services. This supports iterative work and debugging.

Third, the architecture needed to work with enterprise CI/CD systems. The goal was automated fuzzing as part of continuous integration pipelines. This meant working with CARIAD's GitHub Enterprise instance, buildah container system, and JFrog Artifactory for storing artifacts.

Fourth, the architecture needed to handle security concerns around generated code. Fuzz drivers run within the build environment and might access source code and secrets. The design needed to support validation and isolation.

Fifth, the architecture needed to manage cost and latency. Cloud LLM APIs charge per token and add network delay. The design should minimize unnecessary API calls and support caching where useful.

### Layered Design

We used a layered architecture with clear interfaces between parts. This supports the need for multiple LLM backends while keeping interfaces consistent for other components.

The input layer handles source code parsing and context preparation. For C and C++ targets, which make up most automotive security testing work, this layer parses header files to get function signatures, type definitions, and documentation comments. We use tree sitter for parsing syntax and custom code for extracting meaning like parameter constraints inferred from naming patterns.

The input layer produces structured descriptions of target functions suitable for LLM prompts. Each description includes the function signature, any available documentation, parameter types with their definitions, and examples of function usage from test files or sample code. This structured information allows consistent prompt building across different LLM backends.

The generation layer manages interaction with LLM services. An interface defines operations for sending prompts and receiving responses. Concrete implementations exist for different backends: Ollama for local models, the OpenAI API for cloud models, and Azure OpenAI Service for enterprise deployment. This abstraction proved essential during Phase 3, when we had to switch between endpoints depending on network access.

The generation layer also handles response parsing and validation. LLM outputs are not always clean code. The layer includes logic for extracting code blocks from responses, fixing common formatting problems, and rejecting clearly broken outputs before they reach later stages.

The execution layer works with cifuzz for fuzzing campaign management. cifuzz builds on LLVM libFuzzer and provides features for corpus management, coverage tracking, and crash grouping. The execution layer takes generated drivers and seed inputs, compiles them with sanitizer instrumentation, runs fuzzing campaigns, and collects metrics.

Communication between layers uses files. The input layer produces JSON files with parsed source information. The generation layer reads these and produces driver source files and seed input files. The execution layer picks up these files and sets up campaigns. This loose connection made debugging easier and let team members work on different parts at the same time.

### Prompt System

An important design choice was separating prompt building from model calls. Early versions had prompt templates written directly in the code, making experiments difficult. We changed to use external template files with placeholder replacement.

The template system supports several useful features. Templates can include conditional sections that appear or not based on available context. For example, if documentation exists for a target function, the template includes a documentation section. Otherwise it is left out to avoid confusing the model.

Templates can also reference other templates for composition. A base template sets the overall structure, while specialized templates add specific context. For automotive security testing, we made templates that include information about common automotive protocols and safety considerations.

The system logs all prompts used during generation, allowing later analysis of which prompt forms led to successful outputs. This logging helped with prompt improvement in Phase 2.

### Integration with cifuzz

The execution layer builds on cifuzz and its spark feature. cifuzz spark handles LLM integration for fuzz test generation within the cifuzz system. After setting up cifuzz in a project and configuring CMake, running cifuzz spark examines the code and generates fuzz test files.

Integration requires adding cifuzz to the build system. For CMake projects, this means adding find_package(cifuzz) and enable_fuzz_testing() commands. A subdirectory called cifuzz spark holds generated tests. The build is configured to compile with appropriate sanitizers, typically AddressSanitizer and UndefinedBehaviorSanitizer.

Environment variables configure the LLM backend for cifuzz spark. CIFUZZ_LLM_API_URL sets the inference endpoint. CIFUZZ_LLM_MODEL or CIFUZZ_LLM_AZURE_DEPLOYMENT_NAME sets the model. CIFUZZ_LLM_API_TOKEN provides authentication. CIFUZZ_LLM_MAX_TOKENS controls response length. This setup allows switching backends without code changes.

The execution layer also manages corpus storage. Fuzzing corpora collect inputs that exercise different code paths. Keeping corpora across CI/CD runs lets coverage build up over time rather than starting fresh each build. We set up corpus upload and download through JFrog Artifactory, with compression and checksums to manage storage costs.


## 3.3 Research Strategy: Three Phase Approach

Rather than trying to build everything at once, we split the research into three distinct phases. This reflected both practical limits and good methodology. Each phase required different infrastructure and skills. Breaking work into phases with evaluation checkpoints let us adjust our approach based on what we learned.

The three phases were: local LLM evaluation to establish baseline abilities and find promising model setups, model optimization using LoRA fine tuning to address limits found in Phase 1, and enterprise CI/CD integration to test practical deployment. Each phase had clear goals, success criteria, and decision points.


### 3.3.1 Phase 1: Local LLM Evaluation

The first phase established what current LLMs can achieve for fuzz driver generation. We focused on local model deployment for several reasons.

Local deployment removed cost as a limit on experiments. Cloud API pricing would have restricted how many setups we could test. With local models, we could run many experiments without ongoing costs beyond electricity and hardware.

Local deployment avoided network problems that could affect reproducibility. Cloud API behavior can change over time as providers update their models. Local models stay stable, allowing reliable comparisons.

Local deployment also matched our target for some use cases. Not all organizations can connect CI/CD systems to cloud services because of security policies. Understanding local model performance was directly relevant to these situations.

#### What We Aimed to Learn

Phase 1 had three main goals. First, find out whether current LLMs can generate fuzz drivers that compile and run at all. Before our research, this was not proven for C and C++ automotive libraries with complex APIs.

Second, understand how performance varies across models, model sizes, and prompt strategies. This knowledge would guide model selection for production and show where optimization might help.

Third, identify failure patterns that training or prompt changes might fix. Not all failures are the same. Syntax errors from misunderstanding language rules suggest different fixes than meaning errors from misunderstanding API contracts.

#### Model Selection

We picked models based on three criteria: availability through Ollama for consistent serving, documented performance on code benchmarks, and memory needs that fit our GPU hardware.

The models tested covered a range of sizes. Smaller models around 7 billion parameters included CodeLlama 7B and Qwen2.5 coder 7B. These showed the lower limit of what might work for local deployment with fast inference. Medium models around 13 to 14 billion parameters included CodeLlama 13B and Qwen2.5 coder 14B. Large models around 27 to 34 billion parameters included Qwen2.5 coder 32B, Gemma3 27B, DeepSeek Coder 33B, CodeLlama 34B, and WizardCoder 34B. Very large models beyond 40 billion parameters included Mixtral at 46.7 billion parameters.

We also tested cloud models as reference points. GPT 4o through Azure OpenAI showed what current cloud technology can achieve. Comparing local models against this reference helped measure the gap that training might close.

#### Target Selection

Choosing the right target code was important because results might not apply to all code types. We picked 25 C and C++ repositories with variety along several dimensions.

Complexity ranged from small focused libraries to large feature rich frameworks. API style ranged from C function interfaces to heavily templated C++ designs. Documentation ranged from complete API references to minimal comments. Existing test coverage ranged from extensive test suites to almost no testing.

Main targets included yaml cpp for YAML parsing, pugixml for XML processing, jsoncons for JSON handling, fmt for text formatting, spdlog for logging, and glm for graphics math. These libraries are commonly used in automotive software for configuration, data handling, logging, and graphics in infotainment systems.

For yaml cpp specifically, we found 35 source files with 1061 functions as fuzzing candidates. This size gave enough data points for analysis while staying manageable for detailed study of failures.

#### Testing Protocol

Each model was tested using a standard process to allow fair comparison. For each target function, we generated 10 driver candidates using the same prompts with different random seeds where the model supported this. Repeating helped separate consistent failures from random variation.

Generated drivers were compiled without changes using Clang with AddressSanitizer and UndefinedBehaviorSanitizer. We recorded outcomes as success, syntax errors, type errors, linker errors, or other errors. This grouping helped us understand failure patterns.

Drivers that compiled were run in 10 minute fuzzing campaigns. We recorded whether execution worked, whether target functions were actually called, and what coverage was achieved. Coverage was measured using LLVM instrumentation.

Drivers that passed initial tests were extended to 24 hour campaigns to check crash detection. Any crashes were checked to separate bugs in target code from bugs in generated drivers.


### 3.3.2 Phase 2: Model Optimization with LoRA

Phase 1 showed large performance gaps between smaller local models and cloud models. GPT 4o achieved about 91 percent compilation success on good targets, while local 7B models achieved around 40 percent. Phase 2 tested whether fine tuning could narrow this gap.

#### Why LoRA

We chose Low Rank Adaptation as the fine tuning method for practical reasons.

LoRA adds small trainable matrices to attention layers while keeping base model weights fixed. This greatly reduces memory needs compared to full fine tuning. Full fine tuning of a 7 billion parameter model might need 60 gigabytes of GPU memory, but LoRA fine tuning can work with 16 to 24 gigabytes. This made training possible on our available hardware.

LoRA also cuts training time substantially. Full fine tuning updates all model parameters and needs many calculations per example. LoRA updates only the small adapter matrices. Training runs finish in hours rather than days.

The adapter design allows modular experiments. We could train multiple adapters with different data and swap them for testing without running expensive training jobs again. This supported the iterative experiments our work required.

#### Training Data

Training data came from two sources selected to show good fuzzing practice.

The first source was OSS Fuzz, the project maintained by Google. OSS Fuzz continuously fuzzes important open source software and keeps fuzz drivers written by experienced security engineers. We extracted drivers from 47 projects, picking examples that showed clear API usage, proper resource handling, and effective seed generation. After filtering and removing duplicates, this gave 623 examples.

The second source was successful outputs from Phase 1. When LLMs generated drivers that compiled, ran well, and achieved good coverage, we reviewed them and added quality examples to the training set. This added 184 examples in the style our prompts encouraged.

We tested two dataset sizes: a smaller set of 172 examples for quick iteration on settings, and a larger set of 709 examples for final training. The smaller set allowed faster experiment cycles while developing the training process.

Training examples were formatted as instruction and completion pairs. The instruction included the function signature, documentation, and type definitions. The completion was the fuzz driver code. This format matched how we structured prompts for generation.

#### LoRA Settings

After initial experiments with various settings, we used the following configuration.

Rank was set to 16. Rank controls the size of the adaptation matrices and thus how much the model can learn. Higher ranks give more capacity but risk learning the training data too specifically. Our tests suggested rank 16 balanced these concerns for our dataset sizes.

Alpha was set to 32, following common practice of setting alpha to twice the rank. Alpha controls how much the adapters affect the final output.

Dropout was set to 0.1 to prevent learning the training data too specifically. With our relatively small training sets, some regularization was needed.

Target modules included the query, key, value, and output projection matrices in attention layers. This is the standard setup for code generation and covers the parts most relevant to output.

Training ran for 3 epochs with batch size 4 and learning rate 0.0002 with cosine scheduling. Precision was float16 for memory efficiency.

#### Results

We tested fine tuned models using the same process as Phase 1 to allow direct comparison. For each target, we compared base model performance against fine tuned versions.

Results showed clear improvements. On yaml cpp, compilation success went from 40 percent to 58 percent for the 7B model with LoRA trained on 709 examples. Token use decreased and generation got faster, suggesting the model became more efficient at this specific task.

Comparing the 172 and 709 example training showed the larger dataset gave about 33 percent efficiency gain over the smaller set, which itself showed 24 percent gain over baseline. This suggested more training data would likely give further improvements.

We also found that combining fine tuning with optimized prompts gave additive benefits. The fine tuned model with carefully designed prompts approached the performance of much larger base models on some targets.

#### Generalization Check

A concern with fine tuning on narrow datasets is learning the training examples too specifically rather than general skills. To check generalization, we held out some target functions that were not in training data and tested on these specifically.

Results showed some drop in performance on held out functions. This indicated the fine tuned models had partly memorized patterns from training targets rather than learning fully general driver construction skills. This finding suggested production deployment would benefit from larger and more varied training sets than we could create in the project time.


### 3.3.3 Phase 3: Enterprise CI/CD Integration

The final phase moved from controlled experiments to deployment within CARIAD's production CI/CD systems. This brought challenges that needed solutions beyond the technical design from earlier phases.

#### Enterprise Environment

CARIAD's CI/CD system reflects security needs appropriate for automotive software development. Source code for vehicle systems is valuable intellectual property and a potential attack target if exposed. The system therefore uses defense in depth.

GitHub Enterprise runs on infrastructure isolated from the public internet. CI/CD workflows run on self hosted runners set up as temporary containers. These runners have no default network access beyond the internal corporate network. Reaching external services needs explicit setup and security review.

Build environments are temporary to limit damage from any compromise. Runners are destroyed after each workflow run and rebuilt from clean images. This prevents attacks that need persistent access.

Secrets management uses GitHub's built in storage with access controls based on repository and workflow settings. Secrets are given to workflows as masked environment variables.

Artifact management uses JFrog Artifactory with authentication needed for all access. Build outputs including fuzzing corpora are stored in Artifactory with appropriate retention rules.

#### Integration Goals

Phase 3 had three goals beyond simply making things work. First, integration should need minimal changes to existing workflows. Teams already have established CI/CD patterns, and disrupting these creates friction that slows adoption. Second, integration should provide clear value visible to developers. Abstract security improvements are less motivating than concrete outputs like coverage reports or bug discoveries. Third, integration should keep the security properties that existing systems provide.

#### Network Challenge

The basic challenge was connecting CI/CD runners to LLM services. This showed up differently depending on deployment approach.

For cloud LLM APIs, runners needed to reach endpoints on the public internet. Default network policy blocked this completely. Proxy setups that worked in development did not work on runners because the proxy systems were not available in that network area.

For local model deployment, runners lacked GPU resources for acceptable speed. CPU inference was possible but too slow. Generating one driver took over five minutes, far too long for CI/CD pipelines.

#### Solution

After exploring multiple approaches that did not work, which we describe in Chapter 4, we found a solution using Azure Private Link. Azure Private Link lets Azure services be reached through private IP addresses within a virtual network rather than through public internet endpoints. This met security needs while providing access to Azure OpenAI Service.

Setting this up required coordination across teams. The infrastructure team configured network policies. The Azure platform team set up the private endpoint. The security team reviewed and approved the configuration. This coordination took about six weeks, making up most of the Phase 3 timeline.

With network access established, the integration itself was straightforward. We configured self hosted runners with needed environment variables, updated workflow definitions to call cifuzz spark before fuzzing, and set up corpus management through Artifactory.

#### Rollout

We tested the integration through staged rollout starting with volunteer teams whose libraries had been part of earlier testing. Initial deployment covered one library for two weeks of observation. After confirming stable operation, we extended to more teams gradually.

Monitoring covered both technical measures and developer experience. Technical measures included API response time, generation success rate, and resource use. Developer experience was checked through informal feedback and watching how teams worked with generated drivers.


## 3.4 Evaluation Methodology

Good evaluation was essential for producing useful research conclusions. This section describes how we evaluated the work, including what we measured, how we validated measurements, and what limitations we acknowledge.


### 3.4.1 Metrics and Performance Indicators

We collected metrics in four categories: generation quality, fuzzing effectiveness, resource use, and practical value. Each category addresses different aspects of the research questions.

#### Generation Quality

Compilation success rate measured what percentage of generated drivers compiled without errors. This is the most basic quality check. A driver that does not compile provides no value. We measured this before any manual fixes to assess raw LLM ability.

For failed compilations, we grouped errors to understand failure patterns. Categories included syntax errors showing misunderstanding of language grammar, type errors showing misunderstanding of API types, linker errors showing missing dependencies, and other errors. This grouping informed both prompt design and training data selection.

Functional correctness checked whether compiled drivers actually used target functions meaningfully. A driver might compile but call functions with obviously wrong parameters that cause immediate errors. We detected this through coverage analysis. Drivers that achieved no coverage of target function code were marked as functionally incorrect.

#### Fuzzing Effectiveness

Code coverage measured what percentage of target code was run during fuzzing. We used LLVM source based coverage, reporting both line coverage and branch coverage. Coverage was measured after fixed time campaigns to allow comparison.

Coverage improvement measured the gain from LLM generated drivers compared to baseline approaches. For targets with existing OSS Fuzz drivers, baseline was coverage from those established tests. For targets without existing fuzz setup, baseline was coverage from AFL with random starting inputs.

Crash detection checked whether generated drivers found actual bugs in target code. This measure is naturally noisy because finding crashes depends on both driver quality and random factors in fuzzing. We used long campaigns and multiple runs to reduce noise.

#### Resource Use

Token use tracked LLM API usage per driver generation. This directly relates to cost for cloud deployment. We recorded prompt tokens and completion tokens separately because they have different prices.

Generation time measured how long driver generation took. This affects CI/CD integration because long generation extends pipeline time. We measured total time including network delays for API calls.

Compute resource use tracked GPU memory, CPU use, and RAM during local model inference and training. These measures informed sizing for production deployment.

#### Practical Value

Developer time saved estimated manual effort avoided through automated generation. We established baseline estimates through interviews with security engineers about typical driver development times. Automated generation got credit for saving time when it produced usable drivers, adjusted for time spent reviewing and fixing generated output.

Adoption measures tracked how development teams engaged with the automated fuzzing system. Measures included number of teams using the system, how often fuzzing ran, and whether teams kept automated generation enabled or turned it off after initial trial.


### 3.4.2 Validation and Verification Approach

Numbers alone need validation to ensure they measure what we intend. We used several validation methods.

#### Expert Review

A sample of generated drivers was reviewed by experienced security engineers using the same standards they apply to manually written drivers. Reviewers assessed code quality, correct API usage, test effectiveness, and maintainability. This qualitative check complemented the numbers and helped interpret results.

Expert review sometimes disagreed with the numbers. Some drivers achieved high coverage scores while showing patterns that experts marked as problems, like excessive resource use or depending on implementation details that might change. These cases helped us understand the limits of our measures.

#### Baseline Comparison

Where possible, we compared LLM generated drivers against established baselines. For targets with mature OSS Fuzz coverage, comparison against existing drivers gave a ceiling estimate of achievable quality. The gap between generated and expert written drivers showed remaining room for improvement.

For targets without existing fuzz setup, we created baselines through manual driver development. Security engineers wrote drivers for a subset of targets following their normal process. Generated drivers were compared against these manual baselines on the same measures.

#### Reproducibility

LLM outputs are not deterministic, adding variation that could hide true performance differences. We addressed reproducibility through repetition. Each test condition was repeated multiple times with different random seeds where supported. We report means with standard deviations to show variation.

Fuzzing itself is random. Finding crashes depends on mutation sequences that differ between runs. We used standard campaign lengths and reported results combined across multiple runs.

For key findings, we ran statistical tests to check significance. When comparing model setups, we used appropriate tests to decide whether observed differences likely reflected true performance differences versus random variation.

#### Limitations

We acknowledge several limitations of our evaluation approach. Target repository selection, while varied, covers only a small sample of possible fuzzing targets. Results might differ for other languages, larger libraries, or code with different characteristics.

The evaluation happened over a limited time with specific model versions. LLM abilities change quickly, and results might differ with updated models. We document model versions used to support future comparison.

Enterprise deployment measures reflect one organization's systems and processes. Other organizations with different CI/CD setups might face different challenges or achieve different results.

These limitations define the scope of our conclusions rather than making our findings invalid. We present results as evidence of what is possible within described bounds rather than universal guarantees of performance.
