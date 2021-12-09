# search-engine-code
Here you can find code for serach engine in python
from dataclasses import dataclass
@dataclass
class Abstract:
    """Wikipedia abstract"""
    ID: int
    title: str
    abstract: str
    url: str

    @property
    def fulltext(self):
        return ' '.join([self.title, self.abstract])
import gzip
from lxml import etree

from search.documents import Abstract

def load_documents():
    # open a filehandle to the gzipped Wikipedia dump
    with gzip.open('data/enwiki.latest-abstract.xml.gz', 'rb') as f:
        doc_id = 1
        for _, element in etree.iterparse(f, events=('end',), tag='doc'):
            title = element.findtext('./title')
            url = element.findtext('./url')
            abstract = element.findtext('./abstract')

            yield Abstract(ID=doc_id, title=title, url=url, abstract=abstract)

            doc_id += 1
           
{
    ...
    "london": [5245250, 2623812, 133455, 3672401, ...],
    "beer": [1921376, 4411744, 684389, 2019685, ...],
    "flood": [3772355, 2895814, 3461065, 5132238, ...],
    ...
}

import Stemmer

STEMMER = Stemmer.Stemmer('english')

def tokenize(text):
    return text.split()

def lowercase_filter(tokens):
    return [token.lower() for token in tokens]

def stem_filter(tokens):
    return STEMMER.stemWords(tokens)

import re
import string

PUNCTUATION = re.compile('[%s]' % re.escape(string.punctuation))

def punctuation_filter(tokens):
    return [PUNCTUATION.sub('', token) for token in tokens]

STOPWORDS = set(['the', 'be', 'to', 'of', 'and', 'a', 'in', 'that', 'have',
                 'I', 'it', 'for', 'not', 'on', 'with', 'he', 'as', 'you',
                 'do', 'at', 'this', 'but', 'his', 'by', 'from', 'wikipedia'])

def stopword_filter(tokens):
    return [token for token in tokens if token not in STOPWORDS]
def analyze(text):
    tokens = tokenize(text)
    tokens = lowercase_filter(tokens)
    tokens = punctuation_filter(tokens)
    tokens = stopword_filter(tokens)
    tokens = stem_filter(tokens)

    return [token for token in tokens if token]

class Index:
    def __init__(self):
        self.index = {}
        self.documents = {}

    def index_document(self, document):
        if document.ID not in self.documents:
            self.documents[document.ID] = document

        for token in analyze(document.fulltext):
            if token not in self.index:
                self.index[token] = set()
            self.index[token].add(document.ID)

ef _results(self, analyzed_query):
    return [self.index.get(token, set()) for token in analyzed_query]

def search(self, query):
    """
    Boolean search; this will return documents that contain all words from the
    query, but not rank them (sets are fast, but unordered).
    """
    analyzed_query = analyze(query)
    results = self._results(analyzed_query)
    documents = [self.documents[doc_id] for doc_id in set.intersection(*results)]

    return documents

def search(self, query, search_type='AND'):
    """
    Still boolean search; this will return documents that contain either all words
    from the query or just one of them, depending on the search_type specified.

    We are still not ranking the results (sets are fast, but unordered).
    """
    if search_type not in ('AND', 'OR'):
        return []

    analyzed_query = analyze(query)
    results = self._results(analyzed_query)
    if search_type == 'AND':
        # all tokens must be in the document
        documents = [self.documents[doc_id] for doc_id in set.intersection(*results)]
    if search_type == 'OR':
        # only one token has to be in the document
        documents = [self.documents[doc_id] for doc_id in set.union(*results)]

    return documents
from collections import Counter
from .analysis import analyze

@dataclass
class Abstract:
    def analyze(self):
        self.term_frequencies = Counter(analyze(self.fulltext))

    def term_frequency(self, term):
        return self.term_frequencies.get(term, 0)
def index_document(self, document):
    if document.ID not in self.documents:
        self.documents[document.ID] = document
        document.analyze()

def search(self, query, search_type='AND', rank=True):

    if rank:
        return self.rank(analyzed_query, documents)
    return documents


def rank(self, analyzed_query, documents):
    results = []
    if not documents:
        return results
    for document in documents:
        score = sum([document.term_frequency(token) for token in analyzed_query])
        results.append((document, score))
    return sorted(results, key=lambda doc: doc[1], reverse=True)

import math

def document_frequency(self, token):
    return len(self.index.get(token, set()))

def inverse_document_frequency(self, token):

    return math.log10(len(self.documents) / self.document_frequency(token))

def rank(self, analyzed_query, documents):
    results = []
    if not documents:
        return results
    for document in documents:
        score = 0.0
        for token in analyzed_query:
            tf = document.term_frequency(token)
            idf = self.inverse_document_frequency(token)
            score += tf * idf
        results.append((document, score))
    return sorted(results, key=lambda doc: doc[1], reverse=True)
