# Fake News Detection System - Interview Q&A Guide

## 📋 Project Overview

### Q1: What is this project about?
**A**: This is an AI-powered Fake News Detection and Credibility Assessment System that evaluates the credibility of news articles and claims using multiple signals including machine learning models, source credibility scoring, and evidence aggregation from trusted RSS feeds.

### Q2: What are the three input modes?
**A**: 
1. **Claim Verification** (Primary) - User enters a claim like "Is PM Modi resigning?" and system searches RSS feeds for evidence
2. **URL Analysis** - User pastes a news article URL, system extracts content and analyzes
3. **Text Analysis** (Fallback) - User pastes article text manually

---

## 🧠 Machine Learning & Algorithms

### Q3: What ML models are used?
**A**: The system uses **Classical ML Models**:
| Model | Purpose | Library |
|-------|---------|---------|
| Logistic Regression | Binary classification (real/fake) | scikit-learn |
| Support Vector Machine (SVM) | Text classification with RBF kernel | scikit-learn |
| Random Forest | Ensemble classification | scikit-learn |
| TF-IDF Vectorizer | Text feature extraction | scikit-learn |

### Q4: What is TF-IDF and why is it used?
**A**: **TF-IDF (Term Frequency-Inverse Document Frequency)** converts text into numerical vectors.
- **TF**: How often a word appears in a document
- **IDF**: How rare a word is across all documents
- **Why**: Gives higher weight to distinctive words, lower weight to common words like "the", "is"

```python
from sklearn.feature_extraction.text import TfidfVectorizer
vectorizer = TfidfVectorizer(max_features=5000, ngram_range=(1, 2))
features = vectorizer.fit_transform(texts)
```

### Q5: What preprocessing is done on text?
**A**: The `TextCleaner` class performs:
1. **Lowercasing** - Normalize case
2. **HTML tag removal** - Strip `<p>`, `<div>` etc.
3. **URL removal** - Remove http links
4. **Special character removal** - Keep only alphanumeric
5. **Stopword removal** - Remove "the", "is", "and"
6. **Lemmatization** - Convert "running" → "run"

### Q6: How does the model make predictions?
**A**: 
1. Text is cleaned using `TextCleaner`
2. Cleaned text is vectorized using TF-IDF
3. Vector is passed to trained model (Logistic Regression/SVM/Random Forest)
4. Model outputs probability score (0.0 to 1.0)
5. Score is combined with other signals for final credibility

---

## 📊 Credibility Assessment

### Q7: What is multi-signal credibility aggregation?
**A**: Final credibility combines multiple signals with weights:

| Signal | Weight | Description |
|--------|--------|-------------|
| **Model Prediction** | 50% | ML model's fake/real score |
| **Source Credibility** | 30% | Pre-compiled source reputation database |
| **Evidence Agreement** | 20% | Cross-source similarity from RSS feeds |

### Q8: How is source credibility calculated?
**A**: Using `SourceCredibilityScorer`:
- **Trusted sources** (score 0.85-0.95): Reuters, BBC, The Hindu, NDTV
- **Unknown sources** (score 0.5): Base neutral score
- **Unreliable sources** (score 0.15-0.30): Known fake news sites

```python
score, explanation = source_scorer.get_score("thehindu.com")
# Returns: (0.90, "Highly trusted Indian publication")
```

### Q9: How does evidence aggregation work?
**A**: The `EvidenceAggregator` class:
1. Fetches articles from RSS feeds of trusted sources
2. Computes TF-IDF similarity between input and RSS articles
3. High similarity with trusted sources → Higher credibility
4. High similarity with untrusted sources → Lower credibility

---

## 🔍 Claim Verification (Key Feature)

### Q10: How does claim verification work?
**A**: Step-by-step process:

1. **Entity Extraction** (`ClaimExtractor`)
   - Extract people: "Ajit Pawar"
   - Extract places: "Delhi", "Mumbai"
   - Uses regex patterns + known entity database

2. **Keyword Extraction**
   - Remove stopwords
   - Keep meaningful words: "dead", "plane", "crash"

3. **Evidence Search** (`ClaimVerifier`)
   - Search RSS corpus for matching articles
   - Match by entity + keyword overlap

4. **Evidence Scoring**
   - **CONFIRMATION**: Trusted source confirms claim
   - **CONTRADICTION**: Source denies claim
   - **PARTIAL_MENTION**: Entity mentioned but not event

5. **Verdict Generation**
   - LIKELY TRUE, LIKELY FALSE, UNVERIFIED, DISPUTED

### Q11: What is the entity extraction approach?
**A**: Uses **rule-based NLP** (no external AI APIs):
- Regex for capitalized name patterns
- Pre-compiled database of Indian public figures
- Pre-compiled database of Indian locations

```python
# Pattern for names like "Narendra Modi"
name_pattern = re.compile(r'\b([A-Z][a-z]+(?:\s+[A-Z][a-z]+)+)\b')
```

