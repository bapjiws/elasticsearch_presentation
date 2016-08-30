# <p style="text-align: center;">Elasticsearch as we know it<p>

---

# Some key concepts

**Index** 
> An index is a collection of documents that have somewhat similar characteristics[...] An index is like a *database* in a relational database. It has a mapping which defines multiple types.

**Mapping**
> A mapping is like a *schema definition* in a relational database.

---
# Lucene practical scoring formula

<img src="https://dzone.com/storage/rc-covers/15333-thumb.png" alt="Lucene practical scoring formula" style="height:160px; width: 990px;"/>

- "q" means "query" 
- "d" means document 
- "t" "term"
- "t in q" means "The sum of the weights for each term t in the query q for document d".


---
# Scoring factors

Factor       |	Explanation
------------ | -------------
**coord(q,d) aka "coordination factor"** | "Typically, a document that contains more of the query’s terms will receive a higher score than another document with fewer query terms".
**queryNorm(q) aka "query normalization factor"** |	"...attempts to make scores from different queries (or even different indexes) comparable".

---
# Scoring factors (continued)

Factor       |	Explanation
------------ | -------------
**tf(t in d) aka "term frequency"**	| "Documents that have more occurrences of a given term receive a higher score".
**idf(t) aka "inverse document frequency"**	| "... rarer terms give higher contribution to the total score".
t.getBoost() |	"A search-time boost of term t in the query q."
norm(t,d) aka "field-length norm" | "The shorter the field, the *higher* the weight"

---

# Some key concepts (continued)

**Type**
> A type is like a *table* in a relational database. Each type has a list of fields that can be specified for documents of that type. The mapping defines how each field in the document is analyzed.

**Field**
> A document contains a list of fields, or key-value pairs... A field is similar to a column in a table in a relational database.

---

# Inverted Index
> Elasticsearch uses a structure called an *inverted index*, which is designed to allow very fast full-text searches. An inverted index consists of a list of all the unique words that appear in any document, and for each word, a list of the documents in which it appears.

---
# Sample documents and resulting inverted index

![Inverted Indexes and Index Terms](https://www.elastic.co/assets/bltb03758c3e981d9e4/inverted-index.svg)

---
# Why it's useful

> The inverted index maps *terms* to documents (and possibly positions in the documents) containing the term. Since the terms in the *dictionary* are sorted, we can quickly find a term, and subsequently its occurrences in the *postings*-structure. This is contrary to a "forward index", which lists terms related to a specific document.
> [...] 
> Consequently, an *index term* is the *unit of search*. 

---

> The inverted index allows queries to look up the search term in unique sorted list of terms, and from that immediately have access to the list of documents that contain the term.

> Sorting, aggregations, and access to field values in scripts requires a different data access pattern. Instead of looking up the term and finding documents, we need to be able to look up the document and find the terms that it has in a field.


---

> When you sort on a field, Elasticsearch needs access to the value of that field for every document that matches the query. The inverted index, which performs very well when searching, is not the ideal structure for sorting on field values:

> - When searching, we need to be able to map a term to a list of documents.
> - When sorting, we need to map a document to its terms. In other words, we need to “uninvert” the inverted index.

> This “uninverted” structure is often called a “column-store” in other systems. Essentially, it stores all the values for a single field together in a single column of data, which makes it very efficient for operations like sorting.

> In Elasticsearch, this column-store is known as *doc values*, and is enabled by default. Doc values are created at index-time: when a field is indexed, Elasticsearch adds the tokens to the inverted index for search. But it also extracts the terms and adds them to the columnar doc values.

https://www.elastic.co/guide/en/elasticsearch/guide/current/docvalues-intro.html

---

> Doc values are used in several places in Elasticsearch:

> - Sorting on a field
> - Aggregations on a field
> - Certain filters (for example, geolocation filters)
> - Scripts that refer to fields

---
# Uninverting a field into a field cache

> The inverted index cannot give you the value of a field given a document ID; it's good for finding documents given a value. Therefore, as shown in the figure below, all the documents' values for a field are loaded entirely into memory the first time you try use it for aggregations, sorting or scripting.

![Uninverting a field into a field cache](https://www.elastic.co/assets/bltb6115f73315ac310/uninverting.svg)



---
# The Damerau-Levenshtein distance

> [Elasticsearch] uses the Damerau-Levenshtein distance to find all terms with a maximum of two changes, where a change is the insertion, deletion or substitution of a single character, or transposition of two adjacent characters.

> The default edit distance is 2, but an edit distance of 1 should be sufficient to catch 80% of all human misspellings

---

#  <p style="text-align: center;">The Components of an Analyzer</p>

![ The Components of an Analyzer](https://www.elastic.co/assets/blt51e787daed39eae9/Signatures.svg)

--- 

#  <p style="text-align: center;"> Custom Analysis Flow </p>
<div style="text-align: center;">
<img src="https://www.elastic.co/assets/bltee4e0b427d8fdad4/custom_analyzers_diag.png" alt="Custom Analysis Flow" style="height:550px; width: 350px;"/>
</div>

---
# Standard analyzer

> The standard analyzer is the default analyzer that Elasticsearch uses. It is the best general choice for analyzing text that may be in any language. It splits the text on word boundaries, as defined by the Unicode Consortium, and removes most punctuation. Finally, it lowercases all terms.
<div style="text-align: center;">

```Set the shape to semi-transparent by calling set_trans(5)```
becomes

```set, the, shape, to, semi, transparent, by, calling, set_trans, 5```
</div>

---

# Nested objects

Let's say we have a followers array that looks like this:
```
{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}
```
The result will look like this:

```
{
    "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}
```
The correlation between **{age: 35}** and **{name: Mary White}** is lost.

---
# Nested objects (continued)

*Correlated* inner objects, which are able to answer queries like these, are called *nested objects*.

---

# Resources:

Key concepts:
- https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/glossary.html



Nested objects:
- https://www.elastic.co/guide/en/elasticsearch/guide/current/complex-core-fields.html#object-arrays

Scoring and Lucene practical scoring formula:
- https://dzone.com/refcardz/lucene
- https://www.elastic.co/guide/en/elasticsearch/guide/current/scoring-theory.html
- https://www.elastic.co/guide/en/elasticsearch/guide/current/practical-scoring-function.html

---

# Resources: 
The Damerau-Levenshtein distance:
- https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#_fuzziness

Text analysis and standard analyzer:
- https://www.elastic.co/blog/found-text-analysis-part-1
- https://www.elastic.co/guide/en/elasticsearch/guide/current/analysis-intro.html

Inverted index:
- https://www.elastic.co/guide/en/elasticsearch/guide/current/inverted-index.html
- https://www.elastic.co/blog/found-elasticsearch-from-the-bottom-up

Doc values:
- https://www.elastic.co/guide/en/elasticsearch/guide/current/docvalues-intro.html
- https://www.elastic.co/blog/found-sizing-elasticsearch



