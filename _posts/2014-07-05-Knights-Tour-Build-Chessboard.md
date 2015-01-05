---
layout: post
title: The Knight's Tour | Building The Chessboard
---
#### (Part 2)

In the previous part of this series of posts, we implemented a Graph data structure. Now lets use that Graph to build something a little more tailored to our needs.

The main function we will be writing in this post is going to build the actual chess board.  The first thing we need to figure out is how we want to represent the different squares on the board.

One option is to represent them as (x,y) coordinates. This would be convenient for constructing the graph, since we could loop through both axes to get a point for every square.

On the other hand, for the Graph it is necessary to represent each square with a single key for our adjacency list.

The approach I chose to go with is to create a helper function which converts from (x,y) coordinates to a numerical index. The numerical indices will start at 0 in the lower left corner and increase going from left to right across the board. I think this makes it easy to conceptualize the board, but if you want to go with another approach that is completely fine. The point is that you need a unique index for every square and a convenient way to connect them according to the rules of a knight.

This is my simple coordinate to ID function:

{% highlight javascript%}
var coordToId = function(x,y,n){
  return ((y * n) + x);
};
{% endhighlight %}

The next one we need is a routine to figure out the legal moves for a given square, this is why it is really convenient to have the squares identified by coordinates.

This function basically builds up all the possible positions and then filters the out-of-bounds ones before returning the Array.

{% highlight javascript%}
var getLegalMoves = function(x,y,n){
  var possibleMoves = [
    [x + 1, y + 2],
    [x + 1, y - 2],
    [x + 2, y + 1],
    [x + 2, y - 1],
    [x - 1, y + 2],
    [x - 1, y - 2],
    [x - 2, y + 1],
    [x - 2, y - 1]
  ];

  var legalMoves = [];
  for (var i = 0; i < possibleMoves.length; i++) {
    var pm = possibleMoves[i];
    if (pm[0] > 0 && pm[0] < n && pm[1] > 0 && pm[1] < n){
      legalMoves.push(coordToId(pm[0], pm[1], n));
    }
  }
  return legalMoves;
};
{% endhighlight %}

Finally, We can build the board. All of the code up till now basically allowed us to write this function. It builds a chessboard of side-length: n by looping through all the squares and creating edges for all legal moves.

{% highlight javascript%}
var buildBoard = function(n){
  var graph = new Graph();
  graph.n = n;

  for (var x = 0; x < n; x++) {
    for (var y = 0; y < n; y++) {
      var vertex = coordToId(x,y,n);
      var legalMoves = getLegalMoves(x,y,n);

      for (var k = 0; k < legalMoves.length; k++) {
        lm = legalMoves[k];
        graph.addEdge(vertex, lm);
      }
    }
  }
  return graph;

};
{% endhighlight %}

Thats it for Part 2. In the next post I'll walk through a recursive solution for traversing this graph to find a knight's tour.
