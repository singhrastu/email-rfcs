# email-rfcs

Every current email RFC, indexed with the three things the documents themselves
do not tell you: whether you are reading the right one, what has changed it since
it was published, and what it actually requires.

**99 documents. 3,640 normative requirements. 77 replaced numbers that still
resolve.**

Browse it: **[rastu.tech/rfc](https://rastu.tech/rfc/)**

## Why this exists

Reading an RFC is not the hard part. Working out which one to read is. Three
things trip people up and none of them are visible on the page you land on.

**It has been replaced.** Search for the SMTP RFC and you get RFC 821, which is
still labelled INTERNET STANDARD while the document that replaced it twice over,
RFC 5321, is labelled DRAFT STANDARD. The status field actively misleads.

**It is current and amended.** RFC 5321 is the right document and RFC 7504
changed part of it. Neither page says so. Twenty-eight of the ninety-nine
documents here are in that state.

**What binds you is a few dozen sentences.** An implementation has to satisfy the
RFC 2119 requirements, not the ninety pages around them. Those are extracted here
with the section each came from.

### A worked example

DMARC left RFC 7489 in May 2026 for RFC 9989, 9990 and 9991, and moved from
Informational to the standards track. Nothing announced it. Plenty of
documentation and tooling still cites 7489.

```python
import json
d = json.load(open("rfcs.json"))
d["aliases"]["7489"]["now"]     # ['RFC9989', 'RFC9990', 'RFC9991']
```

One document replaced by three. A resolver that follows only the first successor
silently loses two thirds of DMARC, so this one searches rather than walks.

## The data

`rfcs.json` is the index. `rfc/<number>.json` holds that document's requirements.
They are split because the requirements are a megabyte and most callers want the
index.

```json
{
  "num": 5321, "id": "RFC5321",
  "title": "Simple Mail Transfer Protocol",
  "status": "DRAFT STANDARD", "published": "October 2008",
  "category": "Transport",
  "updates": [], "updated_by": ["RFC7504"],
  "replaces": ["RFC821", "RFC2821"],
  "rel_titles": { "RFC7504": "SMTP 521 and 556 Reply Codes" },
  "req_counts": { "must": 177, "should": 128, "may": 55 },
  "errata": "https://www.rfc-editor.org/errata/rfc5321",
  "doi": "10.17487/RFC5321",
  "note": { "what": "...", "problem": "...", "operate": "..." }
}
```

```json
{
  "num": 7208, "uses_2119": true,
  "requirements": [
    { "section": "3.1", "heading": "DNS Resource Records",
      "level": "must", "keyword": "MUST",
      "text": "SPF records MUST be published as a DNS TXT (type 16) Resource Record (RR) [RFC1035] only." }
  ]
}
```

### The honest parts

**Only uppercase keywords count.** A lowercase "must" is prose and binds nobody.
Treating the two alike is how a requirements list stops being trustworthy.

**`uses_2119` says whether the document adopts the convention at all.** RFC 6152
is Standards Track and contains no uppercase keyword anywhere, because it is a
2011 republication of a 1994 document that never took up RFC 2119. An empty list
there is a fact about the document. An empty list for something that does adopt
2119 would be a parser failure, and the two must not look alike. Nothing in this
set is in that second state, which is the check that says the extraction is not
silently failing.

**Obsolete documents are not entries.** They are aliases, so a search for `821`
lands on `5321` rather than on nothing. HISTORIC is excluded too, that being the
IETF's own word for no longer recommended.

**67 of 99 carry a written explanation**: what the document is, what it solves,
and what it means to operate. The rest render their index data alone. Inventing
commentary so a page looks finished would make the whole thing worth less.

## It keeps itself current

The seed list is topic anchors, not documents. Every build follows the obsoletion
edges forward, so a revision arrives without anyone editing a file: seeding 7489
yields 9989, 9990 and 9991 on its own. A weekly workflow rebuilds from the RFC
Editor's index and commits what moved, with a diff naming it.

```bash
python3 export_rfcs.py --fetch-texts   # rebuild from the published sources
python3 rfc_diff.py old.json new.json  # what changed between two builds
```

Anything that amends a member is pulled in automatically. A keyword sweep of the
full 9,800-document index produces a review queue in `rfc-candidates.json` rather
than publishing itself, because it drags in routing and process RFCs that merely
mention mail.

## Sources

- [The RFC Editor index](https://www.rfc-editor.org/rfc-index.xml) for status,
  obsoletion, amendments, abstracts, errata and DOIs
- The RFC texts themselves, for the requirements

## Licence

Code MIT. The generated dataset is CC BY 4.0: reuse it, and a citation is
appreciated. RFC text quoted in the requirements belongs to the IETF Trust.

Maintained by [Rastu Singh](https://rastu.tech/about/), who runs email
infrastructure for a living.
