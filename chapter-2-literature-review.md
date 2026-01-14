# Chapter 2: Literature Review

## 2.1 Introduction and Scope

I'll be honest—when I first started reading the fuzzing literature, I didn't expect to find papers from 1988 still being cited. But there they were. Barton Miller's original fuzzing experiments keep showing up because the fundamental problem hasn't changed: software crashes when it encounters unexpected input. What has changed, dramatically, is how we generate that input.

This chapter traces the evolution from random byte generation to AI-driven test synthesis. The journey isn't linear. Some approaches that looked promising in 2015 turned out to be dead ends. Others, dismissed as too expensive or impractical, have become viable thanks to cheaper compute and better models. My goal here isn't to catalog every paper ever written on the topic—that would take forever and bore everyone—but to identify the threads that matter for understanding where LLM-based fuzzing fits in the bigger picture.

The scope needs some boundaries. I'm primarily interested in C code because that's what automotive systems run on. Python gets mentioned because most LLM research happens there first, but the challenges differ significantly. Fuzzing a web application is not the same as fuzzing an embedded ECU. Memory safety, real-time constraints, limited resources—these things matter in automotive contexts and they shape what testing approaches are feasible.

One more thing before we dive in. The literature on AI and fuzzing has exploded since 2022. A recent survey by Huang et al. \cite{LLMFuzzSurvey2024} identified over 40 papers on LLM-based fuzzing published in just two years. I've tried to focus on the work most relevant to our research questions, but there's certainly more out there. The field moves fast, and by the time you read this, there will be new papers I haven't covered. That's the nature of working at the frontier.


## 2.2 Traditional AI Approaches in Software Testing

Before GPT-4 could write code, researchers were already trying to make machines smarter about testing. The approaches were different—less "generate a test from scratch" and more "optimize which tests to run"—but the underlying motivation was the same: humans can't test everything manually, so we need help.

Genetic algorithms were among the first AI techniques applied to testing. The idea was borrowed from biology: maintain a population of test cases, let them "reproduce" through crossover and mutation, and select the fittest for the next generation. Wang et al. \cite{MLFuzzReview2020} reviewed this history systematically, noting that evolutionary approaches dominated the field from the late 1990s through the early 2010s. The appeal was obvious—you didn't need to understand the program deeply, just define a fitness function and let evolution do the work.

But there were problems. Setting up the fitness function wasn't trivial. Coverage seemed like an obvious choice, but maximizing coverage alone didn't necessarily find bugs. You could achieve 90% line coverage with tests that never triggered edge cases. Researchers experimented with multi-objective optimization, trying to balance coverage, execution time, and other metrics. It helped, but the fundamental limitation remained: genetic algorithms couldn't reason about what the code actually did.

Symbolic execution took a different tack. Instead of searching randomly, it tried to think. The program becomes a system of constraints; solving those constraints gives you inputs that reach specific code paths. Godefroid, Levin, and Tauth demonstrated this with SAGE \cite{SAGE2008}, which Microsoft used internally for years. The results were impressive for their time—SAGE found bugs in Windows components that traditional testing missed.

The catch? Path explosion. A program with just 10 if-statements has over 1,000 paths. Add loops, and the number becomes effectively infinite. Yavuz's recent work on DESTINA \cite{DESTINA2024} attempts to address this through targeted execution, but the fundamental scaling challenge persists. Symbolic execution works beautifully on small programs and struggles with real-world codebases.

Machine learning started appearing in testing research around 2015, initially in supporting roles. Researchers at Sandia National Laboratories \cite{MLFuzzReviewSandia2019} surveyed how ML was being used: predicting which modules were likely to have bugs, prioritizing test cases, estimating coverage without running tests. These applications were useful but limited. The ML models weren't generating anything—they were just helping humans make decisions faster.

Neural networks brought the first hints of something different. Pei et al. \cite{DeepXplore2017} introduced DeepXplore for testing deep learning systems, using neuron coverage as a testing criterion. Odena and colleagues \cite{TensorFuzz2019} applied coverage-guided fuzzing to neural networks themselves with TensorFuzz. These papers showed that AI could do more than optimize—it could guide the testing process in ways that surprised even the researchers.

