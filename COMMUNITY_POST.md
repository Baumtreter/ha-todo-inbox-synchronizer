# [Blueprint] To-do Inbox Synchronizer: move new items between to-do lists

> **AI disclosure:** The blueprint and this post were created with AI assistance. I defined the use case and the desired behavior, then reviewed and tested the automation in Home Assistant. No personal entity IDs or secrets are included here.

Alexa is great at taking quick voice notes. The problem starts when the thing you said ends up on a list that is technically correct, but practically useless for the rest of the household.

This blueprint turns one Home Assistant to-do list into an inbox for another list. New items are copied from the source list to the target list, and the source item is removed only after the target add succeeds. The target list can therefore remain the household's source of truth, while the source list handles the quick input.

The blueprint is provider-agnostic. Choose any two different `todo.*` entities that support the Home Assistant to-do trigger and actions used below.

## What it does

- Watches the selected source list for `todo.item_added`.
- Processes only the items delivered by that trigger.
- Copies the summary and description to the selected target list.
- Removes the exact source item by UID after the target add succeeds.
- Avoids duplicates by comparing open target summaries case-insensitively and ignoring surrounding whitespace.
- Leaves existing source items and completed history alone.
- Uses queued mode so several quick additions do not trip over each other.

This is intentionally one-way. Two lists both trying to be the boss is how you end up with duplicate milk and a small domestic incident.

## Import

The repository is currently private while the blueprint is being reviewed. Once it is public, import it in Home Assistant with:

`https://raw.githubusercontent.com/Baumtreter/ha-todo-inbox-synchronizer/main/to-do-inbox-synchronizer.yaml`

In Home Assistant, open **Settings → Automations & scenes → Blueprints**, choose **Import Blueprint**, paste the URL, and create an automation from it.

## Example

```yaml
source_list: todo.source_list
target_list: todo.target_list
```

The example uses generic entity IDs on purpose. Replace them by selecting your own source and target entities in the blueprint UI. The two entities must be different.

## Important behavior

Existing open items in the source list are not migrated when the automation is created. This avoids an unexpected bulk transfer. Completed history is not read or modified.

If adding the item to the target fails, the source item stays where it is. That gives you something to inspect or retry instead of quietly eating the shopping list. Nobody needs another automation with a mysterious appetite.

If an equivalent open item already exists in the target, no duplicate is created and the source item is removed. Quantities and provider-specific metadata are not merged.

For troubleshooting, please include the Home Assistant version, the source and target integrations, the automation trace, and the relevant log message. Do not post access tokens or complete private configuration files.
