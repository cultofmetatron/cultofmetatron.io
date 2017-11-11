---
Categories:
- Development
- Elixir
Description: "message passing with elixir"
Tags:
- Development
- elixir
date: 2016-12-02T00:15:41-05:00
menu: blog
title: concurrency by message passing in elixir
---
In my last post, I talked about Elixir's core syntax. While I love the brevity and pure functional style favored in elixir, I didn't even touch on it primary value added proposition. 
<!--more--> 
Elixir is really a process oriented programming langauge.

Rather than worry about low level details like semaphores or mutexes, The concurrency model in elixir is oriented around message passing between lightweight virtual machine processes. 

I decided to take mergesort and see how easily I could adapt it to run in parrallel.

The primary idea of mergesort is to split the array in half, recursivly call itself on both halves and merge the two sorted halves to get the final sorted array.

First I made a regular non concurrent version

```elixir

  # takes two sorted arrays and merges them
  def merge(list1, list2, acc \\ [])
  def merge([], [], acc), do: acc
  def merge([], list, acc), do: acc ++ list
  def merge(list, [], acc), do: acc ++ list
  def merge([head1 | tail1], [head2 | tail2], acc) do
    if head1 >= head2 do
      merge([head1 | tail1], tail2,   acc ++ [head2])
    else
      merge(tail1, [head2 | tail2], acc ++ [head1])
    end
  end

  def split(list, acc \\ [[], []])
  def split([], acc), do: acc
  def split([x], [left, right]) do
    [[x | left], right]
  end
  def split([x | [y | tail]], [left, right]) do
    split(tail, [[x | left], [y | right]])
  end

  @doc """
    Sorts the numbers
    iex> ParallelMergesort.sort([5, 4, 3, 2, 1])
    [1, 2, 3, 4, 5]
  """
  def sort([]), do: []
  def sort([a]), do: [a]
  def sort(list) do
    [left, right] = split(list)
    merge(sort(left), sort(right))
  end

```

visually, it looks like this,

```
[6, 5, 4, 3, 2, 1]
# breaks down to the expression
merge(
	merge(
    	merge([6], [5]), 
        [4])
	merge(
    	merge([3], [2]), 
        [1]))
```

In the original version, the left hand has to be processed before the right side can be processed. Seems we could gain an advantage by running the mergesort on both sides in parrallel.

To accomplish this, what we need to do is have each recursive call spawn a new process and listen for its output as a message.

In elixir, this is done using spawn or spawn_link

```elixir

pid = spawn MODULENAME, :function, [arg1, arg2 ...]
    
```
Spawn returns the pid of the process we jsut spawned. From within any process, we can send a message to any other process if we know its *pid*.

inside a process, we can listen for messages using *receive.*

```elixir

# value evaluates to msg
value = receive do
	msg ->
    	msg
end

```

So instead of calling the function recursivly on both halves, we spawn a new process running the mergesort recursivly. Then we wait until we get back a message containing the result.

In order for the child process to send back data, it needs to be called with its parent id.

first we take care of its base cases

```elixir

  def psort(parent_id, []) do
    send parent_id, {self, [] }
  end
  def psort(parent_id, [a]) do
    send parent_id, {self, [a] }
  end

```

Notice how instead of returning the value, we are given the parent process id and we use that to send the return value back up. The payload is a tuple containing self and the return value. The self is an elixir value that resolves to the pid of the current process. It's used to help parent processes differentiate its children.

The recursive case becomes easy. split te list, map over it with swan to get an array of pids and map over that to listen for messages from both processes. We then extract the result values and merge them before sending it back to the parent process with the current pid.

```elixir

def psort(parent_id, list) do
    sorted = split(list)
      |> Enum.map(fn(half) ->
          #dispatch mergesort on the half to a 
          #spawned version of itself
          #IO.inspect(half)
          spawn_link(ParallelMergesort, :psort, [self , half])
         end)
      |> Enum.map(fn(pid) ->
          #recieve a message from the pid
          receive do
          	#^pid means only match to the current value
            # of pid in this scope.
            {^pid, sorted_half} ->
              sorted_half
            after 1000 ->
              IO.puts('error')
          end
         end)
      |> (fn([left, right]) -> 
          merge(left, right)
         end).()
     send(parent_id, {self, sorted})
  end

```

now we only have to create the parent function that kicks this whole process off

```elixir

def parallel_sort(list) do
  pid = spawn_link(ParallelMergesort, :psort, [self, list])
  receive do
    {^pid, sorted} ->
      sorted
    after 1000 ->
      {:error}
  end
end

```

And just like that, we have a parallelized mergesort.

The code is up on my [github](https://github.com/cultofmetatron/elixir-parallel-mergesort/blob/master/lib/parallel_mergesort.ex). Thanks for reading!


