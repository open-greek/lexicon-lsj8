# Liddell and Scott, A Greek-English Lexicon, 8th edition (1901 printing)

> **Uncorrected OCR.** The entries are text read by machine from scans of
> the printed book. The headwords have been checked and many of them
> corrected, but the rest of each entry has not been proofread. Check
> anything that matters against the page image, which `pages.json` links
> for every page and column.

Henry George Liddell and Robert Scott, *A Greek-English Lexicon*, eighth
edition, revised throughout (Oxford: Clarendon Press, 1897), in the
printing of 1901: the lexicon, pages 1 to 1774, and the Addenda, pages
1774 to 1776, as structured data. Every entry is a JSON object with its
text, its sense labels, its cross-references and the page, column and line
where it is printed. This is release `lsj8-2026-11-22.2`, with 99,837
entries: 99,545 from the lexicon and 292
from the Addenda.

Not included: the preface, the list of authors and abbreviations and the
other front matter.

## The printed book and the scans

The text is read from the University of Toronto copy of the 1901 printing,
digitized by the Internet Archive as
[`greekenglishlex00lidduoft`](https://archive.org/details/greekenglishlex00lidduoft)
(300 ppi). Other copies supply the pages it lacks or cannot give:

- ten pages (549, 563, 569, 651, 690, 747, 854, 855, 1627 and 1683) come
  from an undated American Book Company issue printed from the same setting
  of type, in the Internet Archive's lending library
  ([`greekenglishlexi0000henr`](https://archive.org/details/greekenglishlexi0000henr)).
  It has no public image, so those pages link to the Toronto leaf of the
  same page where the Toronto copy has one;
- pages 1383-1384 and 1775-1776 come from two University of Illinois copies
  of the 1901 printing, digitized by Google and kept by HathiTrust
  (`uiug.30112002427166` and `uiug.30112004106255`). HathiTrust shows them
  in full in the United States only, so they are linked, not embedded;
- pages 854-855 have no public image: the Toronto copy lacks them, and the
  Illinois copy that has them is linked instead, for readers in the United
  States;
- the Toronto leaf of page 1770 cuts off the left edge of column a, which
  is read from the American Book Company issue.

`pages.json` gives, for every page and column, the scan and leaf, a link to
the leaf in the provider's viewer, a link to the image where it is public,
whether the image is public, United States only or restricted, and the
credit line to show with it. The Toronto leaves are checked against the
item's own leaf list and page labels (in `scans/`, as the Internet Archive
serves them).

## How the text was made

1. Each column of each page was read line by line by a vision-language
   model. The lines keep their place: every entry records the page, column
   and lines it was read from.
2. The lines were split into entries. In the print an entry's first line is
   set in and its other lines are flush, so the indent of each line was
   measured on the scan, and an entry starts at a line set in that begins
   with a headword. Where a line could not be measured, the text and the
   alphabetical order decide. Where the scans show the split wrong (an
   entry run on inside the one before, or a quotation read as an entry of
   its own), a record in `corrections.json` sets or removes the start at
   that line, and an entry id that goes is kept in `aliases.json`. Running
   heads, page numbers, the printer's signatures and the letter headings
   are left out.
3. The headwords were checked: one accent on a word of two or more
   syllables, a breathing on the first vowel and none inside the word,
   Greek letters only, alphabetical order, and agreement with the same
   entry in the ninth edition (LSJ9, in the Perseus Digital Library's
   encoding). A headword that failed was read again, from both the Toronto
   copy and the American Book Company issue. Where the readings agree, the
   headword was corrected; where they agree on the letters only, the
   breathing, accent and long and short marks were taken from LSJ9. A few
   corrections fix whole lines: a faint column on page 562, the cut-off
   column on page 1770 and a running head read inside a column on page
   1422. Then the headwords that the corpus's lemmas reach most often were
   read again on the scans of both copies, and corrected where they differ
   from the print. There are 16,857 corrections, all in
   `corrections.json`.
   Each one that changes an entry's text or references is marked at that place in the entry, with the reading before it and the reason. The 11 that change no entry's text (11 entry starts set or removed: c16833-c16843) have no mark.
4. Each entry of the lexicon is matched to its entry in the Perseus
   encoding of LSJ9 where one can be found in the same alphabetical place
   (83,221 entries), and has its id as `source.perseus_id`,
   so that the two editions can be compared entry by entry.

The OCR text itself is not otherwise changed. Lines are joined, words
hyphenated at a line end are joined, and the sense labels (`A.`, `II.`,
`3.`, `b.`) are found in the text and nested, but citations of authors
are not yet parsed or linked.

## Quality

The text of every entry is OCR. `quality` says how far the headword was
checked:

- `ocr-headword-checked` (87,968 entries):
  the headword and the start of the entry pass the checks above;
- `ocr` (11,869 entries): something makes the entry
  doubtful, and `source.review` says what:
  - `headword` (2,222): the headword cannot be right as
    read (two accents, a missing or misplaced breathing, Latin or Cyrillic
    letters);
  - `no-accent` (1,834): a word of two or more
    syllables without an accent;
  - `lsj9-key` (7,944): its breathing, accent or
    another mark differs from its LSJ9 entry. Many of these are misread,
    but some words are printed otherwise in the two editions;
  - `out-of-order` (1,808): the start breaks the
    alphabetical order, mostly because its headword is misread;
  - `indent` (34): a start at a line the scan shows
    flush.

The checks catch misread headwords; they do not prove a headword right.
Blind samples of the corrected headwords were checked on the scans. Of 178
headwords settled with LSJ9, none had a wrong letter or breathing, and
about one in ten differs from the print, nearly always in a long or short
mark (most often a short mark LSJ9 adds that the 1901 print leaves out). Of 145 headwords settled
by the second reading, 136 match the print exactly, and none has a wrong
letter, breathing or accent.

## Known problems

- The text of the entries is uncorrected OCR: letters, accents and
  breathings are misread, and an OCR slip can hide a sense label or add one
  (`I. IO` for `l. 10`). A few lines still have letters of other scripts
  (Arabic, Hebrew) where the OCR misread Greek.
- The 2,222 headwords flagged `headword` are not
  corrected, and the `no-accent` and `lsj9-key` entries have not been
  checked on the scans yet, apart from those the corpus's most frequent
  lemmas reach.
- A headword settled with LSJ9 follows LSJ9 where the 1901 print differs
  from it, mostly in long and short marks.
- Where the indent of a line could not be measured (about one line in
  twenty), or was measured wrong, an entry can start in the wrong place or
  a derivative can be run into the entry before it.
- 149 cross-references do not find their entry,
  among them a few amendments in the Addenda whose headword is read
  differently.
- Citations are not parsed: `citations` is empty in every entry.

Please report a misreading as an issue on this repository, with the
entry's id. Corrections are made as new records in `corrections.json`.

## Files

The format is open-greek-lexica 0.3.0, described in
[`format.md`](format.md), with JSON Schemas in `schema/`.

| file | content |
|---|---|
| `manifest.json` | title, edition, licence, attribution, the pinned sources, the scans, counts, and every file with its size and SHA-256 |
| `entries/0001.jsonl` to `entries/0200.jsonl` | the entries in printed order, one JSON object per line, 500 per file |
| `keys.json` | lookup key -> entry ids, for headword search |
| `pages.json` | one row per printed page and column: scan, leaf, links, access, credit and the entries on it |
| `corrections.json` | every correction, in the order applied, with its inverse |
| `aliases.json` | retired entry ids -> the ids that replace them |
| `scans/` | the Toronto copy's leaf list and page labels, as the Internet Archive serves them |
| `vectors/` | test vectors for the lookup key, the letters-only fold, the Latin transliteration and the corpus reader's words |

An entry's id is `lsj8-1901:<page>-<col>-<line>`, the place of its first
line: the printed page, the column (`a` left, `b` right) and the line in
that column (`lsj8-1901:2-a-3` for ἄβα). These ids are permanent. An id
that has to go, when entries are split or merged, is kept in
`aliases.json` with the ids that replace it.

`source.lines` gives each entry's lines: page, column and line numbers,
the scan and leaf they were read from and the strip of the leaf the OCR
read. The OCR's base text and its strip images are not published; the
manifest identifies the base text by its SHA-256.

`corrections.json` keeps, for each record, the reading before and after, a
one-line reason, the page, column, line and offset it changes, the scan
and leaf where it can be seen, and the inverse change. The working
evidence behind each record (the individual readings compared) is not part
of this repository.

## How to cite

Cite the release you used, by its tag:

> Henry George Liddell and Robert Scott, A Greek-English Lexicon, 8th
> edition (1901 printing), Open Greek OCR edition, release `lsj8-2026-11-22.2`
> (2026-11-22), https://github.com/open-greek/lexicon-lsj8.

`CITATION.cff` has the same in machine-readable form. Cite an entry by its
id and the release.

## Licence and attribution

The 1901 printing is in the public domain. The OCR text, its corrections
and the entry structure are released under CC BY-SA 4.0: see `LICENSE`.
Credit them as follows:

> Henry George Liddell and Robert Scott, A Greek-English Lexicon, eighth edition (Oxford: Clarendon Press, 1897; printing of 1901). OCR from the University of Toronto copy (Internet Archive), with pages from an undated American Book Company issue and the University of Illinois copies of the 1901 printing (HathiTrust), by the Open Greek project, CC BY-SA 4.0. Some headword breathings, accents, breves, macrons and letters are taken from Henry George Liddell, Robert Scott, Henry Stuart Jones and Roderick McKenzie, A Greek-English Lexicon, ninth edition (Oxford: Clarendon Press, 1940), as encoded by the Perseus Digital Library, Tufts University (http://www.perseus.tufts.edu/; https://github.com/PerseusDL/lexica), CC BY-SA 4.0.

Some corrected headwords take their breathing, accent, long and short
marks, and a few their letters, from Henry George Liddell, Robert Scott,
Henry Stuart Jones and Roderick McKenzie, *A Greek-English Lexicon*, ninth
edition (Oxford: Clarendon Press, 1940), as encoded by the Perseus Digital
Library, Gregory Crane, Editor-in-Chief, Tufts University
(http://www.perseus.tufts.edu/), prepared under the supervision of Lisa
Cerrato and David Smith with funding from the National Endowment for the
Humanities: the files `grc.lsj.perseus-eng1.xml` to
`grc.lsj.perseus-eng27.xml` in `CTS_XML_TEI/perseus/pdllex/grc/lsj/` of
https://github.com/PerseusDL/lexica at commit `b1f73c08`,
released under CC BY-SA 4.0. The `source.perseus_id` of an entry is the id
of its entry in that encoding. Apart from those headword forms, nothing of
LSJ9 is included.

The page images are the University of Toronto copy, digitized by the
Internet Archive (not in copyright); the American Book Company issue in the
Internet Archive's lending library; and the University of Illinois copies,
digitized by Google (HathiTrust).
