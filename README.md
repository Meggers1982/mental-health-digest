# Mental Health & Psychiatry Research Digest

A GitHub Actions workflow that searches curated psychiatry, psychology, behavioral science, and neuroscience research relevant to mental health journals on PubMed, filters out widely covered stories, runs a single Claude pass for journalist-ready summaries and pitch angles, and publishes results to a GitHub Pages dashboard.

## How it works

1. **PubMed search** - Queries journals by ISSN for studies published in the past 7 days
2. **Title screening** - Prioritizes studies with novelty signals and excludes animal-only studies
3. **SERPAPI media filter** - Checks Google News and skips any study with 3+ news results
4. **Abstract fetch** - Retrieves full abstracts for shortlisted studies
5. **Claude pass** - Writes structured JSON: headline, summary, why it matters, caveats, relevance score, and pitch angles per publication type
6. **Artifact upload** - Saves JSON results as a GitHub Actions artifact
7. **Deploy job** - Downloads all job artifacts, merges and deduplicates by PMID, commits `data/results.json`, serves via GitHub Pages
8. **Email notification** - Sends a short email with study count and a dashboard link

## Dashboard

Features:
- Card view per study with headline, summary, caveats, fact-check notes
- Expandable pitch angles section for publications such as The Atlantic, STAT News, Psychology Today, HuffPost Health, Verywell Mind, The New York Times (Health/Well section), NPR Health, Health.com, Women's Health Magazine, Everyday Health, Scientific American, Monitor on Psychology, The Transmitter, and general health outlets
- Filter by category, groundbreaking type, status, date range, and score
- Search across all study text and pitches
- Status tracking (New / Saved / Pitched / Passed) saved to localStorage
- Deduplication across runs by PMID

## Schedule

Runs automatically every morning at 7:00 AM ET. All jobs run in parallel; the deploy job merges results and publishes the dashboard once complete.

Can also be triggered manually via **Actions -> Mental Health & Psychiatry Research Digest -> Run workflow**.

## Categories

| Category | Journals | Jobs |
|---|---:|---|
| Psychiatry | 222 | 2 (chunks 1-2) |
| Behavioral Sciences | 93 | 2 (chunks 1-2) |
| Neurology | 384 | 2 (chunks 1-2) |
| Psychology | 197 | 2 (chunks 1-2) |

Large categories are split into chunks to keep run times under 20 minutes.

The category CSVs in `data/` are now hand-maintained. They were originally generated from `PubMed_Journals_Categorized.xlsx` by `scripts/extract_journals.py`, but that workbook no longer exists, so the script has been deleted.

## Journal list audit (2026-09-14)

**Method:** Pulled OpenAlex's top sources for this digest's subject areas over the prior year, diffed them against every CSV by ISSN and title, and kept only titles that NCBI lists and that PubMed indexed at least 20 articles from in the past 12 months. Every row in a category CSV is searched with no topic filter, so a journal's entire output has to fit the beat.

**Added (11):**

| Journal | ISSN | CSV | PubMed articles/yr |
|---|---|---|---:|
| BJPsych Open | 2056-4724 | Psychiatry | 309 |
| Child and Adolescent Psychiatry and Mental Health | 1753-2000 | Psychiatry | 188 |
| PLOS Mental Health | 2837-8156 | Psychiatry | 266 |
| Schizophrenia (formerly npj Schizophrenia) | 2754-6993 | Psychiatry | 125 |
| Borderline Personality Disorder and Emotion Dysregulation | 2051-6673 | Psychiatry | 49 |
| Schizophrenia Research: Cognition | 2215-0013 | Psychiatry | 87 |
| Journal of Eating Disorders | 2050-2974 | Behavioral Sciences | 334 |
| European Eating Disorders Review | 1099-0968 | Behavioral Sciences | 173 |
| Eating Disorders | 1532-530X | Behavioral Sciences | 109 |
| Eating and Weight Disorders | 1590-1262 | Behavioral Sciences | 108 |
| Journal of Child & Adolescent Trauma | 1936-153X | Psychology | 169 |

The eating disorder titles go in Behavioral Sciences alongside the *International Journal of Eating Disorders*, and the trauma title goes in Psychology with the other trauma journals. No category grew enough to need new workflow chunks.

