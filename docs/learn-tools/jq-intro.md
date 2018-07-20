# An Introduction to jq

## TL;TR

> Command-line JSON processor http://stedolan.github.io/jq/

- It is functional
- You can pipe: `curl` `cat`

[Developer Guild](https://stedolan.github.io/jq/manual/)

[More Advanced Topics](https://github.com/stedolan/jq/wiki/Cookbook)

## Pipe to jq

[sample json](./suburb_house.json)

Open file in terminal using cat, then pipe the output to jq and output the whole content

`cat suburb_house.json | jq '.'`

Fetch from API, then pipe the response to jq, and output the whole content

`curl https://jsonplaceholder.typicode.com/posts | jq '.'`

Open file by giving jq the full file path

`jq '.' suburb_house.json`

## Basic Syntax

### The root object

`.`

### Drill into an object

`.|.data|.title`

Drill into an object `curl https://jsonplaceholder.typicode.com/posts/1 | jq '.title'`

Drill into an array of objects `curl https://jsonplaceholder.typicode.com/posts | jq '.[]|.id'`

### Iterator

`.[]` [JSON Lines](http://jsonlines.org/)

Turn to array `curl https://jsonplaceholder.typicode.com/posts | jq '.[]'`

Access array items `curl https://jsonplaceholder.typicode.com/posts | jq '.[3]'`

### Turn Iterator back to Array

Wrap the jq statement with `[]` will construct back to a new array

`curl https://jsonplaceholder.typicode.com/posts | jq '[.[]|.title]'`

### Length

Access array length `curl https://jsonplaceholder.typicode.com/posts | jq 'length'`

### Chain

`|` aka PIPE

turn to an array first -> access the value of 2007 `jq '.[]|."2007"' suburb_house.json`

```
"607000"
"1101000"
"330000"
"304000"
"230000"
"264500"
"149500"
"205000"
"350000"
"265000"
"226000"
"1301000"
"605000"
"330000"
"404000"
"165000"
"231500"
"385000"
"418000"
"450000"
```

### Multi properties

`curl https://jsonplaceholder.typicode.com/posts | jq '.[3]|.id,.title'`

```
"eum et est occaecati"
```

###