---
name: 23blocks-rag-ingestion-api
description: Process files for RAG vector embeddings and object detection. Use when ingesting documents for semantic search, detecting objects in images with YOLOv8, or generating CLIP embeddings for visual search.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# RAG Ingestion API

Process files to generate vector embeddings for semantic search. Supports OCR text extraction and YOLOv8 object detection with per-object CLIP embeddings.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | RAG API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### POST /products/:product_id/files/:file_id/process - Process File

Processes an uploaded file to generate vector embeddings. Supports OCR text extraction and object detection modes.

**Resource scoping:** Replace `/products/:product_id` with the appropriate resource path:
- `/products/:id` — Product catalog files
- `/accounts/:id` — CRM account files
- `/contacts/:id` — CRM contact files
- `/users/:id` — User profile files
- `/storage/:company_url` — Company storage files

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/$PRODUCT_ID/files/$FILE_ID/process" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "processing_mode": "ocr_text"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `processing_mode` | string | Yes | Processing mode: `ocr_text` or `object_detection` |

**Response 200 (OCR Text mode):**
```json
{
  "success": true,
  "job_id": "job_xyz",
  "file_unique_id": "file_abc",
  "processing_mode": "ocr_text",
  "chunks_created": 4,
  "vectors_upserted": 4,
  "tokens_used": 250,
  "metadata": {
    "is_image_based": false,
    "ocr_used": false
  },
  "phase_1_completed": true,
  "phase_2_completed": true
}
```

**Request (Object Detection mode):**
```bash
curl -X POST "$BLOCKS_API_URL/products/$PRODUCT_ID/files/$FILE_ID/process" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "processing_mode": "object_detection"
  }'
```

**Response 200 (Object Detection mode):**
```json
{
  "success": true,
  "job_id": "job_xyz",
  "file_unique_id": "file_abc",
  "processing_mode": "object_detection",
  "chunks_created": 2,
  "objects_detected": 3,
  "vectors_upserted": 5,
  "tokens_used": 100,
  "metadata": {
    "is_image_based": true,
    "ocr_used": true
  },
  "detection_metadata": [
    {
      "detection": "multi_object",
      "object_index": 0,
      "total_objects": 3,
      "class": "bottle",
      "confidence": 0.87,
      "bbox": [100, 200, 300, 500],
      "width": 200,
      "height": 300,
      "area": 60000,
      "is_crop": true
    },
    {
      "detection": "multi_object",
      "object_index": 1,
      "total_objects": 3,
      "class": "bottle",
      "confidence": 0.92,
      "bbox": [350, 180, 550, 480],
      "width": 200,
      "height": 300,
      "area": 60000,
      "is_crop": true
    },
    {
      "detection": "multi_object",
      "object_index": 2,
      "total_objects": 3,
      "class": "wine glass",
      "confidence": 0.78,
      "bbox": [600, 250, 750, 550],
      "width": 150,
      "height": 300,
      "area": 45000,
      "is_crop": true
    }
  ],
  "phase_1_completed": true,
  "phase_2_completed": true
}
```

---

## Data Models

### ProcessingResult

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether processing completed successfully |
| `job_id` | string | Unique job identifier |
| `file_unique_id` | string | Processed file ID |
| `processing_mode` | string | Mode used: `ocr_text` or `object_detection` |
| `chunks_created` | integer | Number of text/embedding chunks created |
| `objects_detected` | integer | Number of objects found (object_detection mode only) |
| `vectors_upserted` | integer | Number of vectors stored in the vector database |
| `tokens_used` | integer | Tokens consumed for processing |
| `metadata` | object | Processing metadata |
| `metadata.is_image_based` | boolean | Whether the file is an image |
| `metadata.ocr_used` | boolean | Whether OCR was applied |
| `detection_metadata` | array | Per-object detection details (object_detection mode only) |
| `phase_1_completed` | boolean | Whether phase 1 (extraction) completed |
| `phase_2_completed` | boolean | Whether phase 2 (embedding) completed |

### DetectionMetadata (per object)

| Field | Type | Description |
|-------|------|-------------|
| `detection` | string | Detection type: `single_object`, `multi_object`, or `failed` |
| `object_index` | integer | Zero-based index of this object in the detection set |
| `total_objects` | integer | Total number of objects detected in the image |
| `class` | string | COCO class name (e.g., `bottle`, `wine glass`, `cup`) |
| `confidence` | float | Detection confidence score (0.0-1.0) |
| `bbox` | array | Bounding box coordinates `[x1, y1, x2, y2]` in pixels |
| `width` | integer | Object width in pixels |
| `height` | integer | Object height in pixels |
| `area` | integer | Object area in square pixels |
| `is_crop` | boolean | Whether a cropped image was generated for this object |

---

## Processing Modes

### `ocr_text`
Extracts text from documents and images using OCR, chunks the text, and generates vector embeddings for each chunk. Best for documents, PDFs, and images with text content.

### `object_detection`
Detects individual objects in images using YOLOv8, crops each object, and generates CLIP embeddings per object. Best for multi-object images like shelf photos, group product images, or inventory photos.

**Detection pipeline:**
1. YOLOv8 runs object detection (~500ms)
2. Each detected object is cropped from the original image
3. CLIP generates an embedding for each crop (~200ms per object)
4. All embeddings are stored in the vector database

---

## Tenant Configuration

Object detection behavior is configured per-tenant via database. No API endpoint exists for configuration changes.

```json
{
  "classes": ["bottle", "wine glass"],
  "confidence_threshold": 0.5
}
```

- `classes`: Array of COCO class names to detect. Defaults to all COCO classes if not set.
- `confidence_threshold`: Minimum confidence score (0.0-1.0). Default: `0.5`.

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "title": "Validation Error",
    "detail": "Invalid processing_mode. Must be 'ocr_text' or 'object_detection'."
  }]
}
```

Common status codes: `401` Invalid API key, `404` Product/file not found, `422` Validation error, `500` Internal server error, `503` Service unavailable.
