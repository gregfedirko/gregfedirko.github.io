---
layout: post
title: The Knight's Tour | Traversing a Graph
---

In the last post of the knight's tour, I am going to post my solution for recursing through the graph in order to find a successful path. Instead of walking through each line with code snippets, I have just displayed a heavily commented file for clarity.

This method utilizes recursion and backtracking.

{% highlight javascript %}
var knightsTour = function(chessBoard, startSquare, depth) {

  var n = chessBoard.n;

  var path = [];
  var actions = [];

  var first = chessBoard.getVertex(startSquare);

  var recurse = function(vertex){
    // push the current vertex to be explored
    // grab square and change class to found before pushing
    // into array
    path.push(vertex);
    // mark the vertex as visited so that it will not be revisited.
    vertex.visited = true;

    // if the function has completed the desired path, return true
    if (path.length === depth) {
      return true;

    } else { // explore the edges of the current node
      var connections = vertex.getEdges();
      var found = false;
      for (var i = 0; i < connections.length; i++) {
        var nextVertex = chessBoard.getVertex(connections[i]);

        // if the vertex has not been visited already, explore this path.
        if (!nextVertex.visited){
          found = recurse(nextVertex);
        }

        // if that path was successful, return true, which breaks
        // out of the loop and tells the function to end
        if (found) {
          return true;
        }
      }
    }
    // if the funcion has reached this point, all paths
    // from this node were dead ends;
    // backtrack by removing this vertex from the path
    path.pop();

    // mark the vertex as unvisited so that it can be
    // explored in further calls.
    vertex.visited = false;
    return false;
  };

  // Execute the subroutine
  if (recurse(first)){
    return path;
  } else {
    return false;
  }
};
{% endhighlight %}
