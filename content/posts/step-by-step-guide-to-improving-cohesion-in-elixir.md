---
title: "Step by Step Guide to Improving Cohesion in Elixir"
featured_image: "images/step_by_step_guide_to_improving_cohesion_in_elixir.jpeg"
images: ["images/step_by_step_guide_to_improving_cohesion_in_elixir.jpeg"]
date: 2020-05-25T11:04:29-04:00
draft: false
---

It is the kind of code that makes you squint. It could be that stray business logic in a controller, a view that queries the database, or even two intertwined features that change for different reasons. I am talking about code that is hard to read.

Cryptic code slows everything down. It is the thing that makes a seemingly simple feature take weeks. Though we all can recognize unreadable code, it seems to keep showing up in our code-bases. This has to do with a lack of *Cohesion*. Read on to explore the concept of *Cohesion*, and how to use it to start taking control of your code.

# What is Cohesion?

Hard to read code often mixes unrelated concepts together in one place. This style often requires the reader to juggle multiple ideas in their mind. It puts the responsibility on them to figure out what is relevant, and what is not. And that is not fun.

**Cohesion is a property of software that doesn't make the reader sort through a puzzle. When applied, this concept leads us to place code that is related together in one location, and any unrelated code at a distance.**

For a more concrete example, please take a look at the module below. Ask yourself, “Is the `is_admin/1` function related to the `determine_score/3` function?”

```elixir
defmodule User do
  def is_admin(user_id)
    #...
  end
  
  def determine_score(game, user_id, date) do
     # ...
  end
end
```

If the answer doesn't spring forth, ask yourself, "Can I think about these parts separately? Why?" 

We can come to the answer of yes, when we notice that identifying a user as an admin, has no effect on what their score is in the game. 

Here are a few other questions to ask so you can determine if code is not Cohesive:

1. Do the functions change for different reasons?
1. Do the functions deal with different types of unrelated behavior?
1. Are there lots of nested conditionals?

## Context Matters

The questions above can provide valuable hints that the code may lack Cohesion. However, Cohesion exists on a continuum. The ideal amount of Cohesion depends on the size and complexity of your code. The larger and more complex the parts become, the more you will need to separate them. 

We can solidify this idea by considering how two unrelated functions interact at different scales.

First, imagine two tiny unrelated functions nestled together in the same module—like the example above. This doesn’t seem like a big deal. Both pieces are small, and we can very quickly determine that they are unrelated.

Next, consider two 65+ line unrelated functions in the same module—along with their associated helpers functions. All of those moving parts provide the opportunity to create confusion. Trying to determine which parts are related, is enough to make anyone's head spin. When the reader can not decide what to focus on, then they will have to re-read code several times.

*As a general rule of thumb, it is a good idea to separate unrelated code as its size and complexity increases*. However, sometimes the damage is done, and unrelated code is already in one place. In the next section we explore how to refactor it apart.

# Separating Unrelated Code

So you have identified some code that should be separate! How do you move it apart? Not to worry, *the Extract Module* refactoring technique is here to save the day.

