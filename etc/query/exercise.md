1. search for 'star wars' moview released after 1980, using both a url search and a request body search.

~~~
curl -XGET "127.0.0.1:9200/movies/_search?q=%2Byear%3A%3E1980+title%3Astar%20wars&pretty"

curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "bool": {
            "must": {
                "match_phrase": {
                    "title": "Star Wars"
                }, 
            },
            "filter": {
                "range": {
                    "year": {"gte":1980}
                }
            }
        }
    }
}'
~~~

2. search for science fiction movies before 1960 sorted by title
~~~
{
    "query": {
        "bool": [
            "must": {
                "match": {
                    "genre":"Sci-Fi"
                }
            },
            "filter": {
                "range":{
                    "year":{"lt":1960}
                }
            }
        ]
    },
    "sort":"title.raw"
}

