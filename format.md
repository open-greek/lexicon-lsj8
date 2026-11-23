# Lexicon entry format (open-greek-lexica 0.3.0)

> This copy of the format comes with release `lsj8-2026-11-23.1` of
> `lsj8-1901`. The programs it names under `scripts/` build the lexica and
> are not part of this repository, and neither is `lookup/`, the lemma
> lookup. The JSON Schemas are in `schema/` and the test vectors in
> `vectors/`. `corrections.json` here leaves out the `evidence` of each
> record; `schema/lexicon-public-corrections.schema.json` describes it.

One format for every public-domain lexicon in the lexica dataset: Cunliffe's
*Lexicon of the Homeric Dialect* first, then LSJ (8th edition), Greenfield's
New Testament lexicon and the Didot Stephanus. The JSON Schemas in
`schema/` are normative; this page explains them. The version is in every
manifest (`"format": "open-greek-lexica/0.3.0"`). A change that breaks a
reader bumps the minor version while the format is 0.x, and the major
version from 1.0 on. 0.2.0 added the `correction` mark and the `error_in`
and `summary` fields of correction records. 0.3.0 added `pages.json`,
`aliases.json`, the entry fields `fold` and `quality`, LSJ8's
`source.perseus_id` and the manifest's `perseus_ids`, the rule that a key
starts with a Greek letter or a hyphen, the `access` and `credit` of every
scan, and the test vectors in `vectors/`.

The reader site does not read these directories; it serves a release of
SQLite files exported from them.

## Layout

Each lexicon is one directory named by its id (`cunliffe-1924`):

| file | content |
|---|---|
| `README.md` | what the lexicon covers, its signs, conversion rules and known problems |
| `manifest.json` | title, edition, licence, attribution, pinned sources, scans, counts, and every file with its size and SHA-256 |
| `entries/0001.jsonl`, ... | entries in printed order, one JSON object per line, 500 per file |
| `keys.json` | lookup key -> entry ids, for headword search |
| `corrections.json` | the reversible corrections applied to the source |
| `pages.json` | one row per printed page and column: its scan image, access and the entries on it |
| `aliases.json` | retired ids -> the ids that replace them |
| `reverse/<work>.json` | corpus locus -> ids of the entries that cite it |
| `lookup/lemmas.json` | annotation lemma -> entry ids |

The manifest lists each entries file with its first and last `seq` and key,
so a static site can load `keys.json`, find an id, and fetch only the file
that holds it. Ids sort in printed order through `seq`. Builds are
deterministic: the same inputs give the same bytes, and nothing records the
build time.

The test vectors in `vectors/` (one JSON object per line,
`{"input": ..., "output": ...}`) hold the expected output of `lookup_key`,
`fold`, `translit` and `greek_runs` for several thousand inputs, so that
another implementation (a site's query code) can check itself against the
builders. `python scripts/lexica_format.py vectors` checks them, and with
`--write` rewrites them from the current functions (`--add NAME=FILE` adds
the lines of a file as inputs); a change to a function that changes a
vector is a new normalization version.

## Entries

```json
{"id": "cunliffe-1924:agathos", "seq": 19, "kind": "entry",
 "headword": "ἀγαθός", "key": "ἀγαθός", "alt_keys": [], "fold": "αγαθοσ", "homograph": null,
 "flags": [], "quality": "transcribed-checked",
 "text": "ἀγαθός, -ή, -όν. (1) Well-born, noble Ξ 113, Φ 109: δ 611, ...",
 "senses": [{"label": "1", "path": "1", "span": [17, 144]}, ...],
 "citations": [{"printed": "Ξ 113", "encoded": "Il. 14.113",
                "urn": "urn:cts:greekLit:tlg0012.tlg001.perseus-grc2:14.113",
                "address": "homerus-epic.ilias:14.113", "span": [38, 43], "sense": "1"}, ...],
 "xrefs": [...], "marks": [{"type": "headword", "span": [0, 6]}, ...],
 "loc": {"vol": null, "page": 1, "col": null}, "loc_end": {"vol": null, "page": 1, "col": null},
 "page_breaks": [], "source": {"xml_id": "agathos-cunliffe-lex", "lemma": "ἀγαθός"}}
```

- **`id`**: `<lexicon>:<local id>`. The local id comes from the source's own
  identifier where there is one (the Perseus xml:ids for Cunliffe) and is
  never reused. An OCR'd lexicon without source ids uses the position of
  the entry's first line: `<page>-<col>-<line>` for LSJ8 (printed page,
  column, line in that column), with the volume first for a lexicon in
  volumes. An id that has to go (entries merged or split, a running head
  or a quotation read as an entry) is kept in `aliases.json` with the ids that replace it
  (below), so a link to it still resolves.
