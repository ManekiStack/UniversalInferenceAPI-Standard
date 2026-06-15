# Universal Inference API Standard (UIAS)
**Version:** 1.8.0
**Last Updated:** June 15, 2026
## Core Philosophy: The Universal Gateway
The vision behind the **Universal Inference API Standard (UIAS)** is simple: **One standard to handle *any* inference task.** Whether you are generating conversational text, embedding documents into vector space, synthesizing speech, classifying images, or running complex multi-agent workflows, UIAS dictates the exact same endpoint, payload structure, and session management. By explicitly defining the task in the payload, the API routes your multimodal inputs to the appropriate compute engine and standardizes the output, eliminating the need for fragmented, model-specific integrations.
## Base URL
```text
[https://api.example.com/v1](https://api.example.com/v1)
```
## Authentication
UIAS implementations use **API keys** for authentication. Include your API key in the Authorization header of all requests:
```http
Authorization: Bearer YOUR_API_KEY
```
## Endpoints: Discovery
### 1. List Providers
**Endpoint:** GET /providers
**Description:**
Retrieve a list of all AI compute providers supported by the platform.
#### Response
**Status Code:** 200 OK
```json
{
  "providers": [
    {
      "providerId": "openai",
      "name": "OpenAI",
      "website": "[https://openai.com](https://openai.com)"
    },
    {
      "providerId": "mistral",
      "name": "Mistral AI",
      "website": "[https://mistral.ai](https://mistral.ai)"
    }
  ],
  "total": 2
}
```
### 2. List Tasks
**Endpoint:** GET /tasks
**Description:**
Retrieve a list of all UIAS standardized tasks supported by the engine, including descriptions and expected input/output content types.
#### Response
**Status Code:** 200 OK
```json
{
  "tasks": [
    {
      "taskId": "generate_text",
      "name": "Text Generation",
      "description": "Standard LLM completion, chat, and reasoning.",
      "expectedOutputTypes": ["text/plain", "application/json"]
    },
    {
      "taskId": "generate_video",
      "name": "Video Generation",
      "description": "Text-to-video or image-to-video generation.",
      "expectedOutputTypes": ["video/mp4"]
    }
  ],
  "total": 12
}
```
### 3. List Models
**Endpoint:** GET /models
**Description:**
Retrieve a list of available models. You can filter the results by specific providers or tasks to dynamically discover capabilities.
#### Query Parameters

| Parameter | Type | Description | Required |
| :--- | :--- | :--- | :--- |
| providerId | string | Filter models belonging to a specific provider. | No |
| task | string | Filter models that support a specific task. | No |
| limit | integer | Maximum number of models to return. | No |
| offset | integer | Number of models to skip. | No |

#### Request Example
GET /models?task=generate_video&providerId=openai
#### Response
**Status Code:** 200 OK
```json
{
  "models": [
    {
      "modelId": "sora",
      "providerId": "openai",
      "name": "Sora",
      "supportedTasks": ["generate_video"],
      "contextWindow": null
    }
  ],
  "total": 1,
  "limit": 10,
  "offset": 0
}
```
### 4. Retrieve a Model
**Endpoint:** GET /models/{modelId}
**Description:**
Retrieve detailed information about a specific model.
#### Response
**Status Code:** 200 OK *(Returns a Model object)*
## Endpoints: Inference & Interactions
*(Note: The concept of "Sessions" has been deprecated in favor of "Interactions" to support Directed Acyclic Graph (DAG) conversation structures. Backwards compatibility for `/sessions` endpoints is optional but recommended for legacy clients.)*

