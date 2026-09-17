# To-do Inbox Synchronizer

> **VIBE-CODE DISCLAIMER: CODE WRITTEN WITH AI ASSISTANCE**
>
> This blueprint and this README were created with AI assistance. I defined the problem, the constraints, and the acceptance tests. The AI generated the YAML and helped with the documentation. I reviewed the output and tested it in Home Assistant.
>
> The automation works in the tested scenario. This is a useful fact. It is not a guarantee. Please do not confuse the two. That mistake has already caused enough software.

## Why this exists

Alexa was a useful voice inbox for shopping items. Then Amazon improved the Alexa+ experience even further.

No. Not really.

The result was more polished and less useful for the one thing I needed. An impressive achievement, if the objective was to add ceremony to groceries.

In my setup, getting a spoken item onto the shopping list I actually use became unnecessarily awkward. I did not need a new philosophy of shopping. I needed the item to arrive in the right list.

Bring! remains the household's actual shopping list. Alexa is quick to talk to. Home Assistant is available to perform the boring part. This blueprint provides the simple split:

- Alexa is the voice inbox.
- Bring! is the actual shopping list.
- Home Assistant moves the item.

That is all. No grand platform strategy. No bidirectional reconciliation ceremony. One list receives the input. One list remains authoritative. The system remains operational.

## What this does

- Watches the selected **source** to-do list for `todo.item_added`.
- Processes only the UIDs delivered by that trigger.
- Copies the item summary and description to the selected **target** list.
- Removes the exact source item by UID after the target add action succeeds.
- Avoids duplicates by comparing open target summaries case-insensitively and ignoring surrounding whitespace.
- Ignores completed history and does not migrate items that already existed when the automation was created.
- Uses `mode: queued` so quick successive additions do not cancel each other.

The source and target must be different entities. Selecting the same entity twice is not a clever loop. It is a deletion mechanism with extra steps.

## Requirements, because apparently a list needs a contract

The selected integrations must expose the Home Assistant to-do contract used here:

- `todo.item_added`
- `todo.get_items`
- `todo.add_item`
- `todo.remove_item`

The blueprint copies the summary and description only. Provider-specific metadata such as due dates or labels is not copied. The AI was not authorized to invent a data migration strategy. It would have done so.

## Import

The repository is currently private for review. This is deliberate. It is not a GitHub outage.

The import URL becomes usable by other Home Assistant users only after the repository is made public:

`https://raw.githubusercontent.com/Baumtreter/ha-todo-inbox-synchronizer/main/to-do-inbox-synchronizer.yaml`

1. In Home Assistant, open **Settings → Automations & scenes → Blueprints**.
2. Select **Import Blueprint** and paste the URL above. Home Assistant can import a blueprint from GitHub or a Gist.
3. Create an automation from the imported blueprint and choose the source and target entities.
4. Reload automations after changing the blueprint file.

The blueprint includes the matching GitHub page URL as `blueprint.source_url` so future updates can be tracked. It is a small courtesy to the next person who has to determine what changed.

## Example: Alexa to Bring!

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

The source removal is placed after `todo.add_item`. With Home Assistant's normal script error handling, a failed target add stops the sequence before the source removal. The source item remains available for inspection or retry.

This is called failure safety. It is not glamorous. It is preferable.

### Duplicate handling

If an open target item has the same normalized summary, the blueprint does not add another copy and removes the source inbox item. This is intentional for shopping-list inboxes. Shopping lists contain enough duplication without assistance.

The blueprint does not merge quantities or provider-specific metadata. It moves items. It does not conduct negotiations.

### One-way only

This blueprint is not bidirectional synchronization. Changes made in the target list do not affect the source list, and completed target items are not copied back.

Two lists arguing over who owns "milk" is not a synchronization strategy. It is an administrative failure with dairy.

## Before trusting it with groceries

- Parse the YAML with a loader that understands Home Assistant's `!input` tag.
- Verify the trigger is `todo.item_added` and targets the source input.
- Verify `todo.add_item` targets the target input.
- Verify `todo.remove_item` targets the source input and uses the source UID.
- Verify the same-entity guard and the new-items-only behavior.
- Test one temporary source item and remove the temporary target item after the transfer is confirmed.
- The repository is private during review; make it public before sharing the import link.
- Verify the raw import URL after changing repository visibility.