- **`headword`**: as printed, without the flag signs and homograph number,
  which have their own fields. Several printed forms are joined with `, `.
- **`key`** and **`alt_keys`**: normalized lookup keys (below). `alt_keys`
  holds the other printed forms and, where the source gives one, a standard
  lemma spelling (the Perseus encoding has ἀγορά for Cunliffe's ἀγορή). A
  standard spelling is not added when another entry is printed under it.
  A key starts with a Greek letter or a hyphen (a suffix, `-δε`). A Greek
  letter is a capital or small letter of the Greek blocks (U+0370-03FF,
  U+1F00-1FFF), not a sign, numeral sign, ypogegrammeni or
  prosgegrammeni, nor a Coptic letter; the schema's pattern spells out the
  same set. The builders refuse an entry with any other key, so an
  encoder's placeholder (Cunliffe's `crossref`) or a stray sign
  (`¨καταβρόχω`) never becomes one.
- **`fold`**: the letters of `key` (below), for search that forgives
  breathings and accents.
- **`homograph`**: the printed number of homographs, or null.
- **`flags`**: signs printed before the headword: `dagger`, `asterisk`,
  `double-dagger`. Their meaning is the lexicon's own and is stated in its
  README (Cunliffe, preface p. vi and p. ix: † before a verb means that only
  the parts cited occur; * means that the form is not recorded but assumed).
- **`quality`**: how far the entry has been checked against the print.
  `transcribed-checked`: a transcription checked against the print
  (Cunliffe, from the Perseus encoding with its corrections);
  `ocr-headword-checked`: OCR text whose headword and entry start passed
  the build's checks; `ocr`: OCR text the build flags for review (LSJ8: an
  entry with any `source.review` code). The checks catch misread
  headwords; they do not prove a headword right. For LSJ8 they ask that
  the key have an accent (a word of two or more syllables) but not two, a
  breathing on its first vowel and none inside the word, only Greek
  letters, and the marks of its matched LSJ9 entry, and that the entry
  start in alphabetical order and at an indent. A site shows the last two
  values with a warning.
- **`text`**: the whole entry as plain text, in printed order, from the flags
  and headword on. It has no inline markup at all.
- **`senses`**: the sense tree. Each node has the printed `label` (`1`, `a`,
  `β`, `II`, without brackets), a `path` of labels from the top (`7.a`),
  the `span` of its text, and child `senses`. Where the source names its
  senses (Perseus xml:ids), the path is taken from that name, so a nesting
  error in the encoding does not change it.
- **`citations`**: every reference to a text, with `printed` (as in the
  book), `encoded` (as in the source encoding, when different), `urn` (CTS),
  `address` (`work:locus` in the corpus release named in the manifest, or
  null), the `span` of the printed reference, the `sense` path containing
  it, and `quote` and `quote_span` when the reference supports quoted words.
  `problem` explains a reference that could not be read normally, or
  whose printed text and link disagree; a continuation reference (a bare
  line number) whose link leaves the book of the reference before it is
  flagged too (a reference in a parenthesis between them does not count).
  The printed form follows the source text, the address the source link.
- **`xrefs`**: cross-references to other entries (`target` id and `sense`
  path), with their spans, and `encoded`, the reference as the source
  encoding writes it, when the text gives it in the printed form (`ἐπι- 7`
  for the printed `(7)` of `ἐπι- (1) (7)`).
