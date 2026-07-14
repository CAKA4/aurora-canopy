# Aurora Canopy — Live Backend Setup

The field app (`index.html`) talks directly to an ArcGIS Online hosted feature
service. Four one-time setup steps, then it's live. Do this on **your own
ArcGIS account first** (staging); migrate to the Town's org at the end.

Because it uses OAuth sign-in, it must run from a real web address (GitHub
Pages) — not opened as a local file, and not inside a chat preview.

---

## 1 — Publish your geodatabase as a hosted feature service

In ArcGIS Pro (or from the AGOL item page):
- Publish the `trees` feature class **and** the `treeinspections` table into a
  **single hosted feature service** so they share one FeatureServer URL.
- Keep the relationship (`tree_fk2` → `GlobalID`) and turn on **attachments**
  and **editing** for the service.
- Note the service URL — it looks like
  `https://services.arcgis.com/XXXX/arcgis/rest/services/Aurora_Tree_Inventory/FeatureServer`
- Open the URL in a browser and note the **layer index** of the trees (usually
  `0`) and the **table index** of treeinspections (usually `1`).

## 2 — Register the OAuth application

AGOL → **Content → New item → Developer credentials → OAuth 2.0**, or
developers.arcgis.com → your apps → register:
- Add a **Redirect URI** that exactly matches where you'll host the app, e.g.
  `https://YOURNAME.github.io/aurora-canopy` (and `http://localhost:8080` if you
  want to test locally with a dev server).
- Copy the **Client ID**. (No client secret is used — this is a browser app.)

## 3 — Fill in the CONFIG block

Open `index.html`, edit the `CONFIG` object near the top of the first script:

```js
const CONFIG = {
  portalUrl:  "https://www.arcgis.com",      // or your org URL
  appId:      "YOUR_CLIENT_ID",
  serviceUrl: "https://services.arcgis.com/XXXX/arcgis/rest/services/Aurora_Tree_Inventory/FeatureServer",
  treeLayerId: 0,
  inspTableId: 1,
  fkField:    "tree_fk2"
};
```

The tree/inspection field names in the forms already match your geodatabase
schema. If you rename fields later, update the form IDs to match.

## 4 — Deploy to GitHub Pages

- Create a repo, add `index.html`, enable **Settings → Pages → deploy from
  branch** (root).
- Wait for the `https://YOURNAME.github.io/...` URL to go live.
- Confirm that URL is the **exact** redirect URI you registered in step 2.

Open the page → **Sign in with ArcGIS** → the map loads your trees.

---

## Test checklist (first live run)

- [ ] Sign-in redirects to ArcGIS and back, map loads with trees colored by health
- [ ] Tap a tree → detail + inspection history load
- [ ] Log an inspection → new row appears in `treeinspections` (check the layer),
      with `tree_fk2` = the tree's GlobalID
- [ ] Attach a photo → confirm it lands as an attachment on the inspection row
- [ ] Edit a tree → the change persists and the marker recolors
- [ ] Add a tree (tap + GPS) → new point in the layer, then first activity logs
- [ ] Turn on airplane mode mid-session → submit → "saved on device"; turn it
      back on → the buffered edit syncs automatically
- [ ] Item page → **Export Data → File Geodatabase** → downloads a `.gdb` with
      attachments (the Town's deliverable)

## Notes & gotchas

- **Can't test in a chat preview or as a local file** — OAuth needs the deployed
  origin registered in step 2.
- **Field names are case-sensitive** and must match the service exactly; a wrong
  name shows as a failed edit.
- **Shared vs. named login:** if crews sign in with individual org accounts you
  get per-user attribution automatically. A shared login still works — you just
  rely on the inspector field.
- **Inspector field domain:** in the geodatabase `inspector_name` has a fixed
  coded-value domain (KC/HS/CG). If you want full names, remove that domain (make
  it a plain text field) on the hosted service before testing, or names outside
  the list will be rejected. Same for the tree layer's `INSPECTR` field.
- **Migration to the Town's org:** publish/clone the finished service into their
  org preserving GlobalIDs (ArcGIS API for Python `clone_items`), then register a
  new OAuth app there and repoint `CONFIG`. Verify the relationship survives.
