# Architecture

## Document ingestion

Google Drive Trigger → Download file → Default Data Loader → Recursive Character Text Splitter → Cohere embeddings → Supabase Vector Store (insert)

The loader records the source filename in metadata. The splitter is configured for 500-character chunks with 200-character overlap.

## Question answering

Webhook → AI Agent → Respond to Webhook

The AI Agent is connected to:
- Google Gemini Chat Model
- Simple Memory, keyed using `body.sessionId` (fallback: `default`)
- Supabase Vector Store in retrieve-as-tool mode
- Cohere `embed-english-v3.0` embeddings for query embedding

The agent prompt instructs it to consult retrieved content and say when the documents do not contain the answer. The prompt currently references a space station handbook; change this to match your actual knowledge base.
