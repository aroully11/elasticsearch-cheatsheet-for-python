# Elasticsearch-Cheatsheet-for-Python3

### Elasticsearch is a distributed, RESTful search and analytics engine capable of addressing a growing number of use cases. As the heart of the Elastic Stack, it centrally stores your data so you can discover the expected and uncover the unexpected.
### Elasticsearch lets you perform and combine many types of searches — structured, unstructured, geo, metric to get the desrired outcome.

### Elasticsearch uses certain APIs to perform search and analysis.
### To search over an index: Search API is used.

	GET index_name/_search
	
#### Example related to DeJoule logs:

	GET mgch_logs_2019-11-09/_search   (for querying over logs site and month wise) 
	GET mgch_data_2019-11-09/_search   (for querying over data site and month wise)
	
###### Above query returns all data present on that particular index which is searched for.

### To query over an index.Write your query inside query block.

	GET index_name/_search
	{	
		"query" : {}
	}
	
### Compound queries can be formed to perfrom operations using queries to give out data in minimalistict way required.The default way to create a compound query is using boolean query (bool).
### Bool requires clasues such as, must, should, must_not, or filter clauses.
###  Clauses description: 
	must : The query must appear in matching documents and will contribute to the score.
	should : The query should appear in the matching document.It may or may not be available in the queried data.
	must_not : The query must not appear in the matching document.Scoring is ignored for must_not clause.
	filter : The query required from this field should appear in matching document.
	
#### Note : One can put more than one "must" and "should" clauses to check for.The must and should clauses have their scores combined — the more matching clauses, the better.

#### Range clause: This clause is used under filter clause.Range queries lets one describe the amount of time for which query needs to be executed for searching documents.

### Queries to filter out values regarding a field : 

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
	{"FIELD": {"gte": start_time,"lte": end_time}
	}}}}}

#### Example related to DeJoule logs:
### match_all field : This clause is used to return all the values from the documents.

	GET index_name/_search
	{"query": {"match_all": {}}}
	
##### boost field can be used to boost the score for the document.

	GET /_search
	{
	"query": {
        "match_all": { "boost" : 1.2 }}}

#### Example related to DeJoule logs:

	GET mgch_logs_2019-11-09/_search
	{"query": {"match_all": {"boost": 10}}}
###### This query returns only first 10 docs from this index since value of boost has been set to 10 and being a match_all query not filters needs to be applied.
	
#### source field : Used for source filtering.To be used for restricting the amout of field that's required by the uesr from the matching document.

	GET index_name/_search
	{"_source": "{field}",
	"query": {}}

#### Example related to DeJoule logs:

	GET mgch_logs_2019-11-09/_search
	{"_source": ["ts","logType"], 
	"query": {"match_all": {"boost": 10}}}
###### This query will return all docs from first 10 entries but only with two fields ts and logType.

#### size field : Whenever a query is done on kibanna then total number of docs returned are 20.This can be incraesed by using size field.One can specify size upto 100000.

	GET /_search
	{"size": size,"query": {}}
	
#### Example related to DeJoule logs:

	GET mgch_logs_2019-11-09/_search
	{"size": 10000}
###### Putting size as maximum value shows 100000 matching values regarding that index.
	
#### aggs field : A multi-bucket value source based aggregation where buckets are dynamically built, where every bucket holds an unique value.

	GET index_name/_search
	{"aggs": {
 	 "NAME": {
   	 "AGG_TYPE": {}
  	}
	}}

#### Example related to DeJoule logs:
	
	{"size": 10000,
	"query": {"bool": {"must": [{"match": {"deviceId": devices}}]}},
	"aggs": {
        "hourly_data": {
        "date_histogram": {"field": "timestamp", "interval": "1h"},
        "aggs": {"dqi": {"cardinality": {"field": "timestamp"}}}}}}
###### This aggregations query is done over field Timestamp where buckets are made for every hour and counts number of unique values, where data_histogram is a multi-bucket aggregation and can only be used with date or date range values. 
