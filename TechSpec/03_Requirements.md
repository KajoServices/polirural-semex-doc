# Overview
The main goal of Text Mining is to help process vast amounts of information from structured and unstructured sources and discover new knowledge at a low cost. The application of text-mining techniques to online content found on forums or social media can provide quantification of rural attractiveness and different pressures on landscape, landscape planning, rural development planning, scenarios matching qualitative and quantitative indicators.

# 1. Sources of information
In this section we give a short overview of all possible sources of information, the type of information that should be extracted from each particular source type and techniques used to solve those tasks. 
## Texts
### General concepts
Here concepts are given related to the project. They do not necessarily correspond to the concepts in the "real world", described by the same term.
#### Topics
***Topic*** is an ultra-short summary of a given semantic entity or a sphere of interest, that can categorize a certain document found on a website, in a paper, or in Social Media feeds. Any given text can contain more than one topic.
Examples : "landscape development", "landscape quality", "new farming business model", "rural policy", "rural policy measures", "landscape dynamics", "leader", "biodiversity", "ecosystem services", "landscape maintenance", "landscape architecture", "historical landscapes and heritage".

Topic can include **sub-topics**.
Examples (in English):

 - "Landscape development" subtopics:
	 - "Landscape quality"
	 - "Landscape dynamics"
	 - "Landscape maintenance"
 - "Rural policy" subtopics:
	 - "Rural policy measures"
	 - "Rural policy proposal"

Topics and their sub-topics can be represented as directed graph, while all topics and subtopics, that are about to be analyzed in the project, together form an **undirected graph**, i.e. some topics can be universally general, while others can be both independent topics or sub-topics of other, more general, topics (i.e. they form a **network**). Examples: ***TODO***
#### Direct quotes
Direct quote is a text, that is very close semantically to another text, already being processed by the system (e.g., a tweet saved in the database or a description of policy measure obtained by the crawler). A text that is compared to another text, for which a similarity ratio is above certain threshold, is being considered as a "direct quote" and should be discarded from processing.
### Web-sites provided by partners
#### PDF documents (papers, etc.)
#### Forums and discussion boards

 **Task:** Topic modelling (see [Topics](#topics)).
 **Description:** Extract text that has the highest semantic similarity with one of the general topics (e.g. "landscape development", "landscape quality", "new farming business model", "rural policy", "rural policy measures", "landscape dynamics", etc.) and create a list of subtopics that are not the part of the original list of topics (e.g. "rural mobility", which is discovered after processing documents and is not a part of the original list of topics).
 **Techniques:**
 - extraction of text: *Crawlers*
 - cleaning of text: *Semantic Explorer / Text Cleaner*
 - topic extraction: *Semantic Explorer / Similarity Estimator* (as compared to topics obtained from [GDELT](#gdelt))
## Social Media (SM)
### Resources
 - Twitter
 - LinkedIn
 - **TODO:** what else ???

### Live streaming
Posts from SM extracted by keywords and/or obtained from certain geographical areas.

**Areas** can be defined as a geo-point (latitude, longitude) and a radius (in km), or as a geo-hash code, or as multi-polygon (list of geo-points).

**Keywords** are defined in two ways:

 1. The list of initial keywords (for each language) are prepared by experts
 2. In order to collect information as close as possible semantically to the given topics, the list of keywords should be continuously updated, based on the information collected and indexed. New keywords will be obtained using Adaptive Keywords technique (see [References #1](references-1) ).

### Normalization
In terms of the current project normalization is a process to convert an original post (a tweet, a post from linkedin, etc.) into a unified form for feeding to the clients of API or/and live feed (end-users).

Normalization includes:

 - Text cleaning
 - Quantification
 - Labelling
 - Keywords extraction
 - Evaluation of representativeness
#### Text cleaning
 - Removal from the original text of all symbols, stop-words, etc., that are not relevant to text processing
 - Removal of personal sensitive information (names, phone-numbers, emails, etc.)
#### Quantification
For each post there should be estimation of opinion polarity on the scale of [-1..1], where -1 is the most negative, 0 is neutral, 1 is maximum positive.
**Technique:** NLP, sentiment analysis.
#### Labelling
Calculating of the probability that the post relates to a certain topic.
**Technique:** ML, classification task.
#### Keywords extraction
Each text should be converted to the list of keywords (bag of words) for Adaptive Keyword processing (see [References #1](references-1) ).
**Technique:** NLP, lemmatization.
#### Evaluation of the representativeness
Non-representative posts (see [Direct quotes](direct-quotes)) should be removed (or marked). Example: re-tweets on Twitter, posts from FB shared via Twitter, etc.
**Technique**: NLP, a ratio based on texts *multiplicity* and *centrality*. A text that is compared to another text, for which this ratio is above certain threshold, is being considered as a "direct quote" and should be discarded from the streaming. 
## GDELT
# System Components
## 1. Streaming (from Social Media)
### 1.1 Resources
Twitter
LinkedIn
### 1.2 Normalizing Data from Social Media
Data incoming from Social Media should pass stages of Normalization and Indexing in order to become available for analysis and queries:

- Normalization (data marked with the asterisk (\*) does not pass through Normalization stage, if it is discarded, see **warning** below):
	- cleaning up and tokenizing original text
	- extraction of hashtags
	- geo-parsing\* (**Warning**: a single document obtained from source can be split to several normalized documents if more than one geographical entity appears in it)
	- semantic analysis:
		- topic extraction\*
		- NER
		- topic modelling
	- sentiment analysis:
		- evaluation of [negative, neutral, positive] on the scale of [-1,..,1]
	- masking out any "sensitive" information (names, phone numbers, emails, etc.)
- Indexing:
	- recording normalized document in Elasticsearch index

**Warning:** Documents that are discarded at the stage of Normalization won't make it to Indexing and will only be available in their raw form, see.

## 2. Crawlers
## 3. Semantic Explorer
### 3.1 Named Entity Recognition (NER)
### 3.2 Geo-parser
### 3.3 Topic Manager
#### 3.3.1 Topic Extraction
#### 3.3.2 Topic Modelling
## 4. Storage
### 4.1 Social Media
There are two separate (and parallel) storage schemes used for data collected from Social Media streaming
#### 4.1.1 Raw data
- ****** (Elasticsearch cluster)

**Raw data** is represented "as-is" (i.e. with preservation of all information that comes through our feed - every document contain all fields). Raw data is used for additional analysis on the later stages of the project (for example, to collect all information from a certain area regarding a chosen event or a policy). Raw data be periodically archived and deleted from the main database (time-frame to be defined in the project settings, and will be fine-tuned regarding the amount of data).

Storage scheme: MongoDB cluster.

**Warning:** the access to raw data will not be given to the "outside world" and can only be delegated by request to admins.
 
#### 4.1.2 Normalized and Indexed data
Normalized data will be indexed in the Elasticsearch cluster. In its turn, indexed data is a product of Normalization, as described in [1.2 Normalizing Data from Social Media](1-2-normalizing-data-from-social-media).

**Warning:** Documents that are discarded at the stage of Normalization won't make it to Indexing and will only be available in their raw form.

### 4.2 Texts
#### 4.2.1 Raw data
#### 4.2.2 Index
## 5. Access to Data
### 5.1 Queries in Domain Specific Language (DSL)
#### 5.1.1 Social Media
#### 5.1.2 Semantic features
### 5.2 API
### 5.3 Web-socket streaming