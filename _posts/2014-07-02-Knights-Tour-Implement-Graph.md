---
layout: post
title: The Knight's Tour | Implementing a Graph
---
#### (Part 1)

The Knight's tour is one of my favorite problems in computer science. The goal is to move a knight across a chessboard so that it lands on every square only once.

This solution to this problem will be broken up into three parts.

1. The first part will be concerned with implementing a graph class in JavaScript.
2. Next, a customized graph will be built to represent the chess board.
3. The last part will explain an algorithm which traverses the data structure to find a path (if one exits).

This post will walk through implementing a graph using an adjacency list which will eventually represent a chessboard. I am not going to create a comprehensive implementation with every bell and whistle, this will just have the necessities required to solve the knight's tour problem.

First thing's first, lets create our classes using the pseudoclassical instantiation pattern in JavaScript.  We will need two types of classes, one to represent the nodes, and one to represent the Graph itself.

The Graph class has a property called ```vertList```. This is just a hash with pointers to each node contained within the graph.

{% highlight javascript %}
var Graph = function () {
  this.vertList = {};
};
{% endhighlight %}


The information regarding the relationships between nodes is contained within the ```Vertex``` class.

The ```connections``` array is a collection of strings, each one represents a key on the ```Graph.vertList``` hash, which you can use to find a reference to a connected Vertex.

Note: this graph will be an unweighted graph, meaning there is no cost to go from one point to another on a chessboard. If we wanted to create a graph that would represent something like point's on a map, then this connections Array would store each node as a tuple or a hash, that way you could keep track of the cost to get from one current point to another one.



{% highlight javascript %}
var Vertex = function(value){
  this.value = value;
  this.connections = [];
};
{% endhighlight %}


Next, lets finish off the Vertex class before going back to the Graph class.

For this implementation, we are going to add in a method for getting the list of edges. We dont really need this method because we could just access the list of edges like we would in any object. However, It's a good idea to create an API to your classes. One, it's just cleaner. And two, it will prevent bugs because you wont have others (or yourself) poking around data in unpredictable ways, you know that you always need to access it through a designated API. If you need to access the data in a different way, then think about the inputs and outputs, and then create the necessary getter or setter method.

{% highlight javascript %}
Vertex.prototype.getEdges = function(){
  var shallowCopy = this.connections.slice();
  return shallowCopy;
}
{% endhighlight %}

For the ```getEdges``` method, we want to create a shallow copy of the Array first, if we just returned the array, then whatever receives it would not just have the connections, It would also have a pointer the the actual data. Creating a copy first prevent's this problem.

So that's all we need on the ```Vertex class```, now lets go back to our ```Graph``` class.

First, lets add in some methods for adding and retrieving Vertices:

{% highlight javascript %}

Graph.prototype.addVertex = function(node){
  this.vertList[node] = new Vertex(node);
};

Graph.prototype.getVertex = function(nodeStr){
  return this.vertList[nodeStr];
};
{% endhighlight %}

We will also need a method for adding a new edge to the graph:

{% highlight javascript %}

Graph.prototype.addEdge = function(fromNodeKey, toNodeKey){
  // if the fromNode does not exist, create it.
  if (!this.vertList[fromNodeKey]) {
    this.addVertex(fromNodeKey);
  }
  // if the toNode does not exist, create it.
  if (!this.vertList[toNodeKey]) {
    this.addVertex(toNodeKey);
  }
  this.vertList[fromNodeKey].connections.push(toNodeKey);
};
{% endhighlight %}


This method first checks if the referenced nodes exist, then it creates them if needed before making the connection.

And that completes the implementation of a Graph Data Structure.  In the next post we will be using it to build a chess board.
