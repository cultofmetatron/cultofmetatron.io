---
Categories:
- Development
- elixir
Description: "first impressions with elixir"
Tags:
- Development
- elixir
date: 2017-01-02T00:13:42-05:00
menu: blog
title: Elixir first impressions
---


Software engineering is a game of tradeoffs. 
Both for building systems that serve our users with reliability and cost that keeps us in buisness. I've delved into learning Elixir as a new toolkit for developing software and experiences with more reliablity and less downtime.
Elixir is a langugae that runs on top of the Erlang VM. While still young for a langugae, it has the benefit of leveraging the Erlang platform; a best of breed high performance system developed at Ericson to run cellphone and telecom operations running with minimal downtime.

When was the last time you had to deal with maintence based outages for your cell phone? 
<!--more--> 
By leveraging the erlang VM, Elixir allows features like zero downtime, clean exception handling, featherweight vm threads that can take advatage of all cores on a cpu and hot code deployment. That last one is huge. I'm a big fan of continuous integration. In a startup, being able to deliver features fast and load new code into an already running vm without having to take down the system, is a tautalogically undeniable advantage when reliablity is a huge part of your value added proposition. The only arguable con I've encountered so far is the learning curve.

Intrigued by the promise of reliability and shortened time to market at zero cost, I delved in. I wanted to first try the features of the core language without relying on the fancy concurrency primatives as I get used to the ruby like syntax so I dug into a toy problem I previously created a solution for in javascript.

