<picture>
  <source media="(prefers-color-scheme: dark)" srcset=".github/assets/banner-dark.png">
  <img alt="The University of Chicago, MS in Applied Data Science. ADSP 32014 course project." src=".github/assets/banner-light.png">
</picture>

# NSF Grant Data Analysis: Did the Language of Funded Research Change in 2025?

> **Example project from Autumn 2025.** This is a copy of [rileymo97/NSF](https://github.com/rileymo97/NSF), shown with the authors' work and commit history intact. Their own overview is [REPORT.md](REPORT.md), with method write-ups in [lda_modeling/README.md](lda_modeling/README.md) and [tfidf_semantic_changepoint/README.md](tfidf_semantic_changepoint/README.md). Course staff wrote this README and made the edits listed under [Changes by course staff](#changes-by-course-staff), including the section on [what to learn from this project](#what-to-learn-from-this-project). The project analyses public records of a live political controversy; the questions, the framing and the conclusions are the authors'.

In 2025 the National Science Foundation terminated more than a thousand active awards, and a list of words that federal grant writers were said to avoid circulated widely. The team asked whether any of this is visible in the text of the awards themselves. On about 82,000 NSF award abstracts from 2019 to 2025 they tracked research topics by division, measured the prevalence of the flagged words, fit a Bayesian changepoint model to the year-by-year drift of the abstracts' meaning, and modeled which awards were terminated from the words in their abstracts.

## Summary

The abstracts come from NSF's own award-search downloads, one JSON file per award, and are joined to NSF's published list of 1,667 terminated awards. Four analyses follow. First, a Latent Dirichlet Allocation topic model per division, 36 divisions with ten topics each, whose topics the team labelled by hand and then grouped by clustering the topics' keyphrase embeddings, giving yearly trend lines per theme. Second, a TF-IDF index over a published list of about 60 flagged words, which rises slowly from 2019 to 2024 and falls by about 40 percent in 2025. Third, the drift of each year's mean abstract embedding from the pre-2025 average, fed to a PyMC changepoint model with a discrete prior on the change year; across all divisions the posterior favors "no change" about three to one, for five selected programs it favors 2025 about two to one, and for Polar Programs it puts essentially all its mass on 2025. Fourth, a Bayesian logistic regression of termination on 100 singular-value components of the abstracts' TF-IDF, whose largest coefficients point at education and outreach awards, social-science and community research, and awards whose abstract text had been replaced by NSF boilerplate, with fellowships and lab-science awards on the safe side. A Gradio app on Hugging Face rewrites an abstract to avoid the flagged words.

## Team and contributions

Team "Softmax Squad", Autumn 2025.

- **Riley Morrison** ([@rileymo97](https://github.com/rileymo97)): the repository and its motivation; the data wrangling notebook with the termination regression; the TF-IDF, semantic-drift and changepoint analysis, as a notebook and as a package.
- **Evan Fantozzi** ([@evanfantozzi](https://github.com/evanfantozzi)): the LDA pipeline as a package with the uv environment; the abstract rewriter on Hugging Face Spaces; the package READMEs.
- **Teddy Kolios** ([@teddykolios](https://github.com/teddykolios)): the subtopic clustering of LDA topics and the yearly trend graphs; the LDA README.

The project ran from November 16 to December 10, 2025. The attributions follow the [commit history](https://github.com/uchicago-dsi-bayesianml/au25-nsf-grant-language/commits/main).

## Getting started

The environment is declared in `pyproject.toml` and pinned in `uv.lock` for Python 3.12 or later, but it covers the text pipeline only: PyMC and ArviZ, which the changepoint and regression code import, are not in it, and the notebooks ran in other kernels. Add them.

```sh
uv sync
uv add pymc arviz
```

Get the data as described under [Data](#data). The LDA pipeline reads three settings from a `.env` file or the environment and does not default all of them:

```sh
cat > .env <<'ENV'
DATA_PATH=data
STOPWORDS_PATH=models/stopwords/english
NLP_PATH=models/en_core_web_sm/en_core_web_sm-3.8.0
ENV
uv run python -m lda_modeling.main
```

That writes `data/grants_df.json`, the topic tables and assignments, and the trend plots, training 36 LDA models along the way. The team then labelled the topics by hand in a copy of the topic spreadsheet named `lda_topics_by_division_labeled.xlsx`, which the clustering step picks up if present. The second package reads `data/grants_df.json`:

```sh
uv run python -m tfidf_semantic_changepoint.main
```

Its first run embeds every abstract with a sentence transformer, about 35 minutes on a laptop CPU. The notebooks under `notebooks/` are the exploratory versions of the same steps; `data_wrangling.ipynb` holds the termination regression, whose sampling took 25 minutes. The rewriter app needs a Google Gemini key in `API_KEY` and the flagged-word list in `BANNED_WORDS`.

## Data

Data files are never committed to this repository, with one small exception noted below.

- **NSF awards 2019 to 2025:** the [award-search download](https://www.nsf.gov/awardsearch/download-awards) files, one JSON per award, a work of the United States government and in the public domain. Place each year's files in `data/2019` through `data/2025`; git ignores those folders.
- **Terminated awards:** NSF's published list of terminated awards, exported on June 5, 2025, kept in `data/NSF-Terminated-Awards.csv` (335 KB): award ID, directorate, recipient, title and obligated amount for 1,667 awards, 805 of them in the education directorate and 296 in social, behavioral and economic sciences.
- **Flagged words:** the list in `models/banned_words.py`, with its source cited in the file.
- **Models:** the spaCy `en_core_web_sm` 3.8.0 model is vendored under `models/` with its MIT license, and the NLTK English stopword list next to it.

## Project structure

| Path | Contents |
|---|---|
| `REPORT.md` | The authors' overview: motivation, the three tools, repository layout. |
| `lda_modeling/` | Loading, cleaning, tokenizing with bigrams and trigrams, lemmatizing, one LDA model per division, Excel export, subtopic clustering and trend plots; its `README.md` is the method write-up. |
| `tfidf_semantic_changepoint/` | TF-IDF of the flagged words and the index, year-centroid similarity heatmaps, semantic drift, the PyMC changepoint model; its `README.md` is the method write-up. |
| `notebooks/data_wrangling.ipynb` | Loading, the termination flag, TF-IDF with 100 singular-value components, and the Bayesian logistic regression of termination with its interpretation. |
| `notebooks/lda_model_generation.ipynb`, `subtopic_clustering.ipynb`, `tfidf_semantic_similarity_and_changepoint.ipynb` | The exploratory versions of the two packages, with outputs. |
| `rewrite_nsf_abstract/` | The Gradio app, with its own README and requirements. |
| `screenshots/` | The figures used in the READMEs. |
| `data/`, `models/` | See [Data](#data). |

## Method

**Topics.** Abstracts are tokenized, phrased into bigrams and trigrams, lemmatized to nouns, adjectives, verbs and adverbs, and filtered to words in at least two and at most half of a division's documents. Each division with at least 50 awards gets its own gensim LDA model with ten topics, Dirichlet hyperparameters alpha 0.1 and eta 0.05, 50 passes. The team named each topic by hand. Topic keyphrases are embedded with all-mpnet-base-v2, averaged into a topic vector, and clustered within each division by Ward linkage on cosine distance with a threshold of 0.15; the resulting clusters' shares of documents are plotted by year.

**Flagged words.** TF-IDF restricted to the flagged-word vocabulary, unigrams to trigrams, averaged per year; the index is the mean over words.

**Semantic drift.** Each abstract is embedded with all-mpnet-base-v2; each year's mean vector is compared by cosine distance to the mean of the 2019 to 2024 vectors, giving one drift value per year. The same is done within selected programs.

**Changepoint model.** With $y_t$ the drift in year $t$,

$$\tau \sim \text{DiscreteUniform}(2019, 2025), \quad \mu_{\text{pre}}, \mu_{\text{post}} \sim \mathcal{N}(0, 1), \quad \sigma \sim \text{HalfNormal}(1), \quad y_t \sim \mathcal{N}(\mu_{\text{pre}} \text{ if } t < \tau \text{ else } \mu_{\text{post}},\ \sigma).$$

PyMC samples $\tau$ by Metropolis and the rest by NUTS, four chains of 2,000 draws after 2,000 tuning steps. Because no year precedes 2019, $\tau = 2019$ means every year is in the "post" regime, so that value is the model's way of saying there is no changepoint.

**Termination regression.** TF-IDF of unigrams and bigrams over all abstracts, reduced to 100 components by truncated SVD; a logistic regression of the termination flag on the components with $\mathcal{N}(0, 1)$ priors on the coefficients and $\mathcal{N}(0, 2)$ on the intercept, NUTS, four chains of 1,000 draws. Components are read through their highest-weighted terms.

### What is Bayesian here

Three pieces. LDA is a Bayesian generative model of text: a Dirichlet prior over each document's topic mixture and over each topic's word distribution, with the hyperparameters 0.1 and 0.05 encoding the belief that documents use few topics and topics use few words; gensim fits it by online variational Bayes, and the posterior topic shares are what the trend plots aggregate. The changepoint model is the course's changepoint material in its simplest form: a uniform prior over the change year, a posterior over it that can be read directly as a histogram, and a boundary value that stands for "no change". The termination model is a Bayesian logistic regression, with priors that regularize 100 coefficients fit to 82,000 documents. What is not Bayesian is also clear: TF-IDF, sentence embeddings and Ward clustering are the feature engineering around the models, and the team says so.

## Results

From the notebooks and the authors' figures.

| Analysis | Result |
|---|---|
| Flagged-word index | About 0.0080 in 2019, rising to about 0.0095 in 2024, about 0.0056 in 2025. |
| Semantic drift, all divisions | 0.0004 to 0.0029 for 2019 to 2024; 0.0077 for 2025. |
| Semantic drift, five selected programs | 0.0019 to 0.0096 for 2019 to 2024; 0.0258 for 2025. |
| Changepoint posterior, all divisions | About three quarters on 2019, the no-change value; about a quarter on 2025. |
| Changepoint posterior, five selected programs | About two thirds on 2025; the rest on 2019. |
| Changepoint posterior, Polar Programs | Essentially all on 2025. |
| Termination regression, largest coefficients | Boilerplate-abstract and quantum component +13.1; education and low-income outreach +9.5; social science, policy and community research +8.4; graduate fellowships −10.3; AI, robotics and cybersecurity workshops −6.6. |

The LDA trend plots show, for example, conservation biology and water topics shrinking within International Science and Engineering, and atmospheric monitoring giving way to cloud computing within Research on Innovative Technologies. The year-centroid similarity heatmaps all sit between 0.97 and 0.999, with 2025 the darkest row in every division shown.

## What to learn from this project

Course staff added this section. The question is real, the data work is substantial and the three models are the right kinds; the points below are about how the changepoint evidence was constructed and read.

- **Don't put the test year outside its own baseline.** Drift is measured from the mean of 2019 to 2024, so each of those years is inside the average it is compared with and 2025 is not. That alone pushes 2025's drift up. Year-over-year drift, or a leave-one-year-out baseline, would put every year on the same footing.
- **Seven points, priors on the wrong scale, warnings ignored.** Each changepoint fit sees seven numbers of order one thousandth under priors with scale one, and two of the three runs reported divergences, R-hat above 1.01 and low effective sample sizes. Rescale the data or the priors, raise the target acceptance, and say what the warnings mean before showing the histogram.
- **Read the posterior you plotted.** Across all divisions the model puts most of its mass on 2019, which is its no-change answer, yet the figure is presented alongside the others without comment. The honest summary is that the pooled series shows no clear break and selected programs do, and that is a more interesting finding than a uniform 2025 verdict.
- **Rule out the boring explanations.** 2025 is a partial year, and the mix of divisions and programs funded in it differs from earlier years; either would move the mean embedding without any change in how abstracts are written. Drift within fixed programs, which the team computed, is the right control, and it deserved to be the headline.
- **Check the word list.** The flagged-word vocabulary includes "community", "status", "increase", "groups" and "political", which carry most of the TF-IDF mass for reasons unrelated to the controversy. An index built on the specific terms would be more convincing, and so would a per-word plot of the ten biggest movers.
- **Give the regression its due.** The termination model is the most direct answer to the project's question and it lives only in a notebook, reports posterior means without intervals, and puts coefficients on unscaled components. A results table with intervals, in the README, would have anchored the write-up.
- **Small things.** The lockfile lacks the Bayesian dependencies; the pipeline needs undocumented environment variables; a 15 MB spaCy model is vendored; ten topics per division were fixed without a coherence check; Ward linkage assumes Euclidean distances; the figure titled "4 selected programs" pools five; the README has methods and figures but no findings section; no license.

What the project does well: a question that matters, asked of the full public record rather than a sample; three complementary models that answer different parts of it; topics labelled by a person rather than left as word lists; the no-change value of the changepoint prior handled correctly in the model, if not in the reading; code organized as packages with a lockfile and per-package documentation; figures that make the findings legible at a glance; a demo anyone can try; the flagged-word list cited to its source; and three contributors whose work is visible in the history.

## About this project

This project was built in Bayesian Machine Learning with Generative AI Applications (ADSP 32014), part of the Master of Science in Applied Data Science at the University of Chicago, in Autumn 2025. It is shown here as an example for later cohorts.

## Changes by course staff

Course staff forked the authors' repository and made these edits. The authors' code, notebooks, packages and data are unchanged.

- Renamed the authors' top-level README to `REPORT.md`, unchanged, and added this README with the program banner. The "Back to Main README" links in the two package READMEs now land here; the authors' overview is `REPORT.md`.

## License

The authors have not added a license, so their work is shown here under GitHub's terms for public repositories. Ask them before reusing it. The NSF award data is in the public domain; the vendored spaCy model is MIT licensed.