The real shift came when people started asking: what if we could generate tests directly, not just select or mutate them? That question led to the current era of LLM-based approaches.


## 2.3 Large Language Models: Evolution and Code Generation

I remember the first time I saw GPT-3 generate code. It was 2021, and a colleague had gotten API access. He typed a comment—"function to reverse a string"—and the model completed it. The code was correct. We tried more complex examples. Most worked. Some didn't. But the fact that any of them worked felt like magic.

That feeling has faded as LLMs have become ubiquitous, but the underlying capability remains remarkable. A model trained primarily to predict the next token in a sequence can, somehow, produce syntactically and often semantically valid programs. How it does this is still being debated. Whether it "understands" code in any meaningful sense is philosophical quicksand I'd rather avoid. What matters for our purposes is that it works, most of the time, for certain kinds of tasks.

The evolution was rapid. GPT-3 in 2020 could generate simple functions. Codex in 2021, trained specifically on code, powered GitHub Copilot and could complete entire methods. GPT-4 in 2023 showed qualitative improvements—better at understanding context, more reliable with edge cases, less likely to produce code that looked right but was subtly wrong. Each generation expanded what was possible.

Open-source alternatives emerged as counterweights to OpenAI's dominance. Meta's LLaMA showed that smaller models could be competitive with careful training. Code-specific variants like StarCoder and CodeLLaMA pushed the state-of-the-art for code generation. By 2024, organizations had multiple options for code-generating LLMs, including models they could run locally without sending proprietary code to external APIs.

But capability isn't the same as reliability. Deng et al. \cite{TitanFuzz2022} found that while LLMs could generate syntactically valid code in most cases, the generated code often had subtle issues. In their experiments with TitanFuzz, models produced tests that compiled successfully but exercised code paths in unexpected ways—sometimes revealing bugs, sometimes just revealing that the model didn't fully understand the API. Wang et al. \cite{LLMMutationTesting2024} systematically studied LLMs for mutation testing and found similar patterns: high success rates on simple cases, degrading performance as complexity increased.

For C specifically, the picture is mixed. Most LLM training data comes from Python, JavaScript, and other high-level languages. C is less represented, and its unique challenges—pointer arithmetic, manual memory management, undefined behavior—aren't things LLMs handle naturally. Papers focusing specifically on C code generation are rarer, and when researchers do evaluate on C, results tend to be worse than on Python.

This matters for automotive applications. Almost all embedded automotive software is written in C. If LLMs struggle with C, their utility for automotive testing is limited regardless of how well they perform on Python benchmarks. This gap between LLM capabilities and automotive needs is one of the motivations for our research.

Context length has been another evolving constraint. Early models had 2K or 4K token windows—barely enough for a single file plus a prompt. GPT-4 expanded to 8K, then 32K, and eventually 128K tokens. This expansion enables new use cases. With 128K tokens, you can include multiple source files, header definitions, example usages, and detailed instructions. Whether models actually use all that context effectively is another question, but at least the possibility exists.


## 2.4 Fuzzing Techniques: Foundations to AI-Guided Methods

Fuzzing began as a happy accident. Barton Miller was teaching a graduate seminar in 1988 and noticed that programs crashed when fed garbage input. His students tested standard UNIX utilities—cat, grep, ls—and found that about a quarter of them failed. The finding was simultaneously embarrassing (this was production software!) and illuminating (there was a systematic way to find bugs).

Early fuzzing was pure chaos. Generate random bytes, feed them to the program, see what happens. This "dumb fuzzing" approach required no understanding of the target and could be applied to anything that accepted input. Its simplicity was its strength. But random input rarely exercises interesting code paths. Most programs validate input early; random garbage fails validation and never reaches the complex logic where bugs hide.

Coverage-guided fuzzing changed everything. Michal Zalewski's AFL, released in 2013, introduced lightweight instrumentation to track which code paths each input explored. If an input discovered a new path, AFL saved it for further mutation. If not, it was discarded. This feedback loop made fuzzing dramatically more efficient. Instead of hoping random mutations would stumble into interesting territory, AFL systematically mapped the program's behavior.

