# To-do Inbox Synchronizer

A reusable Home Assistant automation blueprint for moving newly added items from one `todo.*` list to another.

Typical use case: use an Alexa shopping list as a voice inbox and move new items into a Bring! list, where the target list remains authoritative.

## What it does

- Watches the selected **source** to-do list for `todo.item_added`.
- Processes only the UIDs delivered by that trigger.
- Copies the item summary and description to the selected **target** list.
- Removes the exact source item by UID after the target add action succeeds.
- Avoids duplicates by comparing open target summaries case-insensitively and ignoring surrounding whitespace.
- Ignores completed history and does not migrate items that already existed when the automation was created.
- Uses `mode: queued` so quick successive additions are not cancelled by a restart.

The source and target must be different entities. A guard in the blueprint stops the automation if the same entity is selected twice.

## Requirements

The selected integrations must expose the Home Assistant to-do contract used here:

- `todo.item_added`
- `todo.get_items`
- `todo.add_item`
- `todo.remove_item`

The blueprint copies the summary and description only. Other provider-specific metadata, such as due dates or labels, is not copied.

## Import

The repository currently is private for review. The import URL is ready, but it becomes usable by other Home Assistant users only after the repository is made public:

`https://raw.githubusercontent.com/Baumtreter/ha-todo-inbox-synchronizer/main/to-do-inbox-synchronizer.yaml`

1. In Home Assistant, open **Settings → Automations & scenes → Blueprints**.
2. Select **Import Blueprint** and paste the URL above. Home Assistant can import a blueprint from GitHub or a Gist.
3. Create an automation from the imported blueprint and choose the source and target entities.
4. Reload automations after changing the blueprint file.

The blueprint includes the matching GitHub page URL as `blueprint.source_url` so future updates can be tracked.

## Example: Alexa → Bring!

For the original use case:

```yaml
source_list: todo.source_list
target_list: todo.target_list
```

This mapping is an example only. The shared blueprint itself contains no Alexa- or Bring!-specific entity IDs.

## Important behavior

### New items only

Existing open items in the source list are intentionally left alone. This avoids an unexpected bulk migration when an automation is first created. Add or move those items manually if an initial migration is wanted.

### Failure safety

The source removal is placed after `todo.add_item`. With Home Assistant's normal script error handling, a failed target add stops the sequence before the source removal, so the source item remains available for inspection or retry.

### Duplicate handling

If an open target item has the same normalized summary, the blueprint does not add another copy and removes the source inbox item. This is intentional for shopping-list inboxes. It does not merge quantities or provider-specific metadata.

### One-way only

This blueprint is not bidirectional synchronization. Changes made in the target list do not affect the source list, and completed target items are not copied back.

## Community post

The copy-ready community post lives in [`COMMUNITY_POST.md`](COMMUNITY_POST.md), intentionally outside this technical README.

## Local validation checklist

Before publishing:

- Parse the YAML with a loader that understands Home Assistant's `!input` tag.
- Verify the trigger is `todo.item_added` and targets the source input.
- Verify `todo.add_item` targets the target input.
- Verify `todo.remove_item` targets the source input and uses the source UID.
- Verify the same-entity guard and the new-items-only behavior are documented.
- Test one temporary source item and remove the temporary target item after the transfer is confirmed.
- The repository is private during review; make it public before posting the import link to the community.
- Verify the raw import URL after changing repository visibility.
