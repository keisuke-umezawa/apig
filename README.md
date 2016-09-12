# apig: Golang RESTful API Server Generator
[![Build Status](https://travis-ci.org/wantedly/apig.svg?branch=master)](https://travis-ci.org/wantedly/apig)

apig is an RESTful API server generator.

* Input: Model definitions based on [gorm](https://github.com/jinzhu/gorm) annotated struct
* Output: RESTful JSON API server using [gin](https://github.com/gin-gonic/gin) including tests and documents

## Contents

* [Contents](#contents)
* [How to build and install](#how-to-build-and-install)
* [How to use](#how-to-use)
  + [1. Generate boilerplate](#1-generate-boilerplate)
  + [2. Write model code](#2-write-model-code)
  + [3. Generate controllers, tests, documents etc. based on models.](#3-generate-controllers-tests-documents-etc-based-on-models)
  + [4. Build and run server](#4-build-and-run-server)
* [Usage](#usage)
  + [`new` command](#new-command)
  + [`gen` command](#gen-command)
  + [API Document](#api-document)
* [API server specification](#api-server-specification)
  + [Endpoints](#endpoints)
  + [Available URL parameters](#available-url-parameters)
  + [Data Type](#data-type)
  + [Pagination](#pagination)
  + [Versioning](#versioning)
* [License](#license)

## How to build and install

Prepare Go 1.6 or higher.
Go 1.5 is acceptable, but `GO15VENDOREXPERIMENT=1` must be set.

After installing required version of Go, you can build and install `apig` by

```bash
$ go get -d -u github.com/wantedly/apig
$ cd $GOPATH/src/github.com/wantedly/apig
$ make
$ make install
```

`make` generates binary into `bin/apig`.
`make install` put it to `$GOPATH/bin`.

## How to use

### 1. Generate boilerplate

First, creating by `apig new` command.

```
$ apig new -u wantedly apig-sample
```

generates Golang API server boilerplate under `$GOPATH/src/gihhub.com/wantedly/apig-sample`.
You can omit `-u, --user` and `--vcs` options. The default of `-u, -user` is github username, and `--vcs` is `github.com`

### 2. Write model code

Second, write model definitions under models/. For example, user and email model is like below:

```
// models/user.go
package models

import "time"

type User struct {
	ID        uint       `gorm:"primary_key;AUTO_INCREMENT" json:"id" form:"id"`
	Name      string     `json:"name" form:"name"`
	Emails    []Email    `json:"emails" form:"emails"`
	CreatedAt *time.Time `json:"created_at" form:"created_at"`
	UpdatedAt *time.Time `json:"updated_at" form:"updated_at"`
}
```

```
// models/email.go
package models

type Email struct {
	ID      uint   `gorm:"primary_key;AUTO_INCREMENT" json:"id" form:"id"`
	UserID  uint   `json:"user_id" form:"user_id"`
	Address string `json:"address" form:"address"`
	User    *User  `json:"user form:"user`
}
```

This models are based on [gorm](https://github.com/jinzhu/gorm) structure.
Please refer [gorm document](http://jinzhu.me/gorm/) to write detailed models.

### 3. Generate controllers, tests, documents etc. based on models.

Third, run the command:

```
apig gen
```

It creates all necessary codes to provide RESTful endpoints of models.

### 4. Build and run server

Finally, just build as normal go code.

```bash
$ go get ./...
$ go build -o bin/server
```

After that just execute the server binary.
For the first time, you may want to use `AUTOMIGRATE=1` when running the server.

```
$ AUTOMIGRATE=1 bin/server
```

When `AUTOMIGRATE=1`, the db tables are generated automatically.
After that, you can run the server just executing the command:

```
$ bin/server
```

The server runs at http://localhost:8080


## Usage

### `new` command
`new` command tells apig to generate API server skeleton.

```
$ apig new NAME
```

### `gen` command
`gen` command tells apig to generate files (routes, controllers, documents...) from [gorm](https://github.com/jinzhu/gorm) model files you wrote.

You MUST run this command at the directory which was generated by `new` command.

```
$ apig gen
```

### API Document

API Documents are generated automatically in `docs/` directory in the form of [API Blueprint](https://apiblueprint.org/).

```
docs
├── email.apib
├── index.apib
└── user.apib
```

[Aglio](https://github.com/danielgtaylor/aglio) is an API Blueprint renderer.
Aglio can be installed by

```
$ npm install -g aglio
```

You can generate HTML files and run live preview server.

```
// html file
$ aglio -i index.apib  -o index.html

// running server on localhost:3000
$ aglio -i index.apib --server
```

`index.apib` includes other files in your blueprint.

## API server specification

### Endpoints

Each resource has 5 RESTful API endpoints.
Resource name is written in the plural form.

|Endpoint|Description|Example (User resource)|
|--------|-----------|-------|
|`GET /<resources>`|List items|`GET /users` List users|
|`POST /<resources>`|Create new item|`POST /users` Create new user|
|`GET /<resources>/{id}`|Retrive the item|`GET /users/1` Get the user which ID is 1|
|`PUT /<resources>/{id}`|Update the item|`PUT /users/1` Update the user which ID is 1|
|`DELETE /<resources>/{id}`|Delete the item|`DELETE /users/1` Delete the user which ID is 1|

### Available URL parameters

#### `GET /<resources>` and `GET /<resources>/{id}`

|Parameter|Description|Default|Example|
|---------|-----------|-------|-------|
|`fields=`|Fields to receive|All fields|`name,emails.address`|
|`preloads=`|Nested resources to preload|(empty)|`emails,profile`|
|`pretty=`|Prettify JSON response|`false`|`true`|

#### `GET /<resources>` only

|Parameter|Description|Default|Example|
|---------|-----------|-------|-------|
|`stream=`|Return JSON in streaming format|`false`|`true`|
|`ids=`|Item IDs|(empty)|`1,2,5`|
|`limit=`|Maximum number of items|`25`|`50`|
|`page=`|Page to receive|`1`|`3`|
|`last_id=`|Beginning ID of items|(empty)|`1`|
|`order=`|Order of items|`desc`|`asc`|
|`v=`|API version|(empty)|`1.2.0`|

### Data Type

#### Request

API server accepts the form of `JSON` or `Form`.

`application/json`

```
curl -X POST http://localhost:8080/resources \
     -H "Content-type: application/json" \
     -d '{"field":"value"}'
```

`application/x-www-form-urlencoded`

```
curl -X POST http://localhost:8080/users \
     -d 'field=value'
```

`multipart/form-data`

```
curl -X POST http://localhost:8080/users \
     -F 'field=value'
```

#### Response

Response data type is always `application/json`.

### Pagination

API server supports 2 pagination types.

#### Offset-based pagination

Retrive items by specifying page number and the number of items per page.

For example:

```
http://example.com/api/users?limit=5&page=2
```

```
+---------+---------+---------+---------+---------+---------+---------+
| ID: 5   | ID: 6   | ID: 7   | ID: 8   | ID: 9   | ID: 10  | ID: 11  |
+---------+---------+---------+---------+---------+---------+---------+
          |                                                 |
 Page 1 ->|<-------------------- Page 2 ------------------->|<- Page 3
```

Response header includes `Link` header.

```
Link:   <http://example.com/api/users?limit=5&page=3>; rel="next",
        <http://example.com/api/users?limit=5&page=1>; rel="prev"
```

#### ID/Time-based pagination

Retrive items by specifying range from a certain point.

For example:

```
http://example.com/api/users?limit=5&last_id=100&order=desc
```

```
+---------+---------+---------+---------+---------+---------+---------+
| ID: 94  | ID: 95  | ID: 96  | ID: 97  | ID: 98  | ID: 99  | ID: 100 |
+---------+---------+---------+---------+---------+---------+---------+
          |               5 items (ID < 100)                |
          |<------------------------------------------------|
```

Response header includes `Link` header.

```
Link:   <http://example.com/api/users?limit=5&last_id=95&order=desc>; rel="next"
```

### Versioning

API server uses [Semantic Versioning](http://semver.org) for API versioning.

There are 2 methods to specify API version.

#### Request header

Generally we recommend to include API version in `Accept` header.

```
Accept: application/json; version=1.0.0
```

#### URL parameter

You can also include API version in URL parameter.
This is userful for debug on browser or temporary use,

```
http://example.com/api/users?v=1.0.0
```

This method is prior to request header method.

## License
[![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)](LICENSE)