AFL spawned an ecosystem. AFL++ \cite{AFLPlusPlus} added performance optimizations and new mutation strategies. LibFuzzer integrated fuzzing into LLVM's toolchain. Syzkaller \cite{Syzkaller2020} adapted coverage-guided techniques for kernel testing. Google's OSS-Fuzz project applied these tools at scale, continuously fuzzing hundreds of open-source projects. Ding and Le Goues \cite{OSSFuzzBugs2021} analyzed bugs found by OSS-Fuzz and found that fuzzing discovered vulnerability classes that other testing methods missed.

Grammar-based fuzzing addressed a different limitation. Random mutations rarely produce structurally valid inputs for complex formats. A mutated JSON file is almost certainly invalid JSON. Grammar-based fuzzers generate inputs according to specified grammars, ensuring syntactic validity while still exploring variations. This approach proved particularly valuable for protocol parsers and file format handlers.

Despite these advances, traditional fuzzing hit walls. Writing fuzz harnesses—the glue code connecting the fuzzer to the target—remained tedious manual work. Understanding which functions to fuzz and how to call them required human expertise. Coverage plateaus emerged when fuzzers explored all paths reachable by random mutation but couldn't progress further. These limitations created openings for AI-enhanced approaches.


## 2.5 AI-Enhanced Fuzzing: State-of-the-Art

The marriage of AI and fuzzing is recent but productive. Research has proceeded along multiple tracks: using neural networks to guide mutation, using reinforcement learning to learn fuzzing strategies, and—most recently—using LLMs to generate tests and harnesses directly.


### 2.5.1 LLM-Based Test Case Generation

The idea is straightforward: if LLMs can generate code, maybe they can generate tests. The first serious attempts emerged in 2022, and the field has accelerated since.

Deng et al. introduced TitanFuzz \cite{TitanFuzz2022}, which used LLMs to generate fuzz targets for deep learning libraries. The approach worked by prompting the model with API documentation and asking it to produce test cases that would exercise the API. Results were encouraging—TitanFuzz found bugs in TensorFlow and PyTorch that existing fuzzers had missed. But the work focused on Python libraries with extensive documentation and active developer communities. Whether similar techniques would work for less-documented C code remained unclear.

FuzzGPT \cite{FuzzGPT2023} from the same research group pushed the approach further, specifically targeting edge cases in deep learning libraries. The insight was that LLMs, having been trained on vast code repositories, had implicit knowledge of how APIs were typically used—and therefore, by inversion, how they might be misused. By prompting for unusual or boundary-case inputs, FuzzGPT found bugs that conventional fuzzers overlooked.

Fuzz4All \cite{Fuzz4All2023} by Xia et al. attempted a more universal approach, targeting multiple languages and domains. The system used LLMs to generate both test inputs and the harnesses to run them. Coverage improvements were significant compared to random generation, though still below what expert-written harnesses achieved.

Zhang et al. \cite{LLMDriverGenEffectiveness2023} directly addressed the question of how effective LLM-generated fuzz drivers really are. Their experiments compared LLM-generated drivers against manually written ones across multiple libraries. Results were mixed: for some libraries, LLM drivers achieved 80-90% of manual driver coverage; for others, they barely reached 50%. The gap correlated with documentation quality and API complexity—exactly the factors you'd expect if LLMs are relying on patterns in training data rather than true understanding.

Recent work has explored combining LLM generation with traditional techniques. WhiteFox \cite{WhiteFox2023} uses LLMs to generate targeted test cases for compiler fuzzing. CKGFuzzer \cite{CKGFuzzer2024} enhances LLM generation with code knowledge graphs, providing structural information that models can use to generate more effective tests. HyLLfuzz \cite{HyLLfuzz2024} combines LLM-generated seeds with traditional mutation, getting the best of both approaches.

One consistent finding across this work: LLMs are better at generating something than generating the right thing. They produce syntactically valid tests at high rates, but semantic quality varies. A test that compiles and runs isn't necessarily a test that exercises meaningful behavior. Evaluating this semantic quality remains an open problem.


