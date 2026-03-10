# Excalidraw Architecture Diagrams Guide

How to create effective architecture diagrams using the Excalidraw MCP server tools.

## Available MCP Tools

Depending on which Excalidraw MCP server is installed, you may have access to these tools:

### `@cmd8/excalidraw-mcp` tools:
- `createNode` — Create a shape with a label (returns node ID)
- `createEdge` — Create an arrow between two nodes (by ID or label)
- `deleteElement` — Remove a node or edge
- `getFullDiagramState` — Get markdown representation of the diagram

### `mcp-excalidraw-server` tools (richer):
- `create_element` — Create any element (rectangle, ellipse, diamond, text, arrow, line)
- `create_from_mermaid` — Convert a Mermaid diagram directly to Excalidraw
- `batch_create_elements` — Create multiple elements at once
- `create_edge` — Connect two elements with an arrow
- `group_elements` — Group related elements visually
- `align_elements` / `distribute_elements` — Organize layout
- `export_scene` — Export as `.excalidraw` file
- `export_to_image` — Export as PNG/SVG
- `export_to_excalidraw_url` — Get a shareable Excalidraw URL

## Diagram Types for Architecture Docs

### 1. System Overview (C4 Context Level)

Show the system boundary, users, and external systems. Use rectangles for services, rounded rectangles for users, and cylinders for databases.

**Layout strategy:**
- Users/clients at the top
- Core system in the middle (grouped in a frame)
- External services and databases at the bottom
- Arrows showing data flow direction
- Color code: blue for your services, gray for external, green for databases

**Example element plan:**
```
[User] (top center)
  → [API Gateway] (center)
    → [Auth Service] (left)
    → [Core Service] (center)
      → [Database] (bottom center)
      → [Cache] (bottom left)
    → [Worker] (right)
      → [Queue] (bottom right)
```

### 2. Data Flow Diagram

Show how data moves through the system end-to-end. Use horizontal or left-to-right layout.

**Layout strategy:**
- Source on the left, sink on the right
- Processing steps in between
- Use different arrow styles for sync vs async
- Label arrows with data types/formats

### 3. Deployment Topology

Show infrastructure: servers, containers, cloud services, networks.

**Layout strategy:**
- Group by environment (VPC, subnet, availability zone)
- Use frames/groups for logical boundaries
- Show network connections between components
- Include ports and protocols on arrows

### 4. ER / Data Model

Show entities and their relationships.

**Layout strategy:**
- Main entities in the center
- Supporting entities around the edges
- Use diamond shapes for many-to-many relationships
- Label arrows with relationship type (1:N, N:M)

## Design Tips

### Spacing and Layout
- Leave 150-200px between elements for readability
- Align elements on a grid (multiples of 50px)
- Use `align_elements` and `distribute_elements` when available
- Group related components using `group_elements` or frames

### Colors (consistent scheme)
- **Blue (#4A90D9)** — Your application services
- **Green (#7BC96F)** — Databases and storage
- **Orange (#F5A623)** — Queues and messaging
- **Purple (#9B59B6)** — External/third-party services
- **Gray (#95A5A6)** — Users and clients
- **Red (#E74C3C)** — Security boundaries, firewalls

### Text
- Use short labels (2-3 words max per node)
- Add detail in the ARCHITECTURE.md prose, not the diagram
- Use consistent naming between diagram labels and document sections

## Workflow: Mermaid First, Then Excalidraw

The most efficient approach:

1. **Write the Mermaid diagram** in the ARCHITECTURE.md first (this is always included)
2. **If `create_from_mermaid` is available**, convert the Mermaid to Excalidraw automatically — then refine positioning and styling
3. **If only basic tools are available**, use the Mermaid as a blueprint and recreate with `createNode` / `createEdge` calls
4. **Export** the final diagram as `.excalidraw` (editable) and optionally `.png` (for embedding)

## Setup Instructions for Users

If the user doesn't have Excalidraw MCP:

### Quick setup (recommended):
```bash
claude mcp add excalidraw -- npx -y @cmd8/excalidraw-mcp --diagram ./architecture.excalidraw
```

### Rich setup (more tools):
```bash
git clone https://github.com/user/mcp-excalidraw-server.git ~/mcp_excalidraw
cd ~/mcp_excalidraw && npm install && npm run build
claude mcp add excalidraw -- node ~/mcp_excalidraw/dist/index.js
```

After adding, restart Claude Code for the MCP server to connect.
