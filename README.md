[README.md](https://github.com/user-attachments/files/29755809/README.md)
# NY Metro Industrial Analytics — Hosted Data Architecture (v26)

The app no longer embeds comp data in the HTML. It fetches JSON from the `data/`
folder at load time, so updating comps never requires regenerating or
re-uploading the 1MB HTML file.

## Folder layout

```
index.html                      ← the app (only changes when features change)
data/
  manifest.json                 ← lists which data files to load, in order
  lease_comps_base.json         ← 632 lease comps (the current dataset)
  sale_comps_base.json          ← 196 sale comps
```

`manifest.json` looks like:

```json
{
  "lease": ["lease_comps_base.json"],
  "sale":  ["sale_comps_base.json"]
}
```

The app fetches every file listed and concatenates them, so you can add comps
by **adding a new small file** — the base file never gets touched again.

## One-time setup (GitHub Pages)

1. Create a new GitHub repo (private repos work with Pages on paid plans;
   otherwise public — note the data would be publicly reachable by URL).
2. Upload `index.html` and the `data/` folder (drag-and-drop in the GitHub web
   UI works, no git required).
3. Repo → Settings → Pages → Source: "Deploy from a branch" → `main` / root.
4. Your site is live at `https://<user>.github.io/<repo>/` a minute later.

Netlify works identically (drag the whole folder into the Netlify deploy zone),
same as the Hyrox tracker.

## Adding new comps (the whole point)

1. Give Claude the new comps (email, PDF, CoStar export — whatever) and ask for
   a JSON batch file matching the comp schema below. This is a tiny task any
   model handles — no need to paste the app or the existing dataset.
2. Upload the file to `data/` (e.g. `lease_comps_2026-07.json`).
3. Edit `manifest.json` in the GitHub web UI — add one line:

```json
{
  "lease": ["lease_comps_base.json", "lease_comps_2026-07.json"],
  "sale":  ["sale_comps_base.json"]
}
```

Commit. Refresh the site. Done — everyone with the link sees the new comps.

## Comp record schema (lease and sale use the same shape)

```json
{
  "lat": 40.123, "lng": -74.456,
  "tenant": "Tenant Name",
  "address": "123 Main St, Town, NJ 07000",
  "city": "Town",
  "rate": "$14.50/SF", "rate_raw": 14.5,
  "lcd": "01/15/2026",
  "leased_size": "150,000 SF", "size_raw": 150000,
  "building_size": "300,000 SF",
  "escalations": "3.50%",
  "clear_height": "36'0\"",
  "owner": "Owner Name",
  "submarket": "Exit 8A",
  "type": "New",
  "term": "60 mos", "free_rent": "2 mos", "ti": "$1.00/SF",
  "bldg_class": "A", "yr_built": "2020",
  "dock_doors": "40", "dock_ratio": "40 / 2",
  "comments": "",
  "needs_verification": false,
  "source": "Source Name",
  "state": "NJ", "zip": "07000"
}
```

Notes: `rate_raw` is numeric $/SF (`null` if unknown), `size_raw` is numeric SF
(used for filtering), all other fields are display strings. Empty string for
unknowns.

## If the data ever fails to load

A red banner appears at the top with the error. Most common cause: opening
`index.html` directly from disk (file://) — browsers block local fetches. The
page must be served from GitHub Pages / Netlify / any web host.
