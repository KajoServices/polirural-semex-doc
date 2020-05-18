# Semantic Explorer API
## Root endpoint
To see all available resources, go to: `/api/v1/?format=json`
## Resource calls
Resource consists of data and schema.

Resource data: `/api/v1/<<resource_name>>/?format=json`
Example: `/api/v1/library/?format=json`

Resource schema: `/api/v1/<<resource_name>>/schema/?format=json`
Example: `/api/v1/user/schema/?format=json`

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

Format: `paramname__modifier=value`
Examples:

	...&text__startswith=Rural
	...&text__icontains=transport

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
Sorting by the owner (owners are users who create sources):
    https://semex.io/api/v1/library/?order_by=owner

Sorting by language descending:
    https://semex.io/api/v1/library/?order_by=-lang

To sorting by multiple fields use *order_by* parameter for each field:
    https://semex.io/api/v1/library/?order_by=-updated_at&order_by=-lang

__WARNING:__ Order matters. Consider the following examples:
Sort by `source_type`, and then - within a set of each source type - sort by `created_at` descending:

	...&order_by=source_type&order_by=-created_at
#### Filtering
Use names of fields for filtering in the same manner as parameters (see "Parameters" above):

    https://semex.io/api/v1/library/
        ?username=username
        &api_key=4f23...d3c4
        &source_type=text/html

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

#### Filtering by created_at and updated_at
##### Time range
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

**WARNING!** In those cases *two values* are necessary: start and end date (divided by vertical bar). The following example will cause **400 Bad Request**:

    https://semex.io/api/v1/library/
        &created_at=1 day ago

If the goal is to filter the records for the last day, use the following request:

    https://semex.io/api/v1/library/
        &created_at=1 day ago|now
##### Reserved keywords
Finally, there are reserved keywords, that don't require a pair of values: `today`, `yesterday`, `this week`, `last week`, `this month`, `last month`, `this year`, `last year`.

    https://semex.io/api/v1/library/
        ?created_at=last month
        &updated_at=yesterday

Filters based on time-ranges and reserved keywords are *inclusive*, i.e. they automatically stretch filters from the beginning of the starting date (0:00, or 12am) to the end of the ending date (23:59:59 or 11:59pm). So, the following examples are equivalent:

    https://semex.io/api/v1/library/
        ?created_at=2019-12-31|2019-12-31

    https://semex.io/api/v1/library/
        ?created_at=2019-12-31T00:00:00|2019-12-31T23:59:59

### Search
Search endpoint is available at the following URL:
https://semex.io/api/v1/search/?query=policy

Search are performed by the text (or list of items) stored the following fields (properties): `text`, `text.<lang>`, `url`, `entities`.

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

#### Aggregated data (GET)
Tweets can be aggregated by geo-location (path in the response content: `["features"][<docindex>]["geometry"]["coordinates"]`) and timestamp (path: `["features"][<docindex>]["properties"]["created_at"]`).

If any of aggregation parameter appear in the request, the response contains additional field "aggregations", where summarized number of documents are gathered in "buckets", and sorted accordingly (see below).

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
#### User feedback (PATCH)
A registered user can leave a feedback for any particular tweet:

    PATCH https://semex.io/api/v1/library/<source_id>/<paragraph_number>/?api_key=<user_api_key>&username=<username-or-email>
    Content-Type: application/json
    Connection: keep-alive
    cache-control: no-cache
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
    Connection: keep-alive
    cache-control: no-cache
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
**Warning**: feedback don't have history, i.e. it is possible only to *re-write* comment, but not to add one.  However, if a user wants to leave a feedback for an article that is already commented by someone else, a new comment will be added.

The structure of the feedback itself is free-form for as long as user keep all valuable information in a form of dictionary under the `feedback` key. However, for the procedural simplicity we suggest keeping the following keys, when giving a feedback:
*  `<fieldname>` - `<list>` of values that are considered as relevant to the subject of a current paragraph. This can either be a subset of original `topics` key, or entirely new topics, or combination of both. Class can also be Named Entity - for example, it is possible to change a label of the entity or discard the entity entirely, if it was detected mistakenly. By "class" here is meant a category or a keyword that somehow describes a topic of a given document. *Proposed implementation*: a checkbox'ed list of `topics`, from which checked items become `classes` in the feedback.
* `relevant` - `<bool>` indicating whether a tweet (in its entirety) is relevant to the topic. This key is optional, if tweet is considered to be relevant, while a user wants to specify a list of classes (see above) or simply to leave a note.
* `note` - `<str>` (optional) comment on the reason of leaving feedback.