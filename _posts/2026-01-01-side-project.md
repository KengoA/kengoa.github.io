---
layout: post
title:  "Side Project of a Side Project"
date:   2026-01-01 00:00:00 +0100
category: software
feature: true
---

Before switching to a platform engineering role within the AI Engineering Group earlier this year, I spent 9 months working on RAG with financial research documents.

The main lesson I personally learnt from this experience is that **you have to solve retrieval first before you solve RAG**. To achieve a desired user experience, you need to own (or at least be able to contribute to) the entire pipeline from indexing to retrieval to reranking to generation, rather than simply adding a generation step on top of an existing search system.

In this post, I will describe my personal journey of trying to build a search system for company documents in the UK, and how I ended up writing a Python client for Apache Solr.

<h2>Search Pipeline</h2>

With the above lesson in mind, I thought about building a sandbox environment where I could tune different knobs of the search pipeline with various sets of requirements. As a starting point, for example, I'd ask myself:

- How often do I need to ingest new documents?
- How fast should search be?
- What is the distribution of keywords like in the document domain?

Answers to these questions will influence the system architecture, often with tradeoffs across different components.

For instance, the frequency and lengths of new documents inform whether we can apply compute-heavy document understanding or enrichment steps. We cannot spend multiple seconds ingesting a single document if new documents keep arriving while processing it, or if the search system needs to expose real-time data such as news events.

If this requirement is not strict, then we can potentially afford to extract metadata from documents such as named entity recognition or rephrasing some of the keywords and setting up specific fields to search over for BM25, which would improve both latency and relevance for sparse retrieval.

Keyword distribution also has consequences on how we handle ambiguity and vocabulary mismatch between the user query space and the document domain, with implications for the choice of the embedding algorithm, vector search databases, and vector search algorithms.

<h2>The Side Project</h2>

The actual project I wanted to work on was a better search system for company information on UK startups using the [Document API](https://developer-specs.company-information.service.gov.uk/document-api/reference/document-location/fetch-a-document) from Companies House [^1]. The main motivation for this side project was based on my personal experience that startups are generally much less transparent when things are not going well, and that the current user experience of Companies House is not ideal in terms of document discoverability and readability.

Providing a better search system for such information could potentially help alleviate the pain point of not having enough realistic information on company financials and governance structures prior to joining a startup. This assumes the existence of documents disclosing such information in the Companies House database already, which is sometimes the case after startups raise Series A or Series B rounds which is when they typically accelerate their hiring efforts.

In Companies House, the documents are almost always indexed as PDFs and that the frequency of document updates is generally low, say, no more than one document per day per company. These facts imply that **(1)** we need to have a document understanding step from PDF to some form of structure that can be indexed into the search system and that **(2)** we can take some time to run this step, opening up the possibility of using something more advanced than simple PDF parsing, such as vision-language model-based [SmolDocling](https://arxiv.org/abs/2503.11576).

With those requirements, I came up with a rough system design for necessary components for setting up the search pipeline, mostly around ingestion.

<img src="/images/system.png" alt="system" class="article-img">

<h2>Searching for Search Servers</h2>

I started to look for the libraries and technologies to fill in those components in the diagram above, and considered multiple options for the search server. With the exception of Meilisearch which is built in Rust, the mainstream enterprise search options are ElasticSearch, OpenSearch, and Apache Solr, which are all built on Apache Lucene. I had experience using Apache Solr from the previous RAG project, and decided to stick to it after seeing that OpenSearch and ElasticSearch are not fundamentally too different in terms of functionalities and interfaces [^2].

This HackerNews comment from [tekkk on Aug 16, 2020](https://news.ycombinator.com/item?id=24178656) echoes my sentiment on Solr:

> Solr is one of those technologies which works but isn't really glorious to use and is a bit stuffy with its XML configurations and Java interfaces. It's a bit of a shame, because search engines are so popular nowadays and everybody seems to be fixated on using ElasticSearch, which from what I've read and heard is resource-hungry and not really suited for simple text-search.

and note that this comment predates the rise of LLMs which subsequently led to the popularity in RAG and has since invited a number of ML practitioners who are typically not familiar with the Java ecosystem, including myself. 

This I believe is where the first friction comes in where the interface becomes the main bottleneck for developing a search system.
For example, the _de facto_ official Python client for Apache Solr is _pysolr_, which has been around for well over a decade and is implemented in [one 1500-line Python file](https://github.com/django-haystack/pysolr/blob/master/pysolr.py).

This unfortunately falls short of expectations for a modern Python experience. For instance, pysolr does not support async operations at all, making concurrent queries inefficient compared to libraries that leverage Python’s async features. Indexing documents is also less ergonomic, and there is no built-in type safety, which can lead to runtime errors and make integration with typed codebases more challenging.

Another source of confusion for new users is that pysolr exposes Solr’s query syntax directly. Parameters like <b>q</b>, <b>fl</b>, and <b>qf</b> are rather terse abbreviations, and the relationship between query parameters and query parsers is not clearly documented or enforced by the client. This can make it difficult to understand the hierarchy and structure of queries, especially for those unfamiliar with Solr’s internals.

Let's take a look at an actual example. As a prerequisite, Solr's search request is constructured as HTTP query parameters and is sent as a GET request to a given collection. In pysolr, those are set directly as Python dictionaries as follows.

```python
from pysolr import Solr

solr = Solr('http://localhost:8983/solr/my_core/', timeout=10)

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

There's no contract on what the returned response will be, and it's hard to understand all the fields straight away wit






Going back to the Extended DisMax query example. Taiyo now allows you to set up a query parser like the following:

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

With `GroupParamsConfig` object passed in as a config object.

Alternatively, it's also possible to set up the same exact query with a pandas-like chaining on the parser object:
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

The usage can be of either personal preference or based on practical concerns, such as how complicated your configuration will be. If you need a highly custom/multiple layers of configuration, you might want to opt for config objects that can be set up in different places (i.e in a iteration loop or loaded from a separate file).

## Making something I want

Aside from returning to the actual side project I set out to do, there are several lines of work planned, including supporting more query parsers such as XML parsers, along with clearer documentation.

Also note that this client doesn't abstract away the Lucene query syntax in the query parameter, which could be provided as a utility function in this library, should I find an intuitive interface.

The inevitable next step is to build out the original side project I set out to work on, which is to build a better full-text search system for company documents. The best thing about this client is that it would not be the end of the world if nobody finds it useful, as long as it helps me on my original journey and I can find value in it myself.

YC's famous motto _Make something people want_ [^3] is great advice for building venture-scale companies, but making something _I_ want is more than enough as a heuristic for building side projects, which was definitely the case for [Visprex] and [Taiyo].

---
[^1]: Here I focus on the UK as I'm based in London and more familiar with the UK startup ecosystem
[^2]: The query language is the same across all three, which is [Lucene query language].
[^3]: More details on Paul Graham's essay [Be Good](https://paulgraham.com/good.html)

[Taiyo]: https://taiyoproj.github.io
[GitHub]: https://github.com/taiyoproj/taiyo
[Solr Query Guide]: https://solr.apache.org/guide/solr/latest/query-guide/query-syntax-and-parsers.html

