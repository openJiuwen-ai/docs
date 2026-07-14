# What is openJiuwen?

With the continuous advancement of artificial intelligence, large language model (LLM) technology has rapidly matured. AI applications have evolved from systems focused on simple tasks such as speech recognition into intelligent agents capable of autonomous reasoning, decision-making, and execution of complex tasks. LLM-based agents possess autonomy, goal orientation, and interactive capabilities, enabling them to perceive information, reason, make decisions, and act effectively in complex and dynamic environments. As a result, AI Agents demonstrate strong potential across a wide range of industries and are increasingly applied in areas such as customer support, sales enablement, medical diagnosis, and financial analysis.

As an open-source agent platform, openJiuwen is dedicated to providing flexible, powerful, and easy-to-use capabilities for AI Agent development and execution. Built on this platform, developers can rapidly construct AI Agents to handle tasks of varying complexity, orchestrate collaboration and interaction among multiple agents, and efficiently develop reliable, production-grade AI Agents. openJiuwen also helps enterprises and individuals quickly build AI Agent systems or platforms, accelerating the adoption and large-scale deployment of commercial-grade Agentic AI technologies.

# Application Scenarios

openJiuwen enables the rapid development of AI Agent applications for both consumer and enterprise use cases, helping individuals and organisations improve development efficiency, execution accuracy, and overall system performance.

**Consumer-oriented Application Scenarios**

By leveraging zero-code, low-code, or SDK-based approaches in combination with prompt self-optimisation techniques, users can quickly build complex single-turn and multi-turn conversational Agents. This significantly improves development efficiency while meeting personalised requirements.

- Chat Assistant: Through natural language processing and contextual understanding, the assistant delivers natural and high-quality conversational experiences. It is widely used in customer service and user support scenarios to improve user satisfaction and engagement.
- Text generation: Leveraging the strong language generation capabilities of LLMs, the system supports article writing, creative copywriting, and press release generation to meet diverse needs in marketing and creative fields.
- Intelligent assistant: Powered by task planning and multi-agent collaboration, the system supports intelligent services such as schedule management, information retrieval, and transaction processing, improving the efficiency of individuals and teams.

**Application Scenarios for Enterprises**

Enterprises can leverage openJiuwen’s workflow engine and multi-agent collaboration capabilities to rapidly build, deploy, and efficiently execute various Workflow Agents. These agents can autonomously plan, decompose, and execute complex tasks, significantly reducing implementation costs and accelerating the commercial adoption of Agentic AI.

- Financial industry: Through multi-agent collaboration, the system supports multi-level intent routing, intelligent task allocation, and dynamic agent scheduling. This enables financial institutions to build integrated service systems with high precision, low latency, strong reliability, and robust security.
- Healthcare: Supports customised development of medical Agents for efficient medical record summarisation, preliminary screening of medical imaging data, and generation of diagnostic and treatment recommendations. By deeply integrating professional medical knowledge with contextual information, the system provides accurate and comprehensive clinical decision support, improving diagnostic efficiency and accuracy.

# Product Advantages

Key advantages of the openJiuwen platform include:

- Full-scenario adaptability: Designed for both enterprise (ToB) and consumer (ToC) use cases, supporting a wide range of application scenarios for organisations and individuals.
- Flexible development methods: Offers zero-code, low-code, and SDK-based development options to accommodate users with varying technical backgrounds and requirements.
- Efficient and precise task execution: Ensures high efficiency and accuracy in AI Agent task execution by optimising task processing workflows.
- Multi-Agent collaboration: Enables coordinated operation of multiple Agents to handle complex business processes and cross-domain tasks more effectively.
- Stable production-grade support: Provides commercial-grade stability and high availability, ensuring reliable operation in large-scale production environments and accelerating real-world deployment of Agentic AI technologies.

# System Architecture

openJiuwen adopts a layered architecture that covers the complete lifecycle of AI Agents, from development and execution to deployment, operations, and maintenance. The overall architecture consists of **DeepAgents**, **Agent Studio**, **Agent Framework**, **Agent Distributed Runtime**, and **Agent System Service**.