- **`marks`**: typographic or editorial spans: `headword`, `italic`, `bold`,
  `gloss` (the encoder's tagging of a definition), `greek`, `latin`, `sup`,
  and `correction` (below).
- **`loc`**, **`loc_end`**: volume, page and column where the entry starts
  and ends. `page_breaks` gives the text offset at which each later page or
  column starts. Null where the source does not say.
- **`source`**: identifiers in the source encoding, for tracing. An LSJ8
  entry of the body has `perseus_id`, the id of the same entry in the
  Perseus encoding of LSJ9 (`n34` for ἀβαθής), where one can be matched
  (*Fitting the other lexica*, below).
- **`corrections`**: ids of the corrections that changed the entry.

### Why spans instead of inline markup

All structure points into `text` by `[start, end)` offsets in Unicode code
points (Python string indices; JavaScript code must count code points, not
UTF-16 units, for characters outside the Basic Multilingual Plane, which
Greek text does not use). Plain text is what an OCR'd lexicon produces
anyway, it can be searched and diffed as it is, and a page that renders it
never has to sanitize markup from a third-party encoding. Spans nest
(senses inside senses, citations inside senses), and the build checks that
every span lies inside the text and every sense inside its parent.

## Lookup key normalization (version 1)

`lookup_key` in `scripts/lexica_format.py`:

1. Unicode NFD.
2. Remove macron, breve and macron below (U+0304, U+0306, U+0331).
3. Replace grave (U+0300, U+0340) and U+0341 by acute (U+0301).
4. NFC, then lower case.
5. Remove leading and trailing spaces, †, ‡, \*, digits (homograph numbers),
   superscript digits, brackets, and `. , ; : ·`.
6. Unify apostrophes to U+2019 and lunate sigma to σ; write ς at the end of
   a word and σ inside it or before a hyphen or bracket (`δυσ-`,
   `ἔλασ(σ)ε`).

Breathings, accents, diaeresis, iota subscript, digamma and hyphens (which
mark prefixes and suffixes) are kept, so ἦ and ἤ or ὅς and ὄς stay distinct,
while ὃς and ὅς share a key. A site normalizes a query the same way before
looking it up in `keys.json`.

The manifest's `key_normalization` gives the version of `lookup_key` and
of the three functions below, and where their test vectors are.

### Letters (fold version 1)

`fold`: the lookup key, then NFD; every combining mark is removed
(breathings, accents, diaeresis, iota subscript); final and lunate sigma
become σ and the other typographic letter forms (ϐ ϑ ϕ ϰ ϱ ϖ ϵ) their usual
letters; everything that is not a letter (hyphens, apostrophes, spaces,
brackets) is removed. Digamma stays. `ἔλασ(σ)ε` folds to `ελασσε`, `ὅς τις`
to `οστισ`. Entries carry their fold; a site folds the query the same way.

### Latin letters (transliteration version 1)

`translit` makes one Latin spelling of a Greek key and of a query typed on
a Latin keyboard, so that `logos`, `psuche` or `Socrates` find λόγος,
ψυχή and Σωκράτης:

1. NFD; every combining mark is removed; lower case; the letter forms as
   in fold.
2. α a, β b, γ g, δ d, ε e, ζ z, η e, θ th, ι i, κ k, λ l, μ m, ν n, ξ x,
   ο o, π p, ρ r, σ s, τ t, υ y, φ ph, χ ch, ψ ps, ω o, ϝ w. Breathings are
   not written.
3. Everything but a to z is removed.
4. In this order: kh -> ch; c not before h -> k; ks -> x; f -> ph; u -> y;
   j -> i; h not after t, p or c is removed (hodos, rhetor); g before g, k,
   x or ch -> n (angelos, ankyra).

The Greek side never produces what step 4 rewrites, so the function is
idempotent: a transliterated key is its own query. Keys are not stored
transliterated; a site makes the index from `keys.json`.

### Words of a passage (Greek run version 1)

The corpus reader makes a word of a passage tappable by splitting it into
runs of the characters U+0370-03FF, U+1F00-1FFF, U+0300-036F, U+1FBD
(koronis), U+02BC and the ASCII apostrophe; U+2019 ends a run. `greek_runs`
gives the same runs as `[start, end, surface]` in code points, so a
lexicon built for the reader can address the words it shows. The `n`th run
with the same surface in a passage is its occurrence number.

## Corrections

### Where they show

Every correction that changes an entry's text or a reference is visible at
the place it applies, as a `correction` mark over the corrected text:

```json
{"type": "correction", "span": [179, 184], "correction": "cunliffe-1924-c0019",
 "original": "ψ 845", "error_in": "encoding",
 "reason": "The print has Ψ 845 (Iliad 23.845); the encoding read ψ 845 and linked the Odyssey 23.845, past the end of the book."}
```

`text` and the citation hold the corrected reading. `original` is the reading
before the correction: for a reference, the printed form it had (`ψ 845`);
for other elements, the source text (`+ καταβρόχω`); null when the
correction moved or inserted text rather than changing it. `error_in`
says whether the printed book (`print`, a correction of the author) or the
source encoding (`encoding`, the print is right) was wrong, and `reason` is
the record's one-line `summary`, so a reader can show the corrected reading
with the original and the reason on hover without loading
`corrections.json`.
The full evidence is in the record named by `correction`. When the print
is wrong in a different way from the source (Cunliffe printed ζ 168, the
source has ξ 163, the line meant is ζ 163), the record and the mark also
have `print_reading`, the print's own reading. A correction that
changes the link but not the printed form (a bare line number that
continues another book) is marked too, with the same reading in `original`.
A record that also changes the text after the element (`tail`) marks the
element and that text, and `original` includes the old text (`Θ 98,` for
`Θ 98. 3 pl.`). A reference removed from an entry leaves an empty mark
(`span[0] == span[1]`) where it stood, with the removed reading in
`original`; a reader should show it as a point, not a range.
Corrections that change no entry text, such as an added page break or a
removed running head, have no mark; the entry's `corrections` list still
names them when they belong to an entry. The build fails if an `element`,
`move`, `insert-element` or entry-level `remove-element` correction leaves
no mark in any entry, or a mark other than a removal's covers no text.

### Records

`corrections.json` lists the changes made to the source before conversion,
in the order applied. Every record has `id` (the lexicon id, `-c` and a
number of four or more digits, as `cunliffe-1924-c0019`), `kind`, `error_in` (`print`
or `encoding`), `target`, `original`, `correction`, `summary` (one line for
readers), `evidence`, `scan` (where the evidence can be seen), `date` and
`inverse`, which is the same change with `original` and `correction`
exchanged. The build fails when a record no longer matches its original,
and the tests apply all records and their inverses to the pinned source and
check that serializing the reverted tree gives the same XML as the pinned
source. Kinds:

- `element`: change an element's attributes and text (a wrong book number,
  a misencoded sign); `target` names the entry and element, and `nth` the
  element's position among the elements of that name in the entry, counted
  in the tree as earlier records have left it, when another one would match
  the same original. An attribute that `original` names and `correction`
  does not is removed, so a record may add one and its inverse takes it
  away again. `original` and `correction` may both give `tail`, the
  exact text after the element, to change it as well: stray text the
  source left beside the element (`4</bibl>.3,` for the printed 413), or
  punctuation and words the print has there; the correction mark then
  covers the element and that text. A record on a paragraph (`p`) changes
  only its own words, and its mark covers the text node it changed, not the
  whole paragraph;
