# InteractionItem Specification
**Version:** 1.0.0-beta.0
**Last Updated:** June 15, 2026
**Status:** Beta

## Overview

The `InteractionItem` specification defines a standardized structure for representing individual items within an interaction workflow. Each `InteractionItem` represents a distinct, typed block of data that can be used as input, output, or reference in inference operations.

## Specification

### Schema Definition

```typescript
interface InteractionItem {
    // Required fields
    type: InteractionItemType;
    format: ContentFormat;
    
    // Optional fields (mutually exclusive content fields)
    key: string;
    mime?: string;
    text?: string;
    data?: string;
    url?: string;
    reference?: string;
    json?: Record<string, any>;
    objectId?: string;
}
```

### Field Specifications

#### 1. `type` (Required)

**Type:** `string`
**Description:** The role or category of the interaction item.
**Allowed Values:**

| Type | Description | Use Case |
|------|-------------|----------|
| `developer` | System-level or developer-defined items | Internal system messages, metadata |
| `input` | User-provided input to the system | User queries, prompts, uploaded files |
| `output` | System-generated output | Model responses, generated content |
| `tool_call` | Request to execute a tool/function | Function calls, API requests |
| `tool_result` | Result from a tool execution | Function responses, API results |
| `tool_error` | Error from a tool execution | Failed function calls |
| `tool_definition` | Definition of available tools | Function schemas, tool descriptions |

**Validation Rules:**
- Must be one of the allowed values
- Case-sensitive
- Required for all InteractionItems

**Examples:**
```json
{
  "type": "input"
}
```

```json
{
  "type": "tool_call"
}
```

#### 2. `key` (Optional)

**Type:** `string`
**Description:** Unique identifier for the item, supporting prefix-based naming.
**Format:** `<prefix>/<name>` or `<name>`

**Prefix Support:**
- `user/` - User-provided items
- `agent/` - Agent/system-generated items
- `tool/` - Tool-related items
- `system/` - System-level items
- `dev/` - Developer-defined items
- Custom prefixes are allowed but should follow the `<prefix>/<name>` pattern

**Validation Rules:**
- Must be a non-empty string
- Maximum length: 256 characters
- Allowed characters: alphanumeric, underscore, hyphen, forward slash, dot
- Must not start or end with a forward slash
- Forward slashes are only allowed as prefix separators

**Examples:**
```json
{
  "key": "user/prompt_1"
}
```

```json
{
  "key": "agent/response_1"
}
```

```json
{
  "key": "tool/weather_api_call"
}
```

```json
{
  "key": "system/config"
}
```

#### 3. `format` (Required)

**Type:** `string`
**Description:** The content format category.
**Allowed Values:**

