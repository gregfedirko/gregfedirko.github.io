---
layout: post
title: Caching Data in AngularJS
---
Many times in web applications, different views rely on the same data.  In situations where it is unlikely to change very frequently, you can make your app more performant by caching your data.  This post presents one approach to caching data in AngularJS.

Lets start with creating a service that will return a resource for a generic Items API:

{% highlight javascript %}
angular
  .module('app')
  .factory('dataService', function($resource) {
      var dataResource = $resource(
          '/api/items/:id',
          {_id: '@id'}
        );
      return dataResource;
    });
{% endhighlight %}

Now if that data is required for a view, we can inject the service into the controller and call the query method like so:

{% highlight javascript %}
angular
  .module('app')
  .controller('MainController', function($scope, dataService) {
    $scope.items = dataService.query();
  });
{% endhighlight %}

Now lets cache this data by creating another service. In order to maintain a consistent API, our method of caching should act just like the normal dataService.  This can be accomplished by injecting the original ```dataService``` into our new ```cachedDataService```:

{% highlight javascript %}
angular
  .module('app')
  .factory('cachedDataService', function(dataService) {
    var data;
    var service = {
      query = function() {
        if (!data) {
          data = dataService.query();
        }
        return data;
      }
    }
    return service;
  });
{% endhighlight %}

As you can see, in the event that the data has not been cached, the consumer of this service will just be talking to the normal dataService, and if it has already been called, the value of the data variable will just be returned.  If at any time your controller requires fresh data, it can just use the original dataService instead, this is one benefit over the built in caching in AngularJS.

{% highlight javascript %}
angular
  .module('app')
  .controller('MainController', function($scope, cachedDataService) {
    $scope.items = cachedDataService.query();
  });
{% endhighlight %}

And there you go! One simple approach to caching data in AngularJS.  If your caching needs are more complex, I recommend you check out an excellent library called [Breeze.js](http://www.getbreezenow.com/).