- `page-break`: insert or remove a page break before an anchor text;
- `move`: move a division to another parent (repairs nesting);
- `insert-element`: insert an element the source lost (a dropped reference)
  before the `nth` element of that name in the entry, with the text that
  follows it (`tail`), or right after that element and its text when the
  target gives `"after": true` (a reference lost at the end of a paragraph);
  the inverse removes it after checking its exact XML;
- `remove-element`: remove a top-level element (a running head encoded as an
  entry), keeping its exact XML and position for the inverse; or, with
  `entry`, `element` and `nth` in the target, remove that element of the
  entry with the text that follows it (`tail`): a reference the source has
  and the print does not. The entry gets an empty correction mark where the
  element stood, with the removed reading as `original`;
- `insert-entry`: insert an entry the source lost, typed from the scan;
  `target` names its `div`, and `correction` gives its `index` among the
  children of the body, its exact `xml` and the whitespace after it
  (`tail`). The inverse removes it after checking its exact XML. The entry
  gets a correction mark over its whole text, with `original` null;
- `text`: for OCR'd lexica: replace text within one line of the OCR text;
  `target` has `page`, `col`, `line` (the line in that column) and
  `offset` (in characters, in the line as earlier records left it), and
  `original` and `correction` are `{"text": ...}`;
- `start`: for OCR'd lexica: whether a line begins an entry, where the
  lexicon's rules for finding entry starts get it wrong (an entry run on
  inside the one before, or a line of an entry read as an entry of its
  own); `target` has `page`, `col` and `line`, and `original` and
  `correction` are `{"start": true}` and `{"start": false}`, one each.
  The build fails when the rules do not give the original, or when a line
  made a start has no headword. The record changes no text, so it has no
  mark; the entry that begins at the line, or that now holds it, names it
  in `corrections`. An entry id that a start record retires goes into
  `aliases.json`.

Systematic conversions that apply to every entry (how a printed citation is
written, how sense labels are shown) are not corrections; each lexicon's
README lists them.

## Citation links

`address` names a passage of the public corpus as `<work>:<locus>`, for
example `homerus-epic.ilias:14.113`, in the release named by
`citation_target` in the manifest (tag, commit and file hashes).
`reverse/<work>.json` maps each locus of that work to the ids of the entries
citing it, in printed order. When the corpus changes a locus, the next
lexicon build relinks it; nothing else needs to change.

The build report also checks every reference that supports quoted words.
Words are compared without diacritics; words of four letters or more may
differ by elision, movable nu or one letter. Half the words must be on the
cited line or the next one either side. A quotation that runs on may be
found up to three lines away, by form only, with at least one word of
five letters, and all of its words if it has one or two. With the
published lemma annotations, function words (articles, pronouns,
particles, prepositions, conjunctions) are not counted, and a content word
may also match by lemma on the cited line or the next one either side,
since Cunliffe often names the word in a dictionary form; those matches
are listed separately in the report for review. Letters in parentheses
inside a word (`ὅσ(σ)ος`) are kept; other text in parentheses and
brackets (glosses, supplied words) and form lists (`ἄγαγον, -ες`) are not
compared, except that Greek words in parentheses are tried as an
alternative reading (`ἦ τοι (ἤτοι)`). The misses are listed for review; nothing is changed
automatically. Moving every quoted reference to the same line of another
book, as a misread book letter would, the check still reports 97% of them.

