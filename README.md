# sqla_yaml_fixtures

Load YAML data fixtures for SQLAlchemy

This package allows you to define some data in YAML and load them into a DB. The yaml data should correspond to SQLAlchemy declarative mappers.

Example:

```YAML
User:
  - __key__: joey
    username: joey
    email: joey@example.com
    profile:
      name: Jeffrey

  - __key__: dee
    username: deedee
    email: deedee@example.com

Profile:
  - user: dee
    name: Douglas

Group:
  - name: Ramones
    members: [joey.profile, dee.profile]
```

- The root of YAML contain `mapper` names i.e. `User`
- Every mapper should contain a *list* of instances
- Each instance is mapping of *attribute* -> *value*
- the attributes are taken from the mapper `__init__()` (usually an attributes maps to a column)
- The special field `__key__` can be used to identify this instnace in a relationship reference .i.e. The `Profile.user`
- Note that any `__key__` MUST be globally **unique**
- In a *to-one* relationship the data can be directly nested in the parent data definition
- References can access attributes using a *dot* notaion, i.e. `joey.profile`
- *to-many* relationships can be added as a list of references


The mapper definition for this example is in the [test file](https://github.com/schettino72/sqla_yaml_fixtures/blob/master/test_sqla_yaml_fixtures.py).


## API

This module expose a single function `load(ModelBase, session, fixture_text)`

Where:

- `ModelBase` is SQLAlchemy declarative base
- `session` is SQLAlchemy session
- `fixture_text` is a string containg the YAML fixtures


```Python
from sqlalchemy import create_engine
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import Session

import sqla_yaml_fixtures

BaseModel = declarative_base()

class User(BaseModel):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    username = Column(String(150), nullable=False, unique=True)
    email = Column(String(254), unique=True)


def main():
    engine = create_engine('sqlite://')
    BaseModel.metadata.create_all(engine)
    connection = engine.connect()
    session = Session(bind=connection)

    fixture = """
    User:
      - username: deedee
        email: deedee@example.com
      - username: joey
        email: joey@example.commit
    """
    sqla_yaml_fixtures.load(BaseModel, session, fixture)
    session.commit()

    print('\n'.join(u.username for u in session.query(User).all()))

if __name__ == '__main__':
    main()
```