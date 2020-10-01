# Semantic Explorer API
## Root endpoint
To see all available resources, go to: `/api/v1/?format=json`
## Resources
Each Resource is represented by two endpoints: data and schema.

Resource data: `/api/v1/<<resource_name>>/?format=json`
Example: `/api/v1/library/?format=json`

Resource schema: `/api/v1/<<resource_name>>/schema/?format=json`
Example: `/api/v1/user/schema/?format=json`

Schema endpoint describes the structure of the response:
- allowed methods
- fields (types, options, etc.)
- possible filtering
- ordering

## Representation
Resource's default mode of representation is a list of objects.

Every single object has a `resource_uri` attribute, which leads to a detailed representation of a particular object.

In the list mode `meta` container displays limit and offset (see parameters below), URLs for previous and next portion of data (in case limit and offset are used) and total number of records in the output.
## Parameters
### Format
    /api/v1/<resource_name>/param1_name=param1_value \
        &param2_name=param2_value& \
        ... \
        &paramN_name=paramN_value
Parameters that serve as filters, allow for modifiers. Each modifier can be applied to a field of a certain type
* `exact` - equality: *any type*
* `iexact` - equality, case insensitive: *strings*
* `exists` - *any type*
* `startswith` - *strings*
* `istartswith` - case insensitive "startswith": *strings*
* `endswith` - *strings*
* `iendswith` - case insensitive "endswith": *strings*
* `contains` - *strings*
* `icontains` - case insensitive "contains": *strings*
* `match` - matching pattern (can include \*, e.g. racoon\*): *strings*
* `in` - inclusion: *lists*
* `nin` - not in: *lists*
* `lt` - less than: *numeric values, dates, timestamps*
* `lte` - less than or equal: *numeric values, dates, timestamps*
* `gt` - greater than: *numeric values, dates, timestamps*
* `gte` - greater than or equal: *numeric values, dates, timestamps*
* `ne` - not equal: *strings, numeric values, dates, timestamps*
* `all` - must include all mentioned values: *lists*
* `any` - must include at least one of given values: *lists*

Format: `paramname__modifier=value`
Examples:

	...&text__startswith=Rural
	...&text__icontains=transport

__WARNING:__ modifiers `all` and `any` can only be applied for the *ListFields* (such as *topics*), and in this case they should be divided by a vertical bar, for example the following param will return records that contain "land transportation" AND "livestock farming":

    topics__all=land transportation|livestock farming
The following parameter will return records that contain either "land transportation" OR "livestock farming":

    topics__any=land transportation|livestock farming
, and is equivalent to the following param:

    topics=land transportation?topics=livestock farming

The list of available filters and their modifiers are available in each endpoint's schema.
### Common parameters
* __format__ - available formats: xml, json, yaml
* __username__ - not a param, but a part of authentication token (together with __api_key__, see below). NB: this can also be sent as Authorization header.
* __api_key__ -  a part of authentication token (together with "username"). NB: this can also be sent as Authorization header.
* __limit__ - limits number of objects returned. Applicable only in case of detailed reports. Default: 36. Example: ...&limit=100
* __offset__ - number of records to skip from the beginning. Together with "limit" is used to divide data to pages (pagination). Default: 20. Example: ...&offset=40

## Endpoints
### Regional Library
#### Plain list of sources (GET)
    https://semex.io/api/v1/library/?username=username&api_key=api_key
#### Sorting library sources
By default sources are sorted by the field *created_by* descending (latest first).
For custom sorting use parameter `order_by` followed by the name of the field.

Examples:
Sorting by the owner (owners are users who create sources - in this particular case it will be sorted by username):
    https://semex.io/api/v1/library/?order_by=owner

Sorting by language descending:
    https://semex.io/api/v1/library/?order_by=-lang

To sort by multiple fields use *order_by* parameter for each field:
    https://semex.io/api/v1/library/?order_by=-updated_at&order_by=-lang

__WARNING:__ Order matters. Consider the following examples:
Sort by `source_type`, and then - within a set of each source type - sort by `created_at` descending:
    https://semex.io/api/v1/library/?order_by=source_type&order_by=-created_at