**Notable exclusions:**

- **Mega-journal:** *Frontiers in Psychiatry* (about 2,800 PubMed articles/yr) would take over the 30 candidate slots.
- **Off-beat (OpenAlex classifier noise):** sexual medicine and urology (*Journal of Sexual Medicine*, *Sexual Medicine*, *Sexual Medicine Reviews*, *International Journal of Impotence Research*), *Innovation in Aging*, *Toxicon*, *Mediastinum*, *Chronobiology International*, *Journal of Dance Medicine & Science*, *Journal of Immigrant and Minority Health*, and *Gerontology & Geriatrics Education*.
- **Speech and linguistics:** *Journal of Fluency Disorders*, *Language and Speech*, and *Phonetica*. Basic language research rarely makes a mental health pitch.
- **Neurology-heavy, weak mental health angle:** *npj Parkinson's Disease*, *Neurology and Therapy*, *Clinical Parkinsonism & Related Disorders*, the Alzheimer's & Dementia companion journals, *npj Dementia*, *Frontiers in Dementia*, *Neuroscience Applied*, and the smaller sleep titles (*SLEEP Advances*, *Sleep Medicine: X*, *Frontiers in Sleep*, *Clocks & Sleep*, *Sleep and Biological Rhythms*). The Neurology list already carries the core dementia and sleep journals.
- **Child welfare scope, not mental health research specifically:** *Child Abuse & Neglect* (529/yr, already in the pediatric-health digest) and *Child Maltreatment*. *OMEGA* was also left out because its grief and end-of-life coverage is uneven and *Death Studies* already covers that niche.
- **Case reports:** *Case Reports in Neurology* and *Epilepsy & Behavior Reports*.
- **Barely in PubMed (fewer than 20 articles/yr) despite high OpenAlex volume:** *Mindfulness*, *Personality and Individual Differences*, *Cognitive Therapy and Research*, *Clinical Psychology: Science and Practice*, *Journal of Child and Family Studies*, *Journal of Aggression, Maltreatment & Trauma*, *Child Abuse Review*, *European Journal of Trauma & Dissociation*, and *Creativity Research Journal*.
- **Not in PubMed at all:** *Psychoanalytic Inquiry*, *Thinking Skills and Creativity*, *Counselling and Psychotherapy Research*, *Journal of Creative Behavior*, and several non-English titles (*Psiquiatría Biológica*, *Revue française de psychanalyse*).

## Manual Trigger

Go to **Actions -> Mental Health & Psychiatry Research Digest -> Run workflow**.

- Leave **category** blank to run all jobs
- Enter an exact category name, such as `Psychiatry`, to run just that category

## GitHub Pages Setup

1. Go to **Settings -> Pages**
2. Set source to **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Save; GitHub will serve `index.html` at the dashboard URL

## Required Secrets

Add these in **Settings -> Secrets and variables -> Actions**:

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `SERPAPI_KEY` | SerpAPI key for Google News filtering |
| `SUPABASE_URL` | Supabase project URL (dashboard save/delete personalization) |
| `SUPABASE_KEY` | Supabase API key (read-only) |
| `DASHBOARD_REPO_TOKEN` | Token with push access to the shared `research-digest-dashboard` repo |

## Repo Structure

```text
.github/
  workflows/
    mental-health-digest.yml
scripts/
  mental_health_digest.py
  merge_results.py
data/
  Psychiatry.csv
  Behavioral Sciences.csv
  Neurology.csv
  Psychology.csv
  results.json
index.html
requirements.txt
```

## Dashboard Study Card Fields

Each study card shows:

- **Headline** - plain-language present-tense summary
- **Relevance score** - 1-10, weighted for mental health, therapy, the brain, and behavior journalism fit
- **Category & journal** - source metadata
- **Groundbreaking type** - counterintuitive, overturns prior research, first-in-class, or domain-relevant finding
- **Media coverage** - SERPAPI verification status
- **The study** - what was done, who participated, and the key finding
- **Why it matters** - real-world significance for the target audience
- **Caveats** - limitations flagged automatically
- **Fact-check note** - corrections made during the Claude pass
- **Pitch angles** - expandable publication-specific pitch blocks
- **Status** - New / Saved / Pitched / Passed, tracked in your browser
