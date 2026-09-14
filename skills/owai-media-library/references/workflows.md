# OwAI call examples

These examples use synthetic names and IDs, never a real user's library.
Replace every example ID with one returned by OwAI and every idempotency key
with one unique to the real operation, stable across its retries. The `tool` /
`arguments` wrapper is explanatory; use the client's exact discovered tool name.
Examples follow the remote schema. For local calls, omit remote-only device
parameters and polling if they are not advertised. No example authorizes a
mutation by itself.

## Find five winter photos featuring a recognized person

User request: five photos of Sam in snow during 2024, in Holiday Photos.

```json
{"tool":"search_media","arguments":{"source_name":"Holiday Photos","media_type":"image","person":"Sam","captured_from":"2024-01-01","captured_to":"2024-12-31","visual_query":"snow","limit":5}}
```

Metadata-only request: browse favorites in that folder. No visual query.

```json
{"tool":"search_media","arguments":{"source_name":"Holiday Photos","favorites_only":true,"sort":"newest","limit":10}}
```

OCR-style request: find photos containing the word STOP. Confirm any claimed
transcription from the returned preview, not from a similarity score.

```json
{"tool":"search_media","arguments":{"source_name":"Holiday Photos","media_type":"image","visual_query":"text 'STOP'","limit":5}}
```

## Inspect and show selected results

```json
{"tool":"locate_media","arguments":{"asset_ids":["example-asset-a"],"include_metadata":true}}
```

Show a preview, including from a remote caller without local-opening eligibility:

```json
{"tool":"read_media","arguments":{"asset_id":"example-asset-a"}}
```

Only if a fresh location result says `local_device`, and the user wants a window
opened, call `open_media` with the returned asset ID. Never equate a local path
string with verified availability.

## Curate a collection

After the user chooses the items, create the collection with those asset IDs.
They must belong to the chosen source. The server creates the ID and routes the
same application operation to its owner; no separate device prompt is needed.

```json
{"tool":"manage_collection","arguments":{"operation":"create","source_name":"Holiday Photos","name":"Winter favorites","asset_ids":["example-asset-a","example-asset-b"],"idempotency_key":"example-create-winter"}}
```

For a remote pending/running result, use the actual returned command ID:

```json
{"tool":"get_library_command","arguments":{"command_id":"example-command","wait_ms":15000}}
```

Read the result's collection ID, item counts and failures. Do not recreate a
collection because a search mirror has not caught up. Reuse its ID for the next
requested operation, such as export:

```json
{"tool":"export_media","arguments":{"selection_type":"collection","selection_id":"example-collection","idempotency_key":"example-export-winter"}}
```

The completed export's path is on the owner, not an assistant attachment.

## Review the least similar duplicate sets

```json
{"tool":"inspect_duplicates","arguments":{"source_name":"Holiday Photos","sort":"least_similar","limit":5,"offset":0}}
```

Return the actual set scores and member filenames. For sets larger than two,
the API's single minimum score does not identify which pair has that score.
Review before deciding whether a set is distinct or a particular original can
be removed. Never select everything except one solely from the score.

## Preview, approve, remove, recover

Preview only the original(s) proposed for removal. Keepers are not in this list.

```json
{"tool":"preview_media_deletion","arguments":{"asset_ids":["example-asset-b"],"idempotency_key":"example-preview-removal"}}
```

Wait for completion before asking for approval. Present exact items and the
preview's retention warning. Updated desktops retain deleted files on the
original drive for 30 days, then permanently remove them while the app is
running and the drive is available. Approval must cover that policy. If the
preview lacks an expected policy or is already expired, do not assume it.

Only after approval, use the real returned owner device and plan IDs:

```json
{"tool":"apply_media_deletion","arguments":{"device_id":"example-device","plan_id":"example-approved-plan","confirm_delete_originals":true,"include_favorites":false,"idempotency_key":"example-apply-removal"}}
```

For local MCP, the same operation omits `device_id` if absent from its schema.
Do not set `include_favorites:true` unless the user explicitly approved those
favorites. An invalidated plan needs a new preview and approval. Poll remote
commands and read per-item results before claiming deletion.

For requested undo, find recovery IDs on the owning device:

```json
{"tool":"inspect_activity","arguments":{"device_id":"example-device","limit":10,"offset":0}}
```

After inspecting completed activity and selecting the relevant non-purged item:

```json
{"tool":"restore_media","arguments":{"device_id":"example-device","recovery_id":"example-recovery","idempotency_key":"example-restore-removal"}}
```

Local MCP omits `device_id` on these two calls if it is not advertised. Remote
pending activity is not an empty history. Report restore conflicts or purged
files instead of attempting direct filesystem recovery.