#### Sorting by nested fields
It is also possible to sort by fields that are represented as objects (for example, `owner`) - in the following example the output is sorted by the name of organization, which is represented by owner  (note that "name" is redundant, since this is a default field that represents an organization).

	...&order_by=-owner__profile__organization
#### Filtering
Use names of fields for filtering in the same manner as parameters (see "Parameters" above):

    https://semex.io/api/v1/library/?source_type=text/html

Filters can be combined:

    https://semex.io/api/v1/library/
        ?source_type=application/pdf
        &lang=it
        &created_at__gte=2020-02-18
        &size_src__lte=500

The resulting list of objects can be sorted:

    https://semex.io/api/v1/library/ \
        ?source_type=application/pdf
        &lang=it
        &created_at__gte=2020-02-18
        &size_src__lte=500
        &order_by=-updated_at

#### Time Range Filters
In addition to the standard modifiers (`__gt`, `__lte`, etc.) filtering by date fields (`created_at` and `updated_at`) can be performed using time ranges. Time-range is a string that consists of two dates (start and end), divided by vertical bar (|):

    https://semex.io/api/v1/library/
        &created_at=2015-05-08T10:00|2015-05-09T12:15

It is possible to use both date- and time-stamps as values for ranges, and to combine them in the same query:

    https://semex.io/api/v1/library/
        &created_at=2015-05-08|2015-05-09T12:15

Time range can be specified in human readable format:

    https://semex.io/api/v1/library/
        &updated_at=2 hours ago|now

It is possible to use other human readable keywords, e.g. "1 day ago", "January 12, 2017", "Saturday", etc.

Examples:

    https://semex.io/api/v1/library/
        &updated_at=2020 Feb|yesterday

    https://semex.io/api/v1/library/
        &created_at=1st of Jul 2012|in 2 hours

    https://semex.io/api/v1/library/
        &created_at=2020 Feb|2020-03-05T12:15
        &updated_at=2 hours ago|now

**WARNING!** In all given examples *two values* are necessary: start and end date (divided by vertical bar). The following example will cause **400 Bad Request**:

    https://semex.io/api/v1/library/
        &created_at=1 day ago

If the goal is to filter the records for the last day, use the following request:

    https://semex.io/api/v1/library/
        &created_at=1 day ago|now
#### Reserved keywords for Time Range Filters
Finally, there are reserved keywords, that don't require a pair of values: `today`, `yesterday`, `this week`, `last week`, `this month`, `last month`, `this year`, `last year`.

    https://semex.io/api/v1/library/
        ?created_at=last month
        &updated_at=yesterday

Filters based on time-ranges and reserved keywords are *inclusive*, i.e. they automatically stretch filters from the beginning of the starting date (0:00, or 12am) to the end of the ending date (23:59:59 or 11:59pm). So, the following examples are equivalent:

    https://semex.io/api/v1/library/
        ?created_at=2019-12-31|2019-12-31

    https://semex.io/api/v1/library/
        ?created_at=2019-12-31T00:00:00|2019-12-31T23:59:59

### Curated Reading List
Curated Reading List (CRL) is a collection of sources on a certain topic or a list of topics (by "topic" here an abstract topic is meant, not the "topic" object in the Semantic Explorer).
#### List of CRLs (GET)
Access to the list of CRL: https://semex.io/api/v1/crl/?username=username&api_key=4f23...d3c4
All mechanisms of filtering and sorting are available in the same way as for Library Sources.
**Warning**: only registered users can access CRL endpoints.

#### CRL detail (GET)
Detailed view of a certain CRL can be accessed in the following way:
https://semex.io/api/v1/crl/crl_id/?username=username&api_key=4f23...d3c4
where `crl_id` is a UUID returned after the creation, or can be found in the list.

#### New CRL (POST)
Example of creating a new CRL (warning: URLs are fictional, use your own list of links):

    POST https://semex.io/api/v1/crl/?username=username&api_key=4f23...d3c4 \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "name": "Rural Demographics",
        "description": "Demographic change",
        "sources": [
            "https://ec.eu/eurostat/statistics_EU",
            "https://cor.eu/en/engage/studies/Documents/The%20impact.pdf",
            "https://espon.eu/sites/default/Regions.pdf",
            "https://www.lulla.com/2018/12/22/the-challenge-of-rural-depopulation/#4d74a7a81295",
            "http://landmobility.gr/",
         ]
    }'

