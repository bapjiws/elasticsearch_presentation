# <p style="text-align: center;">Elasticsearch as we know it<p>

---

# The Damerau-Levenshtein distance

> [Elasticsearch] uses the Damerau-Levenshtein distance to find all terms with a maximum of two changes, where a change is the insertion, deletion or substitution of a single character, or transposition of two adjacent characters.

> The default edit distance is 2, but an edit distance of 1 should be sufficient to catch 80% of all human misspellings

---

#  The Components of an Analyzer

![ The Components of an Analyzer](https://www.elastic.co/assets/blt51e787daed39eae9/Signatures.svg)

From: https://www.elastic.co/blog/found-text-analysis-part-1

--- 

# Custom Analysis Flow

<img src="https://www.elastic.co/assets/bltee4e0b427d8fdad4/custom_analyzers_diag.png" alt="Custom Analysis Flow" style="height:550px; width: 350px;"/>

From: https://www.elastic.co/blog/found-text-analysis-part-1
