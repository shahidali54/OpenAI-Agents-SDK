# OpenAI Agents SDK Learning Guide

This repository provides a beginner-friendly guide to learning the **OpenAI Agents SDK**, a Python library for building intelligent AI agents powered by large language models (LLMs) like GPT-4o. Whether you have zero coding experience or are new to AI, this guide walks you through creating and running agents step by step with detailed explanations and code examples.

## Table of Contents
- [What is the OpenAI Agents SDK?](#what-is-the-openai-agents-sdk)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Code Examples](#code-examples)
  - [Simple Agent](#simple-agent)
  - [Running an Agent](#running-an-agent)
  - [Conversational Agent](#conversational-agent)
  - [Handoffs](#handoffs)
  - [Guardrails](#guardrails)
  - [Dynamic Instructions](#dynamic-instructions)
  - [Cloning Agents](#cloning-agents)
  - [Tracing](#tracing)
  - [Customer Support System](#customer-support-system)
- [How to Run the Examples](#how-to-run-the-examples)
- [Best Practices](#best-practices)
- [Resources](#resources)
- [License](#license)

## What is the OpenAI Agents SDK?

The OpenAI Agents SDK allows you to create **AI agents** that can:
- Follow instructions to perform tasks (e.g., answer questions in haiku).
- Use tools (e.g., a weather API or calculator).
- Delegate tasks to other agents (handoffs).
- Validate inputs/outputs with guardrails.
- Debug workflows using tracing.

This repository includes code examples to help you learn how to build and run these agents, even with no prior programming knowledge.

## Prerequisites

- A computer with internet access.
- [Python 3.10+](https://www.python.org/downloads/) installed.
- An [OpenAI API key](https://platform.openai.com/docs/quickstart#create-and-export-an-api-key).
- Basic familiarity with using a terminal (Command Prompt on Windows, Terminal on macOS/Linux).

## Setup Instructions

1. **Install Python**:
   - Download and install Python 3.10 or later from [python.org](https://www.python.org/downloads/).
   - Verify installation:
     ```bash
     python --version
     ```

2. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-username/openai-agents-learning.git
   cd openai-agents-learning
   ```

3. **Set Up a Virtual Environment**:
   ```bash
   python -m venv .venv
   ```
   Activate it:
   - Windows: `.venv\Scripts\activate`
   - macOS/Linux: `source .venv/bin/activate`

4. **Install the OpenAI Agents SDK**:
   ```bash
   pip install openai-agents
   ```

5. **Set Your OpenAI API Key**:
   ```bash
   export OPENAI_API_KEY=sk-...
   ```
   Replace `sk-...` with your API key. On Windows, use:
   ```bash
   set OPENAI_API_KEY=sk-...
   ```

## Code Examples

This repository contains Python scripts demonstrating key features of the OpenAI Agents SDK. Each script is commented for clarity.

### Simple Agent
Creates an agent that responds in haiku form and uses a weather tool.

**File**: `simple_agent.py`
```python
from agents import Agent, function_tool

# Define a tool to get weather information
@function_tool
def get_weather(city: str) -> str:
    # Simulate an API call
    return f"The weather in {city} is sunny"

# Create an agent that responds in haiku form
agent = Agent(
    name="Haiku Agent",
    instructions="Always respond in haiku form",
    model="o3-mini",  # Use "gpt-4o" if available
    tools=[get_weather]
)
```

### Running an Agent
Runs the haiku agent with a user input.

**File**: `run_agent.py`
```python
from agents import Agent, function_tool, Runner
import asyncio

# Define the weather tool
@function_tool
def get_weather(city: str) -> str:
    return f"The weather in {city} is sunny"

# Create the agent
agent = Agent(
    name="Haiku Agent",
    instructions="Always respond in haiku form",
    model="o3-mini",
    tools=[get_weather]
)

# Define the main function to run the agent
async def main():
    result = await Runner.run(agent, "What's the weather in New York?")
    print(result.final_output)  # Prints the agent's response

# Run the async function
asyncio.run(main())
```

### Conversational Agent
Handles multi-turn conversations by maintaining history.

**File**: `conversation.py`
```python
from agents import Agent, Runner
import asyncio

# Create a concise agent
agent = Agent(name="Assistant", instructions="Reply very concisely.")

# Define the main function for a conversation
async def main():
    # First question
    result = await Runner.run(agent, "What city is the Golden Gate Bridge in?")
    print(result.final_output)  # Expected: San Francisco

    # Continue the conversation
    new_input = result.to_input_list() + [{"role": "user", "content": "What state is it in?"}]
    result = await Runner.run(agent, new_input)
    print(result.final_output)  # Expected: California

# Run the async function
asyncio.run(main())
```

### Handoffs
Demonstrates delegating tasks to specialized agents.

**File**: `handoffs.py`
```python
from agents import Agent

# Define specialized agents
booking_agent = Agent(name="Booking Agent", instructions="Help users book tickets")
refund_agent = Agent(name="Refund Agent", instructions="Help users with refunds")

# Create a triage agent that delegates tasks
triage_agent = Agent(
    name="Triage Agent",
    instructions="Determine if the user wants to book or refund a ticket and hand off accordingly",
    handoffs=[booking_agent, refund_agent]
)
```

### Guardrails
Ensures inputs meet specific criteria.

**File**: `guardrails.py`
```python
from agents import Agent, Guardrail

# Define a guardrail to check for offensive language
def no_offensive_language(input: str) -> bool:
    # Placeholder: assume input is safe
    return True

# Create a guardrail
guardrail = Guardrail(input_checks=[no_offensive_language])

# Create an agent with guardrails
agent = Agent(
    name="Safe Agent",
    instructions="Answer questions safely",
    guardrails=guardrail
)
```

### Dynamic Instructions
Customizes instructions based on context.

**File**: `dynamic_instructions.py`
```python
from agents import Agent, RunContextWrapper
from dataclasses import dataclass

# Define a context class
@dataclass
class UserContext:
    name: str

# Define dynamic instructions
def dynamic_instructions(context: RunContextWrapper[UserContext], agent: Agent[UserContext]) -> str:
    return f"The user's name is {context.context.name}. Help them..."

# Create an agent with dynamic instructions
agent = Agent(
    name="Triage Agent",
    instructions=dynamic_instructions
)
```

### Cloning Agents
Creates a new agent based on an existing one.

**File**: `clone_agent.py`
```python
from agents import Agent

# Create a base agent
pirate_agent = Agent(name="Pirate", instructions="Write like a pirate", model="o3-mini")

# Clone the agent with new properties
robot_agent = pirate_agent.clone(name="Robot", instructions="Write like a robot")
```

### Tracing
Logs agent workflows for debugging.

**File**: `tracing.py`
```python
from agents import Agent, Runner, trace
import asyncio

# Create an agent
agent = Agent(name="Assistant", instructions="Reply very concisely.")

# Run the agent with tracing
async def main():
    with trace(workflow_name="Conversation", group_id="thread_123"):
        result = await Runner.run(agent, "What city is the Golden Gate Bridge in?")
        print(result.final_output)  # Expected: San Francisco

# Run the async function
asyncio.run(main())
```

### Customer Support System
A complete example with a triage agent delegating to booking or refund agents.

**File**: `customer_support.py`
```python
from agents import Agent, function_tool, Runner
from dataclasses import dataclass
import asyncio

# Define a context class
@dataclass
class UserContext:
    uid: str
    is_pro_user: bool

# Define tools
@function_tool
def book_ticket(user_id: str) -> str:
    return f"Ticket booked for user {user_id}"

@function_tool
def refund_ticket(user_id: str) -> str:
    return f"Refund processed for user {user_id}"

# Create specialized agents
booking_agent = Agent(
    name="Booking Agent",
    instructions="Help users book tickets",
    tools=[book_ticket]
)
# Create specialized refund agents
refund_agent = Agent(
    name="Refund Agent",
    instructions="Help users with refunds",
    tools=[refund_ticket]
)

# Create a triage agent
triage_agent = Agent(
    name="Triage Agent",
    instructions="Determine if the user wants to book or refund a ticket and hand off accordingly",
    handoffs=[booking_agent, refund_agent]
)

# Run the system
async def main():
    user_input = "I want to book a ticket."
    context = UserContext(uid="123", is_pro_user=True)
    result = await Runner.run(triage_agent, user_input, context=context)
    print(result.final_output)

# Execute the async function
asyncio.run(main())
```

## How to Run the Examples

1. Ensure your environment is set up (see [Setup Instructions](#setup-instructions)).
2. Navigate to the repository directory:
   ```bash
   cd openai-agents-learning
   ```
3. Run any script using Python:
   ```bash
   python simple_agent.py
   ```
   Replace `simple_agent.py` with the desired script name (e.g., `run_agent.py`, `customer_support.py`).
4. View the output in the terminal or check traces in the [OpenAI Dashboard](https://platform.openai.com/).

## Best Practices

- **Start Simple**: Begin with `simple_agent.py` to understand basic agent creation.
- **Test Incrementally**: Add features like handoffs or guardrails one at a time.
- **Enable Tracing**: Use tracing (`tracing.py`) to debug complex workflows.
- **Experiment**: Modify instructions or tools to explore agent behavior.
- **Read Documentation**: Check the [OpenAI Agents SDK Documentation](https://openai.github.io/openai-agents-python/) for details.

## Resources


- [OpenAI Agents SDK Official Documentation](https://openai.github.io/openai-agents-python/)
- [OpenAI Agents SDK Quickstart](https://openai.github.io/openai-agents-python/quickstart/)
- [OpenAI Platform Developer Resources](https://platform.openai.com/docs/guides/agents)
- [GitHub Repository for OpenAI Agents SDK](https://github.com/openai/openai-agents-python)
- [Building AI Agents with OpenAI Agents SDK](https://medium.com/@sahin.samia/building-ai-agents-with-openai-agents-sdk-a-step-by-step-guide-5f1a4f1133b3)
- [OpenAI Agents SDK Tutorial by DataCamp](https://www.datacamp.com/tutorial/openai-agents-sdk-tutorial)
- 
