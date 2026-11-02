# Near-duplicate and fuzzy-similarity tools for digital archival materials

Research date: 2026-11-01. Scope: similarity hashing / near-duplicate techniques only (exact-match hashing covered elsewhere). Audience: digital archivists building an appraisal-stage, report-only dedup tool. Release dates and activity figures below were pulled from GitHub's API (`pushed_at`, latest release) and PyPI on 2026-11-01 unless otherwise cited.

---

## Q1. Fuzzy hashing of arbitrary files: ssdeep, sdhash, TLSH, Nilsimsa, mrsh-v2 — comparison, false positives, where they ship

### Takeaway
Of the bytewise "approximate matching" tools, TLSH is the only one with active 2026 releases and a maintained Python binding; ssdeep is stable but frozen (last release 2017), and sdhash is effectively dormant (v4.0, Dec 2013). Independent forensic evaluations consistently rank sdhash and TLSH above ssdeep in recall, with ssdeep notably weak on MS Office files and unable to compare files of very different sizes; Nilsimsa has the highest false-positive rate of the group. BitCurator ships ssdeep (listed under Forensics and Reporting) and documents sdhash for "file similarity."

### Cited Findings

**ssdeep (CTPH)**
- ssdeep is the reference implementation of Kornblum's context-triggered piecewise hashing (CTPH), citing "Identifying almost identical files using context triggered piecewise hashing" (Digital Investigation, 2006); the C API (`fuzzy.h`, `-lfuzzy`) writes a hash into a `FUZZY_MAX_RESULT` buffer and `fuzzy_compare` returns a match score 0–100 where zero means no match; license GPL-2.0 — [ssdeep-project/ssdeep README](https://github.com/ssdeep-project/ssdeep)
- Latest release tag `release-2.14.1`, published 2017-11-07; repo last pushed 2026-08-13 (minor activity, no releases); ~800 stars; not archived — GitHub API query, 2026-11-01 ([repo](https://github.com/ssdeep-project/ssdeep))
- The PyPI `ssdeep` wrapper is at version 3.4, released 2019-10-03, LGPLv3+, bundles ssdeep 2.14.1 source, exposes `ssdeep.hash()` and `ssdeep.compare()` (0–100 integer); classifiers only list Python up to 3.7 — [PyPI ssdeep](https://pypi.org/project/ssdeep/)
- `ppdeep` is a pure-Python implementation of ssdeep/SpamSum-style CTPH; latest listed release is dated 2020 (version `20200505`) — [PyPI ppdeep](https://pypi.org/project/ppdeep/20200505/)
- A known ssdeep limitation: it is unable to compare files of very different sizes, since block size is derived from file length — [ssdeep man page](https://www.mankier.com/1/ssdeep)
- `ssdeeper` (DFRWS 2022) is a paper evaluating and improving ssdeep; the PDF could not be fetched (HTTP 403) — [DFRWS PDF link](https://dfrws.org/wp-content/uploads/2022/07/ssdeeper-Evaluating-and-improving-ssdeep-combined.pdf)

**sdhash (similarity digests / Bloom filters)**
- README reads "sdhash 4.0 by Vassil Roussev, Candice Quates [sdhash.org] 12/2013"; license Apache-2.0; compares data "based on common strings of binary data"; includes a SWIG Python binding (`swig/python`), plus `sdhash-gpu`, `sdhash-server`, `sdhash-ui`; `-t/--threshold` default 1 — [sdhash/sdhash](https://github.com/sdhash/sdhash)
- Latest release `v4.0` published 2013-12-09; repo last pushed 2020-10-16; 178 stars — GitHub API, 2026-11-01
- Roussev's 2011 DFRWS evaluation concluded "sdhash consistently outperforms ssdeep across all categories, with ssdeep struggling on some MS Office files" — [Roussev 2011, An evaluation of forensic similarity hashes (DFRWS)](https://dfrws.org/presentation/an-evaluation-of-forensic-similarity-hashes/) (quote via search summary; PDF at [roussev.net](http://roussev.net/pubs/2011-DFRWS--sdhash-vs-ssdeep.pdf) could not be fetched due to TLS error)

**TLSH (Trend Micro Locality Sensitive Hash)**
- Current version 5.0.2 (dated 10/07/2026 in README); default digest prefix changed to "T1" on 17/02/2026; py-tlsh 4.12.1 released 19/01/2026; the 5.0.x line fixed a reentrancy bug and added `TLSH_OPTION_THREADED4` — [trendmicro/tlsh README](https://github.com/trendmicro/tlsh)
- Algorithm: 5-byte sliding window producing six trigrams, Pearson hash into 128 buckets, quartile-based bucket encoding, 1-byte checksum; digest is 35 bytes rendered as "T1" + 70 hex chars (72 chars); minimum input 50 bytes with sufficient randomness (256 bytes with `-conservative`; 20,000 bytes for THREADED4); identical-byte input yields no hash — [trendmicro/tlsh README](https://github.com/trendmicro/tlsh)
- Distance semantics: score 0 = almost identical, higher = more different; a "global similarity" component penalizes size differences and can be disabled; Python `diffxlen` excludes file length — [trendmicro/tlsh README](https://github.com/trendmicro/tlsh)
- TLSH is adopted by VirusTotal and MalwareBazaar; README points to Table II of the TLSH paper for detection/false-positive rates and links a paper on randomization attacks against similarity digests — [trendmicro/tlsh README](https://github.com/trendmicro/tlsh)
- Original paper (Oliver, Cheng, Chen) claims TLSH "is more robust to the attacks and random changes considered" than prior schemes — vendor-authored — [TLSH whitepaper PDF](https://documents.trendmicro.com/assets/wp/wp-locality-sensitive-hash.pdf)
- py-tlsh on PyPI: 5.0.0 released 2026-05-08; license field "Apache or BSD"; API `tlsh.hash(bytes)`, `tlsh.diff`, `tlsh.diffxlen`, incremental `Tlsh()` with `update()/final()`, `conservativehash`, `oldhash`; 5.0.0 adds `toByteArray()` for Pynndescent compatibility — [PyPI py-tlsh](https://pypi.org/project/py-tlsh/)
- GitHub: last pushed 2026-07-10; latest GitHub release tag 4.12.1 (2026-01-21); 860 stars; license shows NOASSERTION (dual Apache/BSD text) — GitHub API, 2026-11-01
- A 2026 privacy-preserving blocklisting paper benchmarked TLSH, ssdeep and others; TLSH gave the fastest response time, ssdeep the lowest communication volume — [arXiv 2604.15641](https://arxiv.org/pdf/2604.15641)

**Nilsimsa**
- Anti-spam-focused LSH proposed by the cmeclax remailer operator in 2001, reviewed by Damiani et al. 2004; similarity scored 0–128 (128 = identical/very similar) — [Wikipedia: Nilsimsa Hash](https://en.wikipedia.com/wiki/Nilsimsa_Hash)
- Comparison work found "Nilsimsa has a significantly higher false positive rate when compared to other similarity digest schemes such as TLSH, Ssdeep and Sdhash" — [Wikipedia: Nilsimsa Hash](https://en.wikipedia.com/wiki/Nilsimsa_Hash) (citing the TLSH paper)
- A Python port `py-nilsimsa` exists; maintenance status not determined — [github.com/tsaost/nilsimsa](https://github.com/tsaost/nilsimsa) (search result; not fetched)

**mrsh-v2 and alternatives**
- Breitinger & Roussev's 2014 Digital Investigation paper ("Automated evaluation of approximate matching algorithms on real data", vol. 11, S10–S17) evaluates sdhash, mrsh-v2 and ssdeep against a longest-common-substring ground truth on the t5 corpus; its main contribution is "automated precision and recall tests on real world data" — [Augsburg OPUS record](https://opus.bibliothek.uni-augsburg.de/opus4/frontdoor/index/index/year/2024/docId/117624); [PDF](https://opus.bibliothek.uni-augsburg.de/opus4/files/117624/117624.pdf)
- Companion paper: Breitinger, Stivaktakis & Roussev, "Evaluating detection error trade-offs for bytewise approximate matching algorithms" (2014) uses DET curves to compare ssdeep and sdhash — [UNIL Serval record](https://serval.unil.ch/notice/serval:BIB_42AEB5C0F87F)
- NIST SP 800-168 "Approximate Matching: Definition and Terminology" (May 2014; Breitinger, Roussev, White, Guttman, McCarrin) is the standard vocabulary document for this class of tools — [NIST CSRC](https://csrc.nist.gov/publications/detail/sp/800-168/final)
- A later executable-file study comparing ssdeep, sdhash, TLSH and MRSHv2 found TLSH most effective when combined with labeled data (dataset-dependent) — [DFRWS EU 2019 presentation](https://dfrws.org/wp-content/uploads/2021/05/2019_EU_pres-towards_exact_and_inexact_approximate_matching_of_executable_binaries.pdf)
- One study concludes "tlsh and sdhash consistently outperform ssdeep and should be recommended in its place, with tlsh preferred for source-code changes" — [Pagani et al., "Beyond Precision and Recall" (CODASPY 2018)](https://www.eurecom.fr/en/publication/5419/download/sec-publi-5419.pdf)
- LZJD (Raff & Nicholas, 2017/2018, Digital Investigation DOI 10.1016/j.diin.2017.12.004) claims to "significantly outperform sdhash and ssdeep" on related fragments and noise-corrupted files and to be "up to 60x faster than sdhash at comparison time"; a Java implementation with sdhash-compatible CLI arguments is described — [arXiv 1708.03346](https://arxiv.org/abs/1708.03346)
- Martín-Pérez, Rodríguez & Breitinger (2021) classify approximate-matching algorithms and review attacks on them — [DFRWS record](https://dfrws.org/presentation/bringing-order-to-approximate-matching-classification-and-attacks-on-similarity-digest-algorithms/)
- A survey notes that "despite a high diversity of algorithms in existence, only SSDEEP and sdhash are widely used" — [academia.edu record, "A Comparison of Fuzzy Hashes: Evaluation, Guidelines, and Future Suggestions"](https://www.academia.edu/7833902) (authorship not confirmed from snippet)
- `fuzzyhashlib` wraps ssdeep, sdhash (sdbf) and tlsh behind a hashlib-style interface; last Python 2 release 0.0.9 and changelog says future versions "if any" will be Python 3 only — [PyPI fuzzyhashlib](https://pypi.org/project/fuzzyhashlib/)

**Where they ship (BitCurator)**
- BitCurator Walkthrough lists "SSDeep — Fuzzy hashing tool" under Forensics and Reporting, states "BitCurator includes Vassil Roussev's sdhash to report on file similarity," and lists FSlint under "Deduplicate Files: Identify and delete duplicate files"; menu layout described for BitCurator 5.x — [BitCurator Walkthrough](https://bitcurator.github.io/documentation/BitCurator%20Walkthrough/BitCurator%20Walkthrough)
- A 2024 PDT&C article states the BitCurator Environment "includes both ssdeep and sdhash, command-line tools that use fuzzy hashing algorithms to determine the degree of similarity between files" — [Karp, "The Interconnectedness of All Things" (PDT&C, DOI 10.1515/pdtc-2024-0042)](https://www.degruyterbrill.com/document/doi/10.1515/pdtc-2024-0042/html) (quote from search snippet; full text returned HTTP 403)
- Hurlbut (2009, AccessData) traces fuzzy hashing to Tridgell's spamsum and notes FTK has a fuzzy-hash utility — [Fuzzy Hashing for Digital Forensic Investigators](https://repo.zenk-security.com/Techniques%20d.attaques%20%20.%20Failles/Fuzzy%20Hashing%20for%20Digital%20Forensic%20Investigators.pdf)

### Inferences
- For a 2026 Python tool, TLSH (py-tlsh 5.0.0, May 2026) is the only bytewise fuzzy hash with current wheels and active upstream; ssdeep is usable via `ppdeep` (pure Python) or the system `ssdeep` binary, but the PyPI C wrapper (2019, Python ≤3.7 classifiers) is a build risk on modern Pythons.
- sdhash's Python binding is SWIG-based and the project has had no release since 2013; treat as "available in BitCurator, not a dependency to build on."
- ssdeep's fixed-length-ratio limitation makes it poor for "same document, different revision" where one draft grew substantially; TLSH's `diffxlen` explicitly addresses this.
- All forensic evaluations are on malware/binary or the t5 corpus, not archival collections; thresholds should be re-calibrated on archival test data.

### Gaps
- Could not retrieve per-algorithm precision/recall numbers from Breitinger & Roussev 2014 or Roussev 2011 (PDFs blocked by 403/TLS errors); the report writer should cite the papers qualitatively.
- Could not confirm that sdhash still builds/ships in the current BitCurator 5.x distribution from BitCurator's own package list; the Walkthrough and the 2024 PDT&C article say it is included.
- mrsh-v2 has no maintained public repository or Python package surfaced by search; status unknown.
- TLSH's license on GitHub registers as NOASSERTION (dual Apache 2.0 / BSD text); confirm before redistribution.

---

## Q2. Image near-duplicates: perceptual hashes (pHash, dHash, aHash, wHash, blockhash, PDQ) and Python libraries

### Takeaway
`imagehash` (BSD-2, v4.3.2 Feb 2025, actively pushed Oct 2026) is the standard Python toolkit for aHash/pHash/dHash/wHash/colorhash/crop-resistant hashing with Hamming-distance comparison; `imagededup` (Apache-2.0, v0.3.3.post2 Aug 2025) adds CNN embeddings, an evaluation framework and duplicate-finding API; Meta's PDQ (256-bit, with a quality score and a documented match threshold of ≤31) is available via `pdqhash` and has active 2025–2026 upstream.

### Cited Findings
- imagehash supports average, perceptual, difference, wavelet, HSV colour and crop-resistant hashing; hash size adjustable (`hash_size=8` default examples); subtraction of two hashes returns the Hamming distance (example: 33 for unrelated images); dependencies Pillow, numpy, scipy; BSD-2-Clause; fork of bunchesofdonald/photohash — [JohannesBuchner/imagehash](https://github.com/JohannesBuchner/imagehash)
- imagehash 4.3.2 released 2025-02-01 on PyPI; wheel tagged py2.py3 — [PyPI ImageHash](https://pypi.org/project/ImageHash/)
- imagehash GitHub: last pushed 2026-10-12; latest GitHub release tag v4.1.0 (2021) — releases are published on PyPI rather than GitHub; 3,884 stars — GitHub API, 2026-11-01
- imagededup (idealo) offers CNN, PHash, DHash, WHash, AHash; API `PHash().encode_images(image_dir=...)`, `find_duplicates(encoding_map=...)`, `plot_duplicates(...)`; includes an evaluation framework against ground-truth mappings; README says CNN "works best for near duplicates and datasets containing transformations" and DHash is fastest on exact duplicates; benchmarks valid only up to v0.2.2; maintainers Tanuj Jain, Christopher Lennan, Dat Tran; Apache-2.0 — [idealo/imagededup](https://github.com/idealo/imagededup)
- imagededup 0.3.3.post2 released 2025-08-15; requires Python ≥3.9 (classifiers 3.9–3.12) — [PyPI imagededup](https://pypi.org/project/imagededup/); GitHub last push 2025-08-15, 5,681 stars — GitHub API
- PDQ (facebook/ThreatExchange): recommended match threshold is Hamming distance ≤31; discard hashes with quality ≤49; implementations deemed correct when quality ≥80 hashes fall within distance 10 of the C++ reference; "As of November 2025 there are C++, PHP, Python, Java, and WASM implementations" plus third-party C# and Rust — [ThreatExchange/pdq](https://github.com/facebook/ThreatExchange/tree/main/pdq)
- ThreatExchange repo last pushed 2026-10-29; latest release `hma-v1.2.1` 2026-10-22; 1,406 stars — GitHub API
- `pdqhash` PyPI bindings: MIT wrapper, PDQ source licensed separately; `compute(image)` returns (hash_vector, quality), `compute_dihedral` returns 8 rotations/flips, `compute_float`; examples use OpenCV; wheels up to cp313 (0.2.8) — [PyPI pdqhash](https://pypi.org/project/pdqhash/)
- Third-party docs describe PDQ as a 256-bit hash tolerant of JPEG compression and minor edits, developed at Facebook/Meta — [imghash PDQ docs (Go library)](https://www.mintlify.com/ajdnik/imghash/algorithms/pdq)
- An NAU Special Collections blog proposed ssdeep for near-identical commercial photographs ("hundreds of nearly identical files") as an appraisal aid — a proposal, not a tested result — [NAU Special Collections blog, 2015](https://library.nau.edu/speccoll/blog/2015/07/appraisal-of-digital-photographs/)
- The 2026 Institutional Books Visual Elements pipeline deduplicates digitized-book images with 144-bit perceptual hashes, LSH and Hamming thresholds — [arXiv 2608.18957](https://arxiv.org/pdf/2608.18957)
- Trevor Owens (LoC, 2013) framed the archival need with "16 nearly identical catfish photos" and "15 PDFs that differ only in cover pages" as cases where fuzzy similarity rather than checksums is needed — [The Signal, Sept 2013](https://blogs.loc.gov/thesignal/2013/09/the-future-of-digital-equivalence-and-significance/)

### Inferences
- imagehash is the lowest-dependency choice (Pillow/numpy/scipy) and ideal for a report-only tool; imagededup is heavier (TensorFlow/PyTorch for CNN) but gives a ready `find_duplicates` with threshold and evaluation tooling.
- PDQ's published threshold (≤31/256) and quality score make it the most "documented" perceptual hash for defensible reporting; its C++ core and OpenCV dependency add build complexity.
- No archive-specific benchmark of these hashes exists; documented archival usage is anecdotal (blogs, Eltrovo).

### Gaps
- blockhash (ISCC/commonsmachinery) was not researched; no source retrieved.
- No published photo-archive case study naming imagehash/imagededup/PDQ was found; only proposals and adjacent library (digitized books) work.
- imagededup's threshold parameters (`max_distance_threshold`, `min_similarity_threshold`) are documented in its external docs, not on PyPI/GitHub README; values not captured.

---

## Q3. Document near-duplicates: shingling, MinHash/SimHash (datasketch), OCR'd scans, Word/PDF versions; e-discovery and archival tools

### Takeaway
`datasketch` (MIT, v2.0.0 July 2026, actively maintained) provides MinHash, MinHash LSH/LSH Forest/LSH Ensemble with Redis/Cassandra backends and is the standard Python route for shingle-based text near-duplicate detection; the digital-library literature (Williams & Giles 2013) found shingling (F=0.99) outperformed simhash (F=0.91). E-discovery platforms (Microsoft Purview) do pairwise textual similarity with a configurable threshold and a "pivot" document per group — a model directly transferable to appraisal reporting.

### Cited Findings
- datasketch provides MinHash (Jaccard/cardinality), Weighted MinHash, HyperLogLog/++; indexes MinHash LSH (threshold queries), LSHBloom, LSH Forest (top-K), LSH Ensemble (containment), HNSW; Redis and Cassandra backends via extras; Python ≥3.9; 2.0.0 changes default permutation scheme to "affine32" so persisted sketches need rebuilding or `scheme="legacy"` — [ekzhu/datasketch](https://github.com/ekzhu/datasketch)
- datasketch 2.0.0 released 2026-07-05 (PyPI); MIT; Python 3.9–3.13 — [PyPI datasketch](https://pypi.org/project/datasketch/); GitHub last pushed 2026-10-26, 2,977 stars — GitHub API
- Williams & Giles, "Near duplicate detection in an academic digital library" (ACM DocEng 2013), CiteSeerX data: best F-scores 0.91 (simhash) vs 0.99 (shingle-based); shingling found a wider variety of duplicate types — [Microsoft Research record](https://www.microsoft.com/en-us/research/?p=415481); [Penn State record](https://pure.psu.edu/en/publications/near-duplicate-detection-in-an-academic-digital-library/)
- Standard shingling method: k-term shingles, Jaccard coefficient over shingle sets against a preset threshold, union-find to build clusters — [Stanford IR Book, Near-duplicates and shingling](https://nlp.stanford.edu/IR-book/html/htmledition/near-duplicates-and-shingling-1.html)
- Microsoft Purview eDiscovery (Premium) near-duplicate detection parses every text-bearing document, compares each pairwise against a configured similarity threshold, groups those above it, and marks one "pivot" per group so reviewers read the pivot then only differences; classic eDiscovery retired 2025-08-31 — [Microsoft Learn: Near duplicate detection](https://learn.microsoft.com/en-us/purview/ediscovery-near-duplicate-detection)
- UK Web Archive (Andrew Jackson, 2013 comment) was "experimenting with using the ssdeep fuzzy-hash of the textual content of web resources" to track significantly similar content and non-identical duplicates, with results promised later — [The Signal comments, Sept 2013](https://blogs.loc.gov/thesignal/2013/09/the-future-of-digital-equivalence-and-significance/)
- A Spanish study applied fuzzy hashing to web document duplicate detection — [USAL record](https://produccioncientifica.usal.es/documentos/5e69520c2999522cccdcecf7?lang=en) (not fetched)
- Rust crates `sketchir` and `txtfp` replicate datasketch's MinHash/SimHash approach; `txtfp` documents byte-parity with Python datasketch via `HashFamily::MurmurHash3_x64_128` — [docs.rs txtfp](https://docs.rs/crate/txtfp/0.3.0); [docs.rs sketchir](https://docs.rs/sketchir)

### Inferences
- A practical pipeline for Word/PDF revisions is: text extraction (Apache Tika or format-specific libs) → normalization → k-shingles → MinHash → datasketch LSH with a Jaccard threshold; this is the same architecture e-discovery vendors describe.
- OCR noise argues for character n-gram shingles (which tolerate scattered recognition errors) rather than word shingles; no source quantified this for archival OCR specifically.
- The Purview "pivot document" pattern is a good report-output model: group → representative → per-member difference summary.

### Gaps
- No source found on Apache Tika combined with similarity hashing as a packaged tool.
- No published archival (as opposed to library/e-discovery) deployment of MinHash/SimHash found.
- Relativity's textual near-duplicate algorithm and default threshold were not found; only Microsoft Purview documentation.
- No study quantifying OCR-error impact on MinHash/SimHash was retrieved.

---

## Q4. Email deduplication: ePADD mechanism; DArcMail, Mailbag, TOMES

### Takeaway
ePADD (Apache-2.0, v11.1.3 June 2026, maintained by the Open Preservation Foundation) deduplicates at ingest using an exact signature hash — Date (to the minute), From, To, Cc, Bcc, Message-ID (when present), plus a body snippet when both date and Message-ID are missing — and records duplicate folder/Message-ID provenance and PREMIS event counts; it does not perform near-duplicate (fuzzy) email matching. DArcMail, TOMES and Mailbag focus on format conversion (EAXS, PDF, WARC) and no deduplication feature was found in their documentation.

### Cited Findings
- ePADD was developed by Stanford Special Collections & University Archives and is "currently hosted and maintained by the Open Preservation Foundation"; software is Apache License 2.0, non-software materials CC BY 2.0 — [ePADD/epadd README](https://github.com/ePADD/epadd)
- ePADD latest release v11.1.3 published 2026-06-03; repo pushed 2026-10-29 — GitHub API, 2026-11-01
- ePADD's `EmailDocument.getSignature()` builds a string from Date (day, month, year, HH:MM), From, To, Cc, Bcc (if present) and `MessageID:` (if present); `uniqueID = Util.hash(getSignature())`; `equals()` delegates to `compareTo()`, which falls back to comparing `uniqueID` — [EmailDocument.java](https://github.com/ePADD/epadd/blob/master/src/java/edu/stanford/muse/index/EmailDocument.java) (source read via GitHub API; lines ~84–122, 157–190, 761–791)
- `setBodySnippet()` stores the first 500 + last 500 characters plus total length "to disambiguate messages that have neither a date nor a messageID," and forces uniqueID recomputation — [EmailDocument.java](https://github.com/ePADD/epadd/blob/master/src/java/edu/stanford/muse/index/EmailDocument.java)
- During fetch, if `fetchConfig.skipDuplicates` and the message has a date or Message-ID and `archive.containsDoc(ed)` is true, the message is skipped, `stats.nMessagesAlreadyPresent` is incremented, and `archive.getDupMessageInfo()` records (folderName, messageID) of the duplicate against the retained document; PREMIS events record "N messages - N errors - Duplicate messages: N" per mbox/IMAP ingest and "overall unique message_count" — [EmailFetcherThread.java](https://github.com/ePADD/epadd/blob/master/src/java/edu/stanford/muse/email/EmailFetcherThread.java) (lines ~1206–1219, 1357–1369, 1708–1775)
- A code comment notes an earlier hack assigning the previous message's date to undated messages "causes the message to be considered a duplicate," so it was disabled for ePADD — [EmailFetcherThread.java](https://github.com/ePADD/epadd/blob/master/src/java/edu/stanford/muse/email/EmailFetcherThread.java) (line ~347)
- A 2023 EA:BCC workshop deck lists message deduplication among ePADD's ingest actions and shows Message ID as a field in the message view — [Using ePADD to Process Email Archives (June 2023)](https://emailarchivesgrant.library.illinois.edu/wp/wp-content/uploads/2023/06/Workshop-Using-ePADD-to-Process-Email-Archives.pdf)
- ePADD described in D-Lib (Schneider et al., May 2017) as computational analysis software for screening, browsing and access to email collections — [D-Lib Magazine](https://www.dlib.org/dlib/may17/schneider/05schneider.html)
- DArcMail (Smithsonian Institution Archives) preserves an email account in EAXS XML, one XML per MBOX; described as open-source for "preservation and limited processing and access" — [SIA blog: latest release DArcMail](https://siarchives.si.edu/blog/announcing-latest-release-darcmail); [SIA DArcMail page](https://siarchives.si.edu/what-we-do/digital-curation/email-preservation-darcmail); [COPTR: DArcMail](https://coptr.digipres.org/index.php/DArcMail)
- TOMES (NC State Archives) components: PST extractor, DArcMail module (EAXS), tagger — [TOMES software overview](https://www.dncr.nc.gov/tomes/20181127-tomessoftwareoverview/download)
- Mailbag: draft specification extending BagIt, plus `mailbagit` tool converting MBOX/PST/EML to derivative formats (EML, PDF, WARC, text); MIT; PyPI shows 0.3.0 while GitHub latest release is v0.7.5 (2025-11-24), repo pushed 2026-09-30 — [PyPI mailbagit](https://pypi.org/project/mailbagit/); [Mailbag project](https://archives.albany.edu/mailbag/); GitHub API
- A GitHub code search for "duplicate" in UAlbanyArchives/mailbagit matched only `docs/design_documents/personas.md` and `mailbagit/derivatives/warc.py` — no dedup feature — GitHub code search, 2026-11-01
- `mail-deduplicate` (PyPI, 7.5.0) is a generic maildir/mbox dedup CLI with strategies based on size, content, timestamp, file path or random choice — [PyPI mail-deduplicate](https://pypi.org/project/mail-deduplicate/7.5.0)
- Patent literature (e.g., US 8,914,331; 8,275,842) describes email near-duplicate detection by hashing selected header + body fields, hashing attachments separately, grouping by hash and comparing content within groups to classify exact vs near duplicates; notes that plain content hashing "would typically only identify emails that are exact duplicates" — [USPTO 8914331](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/8914331); [USPTO 8275842](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/8275842)
- Standard practice keys dedup on Message-ID, which is "required to be unique for each message," though mailers that reuse IDs can cause false duplicates — [IETF Sieve duplicate draft](https://datatracker.ietf.org/doc/draft-ietf-appsawg-sieve-duplicate/06/); [procmail list 1999](https://www.mhonarc.org/archive/html/procmail/1999-05/msg00072.html)

### Inferences
- ePADD's signature is an exact composite key; two copies with different Message-IDs (e.g., sent vs received copies, list re-mailings) or a Date differing by minutes will not be treated as duplicates. Its dedup is cross-folder and cross-source within one archive, and it keeps provenance of suppressed duplicates — a useful model for a report-only tool.
- None of the archival email tools do fuzzy/near-duplicate matching (quoted-reply detection, forwarded-thread containment); that space is covered by e-discovery vendors and patents, so an appraisal tool would need to build it (e.g., MinHash over normalized body text with quoted material stripped).

### Gaps
- ePADD user guide v10/v11 text on duplicates was not retrievable (Google Docs); findings are from source code.
- DArcMail/TOMES documentation was not fetched directly for dedup statements; absence is inferred from summaries.
- Could not confirm whether the ePADD signature's date component is timezone-normalized.

---

## Q5. Audio/video fingerprinting: Chromaprint/AcoustID, videohash; archival use

### Takeaway
Chromaprint (v1.6.2, Oct 2026) with `pyacoustid` (v1.3.1, April 2026, which now includes a pure-Python `compare_fingerprints`) is an actively maintained path for near-identical audio detection; video perceptual hashing in Python (`videohash`, 64-bit, 2022; forks `videohash2`, `vidhash`) is small-community and less maintained, while Meta's TMK+PDQF sits alongside PDQ in ThreatExchange. No documented archival deployment was found.

### Cited Findings
- Chromaprint is the audio fingerprint library for AcoustID, designed to identify near-identical audio with fingerprints "as compact as possible," trading precision/robustness for search performance; use cases listed as "full audio file identification, duplicate audio file detection and long audio stream monitoring"; not a general-purpose fingerprinting solution; `fpcalc` built with `BUILD_TOOLS=ON` needs FFmpeg; FFTW3 build makes the binary GPL, KissFFT is permissive; based on Ke et al. 2005, Kurth & Müller 2008, Jang et al. 2009 — [acoustid/chromaprint](https://github.com/acoustid/chromaprint)
- Chromaprint v1.6.2 released 2026-10-15; repo pushed 2026-10-15; 1,413 stars; license NOASSERTION (LGPL per project; not verified here) — GitHub API
- pyacoustid 1.3.1 released 2026-04-09, MIT, Python ≥3.10; `fingerprint_file(path)` returns (duration, fingerprint) via libchromaprint or `fpcalc`; `compare_fingerprints(a, b)` (added 1.3.0, pure Python) returns a similarity score; raises `NoBackendError` if neither backend found — [PyPI pyacoustid](https://pypi.org/project/pyacoustid/)
- Fingerprinting does not match cover versions, remixes or re-recordings (different performances) — vendor blog, secondary — [interspacemusic glossary](https://interspacemusic.com/blog/glossary/what-is-audio-fingerprinting/)
- videohash (akamhy) 3.0.1 released 2022-05-29, MIT, Python ≥3.6: extracts one frame per second, shrinks to 144×144, builds a collage, takes the collage's wavelet hash bits, XORs with a 64-section dominant-colour pattern to form a 64-bit hash; requires FFmpeg; documented examples show difference 2 → similar, 34 → not similar; explicitly cannot detect whether one video is contained in another — [PyPI videohash](https://pypi.org/project/videohash/); GitHub last pushed 2024-07-12 — GitHub API
- `videohash2` (demmenie fork, 3.0.3) and `vidhash` (0.3.1; 5 frames/sec image hashes, configurable `HashSettings`) are alternatives — [PyPI videohash2](https://pypi.org/project/videohash2/3.0.3); [PyPI vidhash](https://pypi.org/project/vidhash)
- Checksum Count Vectors for duplicate/variant detection across 4,902 decoded legacy tape images: 58% accuracy on variants, 97% on alternative copies of damaged recordings with up to 75% records missing (Grzeszczuk, Skorupska, Wójcik, March 2026) — [arXiv 2604.09657](https://arxiv.org/abs/2604.09657)

### Inferences
- For a report-only appraisal tool, Chromaprint + pyacoustid's local `compare_fingerprints` avoids any call to the AcoustID web service, keeping source material local.
- Video hashing in Python is thin; a pragmatic approach is frame sampling via FFmpeg + imagehash/PDQ per frame, which is essentially what videohash/vidhash do.

### Gaps
- No documented use of Chromaprint or videohash in an archive or digital-forensics workflow was found.
- TMK+PDQF video hash details were not confirmed from the ThreatExchange page retrieved.
- Chromaprint's current license text was not read (GitHub reports NOASSERTION).

---

## Q6. Version detection of office documents and born-digital drafts ("same document, different revision" clustering)

### Takeaway
The only archive-specific tool found that targets draft/version relationships is Eltrovo (St John Karp, Pratt Institute, GPL-3.0, 2024), a proof-of-concept that runs ssdeep and sdhash over a collection and emits an RDF graph of duplicate, related-draft and embedded-file relationships; it reported output volume problems (1,168 triples from 229 files) and has not been updated since April 2024. Forensic literature names "identifying updated document versions" as a core use case for approximate matching, but no production archival tool implements it.

### Cited Findings
- Eltrovo "analyzes sets of files and discovers the relationships between them using fuzzy matching algorithms. The result is an RDF graph that can inform the archivist not only of duplicate files but also files that appear to be related to each other such as drafts of the same document, or files that include other files within them"; presented at BitCurator Forum March 2024; developed as a Pratt Institute independent study; "experimental"; GPL-3.0; last pushed 2024-04-07 — [dazzlepansy/eltrovo README](https://github.com/dazzlepansy/eltrovo); GitHub API
- Karp's PDT&C article (2024, DOI 10.1515/pdtc-2024-0042) reports that running both hashes on the 229-file trial set "resulted in 1,168 RDF triples, a graph so large that the conversion to a PDF could not be completed," and cites Thomas (2011) for an earlier Bodleian suggestion that fuzzy hashing could support archival arrangement — [De Gruyter record](https://www.degruyterbrill.com/document/doi/10.1515/pdtc-2024-0042/html) (quotes via search snippet; full text 403)
- Karp's Pratt Info Show post (April 30, 2024) frames Eltrovo as identifying whether files "may be versions of the same work, such as edited images or drafts of a document" — [Pratt SI Info Show 2024](https://studentwork.prattsi.org/infoshow/2024/the-interconnectedness-of-all-things-automatic-discovery-for-digital-archives)
- DFRWS EU 2014 practical-use paper lists identifying updated document versions as a common approximate-matching use case, with searching, streaming and clustering modes — [DFRWS record](https://dfrws.org/presentation/practical-use-of-approximate-hash-based-matching-in-digital-investigations/)
- Roussev (2011) noted ssdeep "struggling on some MS Office files" — [DFRWS 2011](https://dfrws.org/presentation/an-evaluation-of-forensic-similarity-hashes/)
- Owens (2013) framed version equivalence as a community-dependent judgment and argued fuzzy hashing could help stewards decide which minor differences matter — [The Signal](https://blogs.loc.gov/thesignal/2013/09/the-future-of-digital-equivalence-and-significance/)

### Inferences
- Bytewise hashing of OOXML/ODF (zipped) containers is unreliable for revision detection because compression scrambles byte similarity; text extraction followed by MinHash/shingling (Q3) or TLSH over extracted text is the more defensible approach, consistent with ssdeep's observed weakness on Office files.
- Eltrovo demonstrates the main design risk for an appraisal tool: pairwise fuzzy output explodes combinatorially; clustering (union-find over threshold edges) and a representative-per-cluster report are needed.

### Gaps
- No peer-reviewed evaluation of revision clustering on an archival born-digital corpus was found.
- Eltrovo's thresholds, exact RDF vocabulary and evaluation metrics were not retrievable (full text blocked).

---

## Q7. Research literature from digital preservation / DH on near-duplicate detection in born-digital collections

### Takeaway
The digital-preservation literature on near-duplicate detection is sparse: the Emory Rushdie project and the Kirschenbaum/Farr/Kraus born-digital literary white paper address access and emulation, not similarity detection; the concrete contributions are Owens's 2013 "digital equivalence" essay, Thomas (2011, Bodleian) suggesting fuzzy hashing for arrangement, the UK Web Archive's ssdeep experiment, and Karp's 2024 Eltrovo/PDT&C work. The rigorous evaluation literature is all in digital forensics (DFRWS, Digital Investigation, NIST SP 800-168).

### Cited Findings
- Emory acquired Rushdie's papers in 2006; born-digital material (calendars, sticky notes, email, novel drafts) was the first large digital component of its rare holdings; the project emphasized emulation and searchable databases of files — [Emory Magazine Winter 2010](https://magazine.emory.edu/2010/winter/authors.html); Carroll, Farr, Hornsby & Ranker, "A Comprehensive Approach to Born-Digital Archives" (Archivaria, 2011) — [Archivaria PDF](https://archivaria.ca/index.php/archivaria/article/download/13360/14664)
- Kirschenbaum, Farr & Kraus NEH white paper "Approaches to Managing and Collecting Born-Digital Literary Materials for Scholarly Use" covers three literary collections including Rushdie — [UMD DRUM](https://drum.lib.umd.edu/handle/1903/9787)
- No source describing near-duplicate or version clustering in the Rushdie collection was found — search results, 2026-11-01
- Owens (LoC, 2013) on digital equivalence and fuzzy hashing; links Cathy Marshall's Code4Lib keynote on digital variations — [The Signal](https://blogs.loc.gov/thesignal/2013/09/the-future-of-digital-equivalence-and-significance/)
- Karp (PDT&C 2024) cites Thomas (2011) for the Bodleian suggestion that fuzzy hashing could support archival arrangement — [De Gruyter record](https://www.degruyterbrill.com/document/doi/10.1515/pdtc-2024-0042/html) (via snippet)
- GWU Libraries born-digital accessioning best practice: practitioner "ran scripts to find duplicates and reviewed duplicate files and removed them from the SIP," also using NARA's Electronic Records Accessioning Support Tools — [GWU DigitalServicesDocs](https://gwu-libraries.github.io/DigitalServicesDocs/borndigital/bestpractice)
- Library of Congress MSS processing note: "when duplicate files and empty directories are located, they may be appraised for potential removal" — [LoC MSS Processing History Note](https://findingaids.loc.gov/exist_collections/ead3pdf/mss/2023/MSS-Processing-History-Note.pdf)
- Forensics evaluation canon: Roussev 2011 (DFRWS), Breitinger & Roussev 2014 and Breitinger, Stivaktakis & Roussev 2014 (Digital Investigation), NIST SP 800-168 (2014), Pagani et al. 2018 (CODASPY), Martín-Pérez et al. 2021 (DFRWS), Raff & Nicholas 2017/18 (LZJD) — see Q1 citations
- 2026 arXiv work on legacy tape-image variant detection (Checksum Count Vectors) is the most recent preservation-adjacent paper found — [arXiv 2604.09657](https://arxiv.org/abs/2604.09657)

### Inferences
- An appraisal tool that reports similarity clusters with provenance would fill a documented gap: archival practice (GWU, LoC) currently relies on exact-duplicate scripts, and the one archival near-duplicate prototype (Eltrovo) stalled on output scale.
- NIST SP 800-168 terminology (bytewise / syntactic / semantic approximate matching; precision/recall/F-score reporting) is a ready framework for documenting the tool's methods.

### Gaps
- No iPRES or Code4Lib Journal paper specifically on near-duplicate detection in born-digital collections was located; searches returned only the Code4Lib keynote reference via Owens.
- No Stanford born-digital near-duplicate publication found (Stanford's contribution is ePADD's exact signature dedup).
- COPTR entries for ssdeep/sdhash/ePADD could not be fetched (HTTP 503 on coptr.digipres.org at time of research); existence of a COPTR DArcMail entry was confirmed only via search listing.
- The UK Web Archive's promised results from the 2013 ssdeep experiment were not located.

---

## Summary inventory table

| Tool | Type/algorithm | Targets | License | Latest release | Activity (2026) | Python | Archival/forensic use documented |
|---|---|---|---|---|---|---|---|
| ssdeep | CTPH fuzzy hash | any bytes | GPL-2.0 | 2.14.1 (2017-11) | minor pushes 2026-08; no releases | `ssdeep` 3.4 (2019), `ppdeep` (2020) | BitCurator; UKWA experiment; Eltrovo |
| sdhash | similarity digest (Bloom filters) | any bytes | Apache-2.0 | 4.0 (2013-12) | last push 2020 (dormant) | SWIG binding in repo | BitCurator; Eltrovo |
| TLSH | LSH, quartile buckets | ≥50 bytes | Apache/BSD | 5.0.2 (2026-07) | active | `py-tlsh` 5.0.0 (2026-05) | VirusTotal, MalwareBazaar |
| Nilsimsa | LSH (anti-spam) | text/email | varies | — | ports only | `py-nilsimsa` (status unknown) | spam filtering; high FP rate |
| mrsh-v2 | multi-resolution similarity hash | any bytes | — | — | no public repo found | none found | academic evaluations |
| imagehash | aHash/pHash/dHash/wHash/colorhash/crop-resistant | images | BSD-2 | 4.3.2 (2025-02) | pushed 2026-10 | native | none archival found |
| imagededup | PHash/DHash/WHash/AHash + CNN | images | Apache-2.0 | 0.3.3.post2 (2025-08) | last push 2025-08 | native | none archival found |
| PDQ | 256-bit DCT hash + quality | images | separate PDQ license | ThreatExchange hma-v1.2.1 (2026-10) | active | `pdqhash` 0.2.8 | trust & safety; threshold ≤31 |
| datasketch | MinHash, LSH, LSH Forest/Ensemble | text/sets | MIT | 2.0.0 (2026-07) | pushed 2026-10 | native | digital-library NDD literature |
| ePADD | exact composite signature hash | email | Apache-2.0 | 11.1.3 (2026-06) | active (OPF) | Java | Stanford/OPF; many archives |
| mailbagit | format conversion (no dedup) | email | MIT | 0.7.5 (2025-11) | pushed 2026-09 | native | UAlbany |
| Chromaprint | chroma-feature audio fingerprint | audio | LGPL (per project; unverified) | 1.6.2 (2026-10) | active | `pyacoustid` 1.3.1 (2026-04) | MusicBrainz Picard |
| videohash | frame collage wavelet hash, 64-bit | video | MIT | 3.0.1 (2022-05) | last push 2024-07 | native (needs FFmpeg) | none found |
| Eltrovo | ssdeep+sdhash → RDF graph | mixed collections | GPL-3.0 | none | last push 2024-04 | Python (proof-of-concept) | Pratt/BitCurator Forum 2024; PDT&C 2024 |
