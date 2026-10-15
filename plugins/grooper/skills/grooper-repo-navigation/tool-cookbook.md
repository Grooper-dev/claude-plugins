# Grooper MCP tool cookbook (read-only recipes)

Tool names are shown bare; when installed via the plugin they are prefixed
`mcp__plugin_grooper_Grooper_MCP__`. Argument names below were verified against the
live server (`tools/list`) — trust the live schema over this file.

## Inventory a project

1. `Get_Node` `idOrPath="/"` → root RowId/GUID.
2. `List_Nodes` `parentId=<root id>` `name="*Project*"` or
   `typeName="Grooper.Project"`, `top=50`, `includePath=true`.
3. Children: `List_Nodes` with `parentId` per folder, or one
   `Load_Descendant_Nodes` call with `ancestorId` + `maxNodes`.

## "What is this node configured to do?"

1. `Get_Node` on the id → `properties` (fixed schema per type).
2. Properties referencing other nodes hold ids — resolve each with
   `Get_Node` (configures-by-id is everywhere).
3. Unfamiliar property names/enums: `Get_Type_Info` with the node's
   `typeNames`, `includeSchema: true`.

## Trace a content model

1. `List_Nodes` `typeName="Grooper.Core.ContentModel"` under the project.
2. Its document types / categories: `Load_Descendant_Nodes` (types
   `Grooper.Core.DocumentType`, `Grooper.Core.ContentCategory`).
3. DataModel shape: `Get_Content_Type_Schema` on the ContentType; profile
   differences: `Get_Overrides`.
4. Blast radius of a shared node (FieldClass, Lexicon):
   `Get_Node_References` `inbound: true`.

## Scripts behind behavior

1. `List_Scripts` → nodes with script projects.
2. `List_Script_Files` → source files; `Get_Node_File` to read one.
3. `Get_Implementation_Details` on a base type to understand what an
   override does; `List_Scriptable_Types` for override targets.

## Search processed documents

1. `List_Indexes` → pick an index (Lucene or vector).
2. `Execute_Lucene_Query` `indexName` + `search` (keywords, phrases,
   wildcards, Boolean), optional `filter` (OData, e.g.
   `"Status eq 'Processed'"`), `select`, `orderBy`, `top`.
3. Semantic: `Execute_Vector_Query` with the same `indexName`.
4. Known ContentType: `List_Documents` then `Get_Document_Data` /
   `Get_Document_Content` per document.

## Query connected databases and CMIS

1. Databases: `List_Database_Tables` `connectionId` →
   `Get_Database_Table_Schema` → `Run_Database_Query` (`returnType:
   "Rows"|"Scalar"|"Count"`). Reads only.
2. CMIS: `Get_Cmis_Repository_Info` / `Get_Cmis_Type_Tree` to learn the
   shape, `Run_Cmis_Query` (CMISQL) to find objects, `Get_Cmis_Object` +
   `Get_Cmis_Content_Stream` to read them.

## Runtime status

1. `List_Services` → instances + live state; `Get_Service_Usage` for AI
   Search usage.
2. `List_Jobs` (filters: `status`, `process`, `step`, `batchId`) /
   `Get_Job`; `List_Batches` + `Get_Batch_Info` / `Get_Batch_Stats`.
3. Trouble: `List_Log_Events` with filters → `Get_Log_Event`.
   Performance: `List_Stats_Sessions` → `Explore_Stats` (discover saved
   queries/filter values) → `Run_Stats_Query`.

## Grooper's own documentation

- `Search_Help`, `Get_Type_Help` — product help content, served by the
  same endpoint.
- `Create_Project_Skill`, `Create_Content_Model_Skill`,
  `Create_Data_Model_Skill`, `Create_Batch_Process_Skill`,
  `Data_Model_Styling_Skill`, `Review_Document_Skill` — despite the names,
  these are read-only: each returns a step-by-step Grooper how-to document.
