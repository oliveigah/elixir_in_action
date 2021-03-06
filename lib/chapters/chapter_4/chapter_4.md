# 4 - Data Abstraction

- Data abstractions are built as pure elixir modules
- These modules works as an abstraction of the data type. (eg. If you want to work with a list data type, you use the List module)
- This kind of modules have 2 types of functions
  - Modifier: Transform the input data, and return the modified version that is same type of the input. eg. `String.upcase/1` and `List.insert_at/3`
  - Query: Analyze the input data and return some piece of information about it in a different data type. eg. `String.length/1` and `List.first/1`
- If `struct` is not used, there is no difference between a map and the high level data structure

## 4.1 - Abstracting With Modules

- Higher level abstractions are built with the basic building blocks
- Although the abstractions are in fact some built in types, the user of the module should not rely on it
- These modules free the user of the concerns like implementation or data structures
- The separation of concerns between modules are kind of similar to OO languages
- By convention this kind of modules have a function called `new` that create and return a new "instance" abstracted data

### 4.1.2 - Composing Abstractions

- Just like in OO languages, a usual pattern in elixir is to separate the concerns in different modules.
- **TODO: Add reference to TodoList project**

### 4.1.3 - Structuring Data With Maps

eg. [this module](Chapter4.Fraction.html#content)

- To avoid changes in functions signatures, it is a usual pattern use a map as a single data structure to all the entries
- Instead of expect something like `module.fun(arg1,arg2,arg3)` you expect `module.fun(%{a: arg1, b: arg2, c: arg3})`
- The entry map must contains the expect keys to be used inside the module's functions

### 4.1.4 - Abstracting With Structs

- eg. [this module](Chapter4.Fraction.html#content)
- Structs can be distinguished from any other data types
- The `defstruct` macro allow the creation of specific data structures that are bound to a specific module
- One module can only have one struct, and one struct can only exists inside a module
- Patter matching works on structs
- By use pattern matching like `fun(%struct{a: a, b: b})` it is enforced the usage of an instance of the struct as the entry data
- Structs are in fact maps, so any `Map` module's function should work on it
- Despite it, `Enum` module's functions do not work, because the struct is an abstraction defined inside the module and should behave as defined inside of it. So you must define inside the module if and how the struct is enumerable or not
- Structs are maps, but always have a special key `__struct__`
- Because of it a struct pattern can not match a plain object. eg `%struct{} = %{a:1, b:2}` raises a match error
- But a plain map pattern can match a struct. eg `%{} = %struct{a: 1, b: 2}` succeeds
- `Records` are an alternative from `structs` that uses tuples instead of maps, many pure erlang modules uses records as interfaces

### 4.1.5 - Data Visibility

- All the data structure is always visible to the module's user. There is no private data structure
- In this sense, encapsulation work different. Instead of private attributes and methods, elixir module as whole is reponsible for manage that kind of data, although you can see it data structure, you should not work on it from out side the module
- This data transparency is useful for debugs purpose
- Despite this, as a module's client you should not rely on the internal data structure representation
- The only guarantee of these abstraction modules is that the functions will work if an instance of a proper data structure is sent as an entry

## 4.2 - Hierarchical Data

- Because of the immutable nature of data the updates on hierarchical data has to be done by update the specific field and then all of the parents recursively
- Usually it is done with several simple functions, wich one responsible for a specific part of the update and the more specific functions delegates to the more general ones
- Instead of doing these several functions, the macro `put_in/2` is avaiable for this purpose
- There are similar macros for retrieving data too. eg. `get_in/2`
- These kind of macros relies on the `Access` module

## 4.3 - Polymorphism With Protocols

- The basic way of doing polymorphism in elixir is with protocols

### 4.3.1 - Protocol Basics

- A protocol is kind of a OO interface
- It is a module where a functions is declared but not implemented

```elixir
defprotocol String.Chars do
  def to_string(thing)
end
```

- Data types that implements the protocol can use all the functions defined by it
- If protocol function is called with some data type that does not implement the protocol, an error is raised

### 4.3.2 - Implementing Protocols

```elixir
## Code of the TodoList project
defimpl String.Chars, for: TodoList do
  def to_string(todo_list) do
    "You have #{todo_list.auto_id - 1} tasks to do"
  end
end
```

- Elixir most basic functions relies on protocol
- Each protocol has it's own set of functions that must be implemented
- `defimpl` macro is used to start the implementation
- In `defimpl` you must specify wich protocol will be implemented and the corresponding data type
- On the do-end block are the specific funtions implementations
- Protocols implementations do not need to be part of any specific module
- Because of it, you can implement protocols even for data types that you don't have access to the source code

### Built-in Protocols

```elixir
## Code of the TodoList project
defimpl Collectable, for: TodoList do
  def into(original) do
    {original, &into_callback/2}
  end

  defp into_callback(todo_list, {:cont, entry}) do
    TodoList.add_entry(todo_list, entry)
  end

  defp into_callback(todo_list, :done) do
    todo_list
  end

  defp into_callback(_, :halt) do
    :ok
  end
end
```

- Elixir have some built-in protocols that are very useful, check docs for details. eg. `Enumerable`,`Collectable`,`Stream`
- By implementing the `Enumerable` protocol, your high level data structure gain access to all the `Enum` functions for free
- Besides built-in protocols you can build your own protocols
