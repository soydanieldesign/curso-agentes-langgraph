## Create a new project

```sh
# Install python3 and create a new project.

python --version
mkdir my_agent
cd my_agent
python -m venv .venv
source .venv\Scripts\activate
where python
# install langgraph and langchain y langchain para Gemini y Gemma 4

pip install -U langgraph langchain langchain-ollama langchain-google-genai
pip install "langgraph-cli[inmem]"

# run the agent
langgraph dev
```

### Env with UV

```sh
# Install uv

curl -LsSf https://astral.sh/uv/install.sh | sh
uv --version

# deactivate the virtual environment
deactivate
rm -rf .venv

## init
uv init
uv venv

# add dependencies
uv add --pre langgraph langchain-ollama langchain-google-genai
uv add "fastapi[standard]"

# add dev dependencies
uv add "langgraph-cli[inmem]" --dev
uv add ipykernel --dev
uv add grandalf --dev

# run the agent
uv run langgraph dev



# install the project
uv pip install -e .
```

```toml
[tool.setuptools.packages.find]
where = ["src"]
include = ["*"]
```

- https://mermaidviewer.com/editor

## 🧠 Model Configuration (Local & Cloud)

Para el curso, se alternan entre **Gemma 4** (Local en disco E:) y **Gemini** (API) para mayor precisión.

<details>
<summary>Configuración de Clientes LLM</summary>

```python
import os
from langchain_ollama import ChatOllama
from langchain_google_genai import ChatGoogleGenerativeAI

# Configuración API Cloud
os.environ["GOOGLE_API_KEY"] = "TU_API_KEY_AQUI"

# OPCIÓN A: Gemma 4 (Local via Docker/Ollama)
# Nota: Los modelos residen en E:\Docker\ollama_data
llm_local = ChatOllama(model="gemma4")

# OPCIÓN B: Gemini 1.5 Flash (API)
llm_cloud = ChatGoogleGenerativeAI(model="gemini-1.5-flash")

# Para usar en los templates:
model = llm_local # o llm_cloud

### 3. Pequeña corrección en tu sección de inicio

En la parte de arriba, donde dice `source .venv\Scripts\activate`, el comando correcto es:

```sh
.venv\Scripts\activate

### Orchestrator Template

<details>
<summary>Template</summary>

```py
from langgraph.graph import MessagesState
from langgraph.graph import StateGraph, START, END
from typing import Literal

class State(MessagesState):
    nodes: list[str]


def orchestrator(state: State):
    return state


def node_1(state: State):
    return state


def node_2(state: State):
    return state


def node_3(state: State):
    return state


def aggregator(state: State):
    return state


builder = StateGraph(State)

builder.add_node('orchestrator', orchestrator)
builder.add_node('node_1', node_1)
builder.add_node('node_2', node_2)
builder.add_node('node_3', node_3)
builder.add_node('aggregator', aggregator)

builder.add_edge(START, 'orchestrator')
builder.add_edge('orchestrator', 'node_1')
builder.add_edge('orchestrator', 'node_2')
builder.add_edge('orchestrator', 'node_3')
builder.add_edge('node_1', 'aggregator')
builder.add_edge('node_2', 'aggregator')
builder.add_edge('node_3', 'aggregator')
builder.add_edge('aggregator', END)
agent = builder.compile()

```

</details>






### Evaluator Template

<details>
<summary>Template</summary>

```py
from typing import TypedDict
from langgraph.graph import StateGraph, START, END
from typing import Literal
import random

class State(TypedDict):
    nodes: list[str]

class State(MessagesState):
    nodes: list[str]


def generator_node(state: State):
    return state


def evaluator_node(state: State):
    return state


builder = StateGraph(State)

builder.add_node('generator_node', generator_node)
builder.add_node('evaluator_node', evaluator_node)

builder.add_edge(START, 'generator_node')
builder.add_edge('generator_node', 'evaluator_node')
builder.add_edge('evaluator_node', END)
agent = builder.compile()
```
</details>

### Fast API Checkpoint
Este template permite persistir el estado de los agentes utilizando PostgreSQL en un entorno FastAPI.

<details>
<summary>Ver Código</summary>

```py
import os
from contextlib import asynccontextmanager
from fastapi import FastAPI, Depends
from typing import Annotated
from langgraph.checkpoint.postgres import PostgresSaver

DB_URI = os.getenv("DB_URI")

# Global checkpointer instance
_checkpointer: PostgresSaver | None = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    global _checkpointer
    # Se conecta a la base de datos definida en DB_URI
    with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
        _checkpointer = checkpointer
        _checkpointer.setup()
        yield

def get_checkpointer() -> PostgresSaver:
    if _checkpointer is None:
        raise RuntimeError("Checkpointer not initialized. Make sure lifespan is running.")
    return _checkpointer

CheckpointerDep = Annotated[PostgresSaver, Depends(get_checkpointer)]

```
</details>