Internally CRL is a collection of URLs and therefore in principle it isn't connected to Regional Library. However, every time a new CRL is created, all the links are added to the library for the faster processing in the future. This explains, why upon creation of a CRL the server only returns the status and `_id`: in order to fill CRL with data, a series if asynchronous background processes take place:
- checking the presence of each resource is in the Library
- creating those that aren't: crawling, extracting text, analyzing it semantically
- getting summary from each text and collecting it in a one big bag-of-sentences
- performing a summary analysis on the bag-of-sentences
- indexing CRL's text fields and performing semantic analysis on the resulting summary (NER, geo-parsing, topic extraction, polarity estimation, etc.)

One way to check if the processing of CRL is finished is to periodically request its `_id` (returned after the creation):
https://semex.io/api/v1/crl/crl_id/?username=username&api_key=4f23...d3c4,
where `crl_id` is a UUID returned after the creation.
The indicator that processing is finished would be data in the field `summary` (which is an empty list right after creation).
#### Updating CRL (PUT, PATCH)
To update only certain fields of a CRL a `PATCH` request should be issued:

    PATCH https://semex.io/api/v1/crl/?username=username&api_key=4f23...d3c4 \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "sources": [
            "https://ec.eu/eurostat/new_statistics_EU",
            "https://cor.eu/en/engage/studies/Documents/TheGimpact.pdf",
            "https://espon.eu/sites/default/Regions.pdf",
            "https://www.lulla.com/2018/12/22/the-challenge-of-rural-depopulation/#4d74a7a81295",
            "http://landmobility.gr/",
         ]
    }'
A background processing will take place only if the field `sources` is changed - in this case the fields `summary`, `keywords`, and `urls_ext` are cleared up, and then the process will be repeated in the same way and order as after the creation of a CRL. In all other cases only field update takes place.
**Warning**: if the list of sources should be changed, all URLs must be provided - not only those that are added to existing list.
`PUT` request means full update, i.e. all fields will be re-written. If values for some required fields aren't provided in `PUT`, it will cause an error `400 Bad Request`.
#### Deleting CRL (DELETE)
No users are authorized to delete a CRL except of admins. For CRL to disappear from the list, it is advisable to set their status `is_active` to `"false"` and to display list filtered by adding parameter `&is_active=true` to the list.

### Search
Search endpoint is available at the following URL:
https://semex.io/api/v1/search/?query=policy

Searches are performed by the text (or list of items) stored in the following fields (properties): `text`, `text.<lang>`, `url`, `entities`.

If search by a single field (or by several, but not all) is required, use `match` modifier on a chosen field(s):

    https://semex.io/api/v1/library/
    	&text__match=tourist*
    	&loc_name__match=flanders

Note that in the latter example, search will be performed by all text fields. If a boosted search by a language-specific field is necessary, use `__match` modifier directly on that field:

    https://semex.io/api/v1/library/
    	&text.spanish__match=productividad agrícola

#### Search with filtering
Search can be combined with filters:

    https://semex.io/api/v1/search/?query=agricultural biodiversity
    	&created_at=last month
    	&loc_country=Belgium
    	&order_by=-created_at
All the rules applied to the filtering of Regional Library is applicable in this case. All possible options for filtering can be found in schema of the resource:
    https://semex.io/api/v1/search/schema?format=json

There is one particular field that requires a special mention: `source_type`. Its value explains the origin of the document and in combination with `source_id` is used to track the original document.  The value in `source_type` always consists of three parts, each of which refers to module, application within the module and data model within application. For example, consider the following fragment:

    {
        "created_at": "2020-04-12",
        "updated_at": "2020-04-23",
        "resource_uri": "/api/v1/library/5e92e8ba4dcefb2097391a17",
        "source_id": "5e92e8ba4dcefb2097391a17",
        "source_type": "app:sources:LibrarySource",
        "text": [
            "Migration of youth from <em>rural</em> <em>towns</em> to bigger cities due to lack of opportunities is a common phenomenon nowadays, resulting in the ageing of <em>rural</em> areas. Youngsters should feel themselves addressed by the affairs and future of their <em>towns</em> and the EU.",
            "They should be involved in dialogues, aiming to find solutions for challenges, hence making <em>rural</em> <em>towns</em> and the EU attractive."
        ],
        "topics": [
            "new town",
            "city",
            "inner city"
        ],
        "url": "https://europa.eu/regions-and-cities/programme/sessions/582_en"
    }
