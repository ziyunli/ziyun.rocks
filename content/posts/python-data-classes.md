---
title: "Data Classes in Python"
tags: [python]
categories: [TILs]
date: 2022-08-17
featured: false
draft: false
---

<!--more-->

`dataclass` decorator is introduced since Python 3.7. A decorated class behaves like a regular Python class, but it automatically generates several dunder methods: `__init__`, `__eq__`, and `__repr__`. The auto-generated constructor method is particular helpful if you have many fields in a class (e.g. a wrapper class for JSON objects). It also introduces a new dunder method was defined for any additional processing: `__post_init__`.

Also note that each field has a type hint, which is great in terms of self-documenting.

An example from https://github.com/ErnstHaagsman/swapi/blob/master/main.py

```python
@dataclass
class StarWarsMovie:
    title: str
    episode_id: int
    opening_crawl: str
    director: str
    producer: str
    release_date: datetime
    characters: List[str]
    planets: List[str]
    starships: List[str]
    vehicles: List[str]
    species: List[str]
    created: datetime
    edited: datetime
    url: str
    def __post_init__(self):
        if type(self.release_date) is str:
            self.release_date = dateutil.parser.parse(self.release_date)
        if type(self.created) is str:
            self.created = dateutil.parser.parse(self.created)
        if type(self.edited) is str:
            self.edited = dateutil.parser.parse(self.edited)
```
