# VoiceRAG-AI

A voice-enabled Retrieval-Augmented Generation (RAG) project. The included n8n workflows provide the document-ingestion and question-answering backend intended to be connected to an ElevenLabs Voice AI Agent.

## Workflows included

### 1. Document ingestion — `workflows/document-ingestion.json`

1. Watches a selected Google Drive folder for newly created files (polling every minute).
2. Downloads each detected file.
3. Loads the file content and attaches the source filename as metadata.
4. Splits text using a Recursive Character Text Splitter (500-character chunks, 200-character overlap).
5. Generates embeddings with Cohere `embed-english-v3.0`.
6. Inserts document chunks and embeddings into the Supabase `documents` table.

### 2. RAG query API — `workflows/rag-query.json`

1. Receives a POST request through an n8n Webhook.
2. Reads the question from `body.Question`.
3. Uses a Gemini-powered AI Agent to answer the question.
4. Uses the Supabase Vector Store as a retrieval tool, with Cohere embeddings.
5. Uses Simple Memory keyed by `body.sessionId` (or `default` if no session ID is supplied).
6. Returns the agent response through Respond to Webhook.

The query workflow's system prompt is currently tailored to a **space station handbook**. Update it if your document collection covers another subject.

## Import and configure in n8n

1. In n8n, choose **Workflows → Import from File**.
2. Import `workflows/document-ingestion.json`.
3. Import `workflows/rag-query.json`.
4. Reconnect credentials in the relevant nodes:
   - Google Drive OAuth2
   - Cohere API
   - Supabase API
   - Google Gemini
5. In the Google Drive Trigger, select the folder you want to watch.
6. Confirm that your Supabase project has the required `documents` table and matching vector-search function/schema for your n8n Supabase Vector Store node.
7. In the query workflow, configure the Webhook path and verify the expected request body.
8. Test both workflows manually before activating them.

## Query webhook contract

The query workflow reads these fields:

```json
{
  "Question": "What does the handbook say about emergency procedures?",
  "sessionId": "optional-conversation-session-id"
}
```

Send this JSON as the body of a POST request to the n8n Webhook URL. The `sessionId` should remain stable during a conversation if you want Simple Memory to associate turns with the same session.

The exact response payload depends on the n8n Respond to Webhook node configuration and execution output. Test it in your own n8n instance before configuring ElevenLabs to consume it.

## ElevenLabs integration

The two included files are the RAG backend workflows. They do **not** include an ElevenLabs agent export or agent configuration. Configure an ElevenLabs tool to call the deployed query webhook, pass the user's question in `Question`, and pass a stable conversation/session identifier in `sessionId` if needed. Map the returned n8n response field to the agent's tool output after verifying the actual webhook response.

