---
layout: post
title:  "Storing a Python dict as JSON in SQLite using SQLAlchemy"
date:   2020-09-06 02:00:57 +0200
categories: sqlite python
excerpt: I use this approach in a web app I'm writing to save user-defined templates and data that have arbitrary column names and values.
#proccessors: pymd
---

## Preface

I use this approach in a web app I am writing to save user-defined templates and
data that have arbitrary column names and values. Saving it as JSON feels good to
me. If the project grows I'll probably move over to Postgres, which also also
supports JSON columns.

## Example code

Neither a small, nor simple example, but this is a way to feed a column in SQLite
a `dict` and getting it stored as JSON using SQLAlchemy v1.3.
I am using the [*declarative system*][1] for mapping up the database.

```python
from sqlalchemy import create_engine, Integer, JSON, Column, Sequence
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base

EntityBase = declarative_base()


class Item(EntityBase):
    __tablename__ = "items"
    id = Column(Integer, Sequence("item_id_seq"), primary_key=True, nullable=False)
    information = Column(JSON, nullable=True)


# Setup a database connection. Using in-memory database here.
engine = create_engine("sqlite://", echo=True)

Session = sessionmaker(bind=engine)
session = Session()

# Create all tables derived from the EntityBase object
EntityBase.metadata.create_all(engine)

# Declare a new row
first_item = Item()
first_item.information = dict(a=1, b="foo", c=[1, 1, 2, 3, 5, 8, 13])

# Insert it into the database
session.add(first_item)
session.commit()

# Get all saved items from the database
for item in session.query(Item).all():
    print(type(item.information))
    # <class 'dict'>
    print(item.id, item.information)
    # 1 {'a': 1, 'b': 'foo', 'c': [1, 1, 2, 3, 5, 8, 13]}
```

## References
- <https://www.sqlite.org/json1.html>
- <https://stackoverflow.com/questions/31804378/how-to-query-on-a-json-type-field-with-sqlalchemy>
- <https://docs.sqlalchemy.org/en/13/>

[1]: https://docs.sqlalchemy.org/en/13/orm/extensions/declarative/basic_use.html
