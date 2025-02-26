---
layout: post
title:  "Vertical AI agents"
date:   2025-02-01 18:08:28 +0000
categories: jekyll update
---

>Agentic systems have and are still being a hot topic for [AI researchers](https://paperswithcode.com/task/ai-agent), passionate builders ([YCombinator](https://www.ycombinator.com/companies/industry/ai-assistant), Entrepreneur First and many others), startups & businesses. Demistifying this topic requires understanding both where the intelligence stands and where the business impact starts.

What is new about agents that didn't exist before ? .. function calling on llms. an agent can be defined as mere component that can interact with an environment (some tools, APIs ...). But how is this different than a any other dev system ? The answer lies in the nature of the interaction. without intelligence, we may call it a workflow, but when the component (agent) becomes intelligent in its choice of tools, reasoning on the tools outputs ... we may start calling it an AI agent. and this intelligence arised thanks to llms. Model providers, started supporting this new feature called **function calling** which transcends llms from mere instructed next token predictors to models that are able to 'reason' on tools schema, your previous conversations and tools output to decide when they need to call a tool. This intelligence enabled llms to be called AI agents since their understanding of your tools, makes it possible to rely on them to interact with your environment and solve complex non-determinstic task.

Agentic systems are redefining concepts for both developers and businesses. Development product interaction is evolving, necessitating redesigned agent UIs, such as co-agent interfaces. System evaluation is also undergoing transformation, as evaluating an agent differs significantly from traditional software testing. [Furthermore, business operations are being reshaped](https://blog.langchain.dev/is-langgraph-used-in-production/), with single agents capable of handling multiple use cases. The potential for autonomous agent operation raises concerns about control and intervention. Maintaining oversight of an agent's actions and trajectory is crucial. This is why frameworks like LangGraph incorporate human-in-the-loop features, enabling human intervention before critical tool usage, database access, or when human input or validation is required.

*This article will be about : `agents`, `building them`, `main challenges`, and `good practises`, all based on my experience building AI agents for procurement risk & resilience.*

>YC mentioned [*vertical agents*](https://www.ycombinator.com/rfs#spring-2025-vertical-ai-agents) as a promising build direction.


## Part 1: General Comments on AI Agents 

I believe building agents for business applications differs significantly from creating agents for general human productivity, such as content creation, daily routines, or research. Many productivity and general-purpose agents are developed by open-source communities, contributors to LangChain, and other individuals.

However, constructing specialized agents that integrate with a company's existing processes, data, and business use cases demands a deep understanding of the business itself, beyond mere technical mastery. (See the [technical stack session](#tech_stack) for more details.)

The majority of AI agent developers, excluding specialized startups (including **Y Combinator companies focused on agents**) and companies building in-house solutions, are primarily focused on the general-purpose approach.

### <a name="reasoning_and_tools">Reasoning And tools Are All You Need</a>

Yes, and I would add that function calling & structured output are the best things that happened so far in the LLMs industry. OpenAI introduced their [structured output](https://www.youtube.com/watch?v=kE4BkATIl9c), in terms of research, engineering and API design.

This aligns with Anthropic's insights on building effective agents, as detailed in their [research blog](https://www.anthropic.com/research/building-effective-agents). Function calling empowers LLMs to act as orchestrators, not just text generators, which is a crucial shift. 

Reason enough to believe that working either on new impacting and efficient set of tools and efficient function-calling-enabled llms & llm grounding (and on the data side, working on data connectors, API gateways for agents [[WildCard YC25 startup is building around this](https://www.ycombinator.com/companies/wildcard)] and text-to-sql : [[Lamini and their mixture of memory experts](https://www.lamini.ai/blog/lamini-memory-tuning)]) are major for the AI agent industry. Since an agent is tools+reasoning. 

The complexity of integrating LLMs with diverse enterprise tools underscores the necessity for meta-systems. These systems will distill company knowledge and tool functionalities, making them readily accessible to agents. This is a critical area for development.

Personally, I'm particularly interested in developing multimodal forecasters as a tool for agents. Building AI agents within a startup environment presents a unique opportunity to directly contribute to research and development, creating novel and powerful tools that expand the capabilities of these intelligent systems.

### <a name="tech_stack">Technical Stack</a>

Building effective AI agents requires a robust and well-integrated technical stack. Here's a glimpse into the tools and technologies I'm currently leveraging:

#### Core Agent Framework

* **LangGraph:** For initiating and compiling the agent's core logic and workflow.

#### Data Orchestration and Management

* **Dagster:** ([https://docs.dagster.io/guides/build/assets/metadata-and-tags/](https://docs.dagster.io/guides/build/assets/metadata-and-tags/)) A powerful data orchestrator for managing complex data pipelines and ensuring data observability. I rely on Dagster for data observability within my agents.
* **Data Storage:** Flexibility is key. I integrate with various data storage solutions, including MongoDB and Snowflake, depending on the project's needs. Use your own data engineering stack.
* **Pydantic:** For robust schema validation, ensuring data integrity within the agent's operations.

#### LLM Application Monitoring

* **Langfuse/Langsmith:** Essential for monitoring and debugging LLM applications. I use Langfuse for LLM-app observability.
* *Note: While Langfuse is my current choice for LLM-app observability, Langsmith is also a valuable tool in this space. (langsmith is maintained by langchain)*

#### Development and Debugging

* **LangGraph Studio:** A user-friendly interface for serving and debugging LangGraph agents.

#### LLM and AI Libraries

* **LangChain:** Utilizing pre-built integrations and tools to streamline agent development.
* **Groq:** A free, low-context LLM.
* **LlamaIndex (LlamaParse, etc.):** For efficient data indexing and retrieval, crucial for grounding LLMs.
* **Hugging Face:** Leveraging pre-trained models for various AI tasks.

#### Key Considerations

* This is a non-exhaustive list, and the specific tools used may vary depending on the project's requirements.
* I prioritize tools that provide strong observability and debugging capabilities, especially when dealing with complex agent workflows.
* The ability to integrate with existing data engineering stacks is crucial for seamless deployment in real-world environments.

what I like much about **LangGraph Studio:**

 - easy add of interrupts
 - memory 
 - threads log
 - agents and assitants handler
 - runtime-config management in UI
 - state-fork / time-travel feature (the best)


### <a name="states_langgraph">Agent Graph/SubGraph State</a>

In LangGraph, "state" is a fundamental concept for managing agent behavior. Because agents are structured as graphs with interconnected nodes, we use state to checkpoint and track their progress.

Essentially, **state holds the information that needs to be passed between the agent's nodes.** This information should be:

* **Meaningful for the agent's operation:** It should directly influence the agent's decision-making and actions.
* **Useful for human understanding:** It should provide clarity during testing and post-run analysis, allowing developers to visualize the agent's trajectory.

**Key Considerations for State Management:**

* **State Propagation to Subgraphs:** State information is passed down to subgraphs, requiring careful consideration of input, output, and overall state for each subgraph.
* **Information Overload:** Avoid overwhelming the state with excessive data. "Too much information displayed kills the information." Prioritize the most relevant and critical state elements to maintain clarity and focus.
* **Design for Clarity:** Your state design should be intuitive and easily understandable, ensuring that the agent's behavior and data flow are transparent.
### <a name="agent_ui">Your Agent's UI/UX</a>

The CEO of [Mistral AI recently highlighted](https://www.youtube.com/watch?v=bzs0wFP_6ck) a crucial shift in agent interfaces: we're moving from single-agent, synchronous interactions to multi-agent, asynchronous systems. This raises a key question: **what UI paradigm will best serve users in this new era?**

**Rapid UI Development is Key**

For initial prototyping and testing, the emphasis should be on rapid UI development. Leveraging pre-built UI components and frameworks can significantly accelerate this process.

**LangGraph Studio: A Powerful Tool**

In my experience, LangGraph Studio is an excellent platform for visualizing and debugging agent workflows. Its intuitive interface allows for efficient testing and iteration.

**Exploring CoAgent for Specialized UX**

While I haven't personally used CoAgent yet, it appears to offer promising capabilities for creating specialized user experiences within multi-agent environments. For applications requiring a unique or tailored UX, CoAgent seems like a valuable option to explore.

**The Future of Agent Interfaces**

As multi-agent systems become more prevalent, the design of agent interfaces will become increasingly critical. We need interfaces that can effectively manage complex interactions, provide clear visibility into agent activities, and empower users to seamlessly collaborate with and control these intelligent systems.

### <a name="eval_agent">Agent: Evaluation, Monitoring & Error Handling</a>

Testing AI agents effectively requires a multifaceted approach, encompassing node, subgraph, and tool-specific testing, alongside robust error handling strategies like fallback models to prevent infinite loops and cost overruns. Caching, mocking, and unit testing are essential for efficiency and reliability, while meticulous input data validation and proactive error management, particularly for tool interactions, are critical for preventing unexpected behavior. Agents should be designed to halt upon task completion, balancing LLM decision-making with predefined workflows and human intervention for control. Tool functionality should be tested independently of the LLM to isolate issues and reduce generation costs, and the agent's behavior should be evaluated across different reasoning models to assess its robustness. Addressing the inherent non-deterministic nature of LLMs necessitates careful consideration of parameters like temperature, alongside rigorous testing and validation.

### <a name="modularity_agent">Modularity in The Agent </a>

*(good for testing purposes but also visualisation)*

llm_webagent_subgraph

event_summarizer subgraph

image_similarity_subgraph 

all these subgraphs can be easily changed while modifying only few lines in the graph setting (example : moving from llm_webagent to vlm_webagent ..)


### <a name="web_agents">Reflexion on Webagents</a>

WebAgents unlock a big potential of interesting usecases where you need efficient interaction with the web, thus many are building around this (multiOn, openai deepsearch, Grok deepsearch, anthropic
for webagents, at my knowledge, the two most used approaches so far are :

 - you navigate to the desired website -> scrap the html -> feed it to a llm with your prompt. (scrapgraphAI)

 - you intiate a playwright webpage -> you mark the web page + define a set of tools (click, write, go back ...) -> take screenshots -> feed the annotated image & an edeqaute system promtp to the vlm  -> actions taken until response. ([webvoyager](https://langchain-ai.github.io/langgraph/tutorials/web-navigation/web_voyager/#use-the-graph))

Pros and cons :
llms as webagents are expensive, and are a good solution if (you have a large-context free llm)

using screenshots on vlm is [cheaper](https://openai.com/api/pricing/); you can [check the pricing](https://langtail.com/llm-price-comparison) per feature & model provider, however, a combination of both may be better for complex tasks. (would be good to have real numbers on cost & number of tokens (language and visual tokens))

Surpisingly web agent implementation from webvoyager is less costly than scrapgraphAI (reason: cost of image <<< cost of html tokens)


### <a name="what_matters">What Matters</a>

*do hard things first and focus on what matters*

When building an agent, you can easily be lost in details and unvaluable features; the ability to focus on what matters, thus build a MVP is required at early stage. 
An example of what might matter for you:
 - Handling internal data efficiently
 - Searching for external web data
 - Implemeting existing business processes in your agent (As tools, nodes or subgraphs)
 - Handling a diversity of scenarios (aka user inputs) through reasoning.

### <a name="search">Search & retrieval: web/indexes </a>

add google search api to retrieve the website to scrap


nodes from scrapgraphAi to use : SearchLinkNode, searchNode (uses google ap )


if you need to parse an output as a list : CommaSeparatedListOutputParser

inspire from the prompt refiner in scrapgraphai (helps guide the agent .. and if it doesnt have enough informations asks the human and interrupt)

### <a name="human_in_the_loop">Human In The Loop</a>



### <a name="parallel_states_langgraph">Paralelism in Langgrraph</a>

Read the [blog](https://langchain-ai.github.io/langgraph/how-tos/branching/#next-steps) from langgraph 

**async ? when do u need it ? if u don't read the content of an async call ? manage output of langgraph nodes ? different classes of node type ? subgraph ?**

### <a name="memory_checkpoints_langgraph">Persistence and Memory</a>

![checkpoints_langgraph_img](assets/17384938530856.jpg)


### <a name="good_practises">Good Practises</a>


Start simple, don't go into very complex agent architectures if not needed: *if a workflow is enough, keep it as a workflow; not all UseCases needs reasoning*.

Add [Human In The Loop](#human_in_the_loop).  If LLM needs more informations or agent is about to access a tool/Vault DB for which we want to validate first. 

As for any dev codebase, your agent codebase should be modular (use [subgraphs](https://langchain-ai.github.io/langgraph/how-tos/subgraph/) when it may enhance the testing, updating and understanding of you whole graph)

Think about reducing cost for repetitive workflows. Save threads of your graph and reuse it if needed to reduce the cost. 

Don't neglect Agent testing. (continous integration for AI agents)


To continue reading here are some valuable writings. 
## Part 2: Procurement risk & resilience AI Agent