In this fragment the value of the field `source_type` equals to "app:sources:LibrarySource", which can be read as follows:
- module: `app`
- application: `sources`
- data model: `LibrarySource`

The value of the field `source_id` points to the document within this scheme.

**NB:** This information is necessary only if you want to filter by source types (i.e. `&source_type=feed:twitter:tweet`). If you only want to reach the original document from the search results, the field `resource_uri` serves this purpose.

Possible values:
- `app:sources:LibrarySource`
- `app:sources:Keyword`
- `app:sources:ReadingList`
- `app:accounts:Organization`
- `app:regions:Landscape`
- `app:regions:Region`
- `feed:twitter:tweet`
**Warning:** This list is extendable!

#### Search by specific fields
It is possible to use specified fields for search query. If more than one field specified for a search phrase, for each of them param `match` should be added to the request:

    https://semex.io/api/v1/search/?query=agricultural biodiversity
    	&match=text
    	&match=description
The fields available for selection:
- `text`
- `summary`
- `description`
- `url`
- `domain`
- `author`

**NB**: If parameter `match` is specified, the search query will be applied to all fields mentioned above.

#### Wildcard search
For wildcard search use the star symbol (**\***):

    https://semex.io/api/v1/search/?query=agri*
    https://semex.io/api/v1/search/?query=agri*ral

**Warning:** wildcards can only be used for single words. If applied to phrases, the result of the query will be empty:

    https://semex.io/api/v1/search/?query=agri*ral policy


### Aggregated data (GET)
Documents can be aggregated by geo-location (path in the response content: `["features"][<docindex>]["geometry"]["coordinates"]`) and timestamp (path: `["features"][<docindex>]["properties"]["created_at"]`).

If any of the aggregation parameters appear in the request, the response contains additional field “aggregations”, where a summarized number of documents are gathered in “buckets”, and sorted accordingly (see below).

If you are interested in aggregated results only, it is possible not to include original documents entirely by setting `size` param to zero (see example below). In this case the field `features` will still be present in the response to comply with GeoJSON format, but it will be an empty list.

    https://semex.io/api/v1/library/
	    ?agg_hotspot=true
    	&size=0

##### Aggregation by geo-location

    https://semex.io/api/v1/library/
	    ?loc_country=Poland
    	&agg_hotspot=true
    	&agg_hotspot__precision=4
    	&agg_hotspot__size=1000

`agg_hotspot__precision` parameter (integer) takes values between 1 and 12 and indicates how precise an aggregation is on a map: *1* is 5,009.4km x 4,992.6km, *12* is 3.7cm x 1.9cm (full list - https://www.elastic.co/guide/en/elasticsearch/reference/6.2//search-aggregations-bucket-geohashgrid-aggregation.html#_cell_dimensions_at_the_equator). Default is 5. Be careful to use very high precision, as it is RAM-greedy, while generally not being very useful for in each hotspot it is statistically being aggregated 2-3 documents.

`agg_hotspot__size` - maximum number of buckets to return (default is 10,000). When results are trimmed, buckets are prioritized based on the number of documents they contain (`doc_count`).

Buckets are sorted by `doc_count` descending (bigger at the top).

##### Aggregation by created_at (date-time histogram)
    https://semex.io/api/v1/library/
	    ?loc_country=Poland
    	&agg_timestamp=true
    	&agg_timestamp__interval=90m

`agg_timestamp__interval` defines interval for collecting records. Available expressions for interval:
* year - `1y`
* quarter - `1q`
* month - `1M`
* week - `1w`
* day - `1d`
* hour - `1h`
* minute - `1m`
* second - `1s`

Fractional time values are not supported, but it is possible to achieve the goal shifting to another time unit: e.g., `1.5h` could instead be specified as `90m`

**Warning**: time intervals larger than than days do not support arbitrary values but can only be one unit large at its maximum, e.g. `1w` is valid, `2w` is not. For large intervals use days: for example, for a precision value of two weeks use `14d`.

Buckets are sorted by timestamps of the intervals, ascending.

##### Date-time histogram with average sentiment
In Semantic Explorer database sentiment is stored in the field `polarity`. Date-time histogram with average values of polarity is being obtained on the following way:

    https://semex.io/api/v1/library/
	    ?topics=rural population
	    &topics=rural attractiveness
	    &country=Ireland
    	&agg_polarity=true
    	&agg_polarity__interval=90m

This indicates calculation of the average sentiment for each timestamp bucket. In the example above in each bucket (1.5hrs long) in addition to `doc_count` will contain `avg_polarity`.
### User feedback (PATCH)
A registered user can leave a feedback for any particular document:

    PATCH https://semex.io/api/v1/library/<source_id>/<paragraph_number>/?api_key=<user_api_key>&username=<username-or-email>
    Content-Type: application/json
    data = '{
        "feedback": {
            "topics": [
	            "natural disaster",
                ],
            "relevant": false,
            "note": "This has nothing to do with natural disaster: this paragraph is about population decline!"
        }
    }'

    PATCH https://semex.io/api/v1/library/<source_id>/<paragraph_number>/?api_key=<user_api_key>&username=<username-or-email>
    Content-Type: application/json
    data = '{
        "feedback": {
            "entities": [{
	            "text": "natural disaster",
	            "start_char": 12,
	            "end_char": 28,
	            "label": "GPE"
                ],
            "relevant": false,
            "note": "Not an entity, just a text!"
        }
    }'


