# Elasticsearch profile

Reference - [Elasticsearch Profile API documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-profile.html)

Any `_search` request can be be profiled with `"profile": true` flag set.

We will focus only on the `"profile"` section in the response.

## Overall Structure


```json
{
   "profile": {
        "shards": [
           {
              "id": "[q2aE02wS1R8qQFnYu6vDVQ][my-index-000001][0]",  
              "node_id": "q2aE02wS1R8qQFnYu6vDVQ",
              "shard_id": 0,
              "index": "my-index-000001",
              "cluster": "(local)",             
              "searches": [
                 {
                    "query": [...],             
                    "rewrite_time": 51443,      
                    "collector": [...]          
                 }
              ],
              "aggregations": [...],            
              "fetch": {...}                    
           }
        ]
     }
}
```

- Profile is broken down at a shard level.
- `node_id`, `shard_id`, `index`. Here `node_id` can be useful to find if there hotspots specific a node.
- `searches`, `aggregations`, `fetch` -> 3 phases
- The `search` phase itself is split to `query` and `collector`.


### On multiple searches

> The profile itself may consist of one or more "searches", where a search is a query executed against the underlying Lucene index. Most search requests submitted by the user will only execute a single search against the Lucene index.

So we will consider single search as first class feature.


### On Lucene internals

> The details provided by the Profile API directly expose Lucene class names and concepts, which means that complete interpretation of the results require fairly advanced knowledge of Lucene. 

> With that said, a complete understanding is often not required to fix a slow query. It is usually sufficient to see that a particular component of a query is slow, and not necessarily understand why the advance phase of that query is the cause, for example.

So lucene information becomes first class citizen. But maybe its too much information for initial investigation so it can be probably opt-in.

## Query section

```json
"query": [
    {
       "type": "BooleanQuery",
       "description": "message:get message:search",
       "time_in_nanos": "11972972",
       "breakdown": {...},               
       "children": [
          {
             "type": "TermQuery",
             "description": "message:get",
             "time_in_nanos": "3801935",
             "breakdown": {...}
          },
          {
             "type": "TermQuery",
             "description": "message:search",
             "time_in_nanos": "205654",
             "breakdown": {...}
          }
       ]
    }
]
```


- `time_in_nanos` alone can give a high level overview.
- `breakdowns` can be used for further deep dive.


## Collectors section

We will come back later to this.


## Some of the questions what profile can help answering

- Why is the search taking too much time?
    - Which stages of search is taking too much time?
    - Why is a stage taking too much time?
    - Are there too many stages to start with?
- Why is the search taking too much memory? 
    - Which stage is taking too much memory?