## Page images

`scans` in the manifest names each scan the text was read from or can be
checked against: `url`, `viewer_url` and `image_url` templates (`{leaf}`
is the scan's own image number, `{n}` its position in archive.org's
viewer), `pages` (which printed pages it has, and the leaf offset of each
range), `access` and `credit`. `access` is `public` (anyone can see the
image), `us-only` (HathiTrust full view in the United States only: link,
do not embed) or `restricted` (a lending copy: neither link nor embed). An
archive.org scan has `leaf_files`: its scandata and page-number files,
pinned by SHA-256 and kept gzip-compressed in the lexicon's `scans/`
directory. The build checks every leaf against them: the viewer must show
it, and archive.org's label of the leaf must be the printed page. A label
never decides a leaf, since labels are sometimes wrong (the Toronto LSJ
labels leaves 876-877 as pp. 854-855, which repeat pp. 852-853), and the
viewer position differs from the leaf where the viewer skips leaves.

`pages.json` has one row per printed page and column (one per page in a
lexicon without columns), in page order, one row per line of the file:

```json
{"vol": null, "page": 1, "col": "a", "scan": "greekenglishlex00lidduoft", "leaf": 23,
 "viewer_url": "https://archive.org/details/greekenglishlex00lidduoft/page/n22/mode/1up",
 "image_url": "https://archive.org/download/...", "access": "public",
 "credit": "University of Toronto copy, digitized by the Internet Archive (not in copyright)",
 "fallback": [], "first_seq": 1, "last_seq": 2, "max_line": 78}
```

The scan is the one the page was read from, unless the lexicon's
`page_images.link` sends a reader to another copy of the same page (LSJ8:
pages read from the lending-only American Book Company issue link to the
Toronto leaf). `viewer_url` is null for a restricted scan and `image_url`
for any scan that is not public. `fallback` lists the other scans with the
page that can be linked, public ones first. `first_seq` and `last_seq` are
the first and last entry (by `seq`) that start or continue on that page
and column, null where none does; `max_line` is the last column line an
entry has there, for an OCR'd lexicon with line numbers. `note` explains a
page whose image is missing, partial or from another copy.

## Retired ids

`aliases.json` maps an id that is no longer used to the ids that replace
it:

```json
{"lsj8-1901:1770-a-55": {"successors": ["lsj8-1901:1770-a-10"],
                         "reason": "Read as an entry; the line belongs to ὡς.",
                         "retired_in": "lsj8-2026-11-23"}}
```

`successors` is empty when the entry was removed (a running head).
`retired_in` is the tag of the first release without the id: a dataset
release of the lexicon, which is a tag of its public repository (the
lexicon's id without the year, then the date: `cunliffe-2027-01-02`,
`lsj8-2027-01-02`), or a lexica serving release (`lexica-2027-01-02`) for
an id that no dataset release had. A retired id is never used again, and
the build fails when one is or when a successor is not an id of the
lexicon. A site resolves a link to a retired id through this file.

## Lemma lookup

`lookup/lemmas.json` maps every lemma of the published annotations that
reaches the lexicon to entry ids, built by
`scripts/build_lexicon_lemma_lookup.py` over every sidecar of a pinned
annotation revision. Several ids mean homographs or alternative entries;
show them all, in the order of the table, whose first id is the lemma's
main entry (*Order of a lemma's entries*, below). `methods` gives, for each
lemma, the stage that matched it, and `absent` lists the lemmas pinned as
absent from the lexicon. The file records the annotation revision (with a
digest of the sidecars it read), the Dilemma version, the hashes of the
convention files and pins it used, and the order rule (`entry_order`). The
build refuses a lookup in another format version.

The stages run in order, and the first that finds an entry decides. A
lemma goes to its own entry when the lexicon prints one, so its key,
alternative keys and place in LSJ9 come before the convention, which only
redirects lemmas these miss (Dilemma's `lsj` convention files ὁ under ὅς
and γίγνομαι under the cross-reference γίνομαι, though LSJ prints both
under their own heads):

1. `pinned`: `data/lexica/<lexicon>/lemma_pins.json`
   (`lexicon-lemma-pins.schema.json`) sends a lemma to entries chosen by
   hand, or to the entries another lemma reaches (`as`), each with its
   reason, or marks it absent, and then nothing else is tried. Pins only
   route the lookup; they change no entry. The ids of a pin keep the order
   it gives them, main entry first. Cunliffe's τέμνω goes to the
   τάμνω entry and the cross-reference "τέμνω. See τάμνω."; LSJ8's τίς
   goes to the entry on τις, which treats the interrogative; Πατήρ, which
   the tagger takes for a name, goes as πατήρ. A lemma pinned `as` one that
   reaches no entry stops the build.
2. `key`: the lemma's key is an entry's key.
3. `alt_key`: the lemma's key is one of an entry's alternative keys.
4. `lsj9-align` (LSJ8): the lemma is an LSJ9 headword (the Perseus
   encoding the LSJ8 sources pin). LSJ9 headwords whose key is the key of
   exactly one entry, in an order that agrees with print, are anchors; the
   candidates are the entries printed between the anchors around the
   lemma's place in LSJ9 that have its letters.
5. `convention`: Dilemma's lemma convention for the lexicon (`cunliffe`,
   `lsj`) gives another form; that form's pin, then its key, then its
   alternative keys decide. Then `lsj9-align` is tried on that form.
