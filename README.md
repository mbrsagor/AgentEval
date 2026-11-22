# AgentEval

> AgentEval is a small Python project for experimenting with a conversational refund support agent and evaluating its tool usage with DeepEval. The agent is built with LangChain/LangGraph and uses structured responses to decide whether a conversation should continue. It looks up simulated purchase history, checks refund eligibility, calculates eligible refund amounts, and creates refund tickets in the local demo environment.

## Features

- Interactive, multi-turn customer refund conversations
- Tool calling for purchase history, dates, refund calculations, and tickets
- In-memory conversation state through `InMemorySaver`
- Structured agent responses with Pydantic
- A synthetic purchase-history dataset
- A DeepEval dataset containing refund scenarios and expected tool calls

## How it works

The agent receives a refund request, identifies the customer and requested items,
and uses these tools:

| Tool | Purpose |
| --- | --- |
| `get_user_purchase_history` | Finds a user's purchase history or reports an unknown user |
| `get_today_date` | Returns the current date in `YYYY-MM-DD` format |
| `calculate_right_refund` | Calculates a refund for requested, valid purchases |
| `issue_refund_ticket` | Records a simulated refund-ticket request |

The project applies a 30-day refund rule and only returns items the customer
actually purchased and requested. This is a demo implementation: refund tickets
are not sent to a real banking or ticketing system.

## Project structure

```text
.
├── agent/
│   ├── agent.py       # Creates the LangGraph agent and runs the chat loop
│   ├── data.py        # Synthetic users and purchase records
│   ├── models.py      # Chat model configuration
│   ├── objects.py     # Pydantic response and purchase-item models
│   ├── prompts.py     # Refund-support system prompt
│   └── tools.py       # LangChain tools
├── evaluations/
│   └── dataset.py     # DeepEval goldens and expected tool calls
├── main.py            # Current placeholder entry point
└── pyproject.toml     # Dependencies and Python requirement
```

## Requirements

- Python 3.14 or newer
- [`uv`](https://docs.astral.sh/uv/) for environment and dependency management
- API credentials for the chat model configured in `agent/models.py`

## Setup

Clone the repository and install dependencies:

```bash
git clone <repository-url>
cd agentEval
uv sync
```

Create a local environment file:

```bash
cp .env.example .env
```

For the default OpenAI-compatible configuration, set the API key in `.env`:

```dotenv
OPENAI_API_KEY=your-api-key
```

If the model provider or model name in `agent/models.py` is changed, update the
corresponding provider credentials and environment variables as required by
LangChain.

## Run the agent

Start the interactive refund conversation from the project root:

```bash
uv run python agent/agent.py
```

The demo starts with:

```text
I would like to request a refund
```

Provide a username and the items you want to return. For example:

```text
reply:
I am user123, I would like a refund for the book and pen
```

The conversation continues until the agent sets `conversation` to `false` in its
structured response. Valid sample users include `user123`, `user126`, and
`user127`; add or modify users in `agent/data.py` for new scenarios.

## Evaluations

The evaluation goldens in `evaluations/dataset.py` cover scenarios such as:

- Refund for multiple eligible items
- Refund for a single eligible item
- Purchase-history lookup without a refund
- Unknown customers
- Purchases outside the 30-day window
- A valid item combined with an expired item
- Items that were not purchased

The goldens define expected responses and tool calls, but this repository currently
does not include a separate evaluation runner. Load and execute the dataset with
your preferred DeepEval workflow.

## Configuration

### Chat model

The model is initialized in `agent/models.py` with:

```python
model = init_chat_model(model="gpt-6-luna")
```

Replace `gpt-6-luna` with a model available to your configured provider before
running the project.

### Agent behavior

Update `agent/prompts.py` to change refund policies or response instructions.
Update `agent/tools.py` to connect tools to real services; update `agent/data.py`
to change sample purchase records.

## Limitations

- Purchase data is stored in memory and resets when the process starts
- Dates in the sample data are fixed and require updating for long-running use
- Refund tickets are simulated; no external action is performed
- `main.py` is only a placeholder and does not currently start the agent
- The evaluation dataset does not yet have a built-in CLI runner
