---
name: grooper-repo-navigation
description: >-
  How to explore a Grooper repository through its MCP tools — node tree,
  content models, document types, DataModels, scripts, services, jobs.
  Use when asked anything about the contents or configuration of a
  Grooper project.
when_to_use: >-
  Questions like "what document types can this repo classify", "how is
  this content model extracting", "which scripts drive this batch
  process", "inventory the projects", or any read-only investigation of a
  Grooper repository via its MCP endpoint.
allowed-tools: mcp__plugin_grooper_Grooper_MCP__*
---

# Navigating a Grooper repository over MCP

A Grooper repository is a tree of **nodes**. Every node has a GUID `id`, an
integer `rowId`, a parent (`parentId`/`parentRowId`), and a .NET **typeName**
like `Grooper.Project`, `Grooper.Core.ContentModel`,
`Grooper.Core.DocumentType`, `Grooper.Core.DataModel`,
`Grooper.Core.BatchProcess`, or `Grooper.GPT.MCP.MCPServer`. Nodes are
addressed by RowId, GUID, or "/"-delimited path. All arguments below were
verified against the live server's schemas — the live `tools/list` output
always wins over this file.

## Core loop

1. **Orient** — `Get_Node` with `idOrPath: "/"` returns the repository
   root. `List_Nodes` lists children with filters: `parentId`, `name`
   (exact, or `*` wildcard for partial), and `typeName` — a FULL .NET type
   name (`Grooper.Core.ContentModel`, never the display name
   "Content Model"). It returns only **10 rows by default**: raise `top`,
   page with `skip`, and set `includePath: true` so findings are
   reportable. Unsure of a type name? `Search_Help` or
   `List_Type_Derivatives`.
2. **Whole subtrees in one call** — `Load_Descendant_Nodes` with
   `ancestorId` and required `maxNodes`. Keep `includeDetails` off unless
   you truly need properties; it fattens every node.
3. **Understand the type** — `Get_Type_Info` takes `typeNames` (an
   **array**) and options: `includeChildTypes`, `includeCommands`,
   `includeSchema`, `includeRemarks` (heavy — narrow scope). This is the
   only trustworthy source for a type's property shape.
4. **Read configuration** — `Get_Node` (`includeFiles`,
   `includeValidationMessages` optional), `Get_Node_File` for stored
   files, `Get_Overrides` for profile overrides, `Get_Node_References`
   (`inbound: true` = nodes that reference this one; false = what it
   references).
5. **Behavior questions** — `List_Scripts` → `List_Script_Files` →
   `Get_Implementation_Details`.
6. **Extracted-content questions** — `List_Indexes` first, then
   `Execute_Lucene_Query` (`indexName`, `search`, optional
   `filter`/`select`/`orderBy`/`top`) or `Execute_Vector_Query`.
   DataModel shape: `Get_Content_Type_Schema` on the ContentType.
   CMIS sources: `Run_Cmis_Query`, `Get_Cmis_Object`. Database
   connections: `List_Database_Tables`, `Get_Database_Table_Schema`,
   then `Run_Database_Query` (SELECTs only).
7. **Runtime state** — `List_Services`, `Get_Service_Usage`, `List_Jobs`,
   `Get_Job`, `List_Batches`, `Get_Batch_Stats`, `List_Log_Events`.

Full per-tool recipes: see [tool-cookbook.md](tool-cookbook.md).

## Ground rules

- Discover ids, never guess them. Paths are for display; RowIds/GUIDs are
  for addressing.
- Prefer two cheap reads (`List_Nodes` → `Get_Node`) over one lucky guess.
- Report findings as **node path + type + RowId** so a human can verify.
- Stay read-only. If the answer requires a mutation, say what change would
  be needed and stop.
- Say "I could not determine X" rather than inventing structure.
