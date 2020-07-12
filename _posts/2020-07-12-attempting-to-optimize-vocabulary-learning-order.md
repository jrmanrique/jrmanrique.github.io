---
layout: post
title:  "Attempting to Optimize Korean Vocabulary Learning Order"
date:   2020-07-12 13:00:00 +0800
categories: 
    - programming
tags:
    - programming 
    - languages 
    - korean
    - graph theory
---

I was browsing Reddit when I saw a post proposing a new learning order of Chinese characters. Reading through it, I thought maybe I could also implement this with Korean vocabulary.

There are different methods on how Chinese characters are taught: by frequency and in order of network hierarchy of character components. One of the common ways of learning Chinese characters as a foreign language learner is through Heisig and Richardson's method (as used in *Remembering the Kanji*), where components are introduced directly before their compounds to reinforce the logic of character construction. The paper [*Optimizing the Learning Order of Chinese Characters Using a Novel Topological Sort Algorithm*](https://doi.org/10.1371/journal.pone.0163623) discusses an algorithm which is able to approximate human-curated pedagogically desirable learning order by using both frequency and network hierarchy.

The paper shows how the network of Chinese characters can be represented as a directed  graph: the nodes represent the visual form of the characters, the edges represent the structural relationship between the characters, while taking into account frequency and learning cost as a measure of centrality.

We can implement this in learning Korean vocabulary: given a seed set of words, we can parse the dictionary definitions of each word to create a dependency graph. The idea is one needs to know each word in the definition to fully understand the headword.

Barring the problems on web scraping (because I totally forgot that JSON dictionaries exist), the first problem is parsing the definitions into tokens. For Python, [KoNLPy](https://github.com/konlpy/konlpy) offers many tagging classes. We only extract content words like nouns, verbs, adjectives, and adverbs. 

To populate the network, I decided to iterate through the frequency list published by National Institute of Korean Language to serve as headwords for the parser.

```python
freq = load_json('freq.json')  # freq.json is precleaned from the NIKL data.

for i, word in enumerate(freq):
    if word not in tree:
        hw, children = get_children(word)  # get_children() uses KoNLPy to parse.
        tree[hw] = children
```

To sort this graph, one would normally use [topological sort](https://en.wikipedia.org/wiki/Topological_sorting) (as did the paper): for every directed edge $$uv$$ from vertex $$u$$ to $$v$$, $$u$$ comes before $$v$$ in ordering. One caveat of topological sort is we cannot use it when there is a cycle in the graph. Arguably, a non-circular dictionary cannot exist without setting assumptions on semantic primes (see [Bullock 2010](https://doi.org/10.1093/ijl/ecq035)). This means that by using an unmodified dictionary like [Naver](http://ko.dict.naver.com/), we will end up with a directed graph with cyclic subgraphs.

This got me thinking: we could treat each cyclic subgraph as a node, order the entire graph using topological sort, expand each cyclic subgraph and just order them using frequency as weights. To achieve something like this, we can use either [Kosaraju's algorithm](https://en.wikipedia.org/wiki/Kosaraju%27s_algorithm) or [Tarjan's algorithm](https://en.wikipedia.org/wiki/Tarjan%27s_strongly_connected_components_algorithm). While Kosaraju's algorithm is conceptually simpler, Tarjan's algorithm automatically returns the components in a reverse topological order. I just `reversed()` the output of [py-tarjan](https://github.com/bwesterb/py-tarjan) to get a linear ordering of the network. Each strongly connected component is then sorted by frequency before being flattened.

Is it worth it? Admittedly, I have spent too much time figuring this out when I could have drilled vocabulary instead. I did not end up using this as I have failed to extensively parse words (scraping from an online dictionary takes a while), but I still think this is a good proof of concept for ordering the headwords of a cyclic dictionary. Maybe we can use this to determine which words can qualify as semantic primes. This experiment can be improved by using an offline Korean-Korean JSON dictionary, cutting down processing time.

In the end, I just used the algorithm presented in the paper to optimize the order at which I learn Hanja, the Chinese characters used in Korean.
