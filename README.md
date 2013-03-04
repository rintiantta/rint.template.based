# Rint.template.based
I'm not good at English.... Sorry....

## project composition

## command

```
$ rint_create template.html variables.json output.js
```

## configuration file
configuration file is json file that include the information about variables. the variables must be wraped with `double underscore(__)`.
```javascript
{
    "__rint_variables__": "test_string",
    "__rint_other_variables__": ["test_a", "test_b", "test_c"]
}
```


# Template File
## rint-property about basic
### data-rint="variables"
`data-rint="variables"` allows to declare variables. It must be set on the top of the document. The variables name must be wrap with `double underscore(__)`.
```html
<script>
    __rint_project_name__:string
    __rint_collections__:array
</script>
```
### data-rint="code"
`data-rint="code"` allows to create the code with template.
#### template file
```html
<!DOCTYPE rint>
<script data-rint="variables">
    __rint_project_name__:string
    __rint_collections__:array
</script>
<script data-rint="code">
   var db = require('mongojs').connect('__rint_project_name__', __rint_collections__)
</script>
```
#### configuration file
```json
{
    "__rint_project_name__": "basic",
    "__rint_collections__": ["users", "places", "posts"]
}
```
#### shell
```
$ rint_create template.html variables.json output.js
```
#### output
```javascript
var db = require('mongojs').connect('basic', ['users', 'places', 'posts'])
```

## rint-property with creating the dynamic code.
### data-rint="each"
`data-rint="code"` allows to create the code loop.
#### template file
```html
<!DOCTYPE rint>
<script data-rint="variables">
    __rint_project_name__:string
    __rint_collections__:array
</script>
<script data-rint="code">
    var express = require('express')
    var http = require('http')
    
    var app = express()
    var db = require('mongojs').connect('__rint_project_name__', __rint_collections__)
</script>
<script data-rint="each" data-rint-each="__rint_collection__ in __rint_collections__">
    app.get('/__rint_collection__', function (request, response) {
        db.__rint_collection__.find(function (error, results) {
            response.json(error || result)
        })
    })    
    
    app.get('/__rint_collection__/:id', function (request, response) {
      db.__rint_collection__.findOne({
          _id: db.ObjectId(request.param('id'))
      }, function (error, result) {
          response.json(error || result)
      })
    })
</script>
<script data-rint="code">
    http.createServer(app).listen(3000)
</script>
```
#### configuration file
```javascript
{
    "__rint_project_name__": "basic",
    "__rint_collections__": ["users", "posts"]
}
```
#### output file
```javascript
var express = require('express')
var http = require('http')

var app = express()
var db = require('mongojs').connect('basic', ['users', 'posts'])

app.get('/users', function (request, response) {
    db.users.find(function (error, results) {
        response.json(error || result)
    })
})    

app.get('/users/:id', function (request, response) {
  db.users.findOne({
      _id: db.ObjectId(request.param('id'))
  }, function (error, result) {
      response.json(error || result)
  })
})

app.get('/posts', function (request, response) {
    db.posts.find(function (error, results) {
        response.json(error || result)
    })
})    

app.get('/posts/:id', function (request, response) {
  db.posts.findOne({
      _id: db.ObjectId(request.param('id'))
  }, function (error, result) {
      response.json(error || result)
  })
})

http.createServer(app).listen(3000)
```

## rint-property with the file system.
### data-rint="import"
### data-rint="export"
`data-rint="export"` helps to create a file and import it with replacement. It must be userd with `data-rint-export="__filename__"` and `data-rint-replace="__code__"`.

### data-rint="imports"

#### template file
#### configuration file
#### output

### data-rint="exports"
`data-rint="exports"` helps to create files and import it with replacement. It must be userd with `data-rint-each="__item__ in __items__"`,  `data-rint-export="__filename__"` and `data-rint-replace="__code__"`.

#### template file
```html
<!DOCTYPE rint>
<script data-rint="variables">
    __rint_project_name__:string
    __rint_collections__:array
</script>
<script data-rint="code">
    var express = require('express')
    var http = require('http')
    
    var app = express()
    var db = require('mongojs').connect('__rint_project_name__', __rint_collections__)
</script>
<script
    data-rint="each"
    data-rint-each="__rint_collection__ in __rint_collections__"
    data-rint-exports="__rint_collection__.js"
    data-rint-replacement="require('./__rint_collection__').active(app, db)">
    exports.active = function (app, db) {
        app.get('/__rint_collection__', function (request, response) {
            db.__rint_collection__.find(function (error, results) {
                response.json(error || result)
            })
        })    
        
        app.get('/__rint_collection__/:id', function (request, response) {
          db.__rint_collection__.findOne({
              _id: db.ObjectId(request.param('id'))
          }, function (error, result) {
              response.json(error || result)
          })
        })
    }
</script>
<script data-rint="code">
    http.createServer(app).listen(3000)
</script>
```
#### configuration file
```javascript
{
    "__rint_project_name__": "basic",
    "__rint_collections__": ["users", "posts"]
}
```
#### output
```javascript
var express = require('express')
var http = require('http')

var app = express()
var db = require('mongojs').connect('basic', ['users', 'posts'])

require('./users').active(app, db)
require('./posts').active(app, db)

http.createServer(app).listen(3000)
```
#### users.js
```javascript
exports.active = function (app, db) {
    app.get('/users', function (request, response) {
        db.users.find(function (error, results) {
            response.json(error || result)
        })
    })    
    
    app.get('/users/:id', function (request, response) {
      db.users.findOne({
          _id: db.ObjectId(request.param('id'))
      }, function (error, result) {
          response.json(error || result)
      })
    })
}
```
#### posts
```javascript
exports.active = function (app, db) {
    app.get('/posts', function (request, response) {
        db.posts.find(function (error, results) {
            response.json(error || result)
        })
    })    
    
    app.get('/posts/:id', function (request, response) {
      db.posts.findOne({
          _id: db.ObjectId(request.param('id'))
      }, function (error, result) {
          response.json(error || result)
      })
    })
}
``

## rint-property about supporting the project.


Template Based Programming01
