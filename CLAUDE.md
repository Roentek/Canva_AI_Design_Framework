# Canva AI Design Framework

This project is an AI-powered design automation framework that uses the **Canva Connect API** and **Canva MCP Server** to create, edit, export, and manage designs through natural language. The goal is to let users describe what they want in plain English and have AI translate that into Canva API calls that produce real, editable designs.

## Purpose

Turn natural language requests into professional Canva designs. Examples:

- "Create a 1080x1080 Instagram post for a summer sale with our brand colors"
- "Export my latest presentation as a PDF"
- "Search my designs for anything related to Q3 marketing"
- "Upload this image and add it to the Brand Assets folder"
- "Create 50 personalized flyers from our brand template with this data"

## Architecture

```txt
User (natural language) --> Claude + CLAUDE.md context --> Canva MCP / Connect API --> Canva designs
```

### MCP Servers Available

| Server | Purpose |
| -------- | --------- |
| `canva-mcp` | Design creation, editing, search, export, asset management via Canva |
| `tavily-mcp` | Web research for design inspiration, content, and references |
| `openrouter-mcp` | Multi-model AI for content generation (copy, descriptions) |

---

## Canva MCP Server

**Endpoint:** `https://mcp.canva.com/mcp`
**Auth:** Browser-based OAuth per user (Dynamic Client Registration). Each user authenticates with their own Canva account when first using Canva tools.

### MCP Capabilities

| Capability | Description |
| ------------ | ------------- |
| **Design Generation** | Create designs from text descriptions and specifications |
| **Design Editing** | Modify existing designs through natural language commands |
| **Design Discovery** | Search and retrieve designs, pages, and folders |
| **Asset Management** | Upload assets, organize folders, access brand kits and templates |
| **Design Export** | Export to PDF, PNG, JPG, GIF, PPTX, MP4, HTML |
| **Collaboration** | Add comments and feedback to designs |

### Plan-Based Feature Access

| Feature | Free/Pro | Pro+ | Enterprise |
| --------- | ---------- | ------ | ------------ |
| Create/Edit/Search designs | Yes | Yes | Yes |
| Export designs | Yes | Yes | Yes |
| Upload assets | Yes | Yes | Yes |
| Comments | Yes | Yes | Yes |
| Design resizing | No | Yes | Yes |
| Brand template autofill | No | No | Yes |
| Brand kits | No | No | Yes |

---

## Canva Connect API Reference

**Base URL:** `https://api.canva.com/rest`
**Auth:** OAuth 2.0 with PKCE (Authorization Code flow)
**OpenAPI Spec:** `https://www.canva.dev/sources/connect/api/latest/api.yml`

### OAuth Scopes

| Scope | Access |
| ------- | -------- |
| `asset:read` | View asset metadata (uploaded images, etc.) |
| `asset:write` | Upload, update, or delete assets |
| `brandtemplate:content:read` | Read brand template content |
| `brandtemplate:meta:read` | Read brand template metadata |
| `collaboration:event` | Receive webhook notifications |
| `comment:read` | View design comments |
| `comment:write` | Create comments and replies |
| `design:content:read` | View design contents |
| `design:content:write` | Create designs |
| `design:meta:read` | View design metadata |
| `folder:read` | View folder metadata and contents |
| `folder:write` | Create, move, delete folders |
| `folder:permission:write` | Manage folder permissions |
| `profile:read` | Access user profile info |

### API Endpoints

#### Designs

| Method | Endpoint | Scope | Rate Limit | Description |
| -------- | ---------- | ------- | ------------ | ------------- |
| `POST` | `/v1/designs` | `design:content:write` | 20/min | Create a new design |
| `GET` | `/v1/designs` | `design:meta:read` | - | List/search user designs |
| `GET` | `/v1/designs/{designId}` | `design:meta:read` | - | Get design metadata |
| `GET` | `/v1/designs/{designId}/pages` | `design:content:read` | - | List design pages |
| `GET` | `/v1/designs/{designId}/export-formats` | `design:content:read` | - | Available export formats |

##### Create Design Request

```json
{
  "design_type": {
    "type": "preset",
    "name": "doc | whiteboard | presentation"
  },
  "title": "My Design"
}
```

Or with custom dimensions (40-8000px):

```json
{
  "design_type": {
    "type": "custom",
    "width": 1080,
    "height": 1080
  },
  "asset_id": "optional-image-asset-id",
  "title": "Instagram Post"
}
```

##### Design Response

```json
{
  "design": {
    "id": "DAGabc123",
    "title": "My Design",
    "owner": { "user_id": "...", "team_id": "..." },
    "urls": {
      "edit_url": "https://www.canva.com/design/...",
      "view_url": "https://www.canva.com/design/..."
    },
    "created_at": 1234567890,
    "updated_at": 1234567890,
    "thumbnail": { "width": 595, "height": 335, "url": "..." },
    "page_count": 1
  }
}
```

#### Exports