The comment will automatically be stamped with a date-time mark and a link to a user profile.
**Warning**: feedback is a paragraph based. It is impossible (and useless) to leave a feedback for the whole document.
**Warning**: 
feedback don’t have history, i.e. it is possible only to re-write a comment, but not to add one. However, if a user wants to leave a feedback for an article that is already commented by someone else, a new comment will be added.

The structure of the feedback itself is free-form for as long as user keep all valuable information in a form of dictionary under the `feedback` key. However, for the procedural simplicity we suggest keeping the following keys, when giving a feedback:
*  `<fieldname>` - `<list>` of values that are considered as relevant to the subject of a current paragraph. This can either be a subset of original `topics` key, or entirely new topics, or combination of both. Class can also be Named Entity - for example, it is possible to change a label of the entity or discard the entity entirely, if it was detected mistakenly. By "class" here is meant a category or a keyword that somehow describes a topic of a given document. *Example of  implementation*: a checkbox'ed list of `topics`, from which checked items become `classes` in the feedback.
* `relevant` - `<bool>` indicating whether a found document (a tweet in its entirety or a paragraph of a library source) is relevant to the topic. This key is optional, if tweet is considered to be relevant, while a user wants to specify a list of classes (see above) or simply to leave a note.
* `note` - `<str>` (optional) comment on the reason of leaving feedback.

### Semantic Analysis

All endpoints described above provide access to data that has already been analyzed by the Semantic Explorer and  saved in the database or indexes in the search engine. In addition to that there is a set of endpoints that provide direct access to the semantic features on the text.

#### Request type and structure
Semantic features can be accessed directly only by POST requests.

General form of the request (example made with `curl` command):

    curl --request POST 'https://semex.io/api/v1/analyze/?username=<username>&api_key=<api_key>' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "text": "<text>",
        "pipeline": [<item1>, <item2>, ..., <itemN>],
        "lang": "<lang>"
    }'

Only registered users can access semantic features, therefore absent or wrong `username` and `api_key` will generate `401 Unauthorized` error.

**Warning!** It is very important to set a correct language, if it is known. If the language is not provided, the system will detect it, but this would cost an overhead. If the language is set incorrectly, Semantic Explorer will use it for analysis, which can make outcome quite surprising.

#### The Pipeline
The main parameter of Semantic Analysis endpoint is the `pipeline`. It is a list of strings or dictionaries that describe what kind(s) of analysis should be performed on the text.

Each element in the list can either be a string (name of action) or a dictionary (name of action with parameters).

Example of plain list (actions are performed with the default parameters):

    {
        "text": "<text>",
        "lang": "<lang>",
        "pipeline": ["tokens", "ner", "topics"]
    }

