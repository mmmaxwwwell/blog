# Blog Embeddings Search — Feature Prompt

## What we're building
A local MCP (Model Context Protocol) server that indexes all blog posts from the Astro blog at `/home/max/git/blog` into a vector embeddings store, enabling semantic search so agents can quickly find relevant posts and knowledge by meaning rather than just keywords. The server runs locally, integrates with the blog's build pipeline, and exposes tools via MCP that any compatible agent (Claude Code, etc.) can call.

## Context
- **Blog repo:** `/home/max/git/blog` — Astro static site, posts are `.mdx` files in `src/content/blog/`
- **Blog structure:** Each post is a directory containing `index.mdx` with YAML frontmatter (title, description, date, category, tags) and MDX body content
- **Current post count:** 1 (will grow to hundreds across categories: electrical, openscad, 3d-printing, software-dev)
- **Goal:** Agents working on any of my projects can query the blog's knowledge base semantically — e.g., "what did I learn about ESP32 deep sleep" or "how did I set up Klipper pressure advance" — and get back the relevant post chunks with source references
- **Nix:** The blog repo uses a Nix flake for its dev environment. This tool should either live in the blog repo or be a standalone tool that can be pointed at the blog

## Tech stack & architecture
- **Language:** Node.js/TypeScript (matches the blog repo)
- **Embeddings:** Ollama with `nomic-embed-text` (local, free, no API key needed). The indexing script calls Ollama's REST API at `http://localhost:11434/api/embeddings`. Ollama must be running locally — the script should check and give a clear error if it's not.
- **Storage:** SQLite with a JSON column for vectors + brute-force cosine similarity (no need for a vector extension at this scale — hundreds of posts, not millions). The DB is committed to the repo so embeddings persist and don't need to be re-computed. Only new/changed posts hit the embeddings API during indexing.
- **Chunking strategy:** Split each post by `##` headings. Each chunk includes the post's frontmatter metadata (title, date, tags, category) plus the heading and body text under that heading. If a post has no `##` headings, treat the whole post as one chunk.
- **MCP server:** Exposes tools over the Model Context Protocol (stdio transport) so any MCP-compatible agent can search the blog
  - `search_posts` tool — takes a query string, returns top-N most similar chunks with post title, date, tags, chunk heading, relevance score, and a snippet
  - `list_posts` tool — returns all indexed posts with metadata (title, date, category, tags)
  - `reindex` tool — triggers a re-index of all posts (or only changed posts based on file mtime vs last indexed timestamp)
- **CLI interface** (for manual use):
  - `npm run index` — re-index all posts
  - `npm run search -- "query text here"` — search from the command line
  - `npm run serve` — start the MCP server

### Key paths (target structure inside blog repo)

    embeddings/
    ├── server.ts         # MCP server entry point — registers tools, handles requests
    ├── index.ts          # Indexing script — reads posts, chunks, embeds, stores
    ├── search.ts         # CLI search script for manual use
    ├── lib/
    │   ├── chunker.ts    # MDX parsing + chunking by ## headings
    │   ├── embeddings.ts # API client for generating embeddings
    │   ├── store.ts      # SQLite read/write for chunks + vectors
    │   └── similarity.ts # Cosine similarity + ranking
    ├── db/
    │   └── blog.db       # SQLite database (committed to repo — small enough to track)
    ├── package.json      # Dependencies (@modelcontextprotocol/sdk, better-sqlite3, etc.)
    └── tsconfig.json

## Data model

### Chunks table (SQLite)

```sql
CREATE TABLE chunks (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  post_path TEXT NOT NULL,          -- relative path to the .mdx file
  post_title TEXT NOT NULL,
  post_date TEXT NOT NULL,
  post_category TEXT NOT NULL,
  post_tags TEXT NOT NULL,          -- JSON array
  chunk_heading TEXT,               -- the ## heading this chunk is under (null if whole-post)
  chunk_text TEXT NOT NULL,         -- the actual text content
  chunk_index INTEGER NOT NULL,     -- position within the post (0-based)
  embedding TEXT NOT NULL,          -- JSON array of floats
  indexed_at TEXT NOT NULL,         -- ISO timestamp
  source_mtime TEXT NOT NULL        -- file mtime at time of indexing, for incremental updates
);

CREATE INDEX idx_chunks_post_path ON chunks(post_path);
```

## How it should work

### Indexing flow
1. Glob `src/content/blog/**/index.mdx`
2. For each post, check if `source_mtime` matches current file mtime — skip if unchanged
3. Parse frontmatter (YAML between `---` delimiters)
4. Split body by `##` headings into chunks
5. For each chunk, prepend context: `"Title: {title}\nTags: {tags}\nCategory: {category}\n\n## {heading}\n{body}"`
6. Batch-embed all new chunks via the embeddings API
7. Delete old chunks for changed posts, insert new ones
8. Report: "Indexed 3 new posts, 12 chunks. Skipped 47 unchanged posts."

### Publishing workflow
Indexing happens locally only — GitHub Actions never generates embeddings. After writing a new post:
1. Ensure Ollama is running locally with `nomic-embed-text`
2. Run `npm run index` to embed new/changed posts (only hits Ollama for changes)
3. Commit the updated `embeddings/db/blog.db` alongside the new post
4. Push — GitHub Actions builds and deploys the Astro site as usual, ignoring embeddings entirely
5. MCP server loads the committed DB on startup — no Ollama needed for search

### Search flow
1. Embed the query string
2. Load all chunk embeddings from SQLite
3. Compute cosine similarity between query embedding and each chunk
4. Return top-N results (default 5) sorted by similarity score
5. Output: post title, date, category, tags, chunk heading, similarity score, first 200 chars of chunk text
