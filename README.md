# Müzli-könyvek — GitHub Pages

Static site. No build step, no framework, no server. You edit one JSON file
to add a book.

## Layout

```
index.html                         the hub
konyvek.json                       the list of books
konyvek/
  a-hetedik-fej.html
  muzli-iskolaba-megy.html
  a-masodik-fej-pere.html
  ...
```

## Publishing it

1. Create a repository, e.g. `muzli-konyvek`.
2. Copy these files in, keeping the folder structure.
3. Settings → Pages → Source: *Deploy from a branch*, branch `main`, folder `/ (root)`.
4. It goes live at `https://<username>.github.io/muzli-konyvek/` within a minute or two.

Don't test by double-clicking `index.html` on your computer. Browsers block
`fetch` on `file://`, so the book list won't load and you'll see the warning
box. It works on GitHub Pages. To test locally, run `python3 -m http.server`
in the folder and open `http://localhost:8000`.

## Adding book four

1. Drop the HTML into `konyvek/`.
2. Change its entry in `konyvek.json`:

```json
{
  "id": "negyedik",
  "sorszam": 4,
  "cim": "A negyedik könyv",
  "leiras": "Egy mondat arról, miről szól.",
  "fajl": "konyvek/a-negyedik-konyv.html",
  "allapot": "kesz",
  "vegek": 6,
  "nyomok": 30
}
```

That's the whole job. `"allapot": "keszul"` shows a dashed "Hamarosan…" card
with no link — use it to tease a book that isn't ready. Anything with
`"allapot": "kesz"` and a `fajl` becomes a working link.

Inside the new book, set its keys to match its `id`:

```js
const KULCS    = "mk:v1:negyedik:veg";
const KULCS_NY = "mk:v1:negyedik:nyom";
```

**Never change an `id` once readers have used the book.** The id *is* the
save file. The title, the description and every word of the text can change
freely — the id must not.

## How progress survives your edits

Progress lives in the reader's browser (`localStorage`), not in the files, so
updating the site never touches it. Two rules keep it that way:

**Endings are stored by number.** `VEGEK` entries look like `[3, "A kulcslyuk"]`.
Only the `3` is saved. Retitle the ending whenever you like.

**Clues are stored by id.** `NYOMOK` entries look like `["n07", "Müzli neve"]`,
and each page carries `"ny": "n07"`. Only `n07` is saved.

This is what broke when you renamed Feri bácsi to Lajos bácsi. Clues used to
be saved by their label, so `"Feri bácsi dúdolása"` became a dead entry and
the clue showed as not found — while still counting toward the total, which
is how a reader could end up seeing `29 / 28`. Now the label is display text
only.

Two safeguards are built in. A one-time import block reads any progress saved
under the old keys and converts it, including a rename table that maps
`"Feri bácsi dúdolása"` to `n01`. And a guard drops any saved id that no
longer exists, so the counter can never exceed the maximum even if you delete
a clue later. Both are marked with comments and can be removed once everyone
has visited the new site at least once.

### Renaming or reordering clues later

Edit the label freely; do nothing else. If you *insert* a clue, give it a new
unused id (`n29`, `n30` …) rather than renumbering — renumbering would shift
everyone's saved clues onto the wrong entries. Ids don't have to be in order.

## Caching

GitHub Pages tells browsers to re-check HTML after about ten minutes, so your
edits reach readers shortly after you push. Nothing is cached permanently.

Don't add a service worker unless you have a specific reason. It's the usual
cause of readers being stuck on an old version, and you don't need offline
support for this.

## One thing to expect at launch

`localStorage` is tied to the web address. Anyone who has been reading the
books at their old address starts from zero on the new one — this is a
browser rule and there is no way around it. It happens once, at the move.
After that, nothing resets.

Also worth knowing: every project on `<username>.github.io` shares the same
storage. That's why every key starts with `mk:v1:` — so these books can't
collide with anything else you ever publish there.
