# 4. IMPLEMENTATION

This chapter describes how we turned the methodology from Chapter 3 into working systems. The implementation work spanned approximately five months from May to September 2025, starting with initial toolchain setup and ending with enterprise deployment. We explain which tools we selected and why, the specific setups used in each phase, and the problems we encountered along the way. The goal is to provide enough detail that others could reproduce or build on this work.


## 4.1 Toolchain Selection and Rationale

The toolchain for this project came from a combination of requirements analysis and practical limits. We needed tools that were mature enough for production use, flexible enough for research experiments, and compatible with CARIAD's enterprise systems.

### Fuzzing Platform

For fuzzing execution, we selected cifuzz from Code Intelligence. cifuzz is built on top of LLVM libFuzzer and provides additional features for corpus management, coverage display, and CI/CD integration. We chose cifuzz rather than raw libFuzzer or AFL for several reasons related to enterprise needs.

First, cifuzz provides audit logging that tracks fuzzing activities. This is important in regulated environments where security testing must be documented. Second, cifuzz includes access controls that fit enterprise permission models. Third, cifuzz integrates with common CI/CD systems including GitHub Actions, which CARIAD uses. Fourth, the cifuzz spark feature specifically handles LLM integration, reducing the custom code we needed to write.

The cifuzz spark workflow operates as follows. After initializing cifuzz in a project with cifuzz init, the tool creates a configuration file named cifuzz.yaml. The CMake build system is then configured to include cifuzz through find_package and enable_fuzz_testing directives. A subdirectory is added for spark generated tests. When cifuzz spark runs, it analyzes the codebase and generates fuzz test files based on the LLM output.

### LLM Serving

For local LLM inference, we used Ollama as the serving layer. Ollama provides a unified API across different model architectures and handles model downloading, memory management, and request batching. This abstraction let us test multiple models without writing model specific code for each.

Ollama runs as a local server that accepts HTTP requests in a format compatible with the OpenAI API. This means the same client code can talk to Ollama locally or to cloud APIs with only endpoint changes. Models are downloaded on first use and cached locally. Ollama handles GPU memory allocation and model loading automatically.

For some experiments we also used KoboldCpp, which provides more control over inference parameters. However, Ollama proved simpler for our main use cases.

For cloud model access, we used Azure OpenAI Service. Azure was required for enterprise deployment because of compliance needs around data handling and access controls. Azure OpenAI provides the same models available through OpenAI's public API but within Azure's compliance framework.

The environment variables for cifuzz spark configuration with Azure include:
- CIFUZZ_LLM_API_URL pointing to the Azure endpoint
- CIFUZZ_LLM_AZURE_DEPLOYMENT_NAME specifying the model deployment name
- CIFUZZ_LLM_MAX_TOKENS set to control response length
- CIFUZZ_LLM_API_TOKEN containing the authentication key

### Fine Tuning Tools

For fine tuning, we used the Hugging Face Transformers library together with PEFT for LoRA implementation. This combination is well documented and widely used in the research community, which made finding help easier when we encountered problems.

Transformers provides a consistent interface for working with different model architectures. We could load models from Hugging Face Hub, add LoRA adapters through PEFT, and train using standard PyTorch training loops. The library handles tokenization, data loading, and gradient computation.

PEFT specifically implements parameter efficient fine tuning methods including LoRA. It integrates cleanly with Transformers and provides utilities for saving and loading adapter weights separately from base models.

### Build and Compilation

Build and compilation used the standard LLVM toolchain. We used Clang 15 for compiling fuzz targets because cifuzz depends on LLVM coverage instrumentation. Clang provides built in support for AddressSanitizer and UndefinedBehaviorSanitizer, which we enabled for all fuzzing runs to maximize bug detection.

AddressSanitizer detects memory errors like buffer overflows, use after free, and memory leaks. UndefinedBehaviorSanitizer catches undefined behavior in C and C++ code including integer overflow, null pointer dereference, and alignment violations. Together these sanitizers turn silent bugs into detectable crashes that fuzzing can find.

