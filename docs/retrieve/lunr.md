# Lunr

`retrieve.Lunr` is a wrapper of [Lunr.py](https://github.com/yeraydiazdiaz/lunr.py). It is a powerful and practical solution for searching inside a corpus of documents without using a retriever such as Elasticsearch when it is not needed. Lunr stores an inverted index in memory.

```python
>>> from cherche import retrieve

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

>>> retriever = retrieve.Lunr(key="id", on=["title", "article"], documents=documents, k=30)

>>> retriever("france")
[{'id': 0, 'similarity': 0.605}, {'id': 2, 'similarity': 0.47}]
```

## Map keys to documents

```python
>>> retriever += documents
>>> retriever("france")
[{'id': 0,
  'article': 'Paris is the capital and most populous city of France',
  'title': 'Paris',
  'url': 'https://en.wikipedia.org/wiki/Paris',
  'similarity': 0.605},
 {'id': 2,
  'article': 'The City of Paris is the centre and seat of government of the region and province of Île-de-France .',
  'title': 'Paris',
  'url': 'https://en.wikipedia.org/wiki/Paris',
  'similarity': 0.47}]
```