### Q12: How are Indian claims prioritized?
**A**: 
- Detects Indian context from entities/locations
- Prioritizes Indian news sources: PTI, ANI, The Hindu, NDTV
- Absence of confirmation from Indian sources lowers credibility

---

## 📡 RSS Feed Integration

### Q13: What RSS feeds are used?
**A**: 
**International**:
- Reuters, BBC, AP News, NPR, The Guardian, NY Times

**Indian**:
- The Hindu, Indian Express, NDTV, Times of India, Hindustan Times

### Q14: How are RSS feeds fetched?
**A**: Using `feedparser` library:
```python
import feedparser
feed = feedparser.parse("https://thehindu.com/rss/feed.xml")
articles = [{'title': e.title, 'summary': e.summary} for e in feed.entries]
```

---

## 🔗 URL Analysis

### Q15: How does URL-based article extraction work?
**A**: `ArticleExtractor` class:
1. Validates URL format
2. Checks `robots.txt` for permission
3. Fetches page content using `requests`
4. Parses with `BeautifulSoup` or `newspaper3k`
5. Extracts: title, body, author, date
6. Caches result for determinism

### Q16: What libraries are used for extraction?
**A**:
- `requests` - HTTP fetching
- `BeautifulSoup4` - HTML parsing
- `newspaper3k` - Article extraction (optional)

---

## 🔒 Deterministic Predictions

### Q17: Why are predictions deterministic?
**A**: For academic validity and reproducibility:
- Same input → Same output (every time)
- All random seeds fixed: `numpy`, `random`, `torch`
- Models loaded once at startup (singleton pattern)
- Article extraction cached

```python
def set_all_seeds(seed=42):
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
```

---

## 🛠️ Technology Stack

### Q18: What is the complete tech stack?
**A**:

| Category | Technology |
|----------|------------|
| **Backend** | Python 3.8+, Flask |
| **ML Framework** | scikit-learn, numpy |
| **NLP** | NLTK, TF-IDF |
| **Web Scraping** | BeautifulSoup4, requests |
| **RSS Parsing** | feedparser |
| **Frontend** | HTML5, CSS3, Bootstrap 5 |
| **Visualization** | JavaScript, Chart.js |

### Q19: What are the key Python files?
**A**:
| File | Purpose |
|------|---------|
| `app.py` | Flask API endpoints |
| `preprocessing/text_cleaner.py` | Text preprocessing |
| `models/classical_models.py` | ML model training |
| `credibility/source_scorer.py` | Source reputation |
| `credibility/claim_extractor.py` | Entity extraction |
| `credibility/claim_verifier.py` | Claim verification |
| `credibility/rss_analyzer.py` | RSS similarity |
| `utils/article_extractor.py` | URL extraction |

---

## ⚠️ Limitations & Ethics

### Q20: What are the system limitations?
**A**:
1. **Cannot verify claims outside RSS corpus** - Only searches available feeds
2. **No absolute truth claims** - Probabilistic assessment only
3. **Language limitation** - Primarily English
4. **Recency bias** - RSS feeds have limited history

### Q21: How does it avoid false positives?
**A**:
- Uses multiple signals, not just one model
- Requires confirmation from multiple sources
- Shows "UNVERIFIED" instead of "FALSE" when no evidence
- Displays disclaimer on every result

---

## 🎯 Interview Tips

### Q22: What makes this project unique?
**A**:
1. **Multi-modal input** - Claim, URL, or text
2. **No external AI APIs** - Fully local, privacy-preserving
3. **India-aware** - Prioritizes Indian news sources
4. **Evidence-based** - Shows supporting/contradicting articles
5. **Deterministic** - Reproducible results

### Q23: How would you scale this system?
**A**:
1. Add more RSS feeds
2. Implement caching layer (Redis)
3. Use distributed RSS fetching
4. Add more Indian regional languages
5. Integrate fact-checking databases

### Q24: What improvements could be made?
**A**:
1. Add deep learning models (BERT, RoBERTa)
2. Implement claim paraphrase detection
3. Add temporal analysis (when was claim first seen)
4. Integrate social media signals
5. Add image/video verification

---

## 📝 Quick Reference Card

```
┌─────────────────────────────────────────────────────────┐
│           FAKE NEWS DETECTION - QUICK REFERENCE          │
├─────────────────────────────────────────────────────────┤
│  INPUT MODES:                                             │
│    1. Claim → Entity extraction → RSS search → Verdict   │
│    2. URL → Article extraction → ML analysis → Score     │
│    3. Text → Preprocessing → TF-IDF → Model → Score      │
├─────────────────────────────────────────────────────────┤
│  ML PIPELINE:                                             │
│    Text → Clean → TF-IDF → Logistic Regression → 0.0-1.0 │
├─────────────────────────────────────────────────────────┤
│  CREDIBILITY FORMULA:                                     │
│    Score = (Model×0.5) + (Source×0.3) + (Evidence×0.2)   │
├─────────────────────────────────────────────────────────┤
│  VERDICTS:                                                │
│    LIKELY TRUE | LIKELY FALSE | UNVERIFIED | DISPUTED     │
└─────────────────────────────────────────────────────────┘
```