The CMake configuration for fuzzing includes:
- CMAKE_C_COMPILER and CMAKE_CXX_COMPILER set to Clang
- Sanitizer flags added through cifuzz configuration
- Export of compile commands for tooling support

### Container and CI/CD Tools

For container management during CI/CD integration, we used buildah rather than Docker. CARIAD's CI/CD infrastructure uses buildah for container operations because it runs without a daemon and integrates better with rootless execution. This matches the security model of ephemeral CI/CD runners.

On development machines, we initially tried Docker Desktop but encountered problems on macOS with M1 processors. We switched to Podman, which provides better compatibility with Linux container tooling. Podman can be configured to work with buildah based workflows, making local development match CI/CD behavior more closely.

For artifact storage, we used JFrog Artifactory. Fuzzing corpora are stored in Artifactory between CI/CD runs so that coverage can build up over time. We implemented corpus upload and download with compression to manage storage costs. Checksums ensure corpus integrity.


## 4.2 Phase 1: Local LLM Evaluation Setup

Phase 1 ran from initial project setup through baseline evaluation. The goal was to understand what current LLMs can achieve for fuzz driver generation. This required setting up local GPU infrastructure, selecting models to test, choosing target code, and defining evaluation procedures.


### 4.2.1 Benchmarked Models

We tested models across several size categories. All local models were accessed through Ollama running on development workstations with GPU support.

#### Large Models

Qwen2.5 coder 32B was our primary large model for code generation tasks. This model was specifically trained for code and showed strong performance on coding benchmarks. At 32 billion parameters, it required substantial GPU memory but provided high quality outputs.

Gemma3 27B provided an alternative architecture from Google DeepMind. We included it to see whether model diversity affected results. Gemma showed somewhat different error patterns from Qwen, suggesting the models learned different aspects of coding practice.

DeepSeek r1 32B and DeepSeek Coder 33B offered reasoning focused variants. DeepSeek models are known for strong performance on complex tasks. However, we encountered context length limits with DeepSeek on larger target functions.

CodeLlama 34B and WizardCoder 34B represented the Meta Llama family with code fine tuning. These models showed acceptable performance but were generally slower than Qwen on our hardware.

#### Extra Large Models

Mixtral at 46.7 billion parameters was the largest model we could run locally. Mixtral uses a mixture of experts architecture that activates only part of the model for each token. This makes inference more efficient than a dense model of similar size. However, loading the full model still required careful memory management.

#### Medium Models

Qwen2.5 coder 7B and 14B provided smaller options for faster iteration. The 7B model could generate drivers in seconds rather than the minutes needed for larger models. This speed was useful during prompt engineering when we needed quick feedback.

CodeLlama 7B and 13B offered alternatives in the same size range. Performance was somewhat lower than Qwen for our specific use case, but still useful for comparison.

#### Cloud Reference

GPT 4o through Azure OpenAI served as our upper bound reference. This model represented the current best available through cloud APIs. Comparing local models against GPT 4o helped us understand the capability gap and what improvement might be possible.

We also tested GPT 3.5 Turbo for comparison with older generation models. Performance was notably lower than GPT 4o, confirming that model improvements matter significantly for this task.

#### Model Download and Setup

Getting large models running locally required handling several challenges. Models over about 10 billion parameters often come as split files that must be merged before use. For example, the Qwen2.5 coder 14B model came as two GGUF files that we merged using llama gguf split.

Some models on Hugging Face are gated and require authentication. We configured huggingface cli login to access these. Download times for 30+ billion parameter models were significant even on fast connections.

Quantization was essential for fitting large models in available GPU memory. We primarily used Q4_K_M and Q5_K_M quantization levels from the GGUF format. These provide reasonable quality while reducing memory needs to roughly one quarter of the full precision requirements.


### 4.2.2 Target Repositories

We selected 25 C and C++ repositories as fuzzing targets. Selection aimed for variety across several dimensions to avoid results that only apply to one type of code.

