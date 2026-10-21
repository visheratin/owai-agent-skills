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

Read an image, including from a remote caller without local-opening eligibility:

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

The completed local-folder/Photos export's path is on the owner, not an assistant
attachment. OneDrive exports return tracked provider results under OneDrive/OwAI.

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
preview's retention warning. Filesystem sources retain deleted files on the
original drive for 30 days, then permanently remove them while the app is
running and the drive is available. OneDrive uses Microsoft's recycle bin;
Apple Photos uses Recently Deleted. Approval must cover the actual policy. If the
preview lacks an expected policy or is already expired, do not assume it.

Only after approval, use the real returned plan ID. OwAI derives its owner from
the preview receipt; do not guess or pass a device ID:

```json
{"tool":"apply_media_deletion","arguments":{"plan_id":"example-approved-plan","confirm_delete_originals":true,"include_favorites":false,"idempotency_key":"example-apply-removal"}}
```

Do not set `include_favorites:true` unless the user explicitly approved those
favorites. An invalidated plan needs a new preview and approval. Poll remote
commands and read per-item results before claiming deletion.

For requested undo, find recovery IDs for the relevant source. A device-bound
connection can omit the source; an unbound remote connection with multiple
owners should pass the real source ID returned by library inspection:

```json
{"tool":"inspect_activity","arguments":{"source_id":"example-source","limit":10,"offset":0}}
```

After inspecting completed activity and selecting the relevant non-purged item:

```json
{"tool":"restore_media","arguments":{"recovery_id":"example-recovery","idempotency_key":"example-restore-removal"}}
```

OwAI derives restore ownership from the recovery/activity receipt. Remote
pending activity is not an empty history. Report restore conflicts or purged
files instead of attempting direct filesystem recovery.

## Connected Apple Photos and OneDrive

Use `inspect_library` to find the actual source ID and kind, then use the same
search and organization calls above. Do not infer source ownership from the
computer running the chat or pass a `cloud:` owner as a local device binding.

| Source | Search/inspection | Reads and edits | Export and original recovery |
| --- | --- | --- | --- |
| Apple Photos (`photo_library`) | Last synced catalog is available remotely while the Mac is offline. | Reads and queued changes require the owning Mac's OwAI app. Media reads additionally need Photos permission and enabled relay. Uncached iCloud items need network access. | Current rendered JPEG/MP4 copies are exported on the Mac. Approved deletion goes through Photos; restore in Photos → Recently Deleted, then sync. |
| OneDrive (`onedrive`) | Server-owned catalog is available without a desktop. | Provider content and cloud commands use the existing OneDrive authorization. Pending file jobs must finish before reporting success. | Cloud exports go to OneDrive/OwAI. Restore original files in OneDrive's recycle bin under Microsoft's policy. |

For example, after discovering a Photos album source named “Summer album”:

```json
{"tool":"search_media","arguments":{"source_name":"Summer album","media_type":"image","visual_query":"beach","limit":5}}
```

Call `read_media` on a returned asset ID to inspect its current rendering. Never
send the `apple-photos://` locator to shell or filesystem tools. Editable Photos
user albums link to OwAI collections. Album renames in Photos are imported on
sync; use only the collection operations actually advertised by the tool, not
an invented rename operation. Collection creation, membership changes and
deletion update the native album. Favorites
synchronize too. OwAI groups publish their selected photos to native albums;
group rules and face labels remain OwAI metadata. Deleting a mapped collection
or group removes its native album but not the originals; removing the OwAI
source does not remove native albums. Poll pending commands and check
`native_sync.operations` before reporting a native edit as applied.
After resolving a native failure and syncing the source, retry the same request
with its original idempotency key; do not create a new command to bypass the error.
The connector indexes the still part of
Live Photos, not a separately searchable motion clip. It does not provide direct
iCloud login, arbitrary `.photoslibrary` access, or in-place optimization of
Photos originals. If an operation is unsupported, report that boundary instead
of changing the Photos package or an exported cache file.
