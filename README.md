# Card Binder

A single-page tracker for a Magic: The Gathering collection and the decks built from it. It imports a ManaBox CSV export, shows which cards are owned, which are already committed to decks, and which are missing or proxied. It syncs everything through this repo.

- **Live site:** https://gutelfuldead.github.io/mtg/
- **Data file:** [`card-binder.json`](card-binder.json), raw at `https://raw.githubusercontent.com/gutelfuldead/mtg/main/card-binder.json`
- **Card data source:** [Scryfall API](https://scryfall.com/docs/api), looked up in the browser and cached per device

> This repo is public, so the collection and deck lists in `card-binder.json` are publicly readable.

## Repository contents

| File | Purpose |
|---|---|
| `index.html` | The entire app: HTML, CSS and JavaScript in one file. No build step and no dependencies apart from Google Fonts. |
| `card-binder.json` | The synced collection and decks. Written by the app through the GitHub API, and read by anyone with the raw URL. |
| `README.md` | This file. |

## Using the app

### Import the collection

1. In ManaBox, export the whole collection as **CSV**.
2. Open the site and go to **Import**, then pick the CSV file (or paste its text).
3. Confirm with **Replace my collection**.

Importing replaces the collection and every deck that came from ManaBox. Decks pasted into the app and proxy marks are kept.

ManaBox rows are handled by their `Binder Type` column:

| Binder Type | Treatment |
|---|---|
| `binder` | Owned copies that aren't in a deck. |
| `deck` | Owned copies **and** a deck named after the `Binder Name`. |
| `list` | Wishlist. Ignored entirely. |

Rows with `Proxy = true` count toward their deck but not toward the collection.

### Tabs

- **Collection:** every owned card, with search (names and type lines), filters (*All*, *In a deck*, *Not in a deck*, *Needed for decks*), a color-identity filter (the WUBRG pips show cards that fit within the selected colors; colorless cards are always included), and a type filter. Tap a card for its image, rules text, printings, and the decks that use it.
- **Decks:** every deck with an ownership bar. Tap **New deck** to paste a list. Lines like `1 Sol Ring`, `1x Sol Ring` and `1 Sol Ring (C21) 263 *F*` all work, and a `Commander` section header marks the commander.
- **Deck view:** group by **status**, **card type** or **mana value** (with a curve chart). Use **Mark proxy** on cards you've printed. **Copy missing list** gives a buy or print list, and **Copy deck list** exports the deck as text.
- **Import:** CSV import, GitHub sync settings, and backup files.

### Card statuses in a deck

| Status | Meaning |
|---|---|
| In your collection | Owned copies cover this deck and every other deck that uses the card. |
| Shared | You own enough for this deck, but other decks also claim the same copies. |
| Own some, need more | You own fewer copies than this deck needs. |
| Not owned | You own no copies. |
| Proxy printed | Not fully owned, but marked as proxied. |
| Basic land | Not tracked. Basics are treated as unlimited. |

## Sync between devices

Every device that opens the site loads `card-binder.json` automatically and can view everything. To **edit** from a device and have the changes reach the others, connect that device once:

1. Create a [fine-grained personal access token](https://github.com/settings/personal-access-tokens/new):
   - **Repository access:** Only select repositories, then `gutelfuldead/mtg`
   - **Repository permissions:** Contents set to **Read and write** (GitHub adds Metadata: Read-only automatically)
   - Everything else: No access
2. On the site, go to **Import → Sync with GitHub**. The username and repo are prefilled, so paste the token and tap **Connect**.

Once connected:

- Changes are committed to `card-binder.json` on `main` about 3 seconds after they're made, or immediately when the page is hidden.
- The latest copy is pulled when the page opens or regains focus.
- If the file changed both locally and in GitHub, the app asks which copy to keep. The other copy remains in commit history.
- The token is stored only in that browser's `localStorage` and is never written to the repo. Revoke it on GitHub at any time. When it expires, make a new one, then disconnect and reconnect.

Devices that aren't connected read the copy published by GitHub Pages, which updates a minute or two after each commit.

### Backups

**Import → Backup → Save backup file** downloads the same JSON format as `card-binder.json`. Loading a backup replaces everything on that device, and on GitHub too if the device is connected.

## Hosting

GitHub Pages serves `index.html` from the `main` branch, `/(root)` folder (**Settings → Pages**). Free accounts require the repo to be public. Every sync commit triggers a Pages rebuild; that's expected and harmless.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Site shows 404 | Use the exact URL from **Settings → Pages**. Check that `index.html` is at the repo root with that exact lowercase name, and that the Pages deployment in **Actions** has a green check. |
| "Card lookup paused" or a countdown banner | Scryfall rate-limited the lookup. Keep the page open and it resumes on its own, or it picks up where it left off on the next visit. |
| "GitHub rejected the token" | The token expired or was revoked. Create a new one and reconnect. |
| Another device doesn't show recent changes | Wait for the Pages rebuild, or connect that device to read straight from GitHub. |
| Some cards show "Not looked up yet" | Their Scryfall lookup hasn't finished, or the card isn't on Scryfall. Reopen the page to retry. |

---

## For agents and scripts

This section is the contract for reading or changing the data without the app.

### Reading the data

Fetch `https://raw.githubusercontent.com/gutelfuldead/mtg/main/card-binder.json`. No auth is needed. The file is about 1.3 MB.

### `card-binder.json` format (version 2)

```jsonc
{
  "app": "card-binder",
  "version": 2,
  "savedAt": "2026-10-07T03:45:38.877Z",   // ISO time of the last write
  "importedAt": 1791345938877,             // ms since epoch of the last CSV import, or null
  "cardFields": ["name","set","num","foil","qty","cond","lang","binder","inDeck","sid"],
  "cards": [                               // one row per printing/condition, positional per cardFields
    ["Hostile Minotaur", "M20", "331", false, 2, "near_mint", "en", "M19 Starter Red PRECON", true, "0e1263ea-adc9-442b-b13e-9afb69596372"]
  ],
  "decks": [
    {
      "id": "mux7gtoyrfb2a",               // stable id; keep it when editing a deck
      "name": "Science!",
      "source": "manabox",                 // "manabox" = replaced on each CSV import; "pasted" = user-made, kept
      "cards": [
        { "name": "Island", "qty": 5, "proxy": false },
        { "name": "Bristly Bill, Spine Sower", "qty": 1, "proxy": false, "commander": true, "t": "Creature" }
      ]
    }
  ]
}
```

**`cards` fields**

| Field | Type | Meaning |
|---|---|---|
| `name` | string | Card name as exported by ManaBox (double-faced cards look like `A // B`). |
| `set` | string | Set code, uppercase. |
| `num` | string | Collector number. |
| `foil` | boolean | Foil or etched. |
| `qty` | number | Copies in this row. |
| `cond` | string | Condition, for example `near_mint`. |
| `lang` | string | Language code. |
| `binder` | string | ManaBox binder or deck name where the copies physically live. |
| `inDeck` | boolean | `true` if the row came from a ManaBox deck binder. |
| `sid` | string | Scryfall card ID for this printing (may be empty). |

**Deck card fields:** `name`, `qty`, `proxy` (boolean). Optional fields are `commander` (boolean, only on pasted decks that had a Commander section) and `t` (cached primary type).

Version 1 files, from older backups, have the same top-level shape but `cards` is an array of objects keyed by the same field names. The app reads both versions and writes version 2.

### Rules the app applies

- **Name matching:** compare card names by the front face (the text before ` // `), trimmed, lowercased, with runs of whitespace collapsed. A deck card matches any owned printing of the same name.
- **Owned copies** of a name: the sum of `qty` across all `cards` rows. Rows from ManaBox deck binders count as owned. Proxies and wishlist rows are never in `cards`.
- **Claimed copies** of a name: the sum of `qty` for that name across **all** decks.
- **Free (spare) copies:** owned minus claimed. Cards that aren't committed to any deck are the ones with free copies above 0.
- **Basic lands** (`Plains`, `Island`, `Swamp`, `Mountain`, `Forest`, `Wastes`, and `Snow-Covered` versions) are treated as unlimited and never flagged as missing.
- The file holds **no color, type or mana value data** apart from the optional cached `t`. Look those up on Scryfall using `sid`, or by name.

Example for finding spare cards:

```python
import json, urllib.request
from collections import Counter

d = json.load(urllib.request.urlopen(
    "https://raw.githubusercontent.com/gutelfuldead/mtg/main/card-binder.json"))
key = lambda n: " ".join(n.split(" // ")[0].split()).lower()
rows = [dict(zip(d["cardFields"], r)) for r in d["cards"]]
owned, claimed = Counter(), Counter()
for r in rows: owned[key(r["name"])] += r["qty"]
for deck in d["decks"]:
    for c in deck["cards"]: claimed[key(c["name"])] += c["qty"]
spare = {k: owned[k] - claimed[k] for k in owned if owned[k] > claimed[k]}
```

### Looking up card details

Use `POST https://api.scryfall.com/cards/collection` with up to 75 identifiers per request (`{"id": sid}`, or `{"name": name}` when `sid` is empty). This endpoint is limited to **2 requests per second**, and exceeding it locks the client out for 30 seconds, so space requests at least 500 ms apart. For whole-database work, use Scryfall's bulk data files instead.

### Writing the data

- Write through the GitHub contents API: `PUT /repos/gutelfuldead/mtg/contents/card-binder.json` with the current file `sha`, a commit message, and base64 content. Read the latest `sha` first; a stale one returns 409.
- Keep the version 2 format exactly, including `cardFields` order, and update `savedAt` to the current time so devices notice the change.
- To **add a deck**, append to `decks` with a new unique `id`, `"source": "pasted"`, and `proxy: false` on each card. Don't use `"source": "manabox"`, or the next CSV import will delete the deck.
- Don't edit `cards` by hand unless you mean to; the next CSV import replaces them.
- Connected devices pick up the change on their next open or focus. A device with unsynced local edits will ask which copy to keep.
