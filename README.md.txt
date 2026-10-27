# Intelligent University Assistant — LangGraph Lab

## Overview

This project implements an intelligent university assistant using LangGraph.
It classifies student questions and routes them to specialized agents.

The final assistant supports five categories:
- COURSES
- PROGRAMMING
- MATHEMATICS
- ADMINISTRATION
- GENERAL

The notebook also includes introductory exercises on routing, tool use,
conversational memory, graph visualization, and evaluation.

## Project files

- langgraph_lab.ipynb: executed notebook containing the exercises,
  final assistant, demonstrations, and outputs.
- source/: source code for the final assistant.
- architecture/: architecture diagram.
- screenshots/: execution screenshots.
- report/: short report answering the conceptual questions.

## Setup

Use the Python environment configured for the lab.

The notebook uses:
- Jupyter
- LangGraph
- LangChain and the model-provider integration configured in the notebook
- langchain-community
- wikipedia
- pandas

Install any missing dependencies in the same environment used by
the Jupyter kernel.

The additional packages used for the evaluation and Wikipedia extension
can be installed with:

```powershell
python -m pip install pandas langchain-community wikipedia
```

Configure the language model and any required credentials using
the setup cells at the beginning of the notebook.
Do not include API keys or other secrets in the submitted files.

An internet connection is required for Wikipedia searches and for
the configured model if it uses a remote API.

## How to run

1. Activate the Python environment used for the lab.
2. Start Jupyter and open langgraph_lab.ipynb.
3. Select the kernel associated with that environment.
4. Check the model configuration and required credentials.
5. Execute the notebook cells in order.

The Wikipedia tool definition must appear before the tools list
and the call to llm.bind_tools(tools).

The submitted notebook includes recorded execution outputs.
Re-executing cells generates new outputs.

## Architecture and design choices

### Classification and specialized agents

The classifier assigns each question to one of five categories.
Conditional routing sends the question to the corresponding agent.

Each agent has a category-specific system prompt.
A fallback node handles empty inputs and classifier API failures.
Unexpected classifier labels default to GENERAL.

### Tools

The final assistant has access to:
- calculator
- current_datetime
- search_wikipedia

The Wikipedia extension is implemented as a function decorated with
@tool and backed by WikipediaQueryRun.

All three tools are included in the tools list before it is passed
to llm.bind_tools(...).

When an agent requests a tool, the graph executes it and routes
the result back to the agent associated with the current category.

### Conversational memory

The final graph uses MemorySaver with a thread_id.
The memory demonstration shows that the assistant recalls the name
Ahmed within the same conversation thread.

This memory is associated with the running checkpointer instance;
it is not a durable database.

### Error handling

The implementation includes:
- A fallback response for empty questions.
- Handling of classifier API failures.
- A default category for unexpected classifier labels.
- Exception handling around specialized-agent model calls.
- ToolNode configured with handle_tool_errors=True.

Tool-error messages allow the agent to respond to a failure, but they
do not guarantee successful recovery.

### Wikipedia configuration

The Wikipedia client is configured with this User-Agent:

UniversityAssistantLab/1.0 (educational notebook)

The wrapper returns at most one search result and limits the returned
content to 2,000 characters.

An initial integration test returned JSONDecodeError messages.
After configuring the User-Agent, both the direct tool test and
the final assistant test succeeded.

## Demonstrations

The notebook includes:
- Questions covering all five categories.
- A calculator tool-call trace.
- A current-date question.
- An empty-input fallback.
- A same-thread conversational-memory test.
- A successful Wikipedia tool-call trace and final response.

## Evaluation

The introductory three-category router correctly classified all six
test questions:

- Mathematics: 2/2
- Programming: 2/2
- Artificial intelligence: 2/2

Overall routing accuracy: 6/6 (100%).

This score applies to the introductory router_graph, not to the
five-category final assistant.

The evaluation measures category selection, not the correctness
of generated answers. The small test set does not establish
performance on unseen or ambiguous questions.

## Limitations

- The assistant has no verified institution-specific data source.
- University rules, fees, schedules, and deadlines require verification.
- Wikipedia availability and model API availability may vary.
- Tool calls can fail, and repeated attempts do not guarantee recovery.
- Broader testing is needed to evaluate routing robustness.