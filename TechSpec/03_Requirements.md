# Overview
The main goal of Text Mining is to help process vast amounts of information from structured and unstructured sources and discover new knowledge at a low cost. The application of text-mining techniques to online content found on forums or social media can provide quantification of rural attractiveness and different pressures on landscape, landscape planning, rural development planning, scenarios matching qualitative and quantitative indicators.

# Sources of information
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
# Modules
## Crawlers
## API
