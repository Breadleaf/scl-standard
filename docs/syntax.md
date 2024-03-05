# Syntax

## Intro

SCL is an extremely simplistic language that strives to abstract away the syntax of popular configuration formats to allow for easy configuration of apps without thinking of semantics.

## Paths

A path is a sequence of identifiers that are to the left of an `Assignment`. A path will resolve such that all items in the path prior to the last element will be considered a map.

SCL
```py
today date = "March 5th 2024"
today time zone = "GMT-7"
today time time = 1435
```

In the above example there are two paths: `today/` and `today/time/`.

## Resolutions

A resolution is always the last item in a path. A resolution must always be followed by an `Assignment` then a value. Note there is a special type of resolution, the `Iterator` which is discussed below.

SCL
```py
today date = "March 5th 2024"
today time zone = "GMT-7"
today time time = 1435
```

In the above example there are three resolutions: `date`, `zone`, and `time`, each with their own values.

## Syntax Elements

As stated in the readme, this language has only three distinct syntax elements as listed below:

- Assignment: `=`
- Iterator: `_`
- Comment: `#`

These three elements are able to support a variaty of config setups; however they are limited.

### Assignment

The assignment operator is used to assign a value to an indentifier (see example below).

SCL
```py
var = 25
```

JSON
```json
{
    "var" : 25
}
```

### Iterator

Similar to json, SCL has lists and maps. The language makes the assumption that everything in a "path" is a map unless the ***last identifier is an underscore***. A underscore in the last position means to make that "resolution" into a list rather than a map. It should be noted that this operator can only take the position of the last item within a "path" therefor you can not nest maps within lists or lists within lists. However as will be discussed later on in this page, there are workarounds to this limitation. Below is an example of the iterator in use.

SCL
```py
party host = "alice"
party guests _ = "rob"
party guests _ = "jane"
party guests _ = "shane"
```

JSON
```json
{
    "party": {
        "host": "alice",
        "guests": [
            "rob",
            "jane",
            "shane"
        ]
    }
}
```

### Comment

Comments in SCL can be placed on their own line or at the end of a line. All characters until the end of the line will be ignored. Comments should be ignored in all implementations.

SCL
```py
# user preferences
name first = "adam"
name last = "smith" # TODO: add last name support
age = 24
```

## Limitation Workaround

As promised there is a workaround to the main limitation and it will require you to be a bit creative. Nevertheless, bellow I will expand on the party example and add what items each person should bring to the party.

SCL : `party.scl`
```py
party host = "alice"
party time start = 1500
party time end = 1900

party guests _ = "rob"
rob _ = "chips"
rob _ = "soda"

party guests _ = "jane"
jane _ = "ribs"

party guests _ = "shane"
shane _ = "salad"
```

PYTHON3 : `main.py`
```py
import scl

information = scl.load(open("party.scl", "r"))
party = information["party"]
guests = party["guests"]

brought_items = []
for guest in guests:
    brought_items.extend(information[guest])

print(brought_items)
```

OUTPUT : `stdout`
```py
$ python3 main.py
['chips', 'soda', 'ribs', 'salad']
```