### 2.5.2 Neural Network-Guided Fuzzing

Before LLMs took center stage, researchers explored using smaller neural networks to guide fuzzing decisions. These approaches remain relevant—they're faster and cheaper than LLM inference, making them more suitable for tight feedback loops.

Wang et al. developed NeuFuzz \cite{NeuFuzz2019}, which trained neural networks to predict which inputs would discover new coverage. The model learned from historical fuzzing data, observing which mutations tended to be productive. In experiments, NeuFuzz achieved 15-30% higher coverage than AFL on the same time budget.

She, Woo, and Brumley introduced NEUZZ \cite{NEUZZ2018}, which took a different approach: using neural networks to approximate the gradient of the target program. In smooth functions, gradients indicate which input bytes most influence output behavior. Programs aren't smooth—they have discrete branches and discontinuities—but neural approximation turned out to work surprisingly well. NEUZZ achieved substantial coverage improvements on several benchmarks.

CoCoFuzzing \cite{CoCoFuzzing2021} applied these ideas specifically to neural code models, testing whether models like Codex were themselves vulnerable to adversarial inputs. They found that code models, like image classifiers before them, could be fooled by carefully crafted inputs—a finding with implications for the reliability of AI coding assistants.

The limitation of neural-guided approaches is the training requirement. Each target program needs its own model, trained on data from fuzzing that program. This bootstrapping problem limits applicability: you need to fuzz a lot to train a model that helps you fuzz. Transfer learning offers partial solutions, but models trained on one program don't always generalize to others.


### 2.5.3 Reinforcement Learning Applications

Reinforcement learning frames fuzzing as a sequential decision problem: at each step, choose a mutation strategy, observe the result, adjust future decisions. This formulation is elegant but challenging—rewards (finding new coverage or bugs) are sparse, and the action space is enormous.

Böttinger, Godefroid, and Singh explored deep reinforcement learning for fuzzing \cite{DeepRLFuzz2018}, demonstrating that RL agents could learn mutation strategies that outperformed static heuristics. The learned policies weren't interpretable—the network knew something humans didn't—but they worked.

BertRLFuzzer \cite{BertRLFuzzer2023} combined BERT language models with reinforcement learning, using the language model to understand input structure and RL to learn where to mutate. The combination proved more effective than either approach alone, suggesting that different AI techniques complement each other.

RL approaches face practical challenges. Training is expensive—millions of interactions with the target program may be required before the agent learns useful policies. During training, performance is often worse than simple baselines. Organizations may not have patience for this investment, especially when simpler techniques work reasonably well.


## 2.6 C and Safety Standards

Automotive software lives in a different world than web applications. Reliability isn't a nice-to-have; it's legally mandated. Cars kill people when software fails. This reality shapes every aspect of development, including testing.

MISRA C defines the subset of C that automotive software may use. The full C language is powerful but dangerous—undefined behavior lurks everywhere, and a single buffer overflow can compromise an entire system. MISRA constraints eliminate the riskiest features: no dynamic memory allocation, no recursion, restricted pointer use. Code that follows these rules is harder to write but safer to run.

ISO 26262 specifies functional safety requirements across the automotive lifecycle. Testing is central to the standard, with different integrity levels (ASIL A through D) requiring different levels of rigor. Fuzzing isn't explicitly mentioned, but it can contribute to required verification activities. The question is how: demonstrating that fuzzing provides adequate coverage for certification purposes isn't straightforward.

AUTOSAR standardizes software architecture for automotive applications. From a testing perspective, AUTOSAR's standardized interfaces are a mixed blessing. On one hand, they make automated testing easier—if you know the interface, you can generate tests against it. On the other hand, AUTOSAR's complexity means that testing individual components doesn't guarantee system-level correctness.

The emerging cybersecurity standard ISO/SAE 21434 explicitly mentions fuzzing as a relevant technique for security testing. This represents a shift in how regulators view automated testing: not just a development convenience, but a required activity for demonstrating security posture. As vehicles become more connected, we can expect fuzzing requirements to become more specific.