| Format | Description | Common MIME Types |
|--------|-------------|-------------------|
| `text` | Plain or formatted text content | text/plain, text/markdown, text/html |
| `image` | Image data | image/png, image/jpeg, image/gif, image/webp |
| `video` | Video data | video/mp4, video/webm, video/quicktime |
| `audio` | Audio data | audio/wav, audio/mpeg, audio/ogg |
| `file` | Generic file data | application/octet-stream, application/* |
| `document` | Structured document data | application/pdf, application/json, text/csv |

**Validation Rules:**
- Must be one of the allowed values
- Case-sensitive
- Required for all InteractionItems

**Examples:**
```json
{
  "format": "text"
}
```

```json
{
  "format": "image"
}
```

#### 4. `mime` (Optional)

**Type:** `string`
**Description:** Specific MIME type of the content. Must be compatible with the specified `format`.

**Validation Rules:**
- Must be a valid MIME type string
- Must be compatible with the `format` field
- If present, must follow RFC 2046 MIME type format
- Maximum length: 100 characters

**Compatibility Matrix:**

| Format | Allowed MIME Types |
|--------|-------------------|
| `text` | text/*, application/json, application/xml |
| `image` | image/* |
| `video` | video/* |
| `audio` | audio/* |
| `file` | application/octet-stream, application/* |
| `document` | application/pdf, application/json, text/csv, text/xml, application/msword, application/vnd.* |

**Examples:**
```json
{
  "format": "text",
  "mime": "text/plain"
}
```

```json
{
  "format": "image",
  "mime": "image/png"
}
```

```json
{
  "format": "document",
  "mime": "application/pdf"
}
```

#### 5. Content Fields (Mutually Exclusive)

**Rule:** Only ONE of the following fields may be present in an InteractionItem:
- `text`
- `data`
- `url`
- `reference`
- `json`
- `objectId`

This ensures that each InteractionItem has exactly one source of content.

##### 5.1 `text` (Optional)

**Type:** `string`
**Description:** Plain text content.
**Use Case:** Text prompts, responses, messages

**Validation Rules:**
- Maximum length: 1,000,000 characters (1MB)
- UTF-8 encoded
- Must not be empty if present

**Examples:**
```json
{
  "type": "input",
  "key": "user/query",
  "format": "text",
  "mime": "text/plain",
  "text": "What is the capital of France?"
}
```

##### 5.2 `data` (Optional)

**Type:** `string`
**Description:** Embedded data, typically base64-encoded.
**Use Case:** Small binary data, encoded content

**Validation Rules:**
- Maximum length: 10,000,000 characters (10MB)
- Typically base64-encoded binary data
- Must not be empty if present

**Examples:**
```json
{
  "type": "input",
  "key": "user/image_1",
  "format": "image",
  "mime": "image/png",
  "data": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg=="
}
```

##### 5.3 `url` (Optional)

**Type:** `string`
**Description:** URL pointing to remote content.
**Use Case:** Large files, external resources

**Validation Rules:**
- Must be a valid HTTP/HTTPS URL
- Maximum length: 2048 characters
- Must use absolute URLs (not relative)
- Must be publicly accessible or authenticated via the API

**Examples:**
```json
{
  "type": "input",
  "key": "user/document_1",
  "format": "document",
  "mime": "application/pdf",
  "url": "https://example.com/documents/report.pdf"
}
```

##### 5.4 `reference` (Optional)

**Type:** `string`
**Description:** Reference pointer to another InteractionItem.
**Use Case:** Cross-references, dependencies, linking items

**Format:** `ref:<key>` or `<key>`

**Validation Rules:**
- Must reference an existing InteractionItem key
- Maximum length: 256 characters
- Must not create circular references

**Examples:**
```json
{
  "type": "reference",
  "key": "ref/user/query_1",
  "format": "text",
  "reference": "user/query_1"
}
```

```json
{
  "type": "reference",
  "key": "ref/tool_result_1",
  "format": "json",
  "reference": "ref:tool/weather_call_1"
}
```

##### 5.5 `json` (Optional)

**Type:** `object`
**Description:** Structured JSON data.
**Use Case:** Structured inputs, tool parameters, metadata

**Validation Rules:**
- Must be a valid JSON object (not array)
- Maximum size: 1MB when serialized
- Keys must be strings
- Values can be any JSON type

**Examples:**
```json
{
  "type": "tool_call",
  "key": "tool/weather_api",
  "format": "json",
  "json": {
    "function": "get_weather",
    "parameters": {
      "location": "Paris",
      "units": "metric"
    }
  }
}
```

```json
{
  "type": "tool_result",
  "key": "tool/weather_result",
  "format": "json",
  "json": {
    "temperature": 22.5,
    "humidity": 65,
    "conditions": "sunny"
  }
}
```

##### 5.6 `objectId` (Optional)

**Type:** `string`
**Description:** Reference to an uploaded Object (from the Objects API).
**Use Case:** Large files, persistent storage references

**Validation Rules:**
- Must reference an existing Object ID
- Format: `obj_<uuid>` or custom implementation-specific format
- Maximum length: 64 characters

**Examples:**
```json
{
  "type": "input",
  "key": "user/audio_1",
  "format": "audio",
  "mime": "audio/wav",
  "objectId": "obj_550e8400-e29b-41d4-a716-446655440000"
}
```

```json
{
  "type": "input",
  "key": "user/video_1",
  "format": "video",
  "mime": "video/mp4",
  "objectId": "obj_1234567890abcdef"
}
```

## Complete Examples

### Example 1: Text Input

```json
{
  "type": "input",
  "key": "user/prompt_1",
  "format": "text",
  "mime": "text/plain",
  "text": "Explain the concept of artificial intelligence in simple terms."
}
```

### Example 2: Image Input (Base64)

```json
{
  "type": "input",
  "key": "user/photo_1",
  "format": "image",
  "mime": "image/jpeg",
  "data": "base64:/9j/4AAQSkZJRgABAQEASABIAAD/2wBDAP..."
}
```

### Example 3: Tool Call

```json
{
  "type": "tool_call",
  "key": "tool/calculator",
  "format": "json",
  "json": {
    "function": "calculate",
    "operation": "add",
    "operands": [5, 3]
  }
}
```

### Example 4: Tool Result

```json
{
  "type": "tool_result",
  "key": "tool/calculator_result",
  "format": "json",
  "json": {
    "result": 8,
    "operation": "add",
    "success": true
  }
}
```

### Example 5: Tool Error

```json
{
  "type": "tool_error",
  "key": "tool/api_error",
  "format": "json",
  "json": {
    "error": "API rate limit exceeded",
    "code": "RATE_LIMIT",
    "retry_after": 60
  }
}
```

### Example 6: Tool Definition

```json
{
  "type": "tool_definition",
  "key": "tool/weather_api_def",
  "format": "json",
  "json": {
    "name": "get_weather",
    "description": "Get current weather for a location",
    "parameters": {
      "type": "object",
      "properties": {
        "location": {"type": "string"},
        "units": {"type": "string", "enum": ["metric", "imperial"]}
      },
      "required": ["location"]
    }
  }
}
```

### Example 7: Reference Item

```json
{
  "type": "reference",
  "key": "ref/previous_response",
  "format": "text",
  "reference": "agent/response_1"
}
```

### Example 8: Object Reference

```json
{
  "type": "input",
  "key": "user/document",
  "format": "document",
  "mime": "application/pdf",
  "objectId": "obj_550e8400-e29b-41d4-a716-446655440000"
}
```

### Example 9: Developer Item

```json
{
  "type": "developer",
  "key": "dev/config",
  "format": "json",
  "json": {
    "max_tokens": 1000,
    "temperature": 0.7,
    "model": "gpt-4"
  }
}
```

### Example 10: URL-based Audio Input

```json
{
  "type": "input",
  "key": "user/audio_recording",
  "format": "audio",
  "mime": "audio/wav",
  "url": "https://example.com/audio/recording.wav"
}
```

## Validation Rules

### 1. Required Fields
- `type` - Must be present and valid
- `key` - Must be present and valid
- `format` - Must be present and valid

### 2. Content Field Exclusivity
- Exactly ONE of the following must be present:
  - `text`
  - `data`
  - `url`
  - `reference`
  - `json`
  - `objectId`
- Having zero or multiple content fields is invalid

### 3. Format-Content Compatibility
- The `format` must be compatible with the content field:
  - `text` format: `text`, `json` content
  - `image` format: `data`, `url`, `objectId` content
  - `video` format: `data`, `url`, `objectId` content
  - `audio` format: `data`, `url`, `objectId` content
  - `file` format: `data`, `url`, `objectId` content
  - `document` format: `text`, `data`, `url`, `objectId`, `json` content

### 4. MIME Type Compatibility
- If `mime` is present, it must be compatible with the `format`
- See the compatibility matrix in the `mime` field specification

### 5. Key Uniqueness
- Within an interaction, all `key` values must be unique
- Keys are case-sensitive

## Error Handling

### Validation Errors

| Error Code | Description | HTTP Status |
|------------|-------------|-------------|
| `INVALID_TYPE` | Invalid or missing `type` field | 400 Bad Request |
| `INVALID_KEY` | Invalid or missing `key` field | 400 Bad Request |
| `INVALID_FORMAT` | Invalid or missing `format` field | 400 Bad Request |
| `NO_CONTENT` | No content field provided | 400 Bad Request |
| `MULTIPLE_CONTENT` | Multiple content fields provided | 400 Bad Request |
| `INCOMPATIBLE_FORMAT` | Content field incompatible with format | 400 Bad Request |
| `INVALID_MIME` | MIME type incompatible with format | 400 Bad Request |
| `INVALID_URL` | Invalid URL format | 400 Bad Request |
| `INVALID_REFERENCE` | Reference points to non-existent key | 400 Bad Request |
| `INVALID_OBJECT_ID` | Object ID does not exist | 404 Not Found |
| `CIRCULAR_REFERENCE` | Circular reference detected | 400 Bad Request |

### Error Response Format

```json
{
  "error": {
    "code": "INVALID_TYPE",
    "message": "Invalid type value. Allowed values are: developer, input, output, tool_call, tool_result, tool_error, tool_definition, reference",
    "field": "type",
    "value": "invalid_type"
  }
}
```

## Usage Guidelines

### 1. Key Naming Conventions
- Use meaningful, descriptive keys
- Include prefixes to indicate the source or purpose
- Use consistent naming patterns within an interaction
- Avoid special characters other than underscore, hyphen, and forward slash

### 2. Content Selection
- Use `text` for plain text content under 1MB
- Use `data` for small binary content (base64 encoded)
- Use `url` for large files or external resources
- Use `objectId` for files uploaded via the Objects API
- Use `reference` to link to content outside interactions.
- Use `json` for structured data

### 3. Format Selection
- Choose the most specific format that applies
- Use `text` for plain text, markdown, HTML
- Use `image` for all image formats
- Use `video` for all video formats
- Use `audio` for all audio formats
- Use `document` for structured documents (PDF, JSON, CSV, etc.)
- Use `file` for generic binary files

### 4. MIME Type Usage
- Always include `mime` when the specific type matters
- Use standard MIME types from IANA registry
- Be consistent with format and content type

### 5. Performance Considerations
- For large files (>1MB), prefer `url` or `objectId` over `data`
- For very large files (>10MB), use `objectId` with the Objects API
- Use `reference` to avoid duplicating content
- Consider the payload size limits of your API implementation

## Implementation Notes

### Backward Compatibility
This specification is designed to be backward compatible with the existing `Item` schema in UIAS. The main differences are:

1. More explicit type definitions
2. Structured prefix support for keys
3. Clearer format and MIME type relationships
4. Stricter validation rules

### Future Enhancements
Potential future additions to this specification:
- Additional content formats
- Metadata fields (timestamps, author, etc.)
- Access control information
- Versioning support for items
- Custom validation rules

## Appendix A: Type Definitions

```typescript
type InteractionItemType = 
    | 'developer'
    | 'input' 
    | 'output'
    | 'tool_call'
    | 'tool_result'
    | 'tool_error'
    | 'tool_definition'
    | 'reference';

type ContentFormat = 
    | 'text'
    | 'image'
    | 'video'
    | 'audio'
    | 'file'
    | 'document';
```

## Appendix B: JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "InteractionItem",
  "version": "1.0.0-beta.0",
  "type": "object",
  "required": ["type", "key", "format"],
  "properties": {
    "type": {
      "type": "string",
      "enum": [
        "developer", "input", "output", "tool_call", 
        "tool_result", "tool_error", "tool_definition", "reference"
      ]
    },
    "key": {
      "type": "string",
      "minLength": 1,
      "maxLength": 256,
      "pattern": "^[a-zA-Z0-9][a-zA-Z0-9_.\\-]*[a-zA-Z0-9]$|^[a-zA-Z0-9][a-zA-Z0-9_.\\-]*/[a-zA-Z0-9][a-zA-Z0-9_.\\-]*[a-zA-Z0-9]$"
    },
    "format": {
      "type": "string",
      "enum": ["text", "image", "video", "audio", "file", "document"]
    },
    "mime": {
      "type": "string",
      "maxLength": 100
    },
    "text": {
      "type": "string",
      "maxLength": 1000000
    },
    "data": {
      "type": "string",
      "maxLength": 10000000
    },
    "url": {
      "type": "string",
      "format": "uri",
      "maxLength": 2048
    },
    "reference": {
      "type": "string",
      "maxLength": 256
    },
    "json": {
      "type": "object",
      "additionalProperties": true
    },
    "objectId": {
      "type": "string",
      "maxLength": 64
    }
  },
  "oneOf": [
    {"required": ["text"]},
    {"required": ["data"]},
    {"required": ["url"]},
    {"required": ["reference"]},
    {"required": ["json"]},
    {"required": ["objectId"]}
  ],
  "additionalProperties": false
}
```

## Appendix C: Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0-beta.0 | 2026-06-15 | Initial release |

---

**Document Status:** Beta
**Next Review Date:** 2026-07-15
**Maintainer:** Universal Inference API Standard Working Group