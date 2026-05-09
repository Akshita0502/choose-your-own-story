# 📖 Choose Your Own Adventure — AI Story Generator

A full-stack web application that generates interactive, AI-powered choose-your-own-adventure stories based on a user-provided theme. Built with **FastAPI** on the backend and **React** on the frontend.


---

# flow

> User enters a theme → AI generates an opening scene → User picks choices → Story branches → Win or lose ending

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI (Python) |
| Database | SQLite via SQLAlchemy ORM |
| Validation | Pydantic v2 + pydantic-settings |
| AI | OpenAI API |
| Frontend | React + React Router DOM |
| Config | `.env` via pydantic-settings |
| Package Manager | uv |
| Deployment | Choreo |

---

## 📁 Folder Structure

```
choose-your-own-adventure/
├── backend/
│   ├── core/
│   │   ├── config.py          # Environment variables & settings
│   │   └── __init__.py
│   ├── db/
│   │   └── database.py        # SQLAlchemy engine & session setup
│   ├── models/
│   │   └── models.py          # SQLAlchemy ORM models (Story, StoryNode)
│   ├── schemas/
│   │   └── schemas.py         # Pydantic request/response schemas
│   ├── routers/
│   │   └── story.py           # FastAPI route handlers
│   ├── story_generation/
│   │   └── generator.py       # OpenAI story generation logic
│   ├── main.py                # FastAPI app entry point
│   └── .env                   # Secret keys (not committed)
├── frontend/
│   └── src/
│       ├── components/
│       │   ├── ThemeInput.jsx      # User inputs a story theme
│       │   ├── StoryLoader.jsx     # Loading state while AI generates
│       │   ├── StoryGame.jsx       # Renders scene + choices
│       │   └── LoadingStatus.jsx   # Loading indicator component
│       └── App.jsx
├── pyproject.toml
├── uv.lock
└── README.md
```

---

## 🗄️ Database Schema

The data is modelled as a **tree structure** — one Story is the root, and StoryNodes are the branches.

```
Story
 └── StoryNode (is_root=True)       ← opening scene
      ├── StoryNode                 ← choice A
      │    └── StoryNode (is_ending=True, is_winning_ending=False)
      └── StoryNode                 ← choice B
           └── StoryNode (is_ending=True, is_winning_ending=True)
```

### `Story`
| Column | Type | Description |
|---|---|---|
| `id` | Integer (PK) | Unique identifier |
| `title` | String | AI-generated story title |
| `session_id` | String | Tracks user without authentication |
| `created_at` | DateTime | Auto-set by the database |

### `StoryNode`
| Column | Type | Description |
|---|---|---|
| `id` | Integer (PK) | Unique identifier |
| `story_id` | Integer (FK) | Links node to its parent story |
| `content` | String | The scene text shown to the user |
| `is_root` | Boolean | Marks the opening scene |
| `is_ending` | Boolean | Marks a terminal node |
| `is_winning_ending` | Boolean | Win vs lose ending |
| `options` | JSON | List of choices with `text` and `next_node_id` |

**`options` example:**
```json
[
  { "text": "Enter the cave", "node_id": 5 },
  { "text": "Turn back",      "node_id": 6 }
]
```

> `options` is stored as JSON rather than a separate table for simplicity. The tradeoff is that referential integrity is not enforced at the DB level — this is acceptable for this project's scope.

---

## ⚙️ Key Design Decisions

**No authentication** — the app uses a `session_id` instead of user accounts. Since there is no personal data, payments, or restricted content, auth would add unnecessary complexity.

**`all_nodes` as a `Dict` not a `List`** — when a user picks a choice, the frontend gets a `node_id` and needs to find that node instantly. A Dict keyed by node ID gives O(1) lookup vs O(n) for a List.

**Separate request and response schemas** — `CreateStoryRequest` only contains what the client needs to send (`theme`). `CompleteStoryResponse` contains what the server returns. Keeping them separate prevents exposing internal fields and gives each schema a single clear responsibility.

**`from_attributes = True`** — SQLAlchemy returns ORM objects, not dicts. This config tells Pydantic to read from object attributes so ORM objects can be directly serialized into response schemas.

---

## 🏃 Running Locally

Backend runs at `http://localhost:8000`
Frontend runs at `http://localhost:5173`

---

## 📌 API Overview

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/stories` | Create a new story from a theme |
| `GET` | `/api/stories/{id}` | Get a full story with all nodes |
| `GET` | `/api/stories/{id}/nodes/{node_id}` | Get a specific story node |

---
(TILL TODAY)

## 📚 References

- [Tech with Tim — The Ultimate FastAPI + React Full Stack Project](https://www.youtube.com/watch?v=_1P0Uqk50Ps)
- [FastAPI Docs](https://fastapi.tiangolo.com)
- [Pydantic Docs](https://docs.pydantic.dev)
- [SQLAlchemy Docs](https://docs.sqlalchemy.org)