#### Primary Targets

yaml cpp is a YAML parsing library for C++. We chose it as a main target because it has extensive existing fuzz coverage through OSS Fuzz, providing ground truth for comparison. The library has clear function signatures and reasonable documentation. In yaml cpp we identified 35 source files containing 1061 functions as potential fuzzing candidates.

pugixml is a lightweight XML processing library. It offers moderate complexity with a smaller API surface than yaml cpp. The library has less existing fuzz coverage, letting us assess whether LLM generated drivers could fill genuine gaps.

jsoncons handles JSON parsing and generation. JSON libraries are common targets for fuzzing because they process external input. jsoncons provides a C++ interface with templates, testing how well models handle template heavy code.

fmt is a text formatting library similar to Python's format function. It processes format strings which can be sources of security bugs. The library is well documented and widely used.

spdlog is a logging library built on fmt. Logging is common in automotive software and represents a typical internal library use case. spdlog has a C++ API with many configuration options.

glm is a mathematics library for graphics. It uses C++ templates extensively and represents a different domain than parsing libraries. Testing glm showed how models handle mathematical code.

#### Selection Criteria

We selected targets based on several criteria. First, the code had to be open source with licenses permitting research use. Second, the code had to compile with our toolchain without extensive modification. Third, the API had to be documented well enough that driver generation was feasible. Fourth, ideally some existing fuzz coverage existed for baseline comparison.

We excluded very small libraries with only a few functions because they would not provide enough data points. We also excluded libraries requiring complex runtime setup like databases or network services.

#### Internal Target

We also tested an internal CARIAD library that handles configuration parsing for vehicle systems. This library could not be described in detail due to confidentiality. It represented a realistic internal use case with moderate documentation and no existing fuzz coverage. Results on this target informed our assessment of practical applicability.


### 4.2.3 Evaluation Environment

The hardware for Phase 1 was a workstation with an NVIDIA RTX 4090 GPU. This GPU has 24GB of VRAM and provides good inference speed for large language models. The system had 64GB of system RAM and a 12 core AMD processor.

This hardware could run models up to about 14 billion parameters in full precision or larger models with quantization. The 32B models required Q4 quantization to fit in 24GB. Very large models like Mixtral 46.7B required aggressive quantization and careful memory management.

#### Ollama Configuration

We configured Ollama with mostly default settings. Context length was increased as needed for longer source files, up to 32768 tokens for models that supported it. Temperature was set to 0.3 for driver generation to balance determinism against creativity. Lower temperatures produce more consistent output while higher temperatures might find unusual solutions.

The Ollama server was started with environment variables controlling GPU layers and context size:
- OLLAMA_NUM_GPU controls how many model layers run on GPU
- OLLAMA_CTX_SIZE sets maximum context length

#### Testing Protocol Details

Each model evaluation followed a consistent protocol. For each target function:

1. Extract function signature and relevant context from source files
2. Build prompt using our template system
3. Generate 10 driver candidates with the model
4. Attempt compilation of each candidate with Clang
5. Record compilation outcome with error category if failed
6. For successful compilations, run 10 minute fuzzing campaign
7. Record coverage achieved and any crashes

For functions where initial results looked promising, we extended to 24 hour campaigns to assess crash detection over longer runs.

Data was logged in JSON format with timestamps, model versions, prompt text, generated code, compilation output, and coverage data. This complete logging enabled later analysis and debugging of unexpected results.

#### Time Requirements

A full evaluation of one model across all target repositories took approximately one week of continuous execution. Most of this time was fuzzing execution rather than generation. We parallelized where possible by running different targets on different machines, but single GPU limits constrained throughput.

The 10 minute fuzzing campaigns were chosen to balance coverage assessment against evaluation time. Longer campaigns find more coverage but the marginal gains diminish. Our experiments suggested 10 minutes captured most of the coverage that would be found in the first hour.


## 4.3 Phase 2: Model Optimization with LoRA Fine Tuning

