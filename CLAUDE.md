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

### Overview

When adding a new paper, create **fully detailed** pages from the start. Each page should contain rich, publication-quality content — not placeholder text to be enriched later. Use subagents (one per paper) to fetch content from the paper's source and generate complete pages in parallel.

### 1. Create detail pages (EN + KO) via subagent

For each new paper, launch a subagent with the following instructions:

1. **Fetch the paper content** from the best available source (priority order):
   - arXiv HTML (`https://arxiv.org/html/{arxiv_id}/`) — best, full text
   - ACL Anthology (`https://aclanthology.org/...`) — abstract + PDF
   - AAAI / ACM / OpenReview / SAGE — abstract + metadata
   - KoreaScience / KCI Portal — Korean domestic papers
2. **Generate both `papers/{slug}.html` and `ko/papers/{slug}.html`** with full content

**Required sections with content depth:**

| Section (EN) | Section (KO) | Content Depth |
|---|---|---|
| One-Line Summary | 한줄 요약 | 1-2 sentences with key contribution + headline numbers |
| Background & Motivation | 배경 및 동기 | 2-3 paragraphs + `highlight-box` with key challenge/insight |
| Proposed Method | 제안 방법 | 3-5 `method-step` blocks with specific technical details |
| Experimental Results | 실험 결과 | `results-table` with specific numbers + `key-points` list (4-6 items) |
| Why It Matters | 의의 | 2-3 paragraphs or structured bullet points |
| Links | 링크 | arXiv, ACL Anthology, GitHub, etc. |
| Tags | Tags | Same in both languages |

**Hero structure:**
```html
<div class="detail-hero">
    <div class="lang-toggle">
        <a href="../papers/{slug}.html" class="active">EN</a>
        <a href="../ko/papers/{slug}.html">KO</a>
    </div>
    <div class="container">
        <a href="../index.html" class="back-link" style="color:white;opacity:0.8;">&#8592; All Publications</a>
        <h1>{Full Paper Title}</h1>
        <div class="meta"><span class="venue-badge">{Venue}</span></div>
        <div class="authors">{Author List}</div>
    </div>
</div>
```

For KO pages of domestic/Korean papers, use the Korean title in `<h1>`.

**CSS classes for content sections:**
```html
<!-- Highlight box for key insights -->
<div class="highlight-box">
    <p><strong>Key Challenge:</strong> ...</p>
</div>

<!-- Method steps (numbered) -->
<div class="method-steps">
    <div class="method-step">
        <div class="step-num">1</div>
        <div class="step-title">Step Title</div>
        <div class="step-desc">Detailed description with technical specifics...</div>
    </div>
</div>

<!-- Results table with highlighted best rows -->
<table class="results-table">
    <thead><tr><th>Model</th><th>Metric</th></tr></thead>
    <tbody>
        <tr><td>Baseline</td><td>72.3</td></tr>
        <tr><td class="highlight-cell">Ours</td><td class="highlight-cell">85.1</td></tr>
    </tbody>
</table>

<!-- Key findings list -->
<ul class="key-points">
    <li><strong>Finding:</strong> Description with specific numbers</li>
</ul>

<!-- Figure (after One-Line Summary section) -->
<figure class="paper-figure">
    <img src="../images/{slug}/figure1.png" alt="...">
    <figcaption><strong>Figure 1.</strong> Caption.</figcaption>
</figure>
```

**Reference examples** (well-structured pages to use as templates):
- `papers/magic.html` — arXiv figure, multiple tables, method-steps, highlight-boxes
- `papers/tact.html` — Dataset statistics, multiple result tables, method + evaluation
- `papers/constituency-parse.html` — Extensive tables, ablation studies, cross-lingual results

### 2. Add figures

- **From arXiv HTML**: Link directly to `https://arxiv.org/html/{arxiv_id}/` figure URLs
- **From PDF**: Download PDF, render at 300 DPI with `pdf2image`, crop key figure with PIL
- Save to `images/{slug}/figure1.png`
- EN paths: `../images/{slug}/figure1.png`, KO paths: `../../images/{slug}/figure1.png`

### 3. Add card to index pages

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

### 4. Update member pages

For each author who is a current lab member:
- Add paper `<li>` to their `members/{slug}.html` and `ko/members/{slug}.html`
- Use Korean title on KO member pages for domestic papers
- Update publication count on `members.html` and `ko/members.html`

### 5. Verify consistency

Run cross-checks:
- Paper title in index card == title in detail page `<h1>`
- Authors in index card == authors in detail page
- Tags in `data-tags` attribute match `<span class="tag">` spans
- Member page paper titles match detail page titles
- KO and EN pages are in sync

### Subagent Prompt Template

When adding multiple papers at once, use this prompt per subagent (batch 5 at a time in parallel):

```
You are creating a paper detail page on a bilingual (EN/KO) static HTML website for HYU NLP Lab.

## Task
Create full, detailed pages for the paper "{slug}" by:
1. Fetching the paper content from: {source_url}
2. Creating EN page at /Users/taeuk/Codes/homepage/papers/{slug}.html
3. Creating KO page at /Users/taeuk/Codes/homepage/ko/papers/{slug}.html

## Paper Info
- Title: {title}
- Authors: {authors}
- Venue: {venue}
- Tags: {tags}

## Rules
- Use the template from papers/magic.html for structure
- Include ALL sections with detailed content (not one-liners)
- Use CSS classes: highlight-box, method-steps, results-table, key-points
- KO version should be a natural Korean translation
- Include specific numbers and technical details from the paper

Write both files directly.
```

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
