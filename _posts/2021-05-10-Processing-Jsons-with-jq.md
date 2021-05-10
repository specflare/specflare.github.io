---
title: "Processing JSON documents with jq - an example-based approach"
toc: true
toc_label: "Contents"
tags:
  - JSON
  - jq

date: May 10, 2021
layout: single
classes: wide
excerpt: "Using the jq command-line JSON processor."
---

## Introduction
**jq** is a flexible command-line JSON processor that can be used to filter, slice, map & transform JSON documents. It can be used in conjuction with Linux pipes and output redirection.  
`jq` is for JSON like `sed/awk/grep` are for structured text.
It accepts input from both a JSON file or stdin through a pipe or `xargs`.

The simplest invocation of jq is to just beautify the JSON input:
```bash
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '.'
```
As you see, for these examples we will use the JsonPlaceholder testing API. It will generate us a Json containig 10 different users with this structure:
```json
[
  {
    "id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz",
    "address": {
      "street": "Kulas Light",
      "suite": "Apt. 556",
      "city": "Gwenborough",
      "zipcode": "92998-3874",
      "geo": {
        "lat": "-37.3159",
        "lng": "81.1496"
      }
    },
    "phone": "1-770-736-8031 x56442",
    "website": "hildegard.org",
    "company": {
      "name": "Romaguera-Crona",
      "catchPhrase": "Multi-layered client-server neural-net",
      "bs": "harness real-time e-markets"
    }
  }
]
```
### Extracting just a subset of users
```bash
# Extracts just the first user
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '.[0]'

# Extracts just the last user
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '.[-1]'

# Array slicing (like in Python)
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '.[2:4]'
```

### Selecting just some fields for each user
```bash
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '[.[] | { id: .id, name: .name, company: .company.name }]'
```
`.[]` returns each element of the array, like a stream of sub-documents, that will be fed one by one into `{ id: .id, name: .name, company: .company.name }`.
The beginning `[` and the ending ']' tell `jq` to make an array out of the result.
It will print something like this:
```json
[
  {
    "id": 1,
    "name": "Leanne Graham",
    "company": "Romaguera-Crona"
  },
  {
    "id": 2,
    "name": "Ervin Howell",
    "company": "Deckow-Crist"
  }
  ...
]
```
### Pipelining multiple JSON operations
```bash
# Returns the last element of the already processed JSON
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '[.[] | { id: .id, name: .name, company: .company.name }] | .[-1]'
```

### Filtering JSON array contents based on specific fields
```bash
# Filter by numeric field
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '.[] | select(.id == 3)'

# Filter by text field
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '.[] | select(.company.catchPhrase | contains("neural-net"))'

# Filterig by multiple fields
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '.[] | select(.id == 1 and .name == "Leanne Graham")'

# Filtering based on a regular expression (keep only the users with .com websites)
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '.[] | select(.website | test("^.*.com"))'
```

### Deleting a field or a sub-tree from the JSON document
```bash
$ curl 'https://jsonplaceholder.typicode.com/users' | jq '.[] | del(.company)'
```

### Sort the array contents by a given field
```bash
$ curl 'https://jsonplaceholder.typicode.com/users' | jq 'sort_by(.name)'
```

## Conclusion
Those are the most common examples for the `jq` command. For more advanced use-cases check the [jq Cookbook](https://github.com/stedolan/jq/wiki/Cookbook).
You can also use [this](jqterm.com) website to test your `jq` queries on custom input data.