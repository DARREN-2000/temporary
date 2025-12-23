# Abstract

## Enhancing Automated Security Testing in CI/CD Pipelines with Large Language Models: A Case Study in Automotive Software Development

The growing complexity of automotive software systems, combined with stringent safety requirements under ISO 26262 and UNECE WP.29 cybersecurity regulations, demands scalable approaches to security testing. Fuzz testing has proven effective for vulnerability discovery in C/C++ codebases, yet adoption remains limited—partly because writing effective fuzz drivers requires specialized expertise that most development teams lack. This thesis investigates whether Large Language Models can automate fuzz driver generation for automotive software libraries, reducing the barrier to comprehensive security testing.

We conducted experiments across 25 open-source C/C++ libraries, evaluating seven LLM models ranging from 7B to 46.7B parameters. GPT-4o achieved a 91% compilation success rate for generated fuzz drivers, though coverage improvements were modest: a 6-percentage-point gain over baseline AFL for RapidJSON (84% vs. 78%), still trailing manually-crafted OSS-Fuzz drivers (88%). The story differed for libraries with minimal existing test infrastructure—yaml-cpp coverage increased from 43% to 55% with LLM-generated drivers, demonstrating clear value where manual effort would otherwise be prohibitive.

Beyond model evaluation, this work addresses the practical challenges of deploying AI-driven testing in enterprise automotive environments. Integrating cifuzz spark with Azure OpenAI services within CARIAD's CI/CD infrastructure required navigating corporate firewall policies, container networking constraints, and authentication mechanisms. Initial attempts at proxy tunneling, boundary client integration, and container network bridging all failed; the solution ultimately required provisioning Azure Private Link endpoints—a process that took several weeks of infrastructure coordination. These integration challenges, rarely discussed in academic literature, represent a significant barrier to enterprise AI adoption.

Cost analysis revealed annual expenses between €219 and €1,452 depending on usage intensity, compared to €100,000+ for dedicated security specialists. However, LLM-generated tests exhibit limitations: occasional syntactically valid but semantically meaningless drivers, inability to reason about automotive safety requirements, and inconsistent output quality across model versions. The Qwen 2.5-coder (32B) and Gemma 3 (27B) models emerged as viable open-source alternatives for organizations seeking to avoid API dependencies.

This thesis contributes a reproducible evaluation framework for LLM-based fuzz driver generation, documents enterprise integration patterns for AI-enhanced CI/CD pipelines, and provides evidence-based recommendations for automotive organizations considering AI-driven security testing adoption. The findings suggest that LLMs offer practical value not by surpassing human experts, but by automating tasks that would otherwise remain undone due to resource constraints.

---

**Keywords:** Large Language Models, Fuzz Testing, Automotive Security, CI/CD Integration, Software Testing Automation, ISO 26262, Azure OpenAI

**Institution:** Friedrich-Alexander-Universität Erlangen-Nürnberg, in collaboration with CARIAD SE

**Author:** Morris Darren Babu

**Supervisors:** Prof. Dr.-Ing. Jürgen Teich, Dr.-Ing. Stefan Wildermann (FAU), Bernd Schaller, Dr.-Ing. Andreas Weichslgartner (CARIAD)