* **DeepAgents**: Provides complex, scenario-oriented Agents such as JiuwenSwarm, JiuwenSymbiosis, and DeepSearch, enabling out-of-the-box use.
* **Agent Studio**: A one-stop AI Agent development platform that provides low-code and no-code visual development capabilities. It supports Agent development, workflow orchestration, prompt optimization, online debugging, and resource management, helping developers quickly build and debug Agents and workflows.
* **Agent Framework**: The core framework and execution engine of openJiuwen that provides developers with intuitive APIs for building, orchestrating, and invoking AI Agents across diverse scenarios. It delivers comprehensive capabilities including complex task planning, iterative execution, tool and skill invocation, context management, memory subsystems, multi-agent collaboration, agent self-evolution, and affinity-aware scheduling, enabling the engineering of both single-agent and multi-agent applications.
* **Agent Distributed Runtime**: Provides the distributed runtime foundation for AI Agents, supporting both low-code and code-first deployment models with one-click deployment and unified lifecycle management. It natively supports multi-tenant isolation, elastic scaling, service registration and discovery, and high-performance cross-cluster communication, enabling reliable operation and large-scale deployment of enterprise-grade multi-agent systems.
* **Agent System Service**: Provides the foundational system services of AgentOS, including system-level security sandboxing, unified persistent memory storage, native CLI utilities, a standardized Agent file system, and a cross-agent message bus. These core capabilities provide a secure runtime environment, unified resource management, and efficient collaboration for AI Agents across the platform.

<img src="./images/architecture.png" alt="openJiuwen Capability Architecture" style="display: block; width: 100%; max-width: 1000px; height: auto; margin: 16px auto;" />

## Implementation Overview

openJiuwen adopts a modular repository design to build the AI Agent development ecosystem layer by layer. Each repository can evolve independently or be used in combination with others, covering the complete workflow from Agent applications, Skill distribution, and visual orchestration to framework development and service-based execution.

<img src="./images/repositories.jpg" alt="openJiuwen Implementation Architecture" style="display: block; width: 100%; max-width: 1000px; height: auto; margin: 16px auto;" />

### Repository Overview

<div style="overflow-x: auto;">
<table style="width: 100%; border-collapse: collapse; table-layout: fixed;">
  <colgroup>
    <col style="width: 18%;" />
    <col style="width: 26%;" />
    <col style="width: 56%;" />
  </colgroup>
  <thead>
    <tr>
      <th style="border: 1px solid; padding: 10px 12px; text-align: left; font-weight: 700;">Module</th>
      <th style="border: 1px solid; padding: 10px 12px; text-align: left; font-weight: 700;">Repository</th>
      <th style="border: 1px solid; padding: 10px 12px; text-align: left; font-weight: 700;">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3" style="border: 1px solid; padding: 10px 12px; font-weight: 700; vertical-align: middle;">DeepAgents</td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/jiuwenswarm">jiuwenswarm</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A multi-Agent collaboration framework and official flagship application that supports complex task collaboration and autonomous Skill evolution.</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/jiuwensymbiosis">jiuwensymbiosis</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">An Agent framework for embodied intelligence that supports embodiment-independent capability reuse and safety control.</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/deepsearch">deepsearch</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A knowledge-enhanced deep search and research Agent designed for search, reasoning, and report generation scenarios.</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 10px 12px; font-weight: 700; vertical-align: middle;">SkillHub</td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/skillhub">skillhub</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A Skill hosting and distribution platform that supports Skill publishing, version management, search and download, sharing and reuse, and private deployment.</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 10px 12px; font-weight: 700; vertical-align: middle;">Agent Studio</td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/agent-studio">agent-studio</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A one-stop visual Agent development platform that supports Agent editing, workflow orchestration, resource configuration, prompt optimization, and online debugging.</td>
    </tr>
    <tr>
      <td rowspan="4" style="border: 1px solid; padding: 10px 12px; font-weight: 700; vertical-align: middle;">Agent Framework</td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">agent-gateway</td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A unified access gateway that provides Channel management, message processing, scheduled tasks, heartbeat management, and other capabilities. Currently opening soon.</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/agent-core">agent-core</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A Python Agent SDK that provides core capabilities including Agent orchestration, runtime management, model integration, tools, retrieval, and evaluation.</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/agent-core-java">agent-core-java</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A Java Agent SDK that provides the Java ecosystem with Agent development capabilities consistent with those of the Python SDK.</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/agent-memory">agent-memory</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A long-term memory system for Agents that supports memory extraction, compression, hybrid retrieval, knowledge accumulation, and autonomous evolution.</td>
    </tr>
    <tr>
      <td rowspan="3" style="border: 1px solid; padding: 10px 12px; font-weight: 700; vertical-align: middle;">Agent Distributed Runtime</td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/agent-runtime">agent-runtime</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A Python Agent Runtime responsible for service-based Agent execution, session management, and lifecycle management.</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/agent-runtime-java">agent-runtime-java</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">A Java Agent Runtime based on Spring Boot that provides service-based Agent execution and deployment capabilities.</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;"><a href="https://gitcode.com/openJiuwen/agent-protocol">agent-protocol</a></td>
      <td style="border: 1px solid; padding: 10px 12px; vertical-align: top;">An Agent interoperability protocol SDK that provides the MCP SDK, A2A SDK, and A2X Registry.</td>
    </tr>
  </tbody>
