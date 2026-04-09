# HYU NLP Lab Publications Website — Maintenance Guide

## Site Structure

```
/                        # EN root
  index.html             # Main publication listing (all papers with filter/tags)
  members.html           # Member index (card grid linking to detail pages)
  style.css              # Shared stylesheet
  papers/{slug}.html     # EN paper detail pages
  members/{slug}.html    # EN member detail pages
  images/{slug}/         # Paper figures (figure1.png, etc.)
/ko/                     # KO mirror
  index.html             # Korean publication listing
  members.html           # Korean member index
  papers/{slug}.html     # KO paper detail pages
  members/{slug}.html    # KO member detail pages
```

- **Bilingual**: Every page exists in EN and KO. Always update both.
- **Static HTML**: No build step. Edit HTML directly, push to deploy via GitHub Pages.
- **Deployment**: `git push origin main` deploys to https://galsang.github.io/hyu-nlp-publications/

## Data Sources

- **Official member/publication list**: https://nlp.hanyang.ac.kr/home/members and /home/publications
- **Paper content**: arXiv, ACL Anthology, KoreaScience (koreascience.kr), DBPIA (dbpia.co.kr)
- **Figures**: arXiv HTML (https://arxiv.org/html/{id}/), or render from PDF at 300 DPI

---

## Adding a New Paper

### 1. Create detail pages (EN + KO)

Create `papers/{slug}.html` and `ko/papers/{slug}.html`. Use an existing paper as a template (e.g., `papers/magic.html` for a full example).

Required sections:
- **EN**: One-Line Summary, Background & Motivation, Method, Results, Why It Matters, Links, Tags
- **KO**: 한줄 요약, 배경, 방법, 결과, 의의, 링크, Tags

Key elements:
```html
<!-- Hero with venue badge and authors -->
<div class="detail-hero">
    <div class="lang-toggle">...</div>
    <div class="container">
        <a href="../index.html" class="back-link">...</a>
        <h1>{Full Paper Title}</h1>
        <div class="meta"><span class="venue-badge">{Venue}</span></div>
        <div class="authors">{Author List}</div>
    </div>
</div>
```

For KO pages of domestic/Korean papers, use the Korean title in `<h1>`.

### 2. Add card to index pages

Add a `paper-card` div in the appropriate year-section of both `index.html` and `ko/index.html`:

```html
<div class="paper-card" data-tags="{space-separated tags}">
    <a href="papers/{slug}.html" class="card-link">
        <div class="paper-header">
            <div>
                <div class="paper-title">{Title}</div>
                <div class="paper-authors">{Authors}</div>
            </div>
            <span class="paper-venue">{Short Venue}</span>
        </div>
        <div class="paper-oneliner">{One-line description}</div>
        <div class="paper-tags">
            <span class="tag tag-{category}">{Label}</span>
        </div>
    </a>
</div>
```

For KO index: use Korean title for domestic papers, Korean oneliner for all papers.

### 3. Update member pages

For each author who is a current lab member:
- Add paper `<li>` to their `members/{slug}.html` and `ko/members/{slug}.html`
- Use Korean title on KO member pages for domestic papers
- Update publication count on `members.html` and `ko/members.html`

### 4. Add figures (optional but recommended)

- **From arXiv HTML**: Use `https://arxiv.org/html/{arxiv_id}/` figure URLs directly
- **From PDF**: Download PDF, render at 300 DPI with `pdf2image`, crop key figure with PIL
- Save to `images/{slug}/figure1.png`

### 5. Verify consistency

Run cross-checks:
- Paper title in index card == title in detail page `<h1>`
- Authors in index card == authors in detail page
- Tags in `data-tags` attribute match `<span class="tag">` spans
- Member page paper titles match detail page titles
- KO and EN pages are in sync

---

## Adding a New Member

### 1. Create detail pages

Create `members/{slug}.html` and `ko/members/{slug}.html` using an existing member as template.

Key structure:
```html
<div class="detail-hero">
    <h1>{Name}</h1>
    <div class="meta"><span class="venue-badge">{Role} ({Period})</span></div>
    <div class="member-detail-info">Research: {Topic}</div>
</div>
<div class="detail-content">
    <h2>Publications</h2>
    <ul class="pub-list">
        <li>
            <div class="pub-title"><a href="../papers/{slug}.html">{Title}</a></div>
            <div class="pub-authors">{Authors}</div>
            <span class="pub-venue">{Venue}</span>
        </li>
    </ul>
</div>
```

### 2. Add card to member index

Add to the appropriate section (Ph.D., MS/Ph.D., MS) in `members.html` and `ko/members.html`:

```html
<a href="members/{slug}.html" class="member-card">
    <div class="member-name">{Name} <span class="view-arrow">&rarr;</span></div>
    <div class="member-name-ko">{Korean Name}</div>
    <div class="member-role">{Role} ({Period})</div>
    <div class="member-research">{Research Topic}</div>
    <div class="member-pub-count"><strong>{N}</strong> publications</div>
</a>
```

### 3. Order

Follow the order on https://nlp.hanyang.ac.kr/home/members exactly.

---

## Available Tags

Current tag categories (used in `data-tags` and as CSS classes):

| Tag | CSS Class | Display Label |
|-----|-----------|---------------|
| multimodal | tag-multimodal | Multimodal |
| rag | tag-rag | RAG |
| knowledge | tag-knowledge | Knowledge |
| dialogue | tag-dialogue | Dialogue |
| benchmark | tag-benchmark | Benchmark |
| safety | tag-safety | Safety |
| unlearning | tag-unlearning | Unlearning |
| repr | tag-repr | Representation Learning |
| parsing | tag-parsing | Parsing & Syntax |
| reasoning | tag-reasoning | Reasoning |
| domain | tag-domain | Domain LLM |
| icl | tag-icl | In-Context Learning |
| efficiency | tag-efficiency | Efficiency |
| ir | tag-ir | Information Retrieval |
| kg | tag-kg | Knowledge Graph |
| detection | tag-detection | Detection |
| multilingual | tag-multilingual | Multilingual |
| abstention | tag-abstention | Abstention |
| confidence | tag-confidence | Confidence Estimation |

To add a new tag: add CSS class in `style.css`, add filter button in both index pages, use in `data-tags`.

---

## Index Page Sections (in order)

1. Preprints
2. 2026, 2025, 2024, 2023, 2022, 2021 and Earlier (international conferences)
3. Journals (학술지)
4. Domestic Conferences (국내 학술대회)
5. International Workshops (국제 워크숍)

---

## Common Pitfalls

- **Tag filter bug**: `data-tags` uses space-separated words. The JS filter splits on space and uses `includes()` on the array (not substring matching).
- **Title sync**: When changing a paper title, update it in: index card, detail page `<h1>`, and all member pages that list it.
- **KO titles**: Domestic/Korean papers should use Korean titles in `ko/index.html` and `ko/members/*.html`, but English titles in `index.html` and `members/*.html`.
- **Author lists**: Member pages must show the FULL author list from the paper, not abbreviated versions.
- **Venue accuracy**: Double-check venue names (e.g., "LREC-COLING 2024" not "LREC 2024", "ACL 2026 Findings" not "ACL 2026").
- **Member order**: Follow the exact order from the official Google Sites page.

## Verification Script

To check for mismatches between index, detail pages, and member pages:

```bash
python3 -c "
import re, os
papers = {}
for f in sorted(os.listdir('papers')):
    if not f.endswith('.html'): continue
    slug = f[:-5]
    content = open(f'papers/{f}').read()
    title = re.search(r'<h1>(.*?)</h1>', content, re.DOTALL).group(1).strip()
    authors = re.search(r'class=\"authors\">(.*?)</div>', content, re.DOTALL).group(1).strip()
    venue = re.search(r'class=\"venue-badge\">(.*?)</span>', content).group(1).strip()
    papers[slug] = {'title': title, 'authors': authors, 'venue': venue}

# Check member pages
for mf in sorted(os.listdir('members')):
    if not mf.endswith('.html'): continue
    content = open(f'members/{mf}').read()
    for slug, title in re.findall(r'href=\"\.\./papers/([^\"]+)\.html\">(.*?)</a>', content):
        if slug in papers and title.strip() != papers[slug]['title']:
            print(f'TITLE MISMATCH {mf} [{slug}]')
"
```