Phase 2 ran from December 2024 through February 2025 with the goal of improving local model performance through targeted fine tuning. The work had several components: dataset preparation, training infrastructure setup, hyperparameter experiments, and evaluation of fine tuned models.

### Dataset Preparation

Training data came from two sources as described in Chapter 3. The OSS Fuzz extraction required parsing fuzzer harness files from the OSS Fuzz repository on GitHub. We wrote scripts to extract driver code, identify the target library and function, and format examples as instruction completion pairs.

Filtering was important because not all OSS Fuzz drivers are equally good training examples. We excluded drivers that were extremely short, that focused on internal functions rather than public APIs, or that used complex macros obscuring the actual code. After filtering, 623 examples remained from 47 projects.

The Phase 1 successful outputs were easier to prepare since they were already in the format we needed. We manually reviewed each one to confirm quality before including it in training data. This added 184 examples.

Training examples were formatted with a consistent structure:
- System prompt establishing the task context
- User message containing function signature, types, and documentation
- Assistant response containing the complete fuzz driver

We tokenized examples using the tokenizer for whichever base model we were training. Maximum sequence length was set to 4096 tokens, with longer examples truncated or excluded.

### Training Infrastructure

Training ran on the same RTX 4090 workstation used for Phase 1 evaluation. For LoRA fine tuning of 7B models, 24GB of GPU memory was sufficient with float16 precision. Larger models required gradient checkpointing or smaller batch sizes.

We used the Hugging Face Trainer class with standard configuration:
- Learning rate 2e-4 with cosine schedule
- Batch size 4 with gradient accumulation when needed
- Weight decay 0.01
- Warmup steps equal to 10% of total steps
- Evaluation every 100 steps on held out validation set

Training typically ran for 3 epochs, completing in 6 to 10 hours depending on dataset size and model. We monitored training loss and validation loss to detect overfitting.

### LoRA Configuration Experiments

We experimented with several LoRA configurations before settling on final parameters.

Rank 8 versus rank 16 versus rank 32 showed that rank 16 provided the best balance. Rank 8 underfitted on our data, not capturing enough patterns. Rank 32 overfit more easily without clear benefit on validation metrics.

Target modules experiments confirmed that including all attention projections (q, k, v, o) worked better than just query and value as some papers suggest. The additional parameters from k and o projections helped without causing overfitting given our data size.

Dropout rates of 0.05, 0.1, and 0.15 showed that 0.1 prevented overfitting best on our validation set. Higher dropout slowed training without clear benefit.

Alpha values of 16, 32, and 64 at rank 16 showed that alpha 32 (ratio 2x) performed best. This matches common practice in LoRA literature.

### Fine Tuned Model Results

The fine tuned 7B model showed clear improvements over baseline on our evaluation targets.

On yaml cpp, compilation success went from 40% to 58%, an improvement of 18 percentage points. Token consumption dropped by about 20%, suggesting the model learned more efficient patterns. Generation time decreased proportionally.

Comparison between training on 172 versus 709 examples showed the larger dataset helped. The 709 example model achieved 63% compilation success versus 55% for the 172 example model. This 8 percentage point improvement suggested more data would help further.

Coverage results were harder to interpret because of fuzzing randomness. On average, fine tuned model drivers achieved slightly higher coverage than baseline, but the difference was within noise for many targets. The clearer win was on compilation success, where fine tuning reliably helped.

### Generalization Testing

To check whether fine tuned models generalized beyond training examples, we evaluated on held out functions. These were functions from target libraries that we deliberately excluded from training data.

Results showed some degradation on held out functions. Where the fine tuned model achieved 58% compilation on training distribution functions, it achieved only 48% on held out functions. This 10 percentage point drop indicated partial overfitting.

The drop was worse for functions very different from training examples. Functions with unusual parameter types or complex return values showed the largest degradation. Functions similar to training examples generalized better.

This finding suggested that production deployment would need larger and more diverse training data to avoid overfitting. The training sets we could create in project time were too narrow for strong generalization.