6. `letters`: the entries with the letters of the lemma, then of its
   convention form, marks, case, hyphens and OCR lookalikes ignored.

The letter stages (`lsj9-align` and `letters`) compare letters only, so a
match is kept only when the candidates are one entry or one homograph
group, and refused when their key is itself an LSJ headword: the lemma,
with other marks, is then another word (ὅρος is not ὄρος). They never
reach an affix from a word, and never take a numeral: a form with a
numeral sign (ʹ, ͵, or ΄ written for one) or a numeral letter (ϛ, ϟ, ϡ), or
one whose letters have no accent or breathing, stand in the order of a
numeral and carry another sign (ιη', κα’, ι̅): ιηʹ is 18, not ἴῃ. Keys are
lower case, so a capitalized lemma tagged mostly as a proper noun across
the corpus reaches only entries whose headword is capitalized, at every
stage but the pins: Δῖος does not reach δῖος. A lemma with a small letter
is never taken for a name.

### Order of a lemma's entries

The stages find a lemma's entries; the rule `open-greek-lexica-entry-order/2`
(in `build_lexicon_lemma_lookup.py`) puts them in order, so that the first
is the entry a reader looking up the word wants: the article on the word,
not a note on one of its forms. It reads only the entries.

A **pointer** is an entry that only sends the reader to another: an entry
of the kind `cross-reference` (LSJ8's "ἀγεννία, v. sub ἀγέννεια."), or one
that names another entry and has no sense, no citation and no gloss. It
names the entry through a cross-reference with a target (Cunliffe's "πολύς.
See πολλός." and "οἶδα, pf. εἴδω (C)."; LSJ8's Addenda notes on the entry
they amend), or, when the build resolved none of its references and it
prints at most 80 code points (a printed line is about 67), in words:

- whatever else the note prints, "v. sub X" (LSJ8's "ἔπομαι, to follow,
  Dep.: v. sub ἔπω."), but not a source's "s. v. X", and "X, q. v."
  ("εἰώθα, pf. 2 (in pres. signf.) of ἔθω, q. v."; a gender before "q. v."
  is passed over: "fem. Θρῆσσα, ἡ, q. v." names Θρῆσσα);
- where the note glosses nothing before them, the parse or dialect note of
  a form, with an abbreviation, a number or a bracket before "of" and an
  abbreviation or nothing before "for" ("εἰσί, εἰσίν, 3 pl. of εἰμί
  (sum).", "ὅπερ, Ep. for ὅσπερ, Il. 7. 114.", "μάλιστα, Adv., Sup. of
  μάλα", "γενή, ἡ, poët. for γενεά"), and "v. X" ("εὐθεῖα, ἡ, v. εὐθύς."),
  but not a source's sub voce ("Suid. v. Σωσίβιος"). A gloss after the word
  is the word's ("θῆϊον, τό, Ep. for θεῖον, brimstone"), except in the note
  of an adverb or a feminine, a word of its own, which glosses itself
  ("δεόντως, Adv. of δέον, as it ought"; "δεσπότειρα, fem. of δεσπότης, a
  mistress"). After a gloss of the note's own these are "see also" or a
  note on the reading ("δί-μετρος, ον, of a verse, having two metres,
  Hephaest. ; v. διποδία."; "ἐξιδίω, ..., to exsude: in Ar. Av. 791 euphem.
  for τιλάω.").

"son of" and "used for" name nothing, nor does a note on a phrase
("τοῦναντίον, crasis for τὸ ἐναντίον", "ἐπιπολύ, Adv. for ἐπὶ πολύ"), one
that derives another word ("Dim. of ζωμός", "strengthd. for πτήσσω", "verb.
Adj. of ἀρέσκω"), a prefix ("v. sub μεσ-"), a word in brackets ("εὐβωλος,
ον, (βῶλον) fertile (v. sub εὐπωλος)."), a word the OCR mixes with Latin
letters ("v. sub δopι-"), a word with the note's own key ("καθίζω, Dor.
aor. I subjunct. of καθίζω"), or, in a form's note, a bare letter ("ξέ,
poët. for ξ, him", where the OCR prints ξ for ἕ; "γάμμα, τό, indecl., the
letter γ, q. v." names the letter). A word runs on through the combining
marks the OCR prints for some accents ("βῶκος, ὁ, Dor. for βου̂κος"). A
cross-reference whose references the build left unresolved names its entry
in words too.

1. The entries that are not pointers come first. The lead is one whose
   headword has the lemma's case, a capital or a small letter, when any
   has (LSJ8's ῥέα, easily, leads before the longer Ῥέα, and κῦρος,
   authority, before Κῦρος; the lemma Κῦρος leads with Cyrus). Of those
   the longest leads, unless one printed before it is at least two thirds
   of its length (a lead margin of 1.5, in code points of the entry's
   text): then the first such entry in print leads. A dialect-form or
   crasis note is a line and the article on the word a page, so LSJ8's καί
   leads with the conjunction, not with "καί, crasis for καὶ αἱ"; and
   printed order, which is the order of the homograph numbers where the
   lexicon prints them, decides between entries of comparable length
   (Cunliffe's ἅλς, the sea, before ἅλς, salt). The other entries follow in
   printed order.
2. Each pointer follows the entry it names, directly or through a pointer
   it names, when that entry is one of the lemma's; the other pointers come
   last, in printed order.
3. When every entry of a lemma is a pointer, the entries they name are
   added and lead by step 1 (they are *followed*): Cunliffe's ἐνέπω leads
   with ἐννέπω, then "ἐνέπω. See ἐννέπω."; πολύς with πολλός. So are the
   entries named by the pointers with the lemma's case, a capital or a
   small letter, when every entry with that case is a pointer: LSJ8's κρῆς
   ("κρῆς, Dor. for κρέας") leads with κρέας, not with Κρῆς, a Cretan.
   Pointers to pointers are followed up to three steps. A pointer that
   prints one form for each of its targets ("ἆθλος, ἀθλοφόρος. See ἄεθλος,
   ἀεθλοφόρος.") names, for a lemma that is one of its forms, only that
   form's target.

The entry a pointer names is read from the word it prints as well as from
the cross-reference's target, which the build takes as the first entry
with the word's key, whatever homograph the pointer prints. Of the entries
with the printed key, the pointer names the one with the homograph mark it
prints: a letter after the word (LSJ8's "χράομαι, v. sub χράω C." names
χράω (C), to use, not χράω (A), to graze) or a superscript number in it
(Cunliffe's "εἴκω¹"), matched with the number the entry records or the
letter in brackets in its head. Without a mark it names the build's
target, except a pointer when the key has an entry that is not one
("δύνω, v. sub δύω." names the verb, not "δύω, v. sub δύο.") or an entry
with another key (Cunliffe's "ἐξ. See ἐκ." names the preposition ἐκ, not
the prefix ἐκ-); then it names the key's main entry by step 1. A target
the build reaches through a pointer with the printed key stands (ἐξ- names
ἐκ- through "ἐξ-. See ἐκ-.").

A word named in words names the entries with its key (a compound's hyphen
left out), else with it as an alternative key, else the one entry with its
letters (the OCR often drops a mark: "εἰώθα, ... of ἔθω, q. v." names the
entry headed ἑθω). Of these it names the one with the homograph mark it
prints, in brackets or as a bare letter that does not begin a section
number ("v. sub εἰ A. VI. 4. c." names section A of εἰ, not a homograph
(A)). Without a mark it names, of those that are not pointers (with the
word's case, if any has it, and passing over notes of a line, if a longer
one is left), the one, or the longest when it is at least three times as
long as the next (the word margin): "θεῖον, τό, the divinity, v. sub θεῖος
II." names θεῖος, divine (2,913 code points), not θεῖος, uncle (448).
Otherwise it names none: "v. sub εἴρω to say" does not say which of εἴρω
(A), to fasten, and εἴρω (B), to say, of a length, it means. A word that
names no entry but the pointer itself makes no pointer. So LSJ8's ἕπομαι
leads with ἔπω, καθάπερ with καθά, εὐθεῖα with εὐθύς and ἔ with the
reflexive οὗ (headed οὖ), then the note.

A pin keeps the order it gives, for the cases the rule cannot see. The rule
reads only the entries, not the corpus, so it cannot tell which of two
homographs the corpus means when the rarer one is the longer (LSJ8's
πρόσειμι and μέτειμι, whose tokens are mostly forms of the verbs with εἰμί;
Cunliffe's μήν, the particle, ἕλιξ, the noun, and σύνειμι, to be with), nor
that a lemma is another word than the form a form entry parses (Cunliffe's
"ἔστε, 2 pl. imp. εἰμί." for the conjunction ἔστε, which the rule would
follow to εἰμί). The pins also cover a main entry the parse does not link
(LSJ8 treats οἶδα under *εἴδω, and its "οἶδα, Aeol. οὔδα Alcae. 141, v. sub
*εἶδω B." is not read as a cross-reference by the build; LSJ8's οὐκ is
treated under οὐ), an entry that defines without a marked gloss (Cunliffe's
"ἆ. Ah! ..."), and a misread headword or a stray line of another article
among the entries (LSJ8's ὄρος, ἐμφάνεια and οἱ). Each pin gives its
reason. A lemma pinned `as` another takes the rule, unless the other lemma
is pinned. Reading pointers in words adds pins where the note's word
reaches the wrong entry: LSJ8's μήτι (the build keys the head of μήτις
"μήτις ορ μή τις", so the note reaches only μῆτις, counsel), κέαρ (the OCR
drops the mark of the word the note prints), σῦς (the article on ὗς is not
split out) and Θουκυδίδης (its note names the entry on Doric θευ-); where
the key reaches only a note because the article's head is misread (ἀνάσσω,
ἄδεια, and the conjunction ἤν, headed ἦν); where the lemma is another word
than the entry with its key (ὕστερον, the adverb, under ὕστερος; κάν, the
corpus's κἄν, under κἂν); and where
the corpus means the word a pointer names but the rule puts another
homograph first, since it follows a pointer only when every entry is one
(εἰλέω, to roll up, under εἶλω; οὐρία, the fair wind, under οὔριος), or the
word margin names none of a word's homographs (πόμα, the drink πῶμα, not
the lid). LSJ8's ἔσκε is pinned to εἰμί, which its note prints εἶμί. LSJ8's
εὀ is pinned to the reflexive οὗ, whose word its note prints without marks.

`lemmas.json` records the rule as `entry_order`: `rule`, `lead_margin`,
`follow_depth`, `pointer_span`, `word_margin`, the lemmas whose order a pin
gives (`pinned`), and the entries the rule added for each lemma by
following its pointers (`followed`). A release refuses a table that records
another rule, or whose order is not the one the rule gives.

`lookup/coverage.json` reports, over tokens whose lemma has a Greek letter:
the share of tokens matched, by method, with and without proper nouns, and
how much of it is exact (pins, convention and keys) rather than by letters;
the same for the top 100, 1,000 and 5,000 lemmas and for the Homeric poems;
the letter matches refused and why; the lemmas whose entries carry review
flags; the letter matches among the top 5,000; the 500 most frequent
unmatched lemmas, as the worklist for reading the scans; and the effect of
the entry order (`entry_order`): over all lemmas and the top 100, 1,000 and
5,000, the lemmas with several entries, those whose lead is not their
first entry in print, those led by a followed entry or a pointer and those
a pin orders (each with its tokens), and every lead change in the top
1,000.

## Fitting the other lexica

- **LSJ (8th edition, OCR)**: `loc` has page and column (`col: "a"`/`"b"`).
  Ids are the position of the entry's first line, `<page>-<col>-<line>`,
  and stay so; where the entry matches an entry of the Perseus LSJ9
  encoding, `source.perseus_id` has its id. The match follows the key
  order of both lexica: the longest run of entries in the same order whose
  keys match, then, between two matched entries, the one LSJ9 entry
  between their matches with the same key or letters. A printed homograph
  number picks the LSJ9 homograph with that number; an entry gets no id
  when another LSJ9 homograph with its key is free in the same place, nor
  in the Addenda. The manifest's `perseus_ids` records the pinned LSJ9
  files, the number of entries with an id and the count of each method;
  it is null for a build made without the LSJ9 files, which has no
  `perseus_id` and is not released. An entry whose key and alternative
  keys all differ from the key of its matched LSJ9 entry (hyphens and
  spaces aside) has the review code `lsj9-key`, so `quality` `ocr`.
  Citations of non-Homeric authors will carry a CTS URN and an address
  when the corpus has the work, and `address: null` otherwise.
- **Greenfield**: citations are New Testament book, chapter and verse; the
  URN uses the corpus's NT edition and the address its NT works.
- **Stephanus (Didot)**: `loc` has `vol` and `col` (columns are numbered
  across the volume, pages are not); `page_breaks` holds column breaks.
  Latin is marked with `latin` spans.