The Extract Module refactoring technique moves functions from one module into a newly created one. This allows us to increase the Cohesion of our code, and therefore increase its readability. This refactoring pattern is based off of [Martin Fowler's Extract Class](https://refactoring.guru/extract-class) and [Move Method](https://refactoring.guru/move-method).

Below you will find the steps to the Extract Module refactoring. Like all refactorings it is very important to verify the code still works after every step. The best way to check is by running the unit tests—though I will omit this important phase in the rest of this article.

Here are the steps for the Extract Module refactoring:

1. Identify the business concept to extract
1. Rename the existing module if it conflicts with the name of the business concept
1. Create a module with the name of the business concept
1. Move data to the newly created module
1. Replace all uses of the local data with the module
1. Use the Extract Function Refactoring on business logic
1. Use the Move Function Refactoring to move a function that references data to the new module
1. Repeat for all functions
1. Repeat for all data

# Applying the Extract Module Refactoring

Below you will find the example code we will use to perform the Extract Module refactoring. This code contains business logic in a structure where it doesn't belong—a GenServer. For the purpose of this article, you can think about GenServer as if it is a controller in Rails, or a middleware function in Express. In both situations, the business logic and routing logic will change for different reasons. 

If you want to better understand GenServers, here is an article that shows you what they are and how to test them: [What is a GenServer in Elixir, and How Do I Write One?](https://www.stridenyc.com/blog/what-is-a-genserver-in-elixir) Yet, all you need to know for maximizing Cohesion is that business logic and GenServers should be separate.

Thinking of GenServer as a controller, let's take a look at the code below. Notice the code implements logic for a BookStore, as well as GenServer wiring code. 

The BookStore is able to give clients all its books, add new books, query by book name, and query by id.

```elixir
defmodule BookStore do
  use GenServer

  def start_link() do
    GenServer.start_link(__MODULE__, :ok)
  end

  def all_books(pid) do
    GenServer.call(pid, :all_books)
  end

  def add(pid, book) do
    GenServer.cast(pid, {:add, book})
  end

  def query(pid, term) do
    GenServer.call(pid, {:query, term})
  end

  def find(pid, id) do
    GenServer.call(pid, {:find, id})
  end

  def purchase(pid, id) do
    GenServer.cast(pid, {:purchase, id})
  end

  def init(:ok) do
    {:ok, []}
  end

  def handle_call(:all_books, _from, books) do
    sorted_books =
      books
      |> Enum.sort_by(fn %Book{purchases: purchases} -> purchases end, &>=/2)

    {:reply, sorted_books, books}
  end

  def handle_call({:find, id_to_find}, _from, books) do
    result =
      books
      |> Enum.find(:book_not_found, fn %Book{id: id} -> id == id_to_find end)

    {:reply, result, books}
  end

  def handle_call({:query, term}, _from, books) do
    downcased_query = String.downcase(term)

    matches =
      books
      |> Enum.filter(
           fn %Book{name: name} ->
             name
             |> String.downcase()
             |> String.contains?(downcased_query)
           end
         )
      |> Enum.sort_by(fn %Book{purchases: purchases} -> purchases end, &>=/2)

    {:reply, matches, books}
  end

  def handle_cast({:add, book}, books) do
    book_with_id = %Book{book | id: UUID.uuid1}
    {:noreply, [book_with_id | books]}
  end

  def handle_cast({:purchase, id_to_purchase}, books) do
    updated_books =
      books
      |> Enum.map(fn %Book{id: id, purchases: purchases} = book ->
        case id == id_to_purchase do
          true ->
            %Book{book | purchases: purchases + 1 }
          false -> book
        end
      end)

    {:noreply, updated_books}
  end
end
```

The first step of this refactoring process is to *Identify the business concept to extract*. As mentioned above, the concept we want to extract is a `BookStore`.

With our goal of extracting a `BookStore`, it is time to *Rename the existing module*. Rename the `BookStore` module—which is a GenServer—to be called `BookStoreGenServer`. This ensures the names will not conflict.

 ```diff
- defmodule BookStore do
+ defmodule BookStoreGenServer do
  # ...
end
 ```

Now you can *Create a module with the name of the business concept*. This new module will ultimately hold all of the BookStore logic. Create a new module named `BookStore`.

```
defmodule BookStore do
end
```

Since we have a module to represent the business concept, it is time to *Move data to the newly created module*. In order to convert the `BookStore` into a data structure add a `defstruct` statement containing the Keyword List with one key, `:books`, with a value of an empty List.

```
defmodule BookStore do
  defstruct [books: []]
end
```

It is time to go through and *Replace all uses of the local data with the module*. This entails changing the initialization of the state to use the `BookStore` struct, as well as changing any callback functions to destructure out the relevant attributes.

```diff
def init(:ok) do
-   {:ok, []}
+   {:ok, %BookStore{}}
end
```

With the state of the GenServer now holding a BookStore struct, the way the state is used in all `handle_*` functions need to change. Below is an example of how to make this change for a `handle_call` functions.

Notice we use pattern matching to pull out the relevant data in the arguments. It is also important to make sure to return the entire BookStore in the reply tuple on the last line of the callback.

```diff
-  def handle_call({:find, id_to_find}, _from, books) do
+  def handle_call({:find, id_to_find}, _from, %BookStore{books: books} = book_store) do
    result =
      books
      |> Enum.find(:book_not_found, fn %Book{id: id} -> id == id_to_find end)

-    {:reply, result, books}
+    {:reply, result, book_store}
  end
```

Next, you will find an example of how to change it in `handle_cast` functions. 

In this type of callback function, we want to pattern match the books out of the BookStore. Bear in mind, we will *not* need a variable for the store since we will create a new one each time.

```diff
-  def handle_cast({:purchase, id_to_purchase}, books) do
+  def handle_cast({:purchase, id_to_purchase}, %BookStore{books: books}) do
    updated_books =
      books
      |> Enum.map(fn %Book{id: id, purchases: purchases} = book ->
        case id == id_to_purchase do
          true ->
            %Book{book | purchases: purchases + 1 }
          false -> book
        end
      end)
-    {:noreply, updated_books}
+    {:noreply, %BookStore{books: updated_books}}
  end
```

Continue converting all callback functions to destructure the books attribute out of the `BookStore` struct. When completed, you will be ready to move business operations into the `BookStore` module. 

This process's first step is to *Use the [Extract Function](https://refactoring.com/catalog/extractFunction.html) Refactoring on all business logic*. Once all function are extracted, we can Use *[Move Function](https://refactoring.com/catalog/moveFunction.html) refactoring to move a function that references data to the new module*. 

As a reference, you can find a detailed explanation of these refactorings in [Fowler's book](https://www.amazon.com/Refactoring-Improving-Existing-Addison-Wesley-Signature-ebook/dp/B07LCM8RG2/ref=sr_1_1?dchild=1&keywords=refactoring&qid=1589929247&sr=8-1). It is necessary to follow the steps that Fowler defines in his book due to the lack of automated refactoring support in Elixir—I hope this will change soon.

Now that you have learned the two steps (Extract Function, and Move Function), the process can be repeated for all operations you wish to extract from the GenServer. 

Below is an example of extracting, then moving the `all_books/1` function.

```diff
defmodule BookStore do
  defstruct [books: []]

+  def all_books(%BookStore{books: books}) do
+    books
+    |> Enum.sort_by(fn %Book{purchases: purchases} -> purchases end, &>=/2)
+  end
end

defmodule BookStoreGenServer do
  # ...
  def handle_call(:all_books, _from, %BookStore{books: books} = book_store) do
-    sorted_books =
-      books
-      |> Enum.sort_by(fn %Book{purchases: purchases} -> purchases end, &>=/2)
-
-    {:reply, sorted_books, book_store}
+    {:reply, BookStore.all_books(book_store), book_store}
  end
  # ...
end
```

Here is another example, this time for the `find_by_id/2`. 

After moving this function, it is also possible to rename the `id_to_find` to `id` in the handle_call callback.

```diff
defmodule BookStore do
  # ...

+  def find_by_id(%BookStore{books: books}, id_to_find) do
+    books
+    |> Enum.find(:book_not_found, fn %Book{id: id} -> id == id_to_find end)
+  end
end

defmodule BookStoreGenServer do
  # ...
+ def handle_call({:find, id}, _from, %BookStore{} = book_store) do
- def handle_call({:find, id_to_find}, _from, %BookStore{books: books} = book_store) do
-    result =
-      books
-      |> Enum.find(:book_not_found, fn %Book{id: id} -> id == id_to_find end)
-
+    {:reply, BookStore.find_by_id(book_store, id), book_store}
-    {:reply, result, book_store}
  end
  # ...
end

```

Continue to follow these steps until all data and functions are moved from the GenServer to the `BookStore` module. 

# Completed Extract Module Refactoring

```elixir
defmodule BookStore do
  defstruct [books: []]

  def all_books(%BookStore{books: books}) do
    books
    |> Enum.sort_by(fn %Book{purchases: purchases} -> purchases end, &>=/2)
  end

  def add(%BookStore{books: books}, book) do
    book_with_id = %Book{book | id: UUID.uuid1}
    %BookStore{books: [book_with_id | books]}
  end

  def find_by_id(%BookStore{books: books}, id_to_find) do
    books
    |> Enum.find(:book_not_found, fn %Book{id: id} -> id == id_to_find end)
  end

  def purchase_book(%BookStore{books: books}, id_to_purchase) do
    updated_books =
      books
      |> Enum.map(
           fn %Book{id: id, purchases: purchases} = book ->
             case id == id_to_purchase do
               true ->
                 %Book{book | purchases: purchases + 1}
               false -> book
             end
           end
         )

    %BookStore{books: updated_books}
  end

  def find_by_name(%BookStore{books: books}, term) do
    downcased_query = String.downcase(term)

    books
    |> Enum.filter(
         fn %Book{name: name} ->
           name
           |> String.downcase()
           |> String.contains?(downcased_query)
         end
       )
    |> Enum.sort_by(fn %Book{purchases: purchases} -> purchases end, &>=/2)
  end
end

defmodule BookStoreGenServer do
  use GenServer

  def start_link() do
    GenServer.start_link(__MODULE__, :ok)
  end

  def all_books(pid) do
    GenServer.call(pid, :all_books)
  end

  def add(pid, book) do
    GenServer.cast(pid, {:add, book})
  end

  def query(pid, term) do
    GenServer.call(pid, {:query, term})
  end

  def find(pid, id) do
    GenServer.call(pid, {:find, id})
  end

  def purchase(pid, id) do
    GenServer.cast(pid, {:purchase, id})
  end

  def init(:ok) do
    {:ok, %BookStore{}}
  end

  def handle_call(:all_books, _from, %BookStore{} = book_store) do
    {:reply, BookStore.all_books(book_store), book_store}
  end

  def handle_call({:find, id}, _from, %BookStore{} = book_store) do
    {:reply, BookStore.find_by_id(book_store, id), book_store}
  end

  def handle_call({:query, term}, _from, %BookStore{} = book_store) do
    {:reply, BookStore.find_by_name(book_store, term), book_store}
  end

  def handle_cast({:add, book}, %BookStore{} = book_store) do
    {:noreply, BookStore.add(book_store, book)}
  end

  def handle_cast({:purchase, id}, %BookStore{} = book_store) do
    {:noreply, BookStore.purchase_book(book_store, id)}
  end
end
```

# Improving Your Code

Our code now separates business logic from the delivery mechanism—the GenServer. You are now able to increase the Cohesion of your code by using the *Extract Module* refactoring. This is a critical step in the journey to writing code that is easy to read, and maintain.


**If you enjoyed this post, please share it on your social media. Thank you and happy coding!**


*Photo by 贝莉儿 DANIST on Unsplash*