| Method | Endpoint | Scope | Rate Limit | Description |
| -------- | ---------- | ------- | ------------ | ------------- |
| `POST` | `/v1/exports` | `design:content:read` | 20/min | Start export job |
| `GET` | `/v1/exports/{exportId}` | `design:content:read` | - | Check export status |

##### Export Formats and Options

**PDF:**

```json
{
  "design_id": "DAGabc123",
  "format": {
    "type": "pdf",
    "quality": "regular|pro",
    "size": "a4|a3|letter|legal",
    "pages": [1, 2, 3]
  }
}
```

**PNG:**

```json
{
  "format": {
    "type": "png",
    "quality": "regular|pro",
    "height": 1080,
    "width": 1080,
    "lossless": true,
    "as_single_image": false
  }
}
```

**JPG:**

```json
{
  "format": {
    "type": "jpg",
    "quality": "regular|pro",
    "height": 1080,
    "width": 1080,
    "export_quality": 80
  }
}
```

**MP4:**

```json
{
  "format": {
    "type": "mp4",
    "quality": "regular|pro",
    "orientation": "horizontal|vertical",
    "resolution": "480p|720p|1080p|4k"
  }
}
```

**PPTX / GIF / HTML:** Minimal config, specify `type` and optional `pages`.

##### Export Rate Limits

- Per user: 75 exports / 5 min, 500 / 24 hr
- Per integration: 750 / 5 min, 5000 / 24 hr
- Per document: 75 / 5 min
- Download URLs valid for 24 hours

#### Assets

| Method | Endpoint | Scope | Rate Limit | Description |
| -------- | ---------- | ------- | ------------ | ------------- |
| `POST` | `/v1/asset-uploads` | `asset:write` | - | Upload asset (async) |
| `GET` | `/v1/asset-uploads/{jobId}` | `asset:write` | - | Check upload status |
| `GET` | `/v1/assets/{assetId}` | `asset:read` | - | Get asset metadata |
| `PATCH` | `/v1/assets/{assetId}` | `asset:write` | - | Update asset name/tags |
| `DELETE` | `/v1/assets/{assetId}` | `asset:write` | - | Delete asset |
| `POST` | `/v1/url-asset-uploads` | `asset:write` | - | Upload from URL (preview) |

#### Folders

| Method | Endpoint | Scope | Description |
| -------- | ---------- | ------- | ------------- |
| `POST` | `/v1/folders` | `folder:write` | Create folder |
| `GET` | `/v1/folders/{folderId}` | `folder:read` | Get folder details |
| `PATCH` | `/v1/folders/{folderId}` | `folder:write` | Rename folder |
| `DELETE` | `/v1/folders/{folderId}` | `folder:write` | Delete folder |
| `GET` | `/v1/folders/{folderId}/items` | `folder:read` | List folder contents |
| `POST` | `/v1/folders/move` | `folder:write` | Move item between folders |

#### Brand Templates (Enterprise)

| Method | Endpoint | Scope | Description |
| -------- | ---------- | ------- | ------------- |
| `GET` | `/v1/brand-templates` | `brandtemplate:meta:read` | List brand templates |
| `GET` | `/v1/brand-templates/{id}` | `brandtemplate:meta:read` | Get template metadata |
| `GET` | `/v1/brand-templates/{id}/dataset` | `brandtemplate:content:read` | Get template fields |

#### Autofill (Enterprise)

| Method | Endpoint | Scope | Description |
| -------- | ---------- | ------- | ------------- |
| `POST` | `/v1/autofills` | `design:content:write` | Create autofill job |
| `GET` | `/v1/autofills/{jobId}` | `design:content:write` | Check autofill status |

##### Autofill Workflow

1. Get template dataset: `GET /v1/brand-templates/{id}/dataset`
2. Submit autofill: `POST /v1/autofills` with template ID + field values
3. Poll status: `GET /v1/autofills/{jobId}` until `success`
4. Result includes new design URL

```json
{
  "brand_template_id": "DAGabc123",
  "data": {
    "headline": { "type": "text", "text": "Summer Sale 2026" },
    "hero_image": { "type": "image", "asset_id": "abc123" }
  }
}
```

#### Design Import

| Method | Endpoint | Description |
| -------- | ---------- | ------------- |
| `POST` | `/v1/imports` | Import file as design (async) |
| `GET` | `/v1/imports/{jobId}` | Check import status |
| `POST` | `/v1/url-imports` | Import from URL (async) |

#### Design Resize (Pro+)

| Method | Endpoint | Description |
| -------- | ---------- | ------------- |
| `POST` | `/v1/resizes` | Resize design to new dimensions (async) |
| `GET` | `/v1/resizes/{jobId}` | Check resize status |

#### Comments (Preview)

