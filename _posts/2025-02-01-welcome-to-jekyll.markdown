---
layout: post
title:  "Vertical AI agents"
date:   2025-02-01 18:08:28 +0000
categories: jekyll update
---

Agentic systems have and are still being a hot topic for [AI researchers](https://paperswithcode.com/task/ai-agent), passionate builders ([YCombinator](https://www.ycombinator.com/companies/industry/ai-assistant), Entrepreneur First and many others), startups & businesses. Demistifying this topic requires understanding both where the intelligence stands and where the business impact starts.

What is new about agents that didn't exist before ? .. function calling on llms. an agent can be defined as mere component that can interact with an environment (some tools, APIs ...). But how is this different than a any other dev system ? The answer lies in the nature of the interaction. without intelligence, we may call it a workflow, but when the component (agent) becomes intelligent in its choice of tools, reasoning on the tools outputs ... we may start calling it an AI agent. and this intelligence arised thanks to llms. Model providers, started supporting this new feature called **function calling** which transcends llms from mere instructed next token predictors to models that are able to reason on tools schema, your previous conversations and tools output to decide when they need to call a tool. This intelligence enabled llms to be called AI agents since their understanding of your tools, makes it possible to rely on them to interact with your environment and solve complex non-determinstic task.

Agentic systems redifine many concepts on both sides (developers and businesses) : how we interact with a dev product (an agent UI is to be redifined, mention coagent). How we evaluate systems (evaluating an agent is different). How the businesses work (one agent can do many usecases). It may be alarming to count on an agent without interupting it when needed. Having control over what it does and its trajectory is very important. reason enough why frameworks like langgraph built features to integrate human-in-the-loop and enable humans to interupt agents before calling some critic tools or accessing a vault database, or just when the agent may require human input or human validation. 

*This article will be about : `agents`, `building them`, `main challenges`, and `good practises`, all based on my experience building AI agents for procurement risk & resilience.*

YC mentioned [*vertical agents*](https://www.ycombinator.com/rfs#spring-2025-vertical-ai-agents) as a promising build direction
Papers that you might wanna read: [SFT Memorizes, RL Generalizes:
A Comparative Study of Foundation Model Post-training](https://arxiv.org/pdf/2501.17161)
## Part 1: General Comments on AI Agents

.. I believe building agents for businesses is different than building agents for basic human life productivity (content creation, daily life routines, research ..); Many productivity and general purpose agents are built by opensource communities, folks from `langchain` and many others, [langgraph @langchain](https://www.langchain.com/langgraph)

building specific-purpose agents on top of companies existing processes, data, business usecases requires a solid understanding of the business. Not mere tech stack mastery, refer to the [technical stack session](#tech_stack). 

Majority of AI agents builders  (except segment startups **list YC companies working on agents** & companies building their own) are building around the second approach. 

### <a name="reasoning_and_tools">Reasoning And tools Are All You Need</a>

Yes, and I would add that function calling & structured output are the best things that happened so far in the LLMs industry. OpenAI introduced their [structured output](https://www.youtube.com/watch?v=kE4BkATIl9c), in terms of research, engineering and API design.

mention [blog from anthropic](https://www.anthropic.com/research/building-effective-agents). and when talking abt function calling, the power of using llms as orchestrators rather than just text generators.

Reason enough to believe that working either on new impacting and efficient set of tools and efficient function-calling-enabled llms & llm grounding (and on the data side, working on data connectors and text-to-sql : you may need to read about llamina) are major for the AI agent industry. Since an agent is tools+reasoning. 

then he mentions at 20min, how hard is to use new entreprise tools by an llm, but this is reason why meta-systems will emerge to distill company knowledge and tools to the agent.

`one of the tools I wanna work on soon is: multimodal forecasters`. Building AI agents as a startup is inline with putting effort in different areas of research to build new capable tools, to include to the agents.

### <a name="tech_stack">Technical Stack</a>
This is a non-exhaustive list of my technical stack for building agents: initiating and compiling the agent (langgraph) + [data orchastrator](https://www.youtube.com/watch?v=Xe8wYYC2gWQ) ([dagster](https://docs.dagster.io/guides/build/assets/metadata-and-tags/)) + LLM app monitoring (langfuse, langsmith) + UI & QA (langgraph studio) + some langchain prebuilt integrations and tools + groq as a free low context llm. (personaly i am not using logfire for observability, since for data observability I am using Dagster and for LLM-app observability langfuse, but looks interesting too) + data storage : use your own data engineering stack, many integrations are buitin mongoDB, snowflake ... and Pydantic for schema validation. 

and obviously llamaindex and huggingface are part of my AI stack too. 

what I like much about langgraph studio : 

 - easy add of interrupts
 - memory 
 - threads log
 - agents and assitants handler
 - config management in UI
 - state fork feature (the best)


### <a name="states_langgraph">Agent Graph/SubGraph State</a>

State is an agent-related concept from langgraph. Since the agents are defined as graphs, containg nodes, we checkpoint their "state". The state contains informations we want to pass through the agent nodes. These should be meaningful for the agent functioning and useful for humans (either when testing the agent or just visualising its states after a run).  

*The state informations are passed to subgraphs too* and you need to reflect on which states are input, output, overall for each of your subgraphs. It should make sense for you and avoid : "too much informations displayed kills the information". 

### <a name="agent_ui">Your Agent's UI/UX</a>

ceo of mistral AI in underscore at 15min, speaks about the importance of the agent interface, and how we are going from one-agent synchrone interface to multi-agent asynchrone interfaces. What interface for serving to user ? 

*the UI (build fast, use pre-built UI)*

langstudio is excellent 

i didn't try coAgent yet, but for a specific UX, looks like a good option. 

### <a name="eval_agent">Agent: Evaluation, Monitoring & Error Handling</a>
testing an agent : testing its nodes, its subgraphs, tools ... 

rasing excpetions + [fallback model](https://langchain-ai.github.io/langgraph/how-tos/tool-calling-errors/#custom-strategies) ... (**error handling section**)

**caching the workflow to lower cost + mocking + recordings (VCR.py) + (node, tools, subgraph) unit testing **

- handling tool errors is crucial, a concrete example, is when i get an error from my tools my agents keeps looping. which ends up in a infinite llm cost and no productivity. (same wrong result due to the tool error raised)

 raise ValueError("Invalid input.")


 must know : The prebuilt ToolNode that executes the tool has some built-in error handling that captures the error and passes it back to the model so that it can try again:



 read about this custom strategy : call_fallback_model


 You have a namespace conflict. One of your import statements is masking PIL.Image (which is a module, not a class) with some class named Image.: CHange the imports to resolve the namespace conflict


always check ur input data (arg of ur tool/function)

example: df.loc[11,'image'] was None, because of df.loc[11,'image_file_path'] which was a url of legrand containing an empty page: no image. 

It is also important to have an agent, that stops when they got the answer, sometimes they continue using other tools while its enough.

letting the llm do all the work will result in uncontrollable behaviors. thus a combination between llm decisions, interrupt & predifined workflows looks like a better approach


test the tools independently than the llm, otherwise u are wasting generations

after webscrap, o1 was sometimes able to retrieve base url and add it to img urls; while other models (gpt4 included couldn't). your agent behavior is not agnostic to the reasoning model. 

how to deal with the non-deterministic nature of llms? manipulate the temperature ??


### <a name="modularity_agent">Modularity in The Agent </a>

*(good for testing purposes but also visualisation)*

llm_webagent_subgraph

event_summarizer subgraph

image_similarity_subgraph 

all these subgraphs can be easily changed while modifying only few lines in the graph setting (example : moving from llm_webagent to vlm_webagent ..)


### <a name="web_agents">Reflexion on Webagents</a>

for webagents, at my knowledge, the two most used approaches so far are :

 - you navigate to the desired website -> scrap the html -> feed it to a llm with your prompt. (scrapgraphAI)

 - you mark the web page + define a set of tools (click, write, go back ...) -> take screenshots -> feed the image to the vlm -> actions taking until response. ([webvoyager](https://langchain-ai.github.io/langgraph/tutorials/web-navigation/web_voyager/#use-the-graph))

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


![img](assets/img.png)

the llm decides to use visual similarity. which will redirect to either 2D or 3D similarity (with the appropriate data format: data conversion tool) depending on the extension provided. (it can be useless to wait for the model reason on which 3D similarity tool to use based on the extension of the files. but hard code this while the agent just chooses to use a "visual similarity" tool, which will redirect to the right node (2D or 3D, and based on the extension))

Objectif now: taking it to the next level (with minimum dev effort): we still in MVP stage.
we need smthg very impacting and very astonishing.