Static analysis tools like Polyspace and Coverity complement dynamic testing. These tools find potential issues without running code, catching problems that fuzzing might take years to trigger. The combination of static analysis (find potential vulnerabilities) and fuzzing (confirm exploitability) provides stronger assurance than either alone.


## 2.7 Research Gaps and Thesis Positioning

Reading through this literature, several gaps become apparent. These gaps motivate our research and position this thesis within the broader field.

First, integration remains underexplored. Papers demonstrate that LLMs can generate fuzz harnesses, but they rarely address how to integrate this capability into real development workflows. Academic benchmarks aren't the same as production pipelines. Klooster et al. \cite{CiCdFuzzing2022} studied fuzzing in CI/CD environments and found that practical deployment faces challenges—speed, reliability, determinism—that benchmark papers don't capture.

Second, C receives insufficient attention. Most LLM fuzzing research uses Python or JavaScript as target languages. These languages matter, but they're not what powers automotive ECUs. The specific challenges of C—memory safety, undefined behavior, preprocessor complexity—warrant dedicated study. Papers like ECG \cite{ECG2024} and GDBFuzz \cite{GDBFuzz2023} address embedded systems, but they're exceptions rather than the norm.

Third, semantic quality lacks good metrics. We can measure whether generated tests compile and whether they achieve coverage. We have no standard way to measure whether they exercise meaningful behavior. A test that calls `json_parse(random_bytes)` achieves coverage by failing immediately—hardly useful for finding bugs in parsing logic. The gap between syntactic validity and semantic utility needs to be closed.

Fourth, automotive-specific evaluation is rare. Papers may claim applicability to safety-critical systems, but few actually evaluate on automotive software. The closest work is SAFLITE \cite{SAFLITE2024}, which fuzzed autonomous system components, and CANBusFuzzAI \cite{CANBusFuzzAI2021}, which applied AI to CAN bus testing. Both demonstrate feasibility but don't provide the comprehensive evaluation needed to guide industrial adoption.

This thesis positions itself at the intersection of these gaps. We investigate LLM-based fuzz harness generation specifically for C libraries used in automotive contexts. We evaluate not just coverage metrics but integration with CI/CD pipelines and practical deployment considerations. The contribution is less algorithmic innovation than applied validation: demonstrating what works, what doesn't, and what remains to be solved.


## 2.8 Automotive Software Development

Understanding automotive software development helps contextualize our research. The industry has characteristics that both enable and constrain AI adoption.

Development cycles are long. A vehicle platform takes 3-5 years from conception to production. Software freezes months before launch, leaving limited time for last-minute changes. This timeline contrasts sharply with web development's continuous deployment. Techniques assuming rapid iteration may need adaptation.

Supply chains are complex. A typical vehicle contains software from dozens of suppliers, each with their own development processes. OEMs receive binaries, not always source code. Testing must work across this fragmented landscape, which limits what tools are feasible.

The V-model dominates. Requirements flow down; verification flows up. Fuzzing fits naturally at unit and integration testing levels but requires thought about how it connects to higher-level verification activities. Simply running a fuzzer isn't enough—results must be documented, analyzed, and fed back into development.

Software-defined vehicles are changing these patterns. Companies like CARIAD are bringing more software development in-house. Over-the-air updates enable post-production changes. These shifts create opportunities for more dynamic testing approaches, including continuous fuzzing integrated into development pipelines. CI Spark \cite{CISpark2023} and similar industry tools point toward this future.

Security has become paramount. Connected vehicles face threats that isolated ECUs never confronted. Regulations now mandate cybersecurity capabilities. Fuzzing, as a technique for finding vulnerabilities, directly supports compliance. The question isn't whether to fuzz automotive software but how to do it effectively at scale.

Our experiments at CARIAD occurred during this transition. The organization was building new capabilities, defining new processes, making mistakes, and learning from them. The results we report reflect this specific context. Other organizations at different stages will face different challenges. But the underlying problem—using AI to improve testing of safety-critical automotive software—is universal.

---

This chapter has traced the evolution from random fuzzing to AI-enhanced testing, identified gaps in current research, and positioned our work within the automotive software context. The next chapter describes our methodology for investigating LLM-based fuzz harness generation in this environment.