> The complete project is available on github @[elixir-lazermaze](https://github.com/cultofmetatron/elixir-lazermaze)


Given a Maze with a beam of light shining from somewhere in it. the maze has a length, width, a starting position and direction, and a set of mirrors oriented in either '\' or '/', return the number of squares traversed and at what location does the beam of light leave the maze.

For example, the file *basics.dat*, 

```
5 6
1 4 E
3 4 /
3 0 /
1 2 \
3 2 \
4 3 \
```

would visually be represented using an xy access starting from the top left.

```
+ 0 1 2 3 4
0       / 
1
2   \   \
3         \ 
4   E   /  
5
```

One particular edge case to be aware of is cycles. If the beam finds itself cycling between several mirrors reapeatedly, it will never exit the maze and we should return an error. The easiest way to check for this is to check the path if the log of previous squares already contains the square. To make this work, we need to store the direction of the beam at each point.

```
let path be an collection
while lookahead(position, direction) is not out of bounds
	1. if there is a mirror here, update the direction
    2. if the current position is already in the path 
       with the current direction, break the loop and return failure
	2. add the current position to the path
    3. update the position to the next position
if lookahead(position, direction) is not out of bounds
	return the count of the path and the current position
    
```

Mix is the build tool used by the elixir community. its basicly some kind of well put together hybrid of npm and gulp/broccolii/grunt.

###### To start a new project
```bash
mix new mirrormaze
```
This will create a project tree with a folder for tests and your source code called "lib". Inside teh lib folder, there is a mirrormaze.ex. for now we need to write a loader that can pull in the data from the input files and convert it into a data structure that elixir can use.

Create a file in lib called *maze_loader.ex*
```elixir
defmodule MazeLoader do
	
end

```

First I needed method **load_raw** that would return a file given a pathname.

```elixir

defmodule MazeLoader do
	def load_raw(file_path) do
    	{:ok, file} = File.read file_path
    	file
    end
end

```

Methods in elixir are declared using def or defp in the case of private methods.

Interestingly, assignment in elixir isn't quite the same as it is in javascript. in javascript, the right hand side is strictly evaluated and the result is assigned to the variable on the left hand side. Elixir instead implements a assignment as a series of constrains for which the runtime will either find a means of declaring values that is valid or throw an error.

**File.read** takes a file_path and returns a *tuple* containing a keyword indicating the status and the contents of the file. Because of this constraint solving property, we can destructure the left hand side to be a tuple. 

By using **:ok** in the first paramater, we are constraining the value returned from the read to a tuple with :ok as the first element. Otherwise, throw an error. This seems to be a common idiom in elixir.

If successful, the second paramater is assigned to *file*. The last statement in a method is implicitly returned like in ruby so by putting file after, we make the method return the file contents while making it throw an error if the file is not found.

The testing harness is already set up by mix. in the test directory, I create maze_loader_test.exs. The exs files are elixir scripts that skip coompilation.

```elixir
# test/maze_loader_test.exs
defmodule MazeLoaderTest do
  # this tests the maze loading components
  use ExUnit.Case
  test "it loads the raw file" do
    file = MazeLoader.load_raw("./samples-data/basic.dat")
    file_contents = "5 6\n1 4 E\n3 4 \/\n3 0 \/\n1 2 \\\n3 2 \\\n4 3 \\\n"
    assert file == file_contents
  end
end

```

To run the tests, run `mix test`

Now that we have the raw contents, we have to convert it to a native elixir data structure. 

### Built in data structures

Elixir data structures are very similar to the immutable types found in clojure. The collection modules like Enum follow a protocol based approach for operating on them. in practice, this means the api exposes a function that can operate on several data structures.

The main structures to know of are map %{}, tuple {} and list []

```elixir
iex(1)> a_tuple = {1, 2, 4}
{1, 2, 4}
iex(2)> a_list = [1, 2, 3]
[1, 2, 3]
iex(3)> a_map = %{:foo => "bar"}
%{foo: "bar"}
```

Tuples are static size. individual elements are accessed using **elem()** 
`elem({1, 2}, 1) => 2`

lists can be broken into a head and a tail but they can also be destructured using `[head | rest]`.

```elixir
iex(1)> list1 = [1, 2, 3, 4]
[1, 2, 3, 4]
iex(2)> list2 = [ 0 | list1 ]
[0, 1, 2, 3, 4]
iex(3)> [first | [second | rest]] = list2
[0, 1, 2, 3, 4]
iex(4)> first
0
iex(5)> second
1
iex(6)> rest
[2, 3, 4]
iex(7)> hd(list1)
1
iex(8)> tl(list1)
[2, 3, 4]

```


maps are key values. the keys can be any value.

```
iex(1)> map1 = %{ :foo => 'bar', "buz" => "cola", 3 => "red" }
%{3 => "red", :foo => 'bar', "buz" => "cola"}
iex(2)> map1[:foo]
'bar'
iex(3)> map1["buz"]
"cola"
iex(4)> map1[3]
"red"
```

*basic.dat* contains the string `"5 6\n1 4 E\n3 4 \/\n3 0 \/\n1 2 \\\n3 2 \\\n4 3 \\\n"`. MazeLoader.extract_data() should get something that looks like this.

```elixir
%{
    size: %{length: 5, width: 6},
    start: %{direction: "E", x: 1, y: 4},
    mirrors: [
      %{type: "/", x: 3, y: 4},
      %{type: "/", x: 3, y: 0},
      %{type: "\\", x: 1, y: 2},
      %{type: "\\", x: 3, y: 2},
      %{type: "\\", x: 4, y: 3}
    ]
}
```

Since we know what the output should be, lets start with the test
```
# test/maze_loader_test.exs
defmodule MazeLoaderTest do
  # this tests the maze loading components
  use ExUnit.Case

  ...

  test "it returns a processed file" do
    file = MazeLoader.load_raw("./samples-data/basic.dat")
    data = MazeLoader.extract_data(file)
    assert data == %{
      size: %{length: 5, width: 6},
      start: %{direction: "E", x: 1, y: 4},
      mirrors: [
        %{type: "/", x: 3, y: 4},
        %{type: "/", x: 3, y: 0},
        %{type: "\\", x: 1, y: 2},
        %{type: "\\", x: 3, y: 2},
        %{type: "\\", x: 4, y: 3}
      ]
    }
  end

end

```

Notice how data can be equality checked against the object literal? In Elixir, there is no pass by reference.  The consequence is that large complex data structures can be compared by deep value with a simple == statement. In javascript, you would have to use a library like [immutablejs](https://facebook.github.io/immutable-js/) to write code like this. The downside is that it is not idiomatic to the community the way immutable types are in elixir. Elixir wins big here in my book.


```elixir
defmodule MazeLoader do
  def extract_data(contents) do
    Enum.map(
    	Enum.filter(
        	String.split(contents, "\n"), fn(str) -> str !== "" end), 
       	fn(str)-> String.split(str, ~r/\s/) end)
    %{
      size: "Todo",
      start: "Todo",
      mirrors: "Todo"
    }
  end
end
```

In javascript, we can use lodash's chain() operator to write this in an easier to follow way.

```js
  let data = _.chain(contents.split("\n))
    .filter((str) => str !== "")
    .map((str) => str.split(/\s/))
    .value()

```

Elixer has pipe operator `|>` which enables the same functionality for any function.
**extract_data()** can instead be written as...

```elixir
defmodule MazeLoader do
  def extract_data(contents) do
    data = contents
      |> String.split("\n")
      |> Enum.filter(fn(str) -> str !== "" end) # remove empty newline without data
      |> Enum.map(fn(str)-> String.split(str, ~r/\s/) end)
    %{
      size: "Todo",
      start: "Todo",
      mirrors: "Todo"
    }
  end
end
```

The pipe operator is my new favorite operator. instead of thinking about about my code as a series of objects, I instead model it as a series of transformations with data from being piped along diffrent small methods.

The *data* value should now equal...
```
 [["5", "6"], ["1", "4", "E"], ["3", "4", "/"], ["3", "0", "/"],
  ["1", "2", "\\"], ["3", "2", "\\"], ["4", "3", "\\"]]
```

```elixir
defmodule MazeLoader do
  # ...

  # takes an array of mirrors and returns a list of mirrors with x, y and type
  defp extract_mirrors([_ | [_ | []]]), do: []
  defp extract_mirrors([_ | [_ | mirrors]]) do
    Enum.map mirrors, fn(mirror) ->
      # elem(Integer.parse(hd(mirror)), 0)
      %{
        x: mirror |> hd |> Integer.parse |> elem(0),
        y: mirror |> tl |> hd |> Integer.parse |> elem(0),
        type: mirror |> tl |> tl |> hd
      }
    end
  end

  #takes the dimmensions
  defp extract_dimension([dim | _ ]) do
    %{ 
      length: dim |> hd |> Integer.parse |> elem(0),
      width: dim |> tl |> hd |> Integer.parse |> elem(0)
    }
  end

  # extracts the start point and direction
  defp extract_start([_ | [ start | _ ]]) do
    %{
      x: start |> hd |> Integer.parse |> elem(0),
      y: start |> tl |> hd |> Integer.parse |> elem(0),
      direction: start |> tl |> tl |> hd
    }
  end
  
  def extract_data(contents) do
    data = contents
      |> String.split("\n")
      |> Enum.filter(fn(str) -> str !== "" end) # remove empty newline without data
      |> Enum.map(fn(str)-> String.split(str, ~r/\s/) end)
    %{
      size: extract_dimension(data),
      start: extract_start(data),
      mirrors: extract_mirrors(data)
    }
  end
  
  @doc """
    Takes the file and returns a maze spec
  """
  def load_file(file), do: file |> load_raw |> extract_data

end
```


`defp extract_mirrors([_ | [_ | []]])` matches to any list with two elemets and an empty or nonexistent list. This keeps type checking logic out of the body of the method. The second which is called if the first does not match can focus on its own specific case. This is a big win for keeping each method small.


Now that we have the loader module, we create the maze module and a test for the solve method. Elixir does not have loops. Instead we have to use tail recursion.

```elixir
defmodule Maze do

  # ...
  
  # starts with the basic maze and we track the position and path as we traverse
  def solver(size, mirrors, direction, position, count, path) do
    #IO.inspect({ size, mirrors, direction, position, count, path})
    # is there a mirror here? if yes we must changed the position accordingly
    new_direction = case get_mirror(position, mirrors) do
      {:success, mirror} -> update_direction(direction, mirror)
      {:fail } -> direction
    end
    next = next_position(position, new_direction)
    # get the next position and check if its out of bounds
    # Base case
    cond do
      # 1.  next position is out of bounds
      out_of_bounds(size, next) ->
        {:ok, position, count}
      # 2. next position is in the path with the same direction (infinite loop)
      in_path?(path, position, direction) ->
        {:fail, "cycle detected"}
      # Recursive step
      # 1. call with next position,
      true ->
        breadcrumb = { elem(position, 0), elem(position, 1), direction}
        solver(size, mirrors, new_direction, next, count + 1, [breadcrumb | path])
    end
  end

  def solve(maze) do
    #size, mirrors, direction, position, count, path
    size = { maze[:size][:width], maze[:size][:length] }
    position = {maze[:start][:x], maze[:start][:y] }
    direction = maze[:start][:direction]
    solver(size, maze[:mirrors], direction, position, 0, [])
  end

end

```


The heart of our solver more or less follows the pseudocode. each recursive call passes along the updated data for iteration till a base case is recieved which causes the function to return.

### case and cond
This code introduces a few new langauge features, cond and case.

```elixir
new_direction = case get_mirror(position, mirrors) do
      {:success, mirror} -> update_direction(direction, mirror)
      {:fail } -> direction # direction has not changed
    end
```

**case** takes an expression and executes code associated with the first matching pattern. get mirror takes the position and the set of mirrors and returns a mirror if successful or a tuple with a fail. The case evaluates to the valuated output of the associated expression. All we need to do is write mirror to follow this contract.

```elixir
def get_mirror({x, y}, mirrors) do
	# find_value returns the first truthy value retruned by the filtering callback.
    # otherwise, it returns false
    Enum.find_value(mirrors, fn(mirror) ->
      if (mirror[:x] == x && mirror[:y] == y) do
        { :success, mirror[:type] }
      else
        false
      end
    end) || { :fail }
  end
```

Similarly, next_position and update_direction can be written to pattern match against directions which keeps switching code out of the function body.

```elixir
  #gives the next pointer on the screen
  def next_position(position, direction \\ "S")
  def next_position({ x, y }, "N"), do: { x, y - 1 }
  def next_position({ x, y }, "S"), do: { x, y + 1 }
  def next_position({ x, y }, "E"), do: { x + 1, y }
  def next_position({ x, y }, "W"), do: { x - 1, y }
  
  def update_direction("S", "/"),   do: "W"
  def update_direction("S", "\\"),  do: "E"
  def update_direction("N", "/"),   do: "E"
  def update_direction("N", "\\"),  do: "W"
  def update_direction("E", "/"),   do: "N"
  def update_direction("E", "\\"),  do: "S"
  def update_direction("W", "/"),   do: "S"
  def update_direction("W", "\\"),  do: "N"
```

I'd rather write unit tests for 4 pure single line functions. the cyclometric complexity is effectivly zero.

Destructuring plays a major role in wringing out some increadibly succinct code.

```elixir
defmodule Maze do

  #gives the next pointer on the screen
  def next_position(position, direction \\ "S")
  def next_position({ x, y }, "N"), do: { x, y - 1 }
  def next_position({ x, y }, "S"), do: { x, y + 1 }
  def next_position({ x, y }, "E"), do: { x + 1, y }
  def next_position({ x, y }, "W"), do: { x - 1, y }

  def out_of_bounds({ l, w }, {x, y}) do
    (x >= l) || (y >= w) || (x < 0) || (y < 0)
  end

  # given a set of mirrors, return the mirror type or fail
  # { :success, "/" }
  # { :fail }
  def get_mirror({x, y}, mirrors) do
    Enum.find_value(mirrors, fn(mirror) ->
      if (mirror[:x] == x && mirror[:y] == y) do
        { :success, mirror[:type] }
      else
        false
      end
    end) || { :fail }
  end

  def update_direction("S", "/"),   do: "W"
  def update_direction("S", "\\"),  do: "E"
  def update_direction("N", "/"),   do: "E"
  def update_direction("N", "\\"),  do: "W"
  def update_direction("E", "/"),   do: "N"
  def update_direction("E", "\\"),  do: "S"
  def update_direction("W", "/"),   do: "S"
  def update_direction("W", "\\"),  do: "N"

  # takes a list of position and returns true or false
  def in_path?(path, {x, y}, direction) do
    # get a count of the amount of previous positions 
    # that match the current direction and path and return true if greater than 0
    Enum.count(path, fn({x1, y1, dir1}) ->
      (x == x1 && y == y1 && direction == dir1)
    end) > 0
  end

  # starts with the basic maze and we track the position and path as we traverse
  def solver(size, mirrors, direction, position, count, path) do
    #IO.inspect({ size, mirrors, direction, position, count, path})
    # is there a mirror here? if yes we must changed the position accordingly
    new_direction = case get_mirror(position, mirrors) do
      {:success, mirror} -> update_direction(direction, mirror)
      {:fail } -> direction
    end
    next = next_position(position, new_direction)
    # get the next position and check if its out of bounds
    # Base case
    cond do
      # 1.  next position is out of bounds
      out_of_bounds(size, next) ->
        {:ok, position, count}
      # 2. next position is in the path with the same direction (infinite loop)
      in_path?(path, position, direction) ->
        {:fail, "cycle detected"}
      # Recursive step
      # 1. call with next position,
      true ->
        breadcrumb = { elem(position, 0), elem(position, 1), direction}
        solver(size, mirrors, new_direction, next, count + 1, [breadcrumb | path])
    end
  end

  @doc """
    takes a maze and solves it
    
    A maze is a map with a size, start and set of mirrors
    maze = %{
      size: %{length: 5, width: 6},
      start: %{direction: "S", x: 1, y: 4},
      mirrors: [
        %{type: "/", x: 3, y: 4},
        %{type: "\", x: 3, y: 0},
      ]
    }
  """
  def solve(maze) do
    #size, mirrors, direction, position, count, path
    size = { maze[:size][:width], maze[:size][:length] }
    position = {maze[:start][:x], maze[:start][:y] }
    direction = maze[:start][:direction]
    solver(size, maze[:mirrors], direction, position, 0, [])
  end

end
```

So far, I'm loving Elixir. The combination of smooth destructuring, immutable types and paramater matching has made for some seriosly clean code. 


