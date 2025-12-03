---
layout: post
title:  "Side Project of a Side Project"
date:   2025-11-27 21:10:00 +0100
category: software
feature: true
---

Before switching to a platform engineering role within the [AI Group]() earlier this year, I spent 9 months working on [RAG]() on financial research documents. I learnt a lot about enterprise search during this time, which is built around open-source software of multiple layers of abstraction. 

One key takeaway from this experience is that <b><i>you have to solve retrieval first before you solve RAG</i></b>. To achieve a desired user experience, you need to be able to own and tune the entire pipeline from indexing to retrieval to reranking to generation, as opposed to slapping a generation step on top of an existing search system. A similar sentiment is echoed in [this blog article](). In this post, I will describe my journey trying to experiment with my own search pipeline from scratch, and how I ended up building and open-sourcing a Python client for Apache Solr called [Taiyo](https://github.com/taiyoproj/taiyo).

<h2>The Side Project</h2>

What I wanted to experiment was to set up a sandbox environemnt where I could tune different knobs of the pipeline where I can assume different requirements. (how often do I need to ingest new documents, latency requirements for retrieval).

Having control over these choices let me iterate quickly and see firsthand how small changes at each layer could impact the overall experience.

<h2>Why Solr?</h2>
With the exception of Meilisearch, the mainstream options of ElasticSearch, OpenSearch, and Apache Solr are all built on Apache Lucene, and the differences.

<h2>Missing Prerequisites</h2>

- Fast ingestion pipeline
- Reproducible, easy-to-document retrieval setup

This HackerNews comment from [tekkk on Aug 16, 2020](https://news.ycombinator.com/item?id=24178656) particularly resonates with my experience:

> Solr is one of those technologies which works but isn't really glorious to use and is bit stuffy with its XML configurations and Java interfaces. It's a bit of shame, because search engines are so popular nowadays and everybody seems to be fixated on using ElasticSearch. Which from what I've read and heard is resource-hungry and not really cut for simple text-search.

Note that this comment predates LLM

### Limitations of pysolr

While pysolr has been around for over a decade [^1] and remains the de-facto official client for interacting with Apache Solr, it falls short of modern Python client expectations. 

Notably, pysolr lacks support for asynchronous operations, making concurrent queries inefficient compared to libraries that leverage Python’s async features. Indexing documents is also less ergonomic, and there is no built-in type safety, which can lead to runtime errors and makes integration with typed codebases more challenging.

Another source of confusion for new users is that pysolr exposes Solr’s query syntax directly. Parameters like <b>q</b>, <b>fl</b>, and <b>qf</b> are terse abbreviations, and the relationship between query parameters and query parsers is not clearly documented or enforced by the client. This can make it difficult to understand the hierarchy and structure of queries, especially for those unfamiliar with Solr’s internals.

The official documentation is comprehensive, it's not obvious what the strucutual relationship is between different query parsers and parameters. If you define 

<h2>Writing Python Client for Solr</h2>

<h3>Implications for Agentic Workflows</h3>

```python

```

[Taiyo]: https://taiyoproj.github.io
[GitHub]: https://github.com/taiyoproj/taiyo
[Solr Query Guide]: https://solr.apache.org/guide/solr/latest/query-guide/query-syntax-and-parsers.html

---

[^1]: pysolr is implemented in pysolr is in [one 1500-line Python file](https://github.com/django-haystack/pysolr/blob/master/pysolr.py).