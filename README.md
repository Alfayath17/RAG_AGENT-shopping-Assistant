# ShopEasy Shopping Assistant

An AI shopping assistant that answers product questions from a PDF catalog (RAG) and manages a shopping cart. It is built with **CrewAI**, exposes its tools through an **MCP server**, and can be used from a **Streamlit** chat UI or a **FastAPI** REST API.

## How it works

```
Streamlit UI (app.py) ─┐
                       ├─► CrewAI agent (agent.py, gpt-4o-mini)
FastAPI (api.py) ──────┘            │
                                    │ MCP over stdio
                                    ▼
                          MCP server (mcp_server.py)
                           ├─ search_products ─► rag.py ─► ChromaDB ◄─ products_catalog.pdf
                           └─ cart tools ──────► cart.xlsx
```

- **`rag.py`**: reads `products_catalog.pdf`, splits it into one chunk per product, and stores the chunks in a local ChromaDB database (`chroma_db/`) using OpenAI `text-embedding-3-small` embeddings. The database is built only on the first run.
- **`mcp_server.py`**: a FastMCP server with these tools:
  - `search_products`: semantic search over the catalog
  - `add_to_cart`, `remove_from_cart`, `view_cart`, `clear_cart`: manage the cart stored in `cart.xlsx`
- **`agent.py`**: starts the MCP server, loads its tools, and defines the CrewAI agent, task and crew.
- **`app.py`**: Streamlit chat interface, with the current cart shown in the sidebar.
- **`api.py`**: FastAPI endpoints for the same agent.

## Requirements

- Python 3.10+
- An OpenAI API key

## Setup

```powershell
# 1. Create and activate a virtual environment
python -m venv venv
.\venv\Scripts\activate        # Windows
# source venv/bin/activate     # macOS / Linux

# 2. Install dependencies
pip install -r requirements.txt

# 3. Add your OpenAI API key
copy .env.example .env         # Windows
# cp .env.example .env         # macOS / Linux
# then edit .env and set OPENAI_API_KEY
```

## Running

Always run with the virtual environment activated.

**Streamlit chat app**

```powershell
streamlit run app.py
```

Then open http://localhost:8501. Example questions:
- *Which smart watch do you have?*
- *Add 2 power banks to my cart*
- *What's in my cart?*

**REST API**

```powershell
uvicorn api:app --reload
```

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET    | `/`      | Health check |
| POST   | `/ask`   | Ask the agent. Body: `{"question": "...", "history": ""}` |
| GET    | `/cart`  | Current cart items and grand total |

Interactive docs are available at http://localhost:8000/docs.

**Quick test from the command line**

```powershell
python agent.py
```

## Notes

- `chroma_db/` and `cart.xlsx` are generated automatically and are not committed. To rebuild the product index after changing the PDF, delete the `chroma_db/` folder.
- If you see `You are missing the 'mcp' package` or `Couldn't connect to the MCP server`, the dependencies are not fully installed in the active environment. Run `pip install -r requirements.txt` inside the venv.