</table>
</div>

# Function Features

openJiuwen provides comprehensive Agent orchestration and construction capabilities during development, enabling developers to rapidly build, configure, and iterate on Agent applications. At runtime, openJiuwen is powered by a high-reliability execution engine that serves as a robust foundation for scalable, efficient, and reliable Agent execution.

**Agent Orchestration**

openJiuwen delivers an extensible and flexible Agent application development framework, enabling users to build intelligent and automated Agent systems capable of handling complex tasks. Currently, openJiuwen provides two built-in Agent types: ReActAgent and WorkflowAgent, offering complementary execution paradigms to meet diverse application requirements.

- ReActAgent: Follows the ReAct (Reasoning + Action) paradigm and completes user tasks through an iterative control loop of “Thought → Action → Observation”. It supports multi-turn reasoning, tool invocation, and self-correction through feedback-driven state updates. With dynamic decision-making capabilities, ReActAgent can adapt to changing environments and is well suited for tasks requiring complex reasoning and strategy adjustment.

  ![image](./images/ReActAgent.png)

- Workflow Agent: A multi-step, task-oriented process automation Agent that executes complex tasks by strictly following predefined execution flows. It emphasises predictability, standardised execution, and efficiency, making it suitable for scenarios where task structures are clearly defined and decomposable into multiple steps.

  ![image](./images/WorkflowAgent.png)

**High-reliability Execution Engine**

openJiuwen provides a high-reliability execution engine that supports distributed deployment and cost-efficient operation. The engine addresses scalability and efficiency challenges in large-scale Agent execution, enabling reliable support for high-throughput workloads and production-grade applications.

- Hybrid batch–stream graph execution architecture: Supports the collaborative execution of batch and streaming data within a unified graph structure. Through componentization and dataflow-driven mechanisms, it enables efficient orchestration of complex workflows and real-time output.
- Automatic state management and interruption recovery: Through session-level state modeling, state persistence, and checkpoint-based resume mechanisms, it ensures continuous task execution under high-frequency interactions and abnormal termination scenarios, while supporting consistency and isolation in multi-instance deployments.
- End-to-end observability and debugging capabilities: Provides real-time monitoring, call tracing, and correlated exception analysis across the entire execution lifecycle, delivering engineering-grade assurance for stable operation and rapid issue localization in complex Agent systems under high-concurrency conditions.

# LLM Support

openJiuwen supports integration with a variety of open-source and commercial LLMs from leading providers such as Pangu, Qwen, and DeepSeek. Support for additional LLMs will continue to be added. Feedback and suggestions are welcome, and we will actively incorporate them to further enhance model support and overall platform capabilities.