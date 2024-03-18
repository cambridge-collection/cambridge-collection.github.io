The Solr Stack

CDCP’s search facility is provided by Apache Solr (<https://solr.apache.org/>) and an associated stack of services.

![The Solr search stack](images%2Fsolr-stack.svg)

## CUDL Solr
The core search facility is provided by Solr and can be found in     <https://github.com/cambridge-collection/cudl-solr>.

## CUDL Search API
Solr’s API is a protected. It can only be accessed from within the CDCP subnet by one component: the cudl-search API (<https://github.com/cambridge-collection/cudl-search>). This API provides the endpoints for all search-related actions. By default, only GET requests (*i.e.* queries) are publicly available. All destructive actions, such as submitting a file for indexing or deleting a file, are implemented via PUT and POST requests. These requests are only available to certain key components in CDCP’s private subnet, like the cudl-solr-listener.

## CUDL Solr Listener
Whenever a Solr JSON file is changed, AWS queues an SNS message. This messages are sent to the cudl-solr-listener (<https://github.com/cambridge-collection/cudl-solr-listener>), which verifies that the JSON file is plausibly valid before submitting it for reindexing.