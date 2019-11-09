# Elasticsearch-Cheatsheet-for-Python3

### Elasticsearch is a distributed, RESTful search and analytics engine capable of addressing a growing number of use cases. As the heart of the Elastic Stack, it centrally stores your data so you can discover the expected and uncover the unexpected.
### Elasticsearch lets you perform and combine many types of searches — structured, unstructured, geo, metric to get the desrired outcome.

### Elasticsearch uses certain APIs to perform search and analysis.
### To search over an index: Search API is used.
	GET index_name/_search
### To query over an index.Write your query inside query block.
	GET index_name/_search
	{	
		"query" : {}
	}
### Compound queries can be formed to perfrom operations using queries to give out data in minimalistict way required.The default way to create a compound query is using boolean query (bool).
### Bool requires clasues such as, must, should, must_not, or filter clauses.
### The must and should clauses have their scores combined — the more matching clauses, the better — while the must_not and filter clauses are executed in filter context.



#### Queries to filter out values regarding a field : 

	GET index_name/search
	{"query": {"bool":
	{"must": [
	{"match": {"FIELD": "TEXT"}}],
	"must_not": [
	{"match": {"FIELD": "TEXT"}}],
	"should": [
	{"match": {"FIELD": "TEXT"}}],
	"minimum_should_match": 1, 
	"filter": {
	"range": 
	{"FIELD": {"gte": 10,"lte": 20}
	}}}}}

	
