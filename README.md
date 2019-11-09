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

	GET index_name/_search
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

#### Example related to DeJoule data:
	
	GET mgch_data_2019-11-09/_search
	{"size": 10000,
	"query": {"bool": {"must": [{"match": {"deviceId": "your_deviceId"}}]}},
	"aggs": {
        "hourly_data": {
        "date_histogram": {"field": "timestamp", "interval": "1h"},
        "aggs": {"dqi": {"cardinality": {"field": "timestamp"}}}}}}
###### This aggregations query is done over field Timestamp where buckets are made for every hour and counts number of unique values, where data_histogram is a multi-bucket aggregation and can only be used with date or date range values. 
###### Above query can be used to caluclate data quantity index for data regarding any particular device.

#### For filtering out the basic structure of packet received from elasticsearch 

	filter_path : This is an accepted parameter by REST APIs which allows to filter out the response received from elasticsearch.
	
#### filter path parameter syntax

	GET index_name/_search?filter_path=took,hits.hits.obj1,hits.hits.obj2
	{"query" : {}}

#### Example related to DeJoule data:
	
	
	GET /mgch_logs_2019-08-01/_search?filter_path=hits.hits._source&_source=ts,logType,status
	{
	"query": {"range": {
	  "ts": {
	    "gte": "2019-08-01 00:00:00",
	    "lte": "2019-08-01 23:00:00"
	  }
	}}}
###### Filter path parameter in above query only returns the data from source wich are ts and logType.This reduces the sapce which normal returned doc will take hence making query to run even faster than earlier.

#### To get the latest value inserted into the database

	sort : Can arrange the database in either ascending or descending order so that it's easier for teh user to look for documents as required.

#### sort field synatx

	GET index_name/_search
	{"sort": [
	  {
	    "FIELD": {
	      "order": "desc"
	    }
	  }
	]}
	
#### Example related to DeJoule logs:

	GET mgch_logs_2019-11-09/_search
	{"sort": [
	  {
	    "FIELD": {
	      "order": "desc"
	    }
	  }
	]}
	
#### Example of combination of all above functions related to DeJoule logs:
	
	{"_source": ["logType","ts","controllerId","deviceId","key","source"],
	"size": 100,
	"query": {"bool": 
	{"must": 
	[{"match": {"deviceId": "2200"}},
	{"match": {"source": "pid"}},
	{"match": {"logType": "command"}}],
	"filter": {"range":
	{"ts": {"gte": "2019-11-01 00:00:00", 
		"lte": "2019-11-05 23:00:00"}}}}}}
###### Above query has used multi fields and queries together to give output which is very precise to a given condition, that is, finding commands sent by source pid on days 1st to 5th of Nov'19 to deviceId 2200.The number of documents required is 100 as specified in the size field and source field contains only those fields which are requied by the user which are ts,logType,cid,did,key and source.In range field time is specified by gte(>=) and lte(<=).

#### Searching over multiple index or performing more than one query over same index
	
	m_search : Helps to perfrom more than one query in one request and also query mutilple indices at same time.

#### Synatx for m_search :
	
	POST /m_search
	{"index" : "index_name1", "type" : "doc_type"}
	{"query" : {}}
	{"index" : "index_name2", "type" : "doc_type"}
	{"query" : {}}
	
	