### 5. Run Inference
**Endpoint:** POST /inference
**Description:**
Submit a request to run an inference task. The task parameter explicitly defines the nature of the operation. You can request synchronous processing, streaming (Server-Sent Events), or asynchronous processing (Promises) for long-running models.
#### Request (Async Text-to-Video Example)
**Content-Type:** application/json
**Body:**
```json
{
  "task": "generate_video",
  "providerId": "openai",
  "modelId": "sora",
  "interactionId": "opaque_string_from_previous_response",
  "fork": true,
  "async": true,
  "options": {
    "resolution": "1080p",
    "durationSeconds": 10
  },
  "input": [
    {
      "type": "input",
      "source": "user",
      "contentType": "text/plain",
      "key": "user/prompt_1",
      "text": "A cinematic shot of a golden retriever surfing in Hawaii."
    }
  ]
}
```
#### Response (Asynchronous - 202 Accepted)
When async: true is passed, the API responds immediately with a promise object tracking the execution state.
**Content-Type:** application/json
**Status Code:** 202 Accepted
```json
{
  "task": "generate_video",
  "providerId": "openai",
  "modelId": "sora",
  "promise": {
    "promiseId": "prom_9a8b7c6d",
    "status": "pending",
    "createdAt": "2026-06-14T12:50:00.000Z",
    "checkUrl": "[https://api.example.com/v1/promises/prom_9a8b7c6d](https://api.example.com/v1/promises/prom_9a8b7c6d)"
  }
}
```
#### Response (Synchronous - 200 OK)
If async: false (default) and stream: false, the API holds the connection and returns the standard inference output block once complete.
```json
{
  "task": "generate_text",
  "providerId": "mistral",
  "modelId": "mistral-medium-3.5",
  "interactionId": "new_opaque_string_for_next_request",
  "startedAt": "2026-06-15T12:05:00.000Z",
  "completedAt": "2026-06-15T12:05:01.123Z",
  "usage": {
    "inputTokens": 15,
    "outputTokens": 45,
    "totalTokens": 60
  },
  "output": [
    {
      "type": "output",
      "source": "agent",
      "contentType": "text/plain",
      "key": "agent/response_1",
      "text": "Artificial intelligence is rapidly evolving..."
    }
  ]
}
```
### 6. Await/Retrieve Promise
**Endpoint:** GET /promises/{promiseId}
**Description:**
Poll or await the status of an asynchronous inference task. If the status is completed, the standard inference response is nested inside the result field.
#### Response (Completed)
**Status Code:** 200 OK
```json
{
  "promiseId": "prom_9a8b7c6d",
  "status": "completed",
  "createdAt": "2026-06-14T12:50:00.000Z",
  "updatedAt": "2026-06-14T12:51:30.000Z",
  "result": {
    "task": "generate_video",
    "providerId": "openai",
    "modelId": "sora",
    "startedAt": "2026-06-14T12:50:02.000Z",
    "completedAt": "2026-06-14T12:51:30.000Z",
    "usage": {
      "computeTimeMs": 88000
    },
    "output": [
      {
        "type": "output",
        "source": "agent",
        "contentType": "video/mp4",
        "key": "agent/video_1",
        "url": "[https://api.example.com/assets/vid_999.mp4](https://api.example.com/assets/vid_999.mp4)"
      }
    ]
  }
}
```
### 7. Create an Interaction
**Endpoint:** POST /interactions
**Description:**
Create a new interaction root for tracking a continuous conversation or workflow. (Optional, as `/inference` will auto-initialize if no `interactionId` is provided).
#### Response
**Status Code:** 201 Created *(Returns Interaction Object)*
### 8. Retrieve Interaction History
**Endpoint:** GET /interactions/{interactionId}/history
**Description:**
Returns the linear history of that specific branch by traversing up to the root.
#### Response
**Status Code:** 200 OK *(Returns array of Items)*
### 9. List Interactions
**Endpoint:** GET /interactions
**Description:**
List root interactions.
#### Response
**Status Code:** 200 OK *(Returns paginated Interaction list)*
### 10. Delete an Interaction
**Endpoint:** DELETE /interactions/{interactionId}
**Description:**
Deletes a branch, or the whole tree if it's the root.
#### Response
**Status Code:** 204 No Content
## Endpoints: Objects (CRUD)
Objects allow you to upload files (images, audio, PDFs, CSVs) once and reuse them across multiple inference calls by passing the objectId.
### 11. Create (Upload) an Object
**Endpoint:** POST /objects
#### Request
**Content-Type:** multipart/form-data