## 4.4 Phase 3: Enterprise CI/CD Integration

Phase 3 ran from March through June 2025 and focused on deploying the LLM fuzzing system within CARIAD's production CI/CD infrastructure. This phase proved more challenging than expected, with organizational and technical obstacles extending the timeline.


### 4.4.1 Architectural Challenges

The enterprise environment created challenges our research setup had not encountered. Understanding these challenges took significant time and shaped our final solution.

#### Network Isolation

CARIAD CI/CD runners execute in network segments with no default internet access. This zero trust security model prevents data exfiltration but also blocks access to external services.

Our first attempts to configure network access failed repeatedly:

Attempt 1: Standard proxy configuration. We set HTTPS_PROXY and HTTP_PROXY environment variables pointing to corporate proxy infrastructure. This failed because the proxy services were not available in the runner network segment. The error was connection refused to the proxy address.

Attempt 2: Boundary client tunneling. CARIAD provides a boundary client tool for secure tunneling to approved external services. We tried using this in the CI/CD workflow. It failed at authentication because even the authentication server was unreachable from runners. DNS lookup for the access server failed.

Attempt 3: Corporate proxy discovery. We investigated proxy infrastructure documentation and found references to a specific proxy address. This also failed with timeout errors because the proxy requires a VPN kit component not present on runners.

Attempt 4: Container network manipulation. We tried various container networking approaches including host network mode and custom bridges. All were blocked by firewall policies that applied regardless of container networking.

These failures consumed approximately three weeks of investigation and testing. Each approach seemed promising based on documentation but failed when actually tested on runner infrastructure.

#### Local Model Limitations

We also explored running models locally on runner infrastructure to avoid network requirements entirely. This failed for resource reasons.

CI/CD runners are provisioned as ephemeral containers with limited CPU and memory. They have no GPU access. Running even small quantized models on CPU was too slow. A single driver generation took over five minutes on runner hardware compared to seconds on GPU.

Waiting five minutes per target function would make LLM fuzzing impractical in CI/CD. A library with 100 fuzzing targets would add over eight hours just for generation. This was unacceptable.

#### Secret Management

Even once network access was possible, secret management needed care. LLM APIs require authentication tokens. These tokens needed to be available to workflows without exposure in logs or source code.

GitHub Actions secrets provide encrypted storage with access controls. Secrets are injected as environment variables and masked in logs. We used this for the Azure OpenAI API token.

However, secret rotation required manual updates. Azure OpenAI tokens expire and must be refreshed. We documented the rotation process but this added operational overhead.

#### Rate Limits

Azure OpenAI enforces rate limits on API usage. During peak load with multiple pipelines running, we could exceed limits and receive throttling errors.

We implemented retry logic with exponential backoff to handle transient throttling. For sustained overload, we added request queuing at the workflow level to spread requests over time. This added complexity but was necessary for reliable operation.


### 4.4.2 Self Hosted Runner Solution

After extensive exploration, we found a working solution using Azure Private Link with dedicated self hosted runners.

#### Azure Private Link

Azure Private Link allows Azure services to be accessed through private IP addresses within a virtual network. Traffic stays within the Azure network rather than going through public internet. This satisfies security requirements while providing connectivity.

Setting up Private Link required coordination across multiple teams:
- The infrastructure team updated network policies to allow private endpoint traffic
- The Azure platform team created the private endpoint in CARIAD's Azure subscription  
- DNS was configured to resolve the Azure OpenAI hostname to the private IP
- Network security groups were updated to permit the traffic

This coordination took approximately six weeks. Most of that time was spent in approval processes and waiting for provisioned resources. The actual technical configuration took only a few hours once approvals were complete.

#### Self Hosted Runners

To use the Private Link endpoint, we needed runners with access to the private network. Standard GitHub hosted runners would not work because they run outside CARIAD's network.

Self hosted runners are GitHub Actions runners that execute on infrastructure we control. We provisioned dedicated runner instances with enhanced specifications:
- 32GB RAM for workflow processing
- 8 vCPU for build compilation
- Network access to the Azure private endpoint
- Labels to identify them for workflow targeting

