# Elastic

The Elastic module is a wrapper for the [Elasticsearch](https://elasticsearch-py.readthedocs.io/en/v8.0.0a1/)
Python client. This module allows us to use our Elasticsearch server to integrate it into a neural search pipeline.

To use Elasticsearch as a retriever, we need to install Elasticsearch and start the server. Pieces of information are available [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html).

We can find all the information associated with the `es` parameter, which establishes the connection to your Elasticsearch session on the [official documentation](https://elasticsearch-py.readthedocs.io/en/v8.0.0a1/api.html#module-elasticsearch).

Once our Elasticsearch server is up and running, we can connect with Cherche to add documents to the index.

The `retrieve.Elastic` retriever is the right solution to implement a neural search pipeline on a large corpus. The retriever allows the documents and pre-computed embeddings of the ranker to be indexed on Elasticsearch to avoid overloading the RAM.

`retrieve.Elastic` has two methods to index new documents. These two methods are compatible with mini-batch indexing. The first `add` and the second `add_embeddings` methods are dedicated to indexing documents and eventually embeddings of a dedicated ranker.

## Elastic retriever

```python
>>> from cherche import retrieve
>>> from elasticsearch import Elasticsearch

>>> documents = [
...    {
...        "id": 0,
...        "article": "Paris is the capital and most populous city of France",
...        "title": "Paris",
...        "url": "https://en.wikipedia.org/wiki/Paris"
...    },
...    {
...        "id": 1,
...        "article": "Paris has been one of Europe major centres of finance, diplomacy , commerce , fashion , gastronomy , science , and arts.",
...        "title": "Paris",
...        "url": "https://en.wikipedia.org/wiki/Paris"
...    },
...    {
...        "id": 2,
...        "article": "The City of Paris is the centre and seat of government of the region and province of Île-de-France .",
...        "title": "Paris",
...        "url": "https://en.wikipedia.org/wiki/Paris"
...    }
... ]

# Elasticsearch client
>>> es = Elasticsearch(hosts="localhost:9200")

# Ask to cherche on title and article, retrieves the top 30 results, uses and create the index cherche if it does not exist.
>>> retriever = retrieve.Elastic(key="id", on=["title", "article"], k=30, es=es, index="cherche")

>>> retriever.add(documents=documents)

>>> retriever(q="science")
[{'id': 1,
  'article': 'Paris has been one of Europe major centres of finance, diplomacy , commerce , fashion , gastronomy , science , and arts.',
  'title': 'Paris',
  'url': 'https://en.wikipedia.org/wiki/Paris',
  'similarity': 0.94782937}]
```

Warning, you can empty the index using:

```python
retriever.reset()
```

## Write your own query

Using `retrieve.Elastic`, we can customize the query to fit our needs.

```python
>>> from cherche import retrieve
>>> from elasticsearch import Elasticsearch

>>> documents = [
...    {
...        "id": 0,
...        "article": "Paris is the capital and most populous city of France",
...        "title": "Paris",
...        "url": "https://en.wikipedia.org/wiki/Paris"
...    },
...    {
...        "id": 1,
...        "article": "Paris has been one of Europe major centres of finance, diplomacy , commerce , fashion , gastronomy , science , and arts.",
...        "title": "Paris",
...        "url": "https://en.wikipedia.org/wiki/Paris"
...    },
...    {
...        "id": 2,
...        "article": "The City of Paris is the centre and seat of government of the region and province of Île-de-France .",
...        "title": "Paris",
...        "url": "https://en.wikipedia.org/wiki/Paris"
...    }
... ]

# Elasticsearch client
>>> es = Elasticsearch(hosts="localhost:9200")

# Ask to cherche on title and article, retrieves the top 30 results, uses and create the index cherche if it does not exist.
>>> retriever = retrieve.Elastic(key="id", on=["title", "article"], k=30, es=es, index="cherche")

>>> q = "science"

>>> query = {
...    "query": {
...        "multi_match": {
...            "query": q,
...            "type": "most_fields",
...            "fields": ["title", "article"],
...            "operator": "or",
...        }
...    },
... }

>>> retriever.add(documents=documents)

>>> retriever(**{"q": q, "query": query})
[{'id': 1,
  'article': 'Paris has been one of Europe major centres of finance, diplomacy , commerce , fashion , gastronomy , science , and arts.',
  'title': 'Paris',
  'url': 'https://en.wikipedia.org/wiki/Paris',
  'similarity': 0.9949781}]
```

## Upload documents and embeddings on Elasticsearch

When working with many documents, we need to store the documents and embeddings on disk rather than in RAM. To do this, we can declare an Elastic retriever and index documents and embeddings in Elasticsearch.

It is mandatory to have a GPU to calculate embeddings to index millions of documents.

```python
>>> from cherche import retrieve, rank
>>> from sentence_transformers import SentenceTransformer
>>> from elasticsearch import Elasticsearch

>>> documents = [
...    {
...        "id": 0,
...        "article": "Paris is the capital and most populous city of France",
...        "title": "Paris",
...        "url": "https://en.wikipedia.org/wiki/Paris"
...    },
...    {
...        "id": 1,
...        "article": "Paris has been one of Europe major centres of finance, diplomacy , commerce , fashion , gastronomy , science , and arts.",
...        "title": "Paris",
...        "url": "https://en.wikipedia.org/wiki/Paris"
...    },
...    {
...        "id": 2,
...        "article": "The City of Paris is the centre and seat of government of the region and province of Île-de-France .",
...        "title": "Paris",
...        "url": "https://en.wikipedia.org/wiki/Paris"
...    }
... ]

# Elasticsearch client, in localhost here but it could be done remotly.
>>> es = Elasticsearch(hosts="localhost:9200")

>>> retriever = retrieve.Elastic(key="id", on=["title", "article"], k=30, es=es, index="cherche")

>>> ranker = rank.Encoder(
...    key="id",
...    on = ["title", "article"],
...    encoder = SentenceTransformer("sentence-transformers/all-mpnet-base-v2").encode,
...    k = 10,
... )

# Store embeddings and documents in Elasticsearch
>>> retriever.add_embeddings(documents=documents, ranker=ranker)

>>> search = retriever + ranker

>>> search("science")
[{'id': 1,
  'article': 'Paris has been one of Europe major centres of finance, diplomacy , commerce , fashion , gastronomy , science , and arts.',
  'title': 'Paris',
  'url': 'https://en.wikipedia.org/wiki/Paris',
  'similarity': 0.0890004881677473}]
```