| Field | Type | Description | Required |
| :--- | :--- | :--- | :--- |
| file | binary | The file to upload. | Yes |
| purpose | string | Purpose of the file (e.g., inference, fine-tune). | Yes |
| metadata | string | Optional JSON string containing custom metadata. | No | <br> #### Response <br> **Status Code:** 201 Created *(Returns Object metadata)* <br> ### 12. Retrieve an Object <br> **Endpoint:** GET /objects/{objectId} <br> #### Response <br> **Status Code:** 200 OK *(Returns Object metadata)* <br> ### 13. List Objects <br> **Endpoint:** GET /objects <br> #### Response <br> **Status Code:** 200 OK *(Returns paginated Object list)* <br> ### 14. Update an Object <br> **Endpoint:** PATCH /objects/{objectId} <br> #### Response <br> **Status Code:** 200 OK *(Returns updated Object metadata)* <br> ### 15. Delete an Object <br> **Endpoint:** DELETE /objects/{objectId} <br> #### Response <br> **Status Code:** 204 No Content <br> ## Schemas <br> ### Provider <br> Represents an AI compute provider.
| Field | Type | Description | Required | Example Values |
| :--- | :--- | :--- | :--- | :--- |
| providerId | string | Unique identifier for the provider. | Yes | "openai", "anthropic" |
| name | string | Human-readable name. | Yes | "OpenAI" |
| website | string | Provider's official website. | No | "https://openai.com" | <br> ### Model <br> Represents a specific AI model and its capabilities.
| Field | Type | Description | Required | Example Values |
| :--- | :--- | :--- | :--- | :--- |
| modelId | string | Unique identifier for the model. | Yes | "gpt-4o", "claude-3-opus" |
| providerId | string | The provider that hosts this model. | Yes | "openai" |
| name | string | Human-readable name. | Yes | "GPT-4o" |
| supportedTasks | array | Array of standard tasks this model supports. | Yes | ["generate_text", "analyze_image"] |
| contextWindow | integer | Maximum tokens supported (if applicable). | No | 128000 | <br> ### Item <br> Represents a distinct, typed block of data in the inference workflow.
| Field | Type | Description | Required | Example Values |
| :--- | :--- | :--- | :--- | :--- |
| type | string | Role of the item. | Yes | input, output, system, tool_call, tool_result |
| source | string | Originator of the item. | Yes | user, agent, tool, system |
| contentType | string | Standard MIME Type of content. | Yes | text/plain, application/pdf, video/mp4 |
| key | string | Unique identifier for the item. | Yes | user/query_1, agent/embedding_1 |
| text | string | Text content. | No* | "What is the weather?" |
| url | string | URL for remote content. | No* | "https://example.com/image.png" |
| data | string | Embedded data (e.g., base64). | No* | "base64:iVBORw0KGgo..." |
| objectId | string | Reference to an uploaded Object. | No* | "obj_abc123" |
| reference | string | Reference pointer to another item. | No* | "ref:user/query_1" |
| json | object | Structured JSON data. | No* | {"location": "Paris"} |
| vector | array | Array of floating-point numbers. | No* | [0.1, -0.02, 0.45] | <br> ***Note:** Only **one** of text, url, data, objectId, reference, json, or vector can be present in an item.* <br> ### Promise <br> Represents an asynchronous task executing on the platform.
| Field | Type | Description | Required | Example Values |
| :--- | :--- | :--- | :--- | :--- |
| promiseId | string | Unique identifier for the asynchronous task. | Yes | "prom_9a8b7c" |
| status | string | Current state of execution. | Yes | "pending", "processing", "completed", "failed" |
| createdAt | string | Timestamp when the promise was generated. | Yes | "2026-06-14T12:50:00Z" |
| updatedAt | string | Timestamp when status was last modified. | No | "2026-06-14T12:51:30Z" |
| checkUrl | string | Convenience URL to poll the promise status. | Yes | "https://api.example.com/v1/promises/prom_9a8b7c" |
| result | object | The final inference response (if completed). | No | *See Response (Synchronous)* |
| error | object | Error details (if failed). | No | {"code": "timeout", "message": "..."} | <br> ## Standardized Tasks (UIAS Core) <br> To ensure predictability across providers, the task field should utilize one of the following standardized strings:
| Task Name | Description | Expected Primary Output Type |
| :--- | :--- | :--- |
| generate_text | Standard LLM completion, chat, and reasoning. | text/plain, text/markdown, application/json (Tool calls) |
| generate_embeddings | Converts input text/images into vector arrays. | application/vector |
| generate_image | Text-to-image or image-to-image generation. | image/png, image/jpeg |
| generate_video | Text-to-video or image-to-video generation. | video/mp4, video/webm |
| generate_audio | Text-to-speech (TTS) or audio synthesis. | audio/wav, audio/mpeg |
| transcribe_audio | Speech-to-text (STT) or translation. | text/plain, application/json (Timestamps) |
| analyze_image | Visual Q&A, OCR, or bounding-box detection. | text/plain, application/json |

## Usage Guidelines
 1. **Async Flag:** Set "async": true in the root of the /inference payload to request a Promise instead of holding a synchronous HTTP connection open. This is strongly recommended for tasks like generate_video or generate_image.
 2. **Polling Promises:** Use the checkUrl returned in the promise block to poll for updates. Implement exponential backoff to avoid hitting rate limits on the GET /promises/{promiseId} endpoint.
 3. **Explicit Routing via Task:** The task field is mandatory in the /inference request. The API uses this to validate input formats and route the request to the correct modality engine.
 4. **File References (Objects):** Use the POST /objects endpoint to upload large files (audio files for transcription, images for analysis). Pass the resulting objectId inside your inference payload instead of base64-encoding the data.
 5. **Discovery:** Use /models?task={task} to query which underlying models are capable of fulfilling your specific multi-modal requests before submitting them to /inference.
 6. **Interaction IDs (Opaque):** Clients must treat `interactionId` as an opaque string. To continue a conversation, pass the `interactionId` received from the previous response. To branch a conversation from a specific point, pass that point's `interactionId` and set `"fork": true`.

## Implementation Guidelines

### Stateless Interaction IDs
To efficiently support explicit forking without heavy database lookups, backend implementations are strongly encouraged to structure the opaque `interactionId` internally as a composite key: `[ParentUUID]-[InteractionUUID]-[SequenceID]`.

*   **ParentUUID (128-bit):** The ID of the branch this interaction was forked from (0s if root).
*   **InteractionUUID (128-bit):** The unique ID of the current branch.
*   **SequenceID (32-bit integer):** The current turn number in this branch.

**Behavior:**
*   **Initialization:** Server generates `0000...-UUID_A-0`.
*   **Standard Turn (`fork: false`):** Server increments sequence and returns `0000...-UUID_A-1`.
*   **Explicit Fork (`fork: true`):** Server takes the current ID, sets it as the parent, generates a new Interaction UUID, resets sequence to 0, and returns `UUID_A-UUID_B-0`.

The server can optionally encrypt or Base64URL-encode this composite string before returning it to the client to ensure it remains fully opaque.