Runners were configured with the necessary environment variables for Azure OpenAI access. The CIFUZZ_LLM_API_URL was set to the private endpoint address. Other variables were set through GitHub secrets.

#### Workflow Configuration

The final CI/CD workflow followed this structure:

1. Checkout source code from repository
2. Login to JFrog Artifactory for artifact access
3. Download existing fuzzing corpus if available
4. Pull cifuzz container image from internal registry
5. Build the project in a buildah container
6. Run cifuzz spark to generate fuzz tests (with LLM environment variables)
7. Rebuild to include generated tests
8. Execute fuzzing campaigns
9. Upload updated corpus to Artifactory

The cifuzz spark step passes LLM configuration through environment variables. The buildah container is configured to access these variables using env flags on the run command.

Fuzzing campaigns run with time limits appropriate for CI/CD. We used 30 second campaigns in the standard workflow with longer campaigns scheduled nightly. Short campaigns catch regressions quickly while long campaigns find deeper issues.

#### Validation Steps

We added validation between generation and fuzzing execution. Generated drivers must compile successfully before being used. Compilation failures trigger notifications to developers who can review and correct if desired.

Static analysis with Clang Tidy runs on generated code to catch common problems. This catches issues that might compile but represent poor practice.

If validation fails, the workflow continues with any existing drivers rather than failing completely. This ensures that generation problems do not block the overall fuzzing process.

#### Monitoring and Observability

We implemented monitoring to track system health and usage:
- API request logging captures prompt metadata, response time, and token counts
- Cost tracking aggregates token usage for billing estimates
- Success rate metrics show what percentage of generations compile
- Coverage trends display how fuzzing coverage changes over time

Logs flow to existing monitoring infrastructure where dashboards display key metrics. Alerts notify on error rate spikes or cost anomalies.

#### Rollout Process

Deployment followed a gradual rollout:

Week 1 to 2: Single volunteer team with library used in Phase 1. Close monitoring and quick iteration on issues discovered.

Week 3 to 4: Extended to three additional teams. Addressed scaling issues and refined documentation based on questions.

Week 5 onward: Continued gradual expansion. At project end, eight libraries across four teams were using the system.

Full organization deployment remains pending. More infrastructure capacity is needed to support all teams, and change management processes require additional approvals for broader rollout.

#### Developer Feedback

Early adopter feedback was generally positive. Developers appreciated not writing fuzz drivers manually. The most common praise was for time savings on initial setup.

The most common complaint was occasional nonsense outputs that compiled but tested nothing useful. These occurred when the model misunderstood the target function. We addressed this partially through prompt improvements, but the issue persists at a low rate.

Some developers wanted more control over generated code style. The current system produces drivers in whatever style the model outputs. Future work could add style enforcement or customization options.

#### Cost Analysis

Monthly Azure OpenAI expenses for the current deployment scope ranged from 120 to 180 euros. This varied with fuzzing activity and number of targets processed.

Extrapolating to full organization coverage suggested annual costs of 2000 to 3000 euros. This estimate carries uncertainty because usage patterns might change with broader adoption.

Compared to manual driver development, these costs are low. Security engineer time for manual driver writing costs far more. Even if LLM drivers require some manual correction, the time savings are substantial.

#### Lessons Learned

Phase 3 taught us that technical capability alone is not enough for enterprise deployment. Organizational factors dominated the timeline:
- Security reviews required detailed documentation and multiple approval stages
- Infrastructure changes needed coordination across teams with different priorities
- Change management processes added overhead but ensured proper review

Future projects should budget substantial time for these non technical factors. In our case, roughly 70% of Phase 3 time was spent on organizational coordination rather than technical implementation.

The technical work itself was straightforward once infrastructure was in place. The architecture developed in earlier phases worked as designed. cifuzz spark integration required minimal custom code. The main technical learning was about Azure Private Link configuration, which was poorly documented for our specific use case.
