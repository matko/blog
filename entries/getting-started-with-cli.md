# start a new terminusdb instance using docker
`docker run --rm -p 6363:6363 --pull always -d -v ./storage:/app/terminusdb/storage --name terminusdb terminusdb/terminusdb-server`

# configure tdb-cli
```
tdb-cli setup
Welcome to tdb-cli, the CLI tool for TerminusDB!

This setup will ask a few questions to generate your configuration.
This configuration file will be stored at /home/matthijs/.tdb.yml.

? What instance of TerminusDB do you intend to connect to? Self-hosted TerminusDB
? What name do you wish to use for this server? local
? Server endpoint URL: http://localhost:6363
? Username: admin
? Password: ****
? Default organization (blank if none): admin
Configuration has been written to /home/matthijs/.tdb.yml. You are all set!
```

# immediately change password
`tdb-cli user set-password`

# create a database
`tdb-cli db create family`

# add a schema
get basic schema out:
`tdb-cli doc get family -g schema > schema.json`

add the following:
```
{
  "@type": "Class",
  "@id": "Person",
  "@key": {"@type": "Lexical",
           "@fields": ["name"]},
  "name": "xsd:string"
}
```

Then

`tdb-cli doc insert family -g schema < schema.json`

# add some basic data directly on cli
```
tdb-cli doc insert family -d '{"name": "Alice"}'
tdb-cli doc insert family -d '{"name": "Arthur"}'
tdb-cli doc insert family -d '{"name": "Bella"}'
tdb-cli doc insert family -d '{"name": "Briana"}'
tdb-cli doc insert family -d '{"name": "Bob"}'
tdb-cli doc insert family -d '{"name": "Clara"}'
tdb-cli doc insert family -d '{"name": "Carl"}'
```

# Change  the schema
change the definition of person to the following
```
{
  "@id":"Person",
  "@key": {"@fields": ["name" ], "@type":"Lexical"},
  "@type":"Class",
  "child_of": {"@class":"Person", "@type":"Set"},
  "name":"xsd:string"
}
```

Then

`tdb-cli doc insert family -g schema < schema.json`

# Change some data
```
tdb-cli doc replace family -d '{"name": "Bella", "child_of": ["Person/Alice", "Person/Arthur"]}'
tdb-cli doc replace family -d '{"name": "Briana", "child_of": ["Person/Alice", "Person/Arthur"]}'
tdb-cli doc replace family -d '{"name": "Clara", "child_of": ["Person/Bella", "Person/Bob"]}'
```

# Launch graphql
`tdb-cli graphql serve family -o`

## some fun queries
### Get all people
```
query {
  Person {
    name
  }
}
```

### Get Bella and her parents
```
query {
  Person(filter:{name:{eq: "Bella"}}) {
    name
    child_of { name }
  }
}
```

### Get the children of Arthur
```
query {
  Person(filter:{name:{eq: "Arthur"}}) {
    name
    _child_of_of_Person {
      name
    }
  }
}
```

### Get all ancestors of Clara
```
query {
  Person(filter:{name:{eq: "Clara"}}) {
    name
		_path_to_Person(path:"child_of*") {
      name
    }
  }
}
```