To change the default parameters of each action use the following form:

    {
        ...
        "pipeline": [
            {
                "action": "tokens",
                "params": {
                    "lemmatize": true
                }
            },
            {
                "action": "vectors",
                "params": {
                    "topn": 50
                }
            }
        ]
    }

It is possible to combine those two forms in the same call:

    {
        "text": "<text>",
        "lang": "<lang>",
        "pipeline": [
            "tokens",
            {
                "action": "vectors",
                "params": {
                    "topn": 20
                }
            },
            "topics"
        ]
    }

#### Actions
Every action is a singular procedure that is performed on a given text. Actions are grouped in the chain and each of them affects results on the later stages in the chain. For example if both `tokens` and `vectors` are requested and the action `tokens` was accompanied by param `lemmatize=true` (see below), then `vectors` will return word vectors of lemmas instead of vectors of original words. All dependencies and parameters are described for each action below. 

Semantic Explorer supports the following actions:

##### tokens
Text cleaned up from stop-words, special symbols and punctuation marks.

Parameters:
- `lemmatize <boolean>` (default `false`) - if it is set to `true`, each token will be represented by its lemma: [https://en.wikipedia.org/wiki/Lemma_(morphology)](https://en.wikipedia.org/wiki/Lemma_(morphology))

##### vectors
L2 norm vector of words found in text - for details see [https://spacy.io/usage/vectors-similarity](https://spacy.io/usage/vectors-similarity).

Parameters:
- `topn <int>` (default 10) - top N words sorted by normalized value of the vector

##### ner
Named Entity Recognition [https://en.wikipedia.org/wiki/Named-entity_recognition](https://en.wikipedia.org/wiki/Named-entity_recognition).

Parameters:
- `distinct <boolean>` (default `false`) - if `true` Named Entities are grouped for the whole text and only labels and text of each entity is returned, otherwise all entities with their appearance in the text (accompanied by `start_char` and `end_char`). Entity labels are described here [https://spacy.io/api/annotation#named-entities](https://spacy.io/api/annotation#named-entities)

##### ner_rendered

HTML formatted version of `ner`. No parameters.

##### noun_chunks
Objects and subjects of all sentences of the text - the result of Parts of Speech tagging [https://spacy.io/usage/linguistic-features#pos-tagging](https://spacy.io/usage/linguistic-features#pos-tagging). No parameters.

##### polarity
Estimated value of sentiment on the scale from -1.0 to 1.0, where values around -1.0 are very negative, close to 0.0 are neutral, and close to +1.0 are very positive.

Parameters:
- `level <str>` of analysis. Possible values:
	- `text` or `t` (default) - get sentiment for the whole text
	- `paragraph` or `p` - split text to paragraphs and return estimation of sentiment for each paragraph
	- `sentence` or `s` - split text to sentences and return estimation of sentiment for each sentence.

##### summary
Text summary, i.e. the most representative sentences from the text.

Parameters:
	- `keywords <int>` (default 15) - defined how many top keywords are used to summarize the document (the bigger the number of keywords, the broader the summary).
	- `sentences <int>` (default 5) - how many sentences should be returned.

##### topics
Extraction of topics as if answering the question "what this text is about, in general?" This is highly dependent on the thesaurus - current implementation of topic extraction in semex.io  is based on GEMET [https://www.eionet.europa.eu/gemet/en/about/](https://www.eionet.europa.eu/gemet/en/about/).

Parameters:
- `keywords <int>` (default 15) - same as in `summary`
- `max_topics <int>` (default 10) - the maximum number of topics to be returned (the response is always ordered by the relevance of the topics descending - the most relevant at the top)


#### Inputs
There can be three types of input:
- `text` - plain text of an arbitrary length.
- `url` - a proper URL pointing to some text in the internet. In this case the URL is first scraped (which adds overhead to the response depending on the source size)
- `source_id` - a unique ID of the source from the Regional Library (this is generally not necessary, since all the documents from Regional Library are being constant;y updated and analyzed).

Example of analyzing a text:

    curl --request POST 'https://semex.io/api/v1/analyze/?username=<username>&api_key=<api_key>' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "text": "The productivity of a region'\''s farms is important for many reasons. Aside from providing more food, increasing the productivity of farms affects the region'\''s prospects for growth and competitiveness on the agricultural market, income distribution and savings, and labour migration. An increase in a region'\''s agricultural productivity implies a more efficient distribution of scarce resources. As farmers adopt new techniques and differences, the more productive farmers benefit from an increase in their welfare while farmers who are not productive enough will exit the market to seek success elsewhere",
        "pipeline": [
        	"polarity",
        	{
        	"action": "topics",
        	"params": {"max_topics": 3, "keywords": 15}
        	}
        ],
        "lang": "en"
    }'

Example of analyzing URL:

    curl --request POST 'https://semex.io/api/v1/analyze/?username=<username>&api_key=<api_key>' \
    --header 'Content-Type: application/json' \
    --data-raw '{
    	"url": "https://en.wikipedia.org/wiki/Agricultural_productivity",
        "pipeline": [
        	{
        		"action": "summary",
        		"params": {
        			"sentences": 6
        		}
        	},
        	"noun_chunks"
        ],
    	"lang": "en"
    }'

Example of analyzing a document from the Regional Library:

    curl --location --request POST 'https://semex.io/api/v1/analyze/?username=<username>&api_key=<api_key>' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "source_id": "5df73a3d5ae4c68142634e9f",
        "pipeline": [
        	"tokens",
        	{
        		"action": "ner_rendered"
        	},
        	{
        		"action": "topics",
        		"params": {
        			"max_topics": 5
        		}
        	}
        ],
        "lang": "en"
    }'

#### Similarity Cluster
Similarity Cluster is a general model based on thesaurus, where topics and subtopics are represented as a directed graph with the topic as a root and its subtopics as nodes. If a node has children, it is considered to be a topic on its level, while his children are subtopics.

NB: It is only a representation up to a certain number of levels (normally 3) that can be considered as a directed graph. In reality, the graph is non-directed because the root topic can be a subtopic of the topic on a deeper levels. Similarity cluster is a dynamic structure and is being built in real-time, which allows to add new topics at any time.

Similarity Cluster for a particular topic can be obtained by GET request:

    https://semex.io/api/v1/similarity_cluster/
        ?topic=zonas rurales
        &lang=es
        &threshold=0.6
        &depth=3
        &username=<username>
        &api_key=<api_key>

Parameters:
- `topic <str>` (mandatory)
- `lang <str>` (optional, default **'en'**) - if the topic is in a language different than English, it SHOULD be accompanied with a correct language code!
- `depth <int>` (optional, default 2) - how deep to dive (**warning:** the deeper, the longer the response!):
	- 1 find only nearest neighbors
	- 2 return subtopics for each subtopics of the main topic
	- 3 one level deeper,
	- 4 etc.
- `threshold <float>` (optional, default 0.6) - minimal similarity to consider a subtopic (internally - a distance between normalized vectors of  topic and subtopics).
- `topn <int>` (optional, default 10) - how many similar topics should be returned. NOTE: the first node has `topn` subtopics, but the nodes in further levels have `topn-1` subtopics.

#### Topic browser
Endpoint that connects Similarity Cluster and data in Regional Library and/or Curated Reading List and/or Social Media feed. In other words, for the defined set of subtopics it returns documents from the selected resource(s) (for example, from Regional Library and Social Media feed) that are categorized by those topics. 

GET request:

    https://semex.io/api/v1/topic_browser/
        ?root=rural area
        &topic=urban area
        &topic=rural population
        &topic=rural environment
        &lang=en
        &source_type=app:sources:LibrarySource
        &topn=10
        &username=<username>
        &api_key=<api_key>

Parameters:
- `root <str>` (mandatory) - root topic from Similarity Cluster
- `topic <str>` (multiple, at least 1 is mandatory) - can be any topic different from the root, but ideally it's subtopics.
- `lang <str>` (optional, default "en") - if the topic is in a language different than English, it SHOULD be accompanied with a correct language code!
- `source_type <str>` (multiple, optional, default "app:sources:LibrarySource") - source of documents. Options to choose from :
  	- `app:sources:LibrarySource`
	- `app:sources:Keyword`
	- `app:sources:ReadingList`
	- `app:accounts:Organization`
	- `app:regions:Landscape`
	- `app:regions:Region`
	- `feed:twitter:tweet`
- `topn <int>` (optional, default 10) - how many documents from each source should be returned.
