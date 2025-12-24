---
layout: post
title:  "Side Project of a Side Project"
date:   2025-11-27 21:10:00 +0100
category: software
feature: true
---

Before switching to a platform engineering role within the AI Engineering Group earlier this year, I spent 9 months working on RAG on financial research documents.

The main lesson I personally learnt from this experience is that **you have to solve retrieval first before you solve RAG**. To achieve a desired user experience, you need to own, or at least be able to contribute to, the entire pipeline from indexing to retrieval to reranking to generation, as opposed to simply slapping a generation step on top of an existing search system.

In this post, I will describe my journey of trying to set up a search pipeline for company information on UK startups, and how I somehow ended up building a Python client for Apache Solr.

<h2>The Side Project</h2>

With the above lesson in mind, I thought about building a sandbox environment where I could tune different knobs of the search pipeline with various sets of requirements. For a given retrieval problem, you should first think about the requirements and nature of the document domain such as:

- How often do I need to ingest new documents?
- How fast should search be?
- What is the distribution of keywords like in the document domain?

Answers to these questions will influence the system architecture, often with tradeoffs across different components.

For instance, the frequency and lengths of new documents informs whether we can apply compute-heavy document understanding or enrichment steps. We cannot spend tens of seconds trying to ingest a single document if new documents keep arriving while processing it, or that the search system needs to expose real-time data such as news events.

If this requirement is not strict, on the other hand, we can potentially take time on enriching documents with rephrasing some of the keywords and set up specific fields to search over, this could help keyword-based retrieval systems with BM25, helping with the latency and relevance concerns.

The distribution of keywords also has consequences on we handle ambiguity and vocabulary mismatch between the user query space and the document domain, with implications on the choices of the embeding algorithm, vector search databses and vector search algorithms.

Specifically, what I wanted to build was a better search system for company information on UK startups using [Document API from Companies House](https://developer-specs.company-information.service.gov.uk/document-api/reference/document-location/fetch-a-document). In this specific setup, the documents are indexed as PDFs and that the frequency of document updates is generally low, say, no more than one document per day per company.

These characteristics imply that **(1)** we need to think of a document understanding step from PDF to some form of structure that can be indexed into the search system and that **(2)** we can take some time to run this step, opening up the possibility of using something more advanced than a simple PDF parsing such as vision-langauge model based [SmolDocling](https://arxiv.org/abs/2503.11576).


<h2>Searching for Search Servers</h2>

With the requirements above, I came up with a rough system design for necessary components for setting up the search pipeline, mostly around ingestion.

<img src="/images/system.png" alt="system" class="article-img">

With the exception of Meilisearch which is built in Rust, the mainstream enterprise search options are ElasticSearch, OpenSearch, and Apache Solr which are all built on Apache Lucene.

This HackerNews comment from [tekkk on Aug 16, 2020](https://news.ycombinator.com/item?id=24178656) particularly resonates with my experience:

> Solr is one of those technologies which works but isn't really glorious to use and is bit stuffy with its XML configurations and Java interfaces. It's a bit of shame, because search engines are so popular nowadays and everybody seems to be fixated on using ElasticSearch. Which from what I've read and heard is resource-hungry and not really cut for simple text-search.

Note that this comment predates LLM where the interest in RAG

### Limitations of pysolr

While pysolr, which is implemented in [one 1500-line Python file](https://github.com/django-haystack/pysolr/blob/master/pysolr.py), has been around for well over a decade and remains the de-facto official client for interacting with Apache Solr, it falls short of modern Python client expectations. 

Notably, pysolr lacks support for async operations, making concurrent queries inefficient compared to libraries that leverage Python’s async features. Indexing documents is also less ergonomic, and there is no built-in type safety, which can lead to runtime errors and makes integration with typed codebases more challenging.

Another source of confusion for new users is that pysolr exposes Solr’s query syntax directly. Parameters like <b>q</b>, <b>fl</b>, and <b>qf</b> are rather terse abbreviations, and the relationship between query parameters and query parsers is not clearly documented or enforced by the client. This can make it difficult to understand the hierarchy and structure of queries, especially for those unfamiliar with Solr’s internals.


<h2>The Side Project ^2</h2>

<h4>Indexing</h4>

```python

import asyncio
from taiyo import AsyncSolrClient, SolrDocument

class MyDocument(SolrDocument):
    title: str

async with AsyncSolrClient("http://localhost:8983/solr") as client:
    client.set_collection("my_collection")

    # Split into batches and process concurrently
    batch_size = 100
    all_docs = [MyDocument(title=f"Doc {i}") for i in range(1000)]
    
    batches = [
      all_docs[i:i + batch_size]
      for i in range(0, len(all_docs), batch_size)
    ]

    # Index all batches concurrently
    await asyncio.gather(*[client.add(batch, commit=False) for batch in batches])
    await client.commit()
```

<h4>Query Parsers</h4>

```python
import pysolr

solr = pysolr.Solr('http://localhost:8983/solr/my_core/', timeout=10)

params = {
    'defType': 'edismax',
    'q': 'python programming',
    'qf': 'title^2.0 content^1.0',
    'group': 'true',
    'group.field': 'author',
    'group.limit': 3,
    'group.ngroups': 'true'
}

results = solr.search(**params)
```


huggingface-like configuration objects

```python
from taiyo.params import GroupParamsConfig
from taiyo.parsers import ExtendedDisMaxQueryParser

parser = ExtendedDisMaxQueryParser(
    query="python programming",
    query_fields={"title": 2.0, "content": 1.0},
    configs=[
      GroupParamsConfig(by="author", limit=3, ngroups=True)
    ],
)
```

pandas-like chaining 
```python
from taiyo.parsers import ExtendedDisMaxQueryParser

parser = ExtendedDisMaxQueryParser(
    query="python programming",
    query_fields={"title": 2.0, "content": 1.0}
).group(
    by="author",
    limit=3,
    ngroups=True
)
```

## Making something I want

Aside from going back to the actual side project I set out to do, there's several lines of work planned, including supporting more query parsers such as XML parsers along with clearer documentation.

Also note that this client doesn't abstract away the Lucene query syntax in the query parameter, which can be provided as a util function in this library, provided I find an intuitive interface.

The inevitable next step that is to build out the original side project I set out to work on which is to build a better full-text search system for company documents. The best thing about this client is that it would not be the end of the world if nobody finds this useful, as long as this helps me on this original journey and I can find value in it myself.

YC's famous motto _Make something people want_ [^1] is a great advice for building venture-scale companies, but making something _I_ want is more than enough as a heuristic for building side projects, which was definitely the case for Visprex and Taiyo.

---
[^1]: More details on Paul Graham's essay [Be Good](https://paulgraham.com/good.html)

[Taiyo]: https://taiyoproj.github.io
[GitHub]: https://github.com/taiyoproj/taiyo
[Solr Query Guide]: https://solr.apache.org/guide/solr/latest/query-guide/query-syntax-and-parsers.html

