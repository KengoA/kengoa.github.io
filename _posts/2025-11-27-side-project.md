---
layout: post
title:  "Side Project of a Side Project"
date:   2025-11-27 21:10:00 +0100
category: software
feature: true
---

Before switching to a platform engineering role within the AI Engineering Group earlier this year, I spent 9 months working on RAG on financial research documents.

The main lesson I personally learnt from this experience is that **you have to solve retrieval first before you solve RAG**. To achieve a desired user experience, you need to own (or at least be able to contribute to) the entire pipeline from indexing to retrieval to reranking to generation, as opposed to simply slapping a generation step on top of an existing search system.

In this post, I will describe my journey of trying to set up a search pipeline for company information on UK startups, and how I somehow ended up building a Python client for Apache Solr.

<h2>The Side Project</h2>

I initially set out to build a sandbox environemnt where I could tune different knobs of the search pipeline with various sets of requirements. For a given retrieval problem, you should first think about the requirements and nature of the document domain such as:

- How often do I need to ingest new documents?
- How fast should search be?
- What is the distribution of keywords like in the document domain?

Answers to these questions will influence the system architecture, often with tradeoffs across different components.

For instance, the frequency and lengths of new documents informs whether we can apply compute-heavy document understanding or enrichment steps. We cannot spend tens of seconds trying to ingest a single document if new documents keep arriving while processing it, or that the search system needs to expose real-time data such as news events.

If this requirement is not strict, on the other hand, we can potentially take time on enriching documents with rephrasing some of the keywords and set up specific fields to search over, this could help keyword-based retrieval systems with BM25, helping with the latency and relevance concerns.

The distribution of keywords also has consequences on we handle ambiguity and vocabulary mismatch between the user query space and the document domain, with implications on the choices of the embeding algorithm, vector search databses and vector search algorithms.

Specifically, what I wanted to build was a better search system for company information on UK startups using [Document API from Companies House](https://developer-specs.company-information.service.gov.uk/document-api/reference/document-location/fetch-a-document). In this specific setup, the available data is almost always in a PDF format and that the frequency of document updates is generally low, say, no more than one document per day per company.

These characteristics imply that **(1)** we need to think of a document understanding step from PDF to some form of structure that can be indexed into the search system and that **(2)** we can take some time to run this step, opening up the possibility of using something more advanced than a simple PDF parsing such as vision-langauge model based [SmolDocling](https://arxiv.org/abs/2503.11576).


<h2>Enterprise Search 101</h2>

With the exception of Meilisearch which is built in Rust, the mainstream enterprise search options are ElasticSearch, OpenSearch, and Apache Solr which are all built on Apache Lucene, and the differences arise in the query interface and cluster coordination along with sharding strategies.

<h2>Missing Prerequisites</h2>

- Fast ingestion pipeline
- Reproducible, easy-to-document retrieval setup

This HackerNews comment from [tekkk on Aug 16, 2020](https://news.ycombinator.com/item?id=24178656) particularly resonates with my experience:

> Solr is one of those technologies which works but isn't really glorious to use and is bit stuffy with its XML configurations and Java interfaces. It's a bit of shame, because search engines are so popular nowadays and everybody seems to be fixated on using ElasticSearch. Which from what I've read and heard is resource-hungry and not really cut for simple text-search.

Note that this comment predates LLM where the interest in RAG

### Limitations of pysolr

While pysolr, which is implemented in [one 1500-line Python file](https://github.com/django-haystack/pysolr/blob/master/pysolr.py), has been around for well over a decade and remains the de-facto official client for interacting with Apache Solr, it falls short of modern Python client expectations. 

Notably, pysolr lacks support for asynchronous operations, making concurrent queries inefficient compared to libraries that leverage Python’s async features. Indexing documents is also less ergonomic, and there is no built-in type safety, which can lead to runtime errors and makes integration with typed codebases more challenging.

Another source of confusion for new users is that pysolr exposes Solr’s query syntax directly. Parameters like <b>q</b>, <b>fl</b>, and <b>qf</b> are rather terse abbreviations, and the relationship between query parameters and query parsers is not clearly documented or enforced by the client. This can make it difficult to understand the hierarchy and structure of queries, especially for those unfamiliar with Solr’s internals.

Although the official documentation is comprehensive, it's not obvious what the strucutual relationship is between different query parsers and parameters. If you define 

<h2>Writing Python Client for Solr</h2>

<h3>Implications for Agentic Workflows</h3>

```python

```

[Taiyo]: https://taiyoproj.github.io
[GitHub]: https://github.com/taiyoproj/taiyo
[Solr Query Guide]: https://solr.apache.org/guide/solr/latest/query-guide/query-syntax-and-parsers.html
