# Compendium — Indigo Plugin

An Indigo plugin that surfaces jokes, facts, riddles, quotes, trivia, "on this day" historical events, and cocktail recipes from the [API Ninjas](https://api-ninjas.com/) API as device states. Useful for Control Page tiles, speech actions, daily triggers, or just adding a bit of personality to your home automation.

## Features

- **Seven device types** — Joke, Fact, Riddle, Quote, Trivia, On This Day, Cocktail of the Day
- **Monthly call-budget management** — set your monthly API cap, click one button, and the budget spreads evenly across all your devices
- **Local usage counter** — tracks calls made by the plugin against your monthly budget, with automatic calendar-month rollover
- **History tracking** of the last three items per device, with paired answer history for riddles, trivia, and cocktails
- **Cycle Back action** to surface previous items
- **Per-content-type deduplication** so the same item won't repeat within a session
- **Auto-refresh** at a configurable per-device interval (minimum 5 minutes)
- **Send Status Request** triggers a fresh fetch — works from right-click menu, iOS app, or any Indigo automation
- **Diagnostic logging** with debug toggle — failed calls produce specific error messages rather than silent failures
- **No bundled vendor libraries** — dependencies declared in `requirements.txt` and resolved by Indigo at install

## Requirements

- Indigo 2024.1+ (Server API 3.0)
- A free API key from [API Ninjas](https://api-ninjas.com/)
- Python `requests` (declared in `requirements.txt`, installed automatically)

## Installation

1. Download the latest release `.zip` from the [Releases](../../releases) page
2. Unzip and double-click `Compendium.indigoplugin` to install
3. Enable the plugin in Indigo
4. Open the plugin configuration:
   - Paste in your API Ninjas key
   - Set your monthly call cap (default 3000) and safety buffer (default 10%)
   - Create your devices first, then click **Distribute Calls** to spread the budget across them
5. Add devices from the device library — pick from Joke, Fact, Riddle, Quote, Trivia, On This Day, or Cocktail of the Day

## Configuration

### Plugin-level

| Setting | Purpose |
| --- | --- |
| API Ninjas API Key | Required. Get one free at [api-ninjas.com](https://api-ninjas.com/). |
| Max API calls per month | Your monthly cap (default 3000). Used by Distribute Calls to size per-device intervals. |
| Safety buffer (%) | Headroom held back from the cap for manual refreshes (default 10). |
| Distribute Calls | Reads the above and applies a safe auto-refresh interval to every configured device. |
| Status | Read-only display of current usage. Auto-populates when the config dialog opens. |
| Refresh / Reset Counter | Recompute the status, or manually reset the local counter to zero. |
| Enable Debug Logging | Verbose log output for troubleshooting. |

### Device-level

Each device exposes a single configuration field:

| Setting | Purpose |
| --- | --- |
| Auto-refresh Interval (minutes) | How often the device fetches fresh content. `0` disables; minimum honoured value is `5`. |

## Budget management

The plugin keeps a local counter of API calls made during each calendar month. Counter and current period are shown in the plugin configuration dialog (auto-populated when the dialog opens, refreshable with the **Refresh** button). When the calendar month rolls over, the counter resets automatically.

**Distribute Calls** computes:

```
usable_calls = max_calls × (1 − buffer% ÷ 100)
interval     = ceil(43,830 × device_count ÷ usable_calls)    minutes per device
```

The 5-minute minimum is enforced; if your budget can't sustain that across all your devices, the status field warns you that projected usage will exceed the cap.

The **Reset Counter** button is useful after rotating an API key, after a billing-cycle change, or when calls have been made against the same key outside of this plugin.

## Device states

| Device | States |
| --- | --- |
| Joke | `current_joke`, `previous_1..3`, `last_updated` |
| Fact | `current_fact`, `previous_1..3`, `last_updated` |
| Quote | `current_content`, `previous_1..3`, `last_updated` |
| On This Day | `current_content`, `previous_1..3`, `last_updated` |
| Riddle | `current_content`, `current_answer`, `previous_1..3`, `previous_answer_1..3`, `last_updated` |
| Trivia | `current_content`, `current_answer`, `previous_1..3`, `previous_answer_1..3`, `last_updated` |
| Cocktail | `current_content` (name), `current_answer` (recipe), `previous_1..3`, `previous_answer_1..3`, `last_updated` |

`last_updated` is an ISO-style timestamp (`YYYY-MM-DD HH:MM:SS`). Its UI display value is formatted as `"Updated at HH:MM on DD Mon YYYY"` for use in Control Pages and tooltips.

For devices with paired answers (Riddle, Trivia, Cocktail) the answer history rotates in lockstep with the question/name history, so cycling back always brings the matching answer.

## Actions

| Action | What it does |
| --- | --- |
| Refresh Content | Fetches a fresh item immediately and shifts the previous current into history. |
| Cycle Back Through History | Brings `previous_1` (and its paired answer where relevant) into the current state and rotates the chain. No-op if there's no history yet. |
| Send Status Request | Indigo's built-in status request — wired to trigger a fresh fetch. |

## Examples

### Joke of the day on a Control Page

1. Create a Joke device
2. Click **Distribute Calls** so the device gets a sensible interval
3. Drop the device's `current_joke` state onto a Control Page as a text element
4. Add an action button bound to **Refresh Content** for when you want a new one on demand

### Trivia question via speech

1. Create a Trivia device
2. Create a trigger on a motion sensor or schedule
3. In the trigger action chain:
   - Call **Refresh Content** on the device
   - Wait 2 seconds
   - Speak Text: `%%d:DEVICE_ID:state:current_content%%`
   - Wait 5 seconds (give the listener a moment to guess)
   - Speak Text: `The answer is: %%d:DEVICE_ID:state:current_answer%%`

### On This Day in your morning routine

1. Create an On This Day device, set Auto-refresh Interval to `1440` (24 hours) so it picks fresh content overnight
2. In your morning routine, speak `%%d:DEVICE_ID:state:current_content%%` to hear today's historical event

## Notes and limitations

- The local usage counter only tracks calls made by this plugin. If your API key is used elsewhere, those calls won't appear here. The **Reset Counter** button gives you a manual escape hatch.
- Deduplication is in-memory per session; restarting the plugin clears the seen-history.
- The Cocktail device queries by rotating through a list of common spirits (vodka, gin, rum, tequila, whiskey, bourbon, brandy, vermouth, campari, champagne) since the API requires a search term. Niche cocktails outside those ingredient lists won't appear.
- On This Day will eventually exhaust — API Ninjas returns up to 10 events per date, and the 500-item dedup cap auto-clears so repeats will start showing once you've seen them all.

## Author

Built by [Durosity](https://github.com/Durosity) as part of a small collection of Indigo plugins. Bug reports and PRs welcome via the [Issues](../../issues) tab.

## License

MIT — see [LICENSE](LICENSE) for the full text.
