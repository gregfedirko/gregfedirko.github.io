---
layout: post
title: Seeding a Database Using Promises
---

This post presents my approach to seeding a database utilizing a promise library called bluebird.  The prompt for this post is a recent project of mine called keydash, which allows programmers to improve their typing skills while writing code in different languages.  

I wanted to create a simple way to seed a database with exercises.  My approach was to import files from an exercise directory.  All the essential information about the exercise would be parsed from a standardized filename format as well as the file contents.  Therefore, any time a new exercise is needed, a file can simply be added to the appropriate directory.

The asynchronous nature of this function made it a perfect candidate for Promises.

## Key Methods
Below are references to the bluebird docs for the key functions utilized.  The completed code follows.

###Promise.promisifyAll

>```Promise.promisifyAll(Object target [, Object options])``` -> ```Object```

>Promisifies the entire object by going through the object's properties and creating an async equivalent of each function on the object and its prototype chain. The promisified method name will be the original method name suffixed with ```"Async"```. Any class properties of the object (which is the case for the main export of many modules) are also promisified, both static and instance methods. Class property is a property with a function value that has a non-empty ```.prototype``` object. Returns the input object.
>
>Note that the original methods on the object are not overwritten but new methods are created with the Async-suffix. For example, if you promisifyAll() the node.js fs object use fs.statAsync() to call the promisified stat method.
Components
>
[Source](https://github.com/petkaantonov/bluebird/blob/master/API.md#promisepromisifyallobject-target--object-options---object)


This method was used to promisify the ```fs``` module in node. The fs module is a singleton, therefore it would only need to be promisified once in an application.

###.map

This method was used following ```fs.readdirAsync```, an array of promises was created representing dataObjects with each exercises' title, language, and contents.

>```.map(Function mapper [, Object options])``` -> ```Promise```
>
>Map an array, or a promise of an array, which contains promises (or a mix of promises and values) with the given ```mapper``` function with the signature ```(item, index, arrayLength)``` where item is the resolved value of a respective promise in the input array. If any promise in the input array is rejected the returned promise is rejected as well.

>The mapper function for a given item is called as soon as possible, that is, when the promise for that item's index in the input array is fulfilled. This doesn't mean that the result array has items in random order, it means that ```.map``` can be used for concurrency coordination unlike ```.all().call("map", fn).all()```.
>
>The return value of .map is a promise that is fulfilled with an array of the mapped values
>
[Source](https://github.com/petkaantonov/bluebird/blob/master/API.md#mapfunction-mapper--object-options---promise)

###.join

The join method was used to return a promise of an array of promises.  After the array of promises were all resolved, Mongo documents could then be created for each one.

>```Promise.join(Promise|Thenable|value promises..., Function handler)``` -> ```Promise```
>
>For coordinating multiple concurrent discrete promises. While ```.all()``` is good for handling a dynamically sized list of uniform promises, ```Promise.join``` is much easier (and more performant) to use when you have a fixed amount of discrete promises that you want to coordinate concurrently.

>[Source](https://github.com/petkaantonov/bluebird/blob/master/API.md#promisejoinpromisethenablevalue-promises-function-handler---promise)

### The Finished Code:

{% highlight javascript %}
var mongoose = require('mongoose');
var Promise = require('bluebird');
var join = Promise.join;
var fs = Promise.promisifyAll(require('fs'));

var exerciseSchema = mongoose.Schema({
  title: {type: String},
  language: {type: String},
  content: {type: String}
  });

var Exercise = mongoose.model('Exercise', exerciseSchema);

function createDefaultExercises() {

  var fileTypeMap = {
    'js': 'JavaScript',
    'py': 'Python',
    'rb': 'Ruby'
  };

  var directory = './exercise_files';

  fs.readdirAsync(directory).map(function(fileName) {
    var fileParts = fileName.split('.');

    var title = fileParts[0]
    .replace('-', ' ')
    .replace(/(?:^|\s)\S/g, function(a) { return a.toUpperCase(); });

    var language = fileTypeMap[fileParts[1]];

    var contents = fs.readFileAsync(directory + '/' + fileName, 'utf-8')
    .catch(function ignore() {});

      return join(contents, function(contents) {
        return {
          title: title,
          language: language,
          content: contents
        };
      })

  }).then(function(exerciseObjectArray) {

    for (var i = 0; i < exerciseObjectArray.length; i++) {
      Exercise.create(exerciseObjectArray[i]);
    }

  });

}

exports.createDefaultExercises = createDefaultExercises;

{% endhighlight %}
