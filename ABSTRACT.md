# Abstract

## Enhancing Automated Security Testing in CI/CD Pipelines with Large Language Models: A Case Study in Automotive Software Development

Automotive software systems have become more complex over time, while safety standards such as ISO 26262 and UNECE WP.29 cybersecurity regulations require thorough security testing. Fuzz testing is an effective method for finding vulnerabilities in C/C++ code, but its adoption remains low. One reason is that writing fuzz drivers requires specialized knowledge that most development teams do not have. This thesis explores whether Large Language Models (LLMs) can automatically generate fuzz drivers for automotive software libraries, making security testing more accessible.

We tested 14 open-source LLM models and one cloud-based model (GPT-4o) across 25 C/C++ libraries. The models ranged from 7 billion to 46.7 billion parameters and included Qwen 2.5-coder, Gemma 3, Deepseek, CodeLlama, and others. GPT-4o achieved a 91% success rate in generating fuzz drivers that compiled correctly. However, the coverage improvements were limited. For RapidJSON, coverage improved by 6 percentage points over baseline AFL (from 78% to 84%), but this was still lower than manually written OSS-Fuzz drivers (88%). Results were better for libraries with little existing test coverage. For yaml-cpp, coverage increased from 43% to 55% using LLM-generated drivers, showing that LLMs are most useful where manual testing effort is limited.

This thesis also examines the challenges of deploying LLM-based testing in an enterprise environment. Integrating cifuzz spark with Azure OpenAI within CARIAD's CI/CD system required working around corporate firewall policies, container networking issues, and authentication requirements. Several approaches failed, including proxy tunneling, boundary client integration, and container network bridging. The final solution required setting up Azure Private Link endpoints, which took several weeks to complete. These practical integration challenges are rarely discussed in research papers, but they are significant barriers to adopting AI tools in enterprise settings.

A cost analysis showed that using LLMs for fuzz testing costs between 219 and 1,452 euros per year, depending on usage. This is much lower than the cost of hiring dedicated security specialists (over 100,000 euros per year). However, LLM-generated tests have limitations. Some generated drivers are syntactically correct but do not test meaningful behavior. LLMs cannot reason about automotive safety requirements, and output quality varies between model versions. For teams that want to avoid cloud API costs, Qwen 2.5-coder (32B) and Gemma 3 (27B) are practical open-source options.

This thesis contributes a reproducible framework for evaluating LLM-based fuzz driver generation, documents the real-world effort required to integrate AI into enterprise CI/CD pipelines, and provides practical guidance for automotive organizations. The main finding is that LLMs do not replace human security experts, but they can automate testing tasks that would otherwise not be done due to resource limitations.

---

**Keywords:** Large Language Models, Fuzz Testing, Automotive Security, CI/CD Integration, Software Testing Automation, ISO 26262, Azure OpenAI

**Institution:** Friedrich-Alexander-Universität Erlangen-Nürnberg, in collaboration with CARIAD SE

**Author:** Morris Darren Babu

**Supervisors:** Prof. Dr.-Ing. Jürgen Teich, Dr.-Ing. Stefan Wildermann (FAU), Bernd Schaller, Dr.-Ing. Andreas Weichslgartner (CARIAD)
