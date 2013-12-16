# Express Resource Plus

Sequelize Express Resource is a fork of [express-resource new](https://github.com/tpeden/express-resource-new) providing a few more routes, refactors and an extra feature or two.

## Installation

npm:
    $ npm install express-resource-plus

## Usage

In your main application file (i.e. app.js or server.js) just add the following:

    var express = require('express'),
        Resource = require('express-resource-plus'), // <- Add this (Resource really isn't needed)
        app = express.createServer();
    
    app.configure(function(){
      // it is important to note that /app directory layout must follow a specific pattern to work properly see [App Layout](#app-layout)
      app.set('app_dir', __dirname + '/app');
      /* ... */
    });

### App Layout
  /app/:version/controllers/:controller_name/index.js
                routes.js


"What if I want to create a resource on the root path or change the id variable name or define middleware on specific actions?" express-resource-plus handles that by allowing you to set an `options` property on the controller object like so:

    module.exports = {
      options: {
        root: true, // Creates resource on the root path (overrides name)
        name: 'posts', // Overrides module name (folder name)
        id: 'id', // Overrides the default id from singular form of `name`
        before: { // Middleware support
          show: auth,
          update: [auth, owner],
          destroy: [auth, owner]
        }
      },
      index: function(request, response) {
        response.send('articles index');
      },
      /* ... */
    };

Lastly just call `app.resource()` with your controller name. Nesting is done by passing a function that can call `app.resource()` for each nested resource. Options can also be passed as the second parameter which override the options set in the controller itself.

    app.resource('articles', { version : 'v1'} function() {
      app.resource('comments', { version : 'v1', id: 'id' }); // You can also call `this.resource('comments')`
    });

You can also create non-standard RESTful routes.

`./controllers/comments/index.js`:

    module.exports = {
      index: function(request, response) {
        response.send('comments index');
      },
      /* ... */
      search: function(request, response) {
        response.send("Search all comments on this article.");
      },
      reply: function(request, response) {
        response.send("Reply to comment: " + request.params.comment);
      }
    };

`./app.js`:

    /* ... */
    app.resource('articles', function() {
      this.resource('comments', function() {
        this.collection.get('search');
        this.member.get('reply');
      });
    });

## Default Action Mapping

Actions are, by default, mapped as shown below. These routs provide `req.params.article` for the substring where ":article" is and, in the case of the nested routes, `req.params.comment` for the substring where ":comment" is as shown below:

    articles:
    index   GET     /articles.:format?
    new     GET     /articles/new.:format?
    create  POST    /articles.:format?
    show    GET     /articles/:article.:format?
    edit    GET     /articles/:article/edit.:format?
    update  PUT     /articles/:article.:format?
    update  PUT     /articles.:format?  ( { where : { username : 'Steve' } })
    destroy DELETE  /articles/:article.:format?
    destroy DELETE  /articles.:format?  ( { where : { username : 'Steve' } })
    query   POST    /articles/query.:format?  ( { where : { username : 'Steve' } })

    article_comments:
    index   GET     /articles/:article/comments.:format?
    new     GET     /articles/:article/comments/new.:format?
    create  POST    /articles/:article/comments.:format?
    show    GET     /articles/:article/comments/:comment.:format?
    edit    GET     /articles/:article/comments/:comment/edit.:format?
    update  PUT     /articles/:article/comments/:comment.:format?
    update  PUT     /articles/:article/comments.:format?  ( { where : { username : 'Steve' } })
    destroy DELETE  /articles/:article/comments/:comment.:format?
    destroy DELETE  /articles/:article/comments.:format?  ( { where : { username : 'Steve' } })
    query   POST    /articles/:article/comments/query.:format?  ( { where : { username : 'Steve' } })

## Content Negotiation

Content negotiation is currently only provided through the `req.params.format` property, allowing you to respond accordingly.

## License

    The MIT License

    Copyright (c) 2013 Simon Townsend <stowns3@gmail.com>

    Permission is hereby granted, free of charge, to any person obtaining
    a copy of this software and associated documentation files (the
    'Software'), to deal in the Software without restriction, including
    without limitation the rights to use, copy, modify, merge, publish,
    distribute, sublicense, and/or sell copies of the Software, and to
    permit persons to whom the Software is furnished to do so, subject to
    the following conditions:

    The above copyright notice and this permission notice shall be
    included in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
    EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
    MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
    IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
    CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
    TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
    SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