| Method | Endpoint | Scope | Description |
| -------- | ---------- | ------- | ------------- |
| `POST` | `/v1/designs/{id}/comments` | `comment:write` | Create comment thread |
| `GET` | `/v1/designs/{id}/comments/{threadId}` | `comment:read` | Get thread |
| `POST` | `/v1/designs/{id}/comments/{threadId}/replies` | `comment:write` | Reply to thread |
| `GET` | `/v1/designs/{id}/comments/{threadId}/replies` | `comment:read` | List replies |

#### User

| Method | Endpoint | Description |
| -------- | ---------- | ------------- |
| `GET` | `/v1/users/me` | Current user + team IDs |
| `GET` | `/v1/users/me/capabilities` | User's API capabilities |
| `GET` | `/v1/users/me/profile` | User display name |

---

## Natural Language Design Patterns

When a user makes a design request, follow this decision tree:

### 1. Understand the Request

Parse the user's intent into:

- **What** to create (type, dimensions, content)
- **Style** (brand, colors, mood, template)
- **Output** (format, quality, destination)

### 2. Map to Canva Operations

| User Intent | Canva Action |
| ------------- | -------------- |
| "Create a poster/flyer/post" | `POST /v1/designs` with custom dimensions |
| "Make a presentation" | `POST /v1/designs` with preset `presentation` |
| "Create a document" | `POST /v1/designs` with preset `doc` |
| "Export as PDF/PNG/..." | `POST /v1/exports` with format config |
| "Find my designs about X" | `GET /v1/designs` with search params |
| "Upload this image" | `POST /v1/asset-uploads` |
| "Organize into folder" | `POST /v1/folders` + `POST /v1/folders/move` |
| "Generate 50 flyers from template" | Autofill workflow (Enterprise) |
| "Resize for Twitter" | `POST /v1/resizes` (Pro+) |
| "Add a comment" | `POST /v1/designs/{id}/comments` |
| "What can I do?" | `GET /v1/users/me/capabilities` |

### 3. Common Design Dimensions

| Platform | Format | Dimensions |
| ---------- | -------- | ------------ |
| Instagram | Post | 1080 x 1080 |
| Instagram | Story/Reel | 1080 x 1920 |
| Facebook | Post | 1200 x 630 |
| Facebook | Cover | 820 x 312 |
| Twitter/X | Post | 1600 x 900 |
| LinkedIn | Post | 1200 x 627 |
| LinkedIn | Banner | 1584 x 396 |
| YouTube | Thumbnail | 1280 x 720 |
| Pinterest | Pin | 1000 x 1500 |
| TikTok | Video | 1080 x 1920 |
| A4 Print | Document | 2480 x 3508 |
| US Letter | Document | 2550 x 3300 |
| Poster | 18x24" | 5400 x 7200 |
| Business Card | Standard | 1050 x 600 |

### 4. Workflow Patterns

**Quick Design:** User says "make me an X" -> Create design with appropriate dimensions -> Return edit URL

**Design + Export:** User wants a deliverable -> Create design -> Export to requested format -> Return download URL

**Bulk Creation (Enterprise):** User has data + template -> Get template dataset -> Autofill for each data row -> Collect all design URLs

**Brand Asset Management:** User wants to organize -> Upload assets -> Create folders -> Move items -> Tag with metadata

**Research + Design:** User needs inspiration -> Use Tavily to research trends/competitors -> Use findings to inform design creation

---

## AI Workflow Integration

### Using Multiple MCP Servers Together

**Content + Design Pipeline:**

1. Use `openrouter-mcp` to generate marketing copy, headlines, descriptions
2. Use `tavily-mcp` to research trends, competitors, or reference designs
3. Use `canva-mcp` to create the design with generated content
4. Use `supabase-mcp` to log the design metadata, track versions

**Template-Based Campaigns (Enterprise):**

1. Query brand templates from Canva
2. Use `openrouter-mcp` to generate personalized content per recipient
3. Use `canva-mcp` autofill to create personalized designs at scale
4. Export all designs and store URLs in `supabase-mcp`

**Design Review Workflow:**

1. Search existing designs via `canva-mcp`
2. Add review comments via Comments API
3. Export approved designs
4. Track approval status in `supabase-mcp`

---

## Important Constraints

- **Blank designs auto-delete** after 7 days if not edited
- **Design URLs expire** after 30 days
- **Thumbnail URLs expire** after 15 minutes
- **Export download URLs expire** after 24 hours
- **Dimensions:** 40-8000px for design creation, 40-25000px for exports
- **Title length:** 1-255 characters
- **Preview APIs** may have breaking changes without notice
- **Rate limits** vary per endpoint (typically 20/min per user)
- **Async operations** (export, upload, import, autofill, resize) require polling for completion

## Documentation Links

- Canva Developer Docs: <https://www.canva.dev/docs/>
- Connect API: <https://www.canva.dev/docs/connect/>
- MCP Server: <https://www.canva.dev/docs/mcp/>
- OpenAPI Spec: <https://www.canva.dev/sources/connect/api/latest/api.yml>
- Apps SDK: <https://www.canva.dev/docs/apps/>
