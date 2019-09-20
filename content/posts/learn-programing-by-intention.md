---
title: "Learn Programing by Intention: A Long Forgotten Programming Technique"
featured_image: "images/learn_programing_by_intention.jpeg"
images: ["images/learn_programing_by_intention.jpeg"]
date: 2018-05-20T21:02:35-04:00
---

I want to tell you about an alternate design technique, it is called programming by intention. Rather than writing code that describes how to perform some action, you call a function. The interesting thing is the function doesn’t exist. Yet, we are going to call it as if someone wrote it ten minutes ago. We are going to follow a loop of call a function that doesn’t yet exist, implement what it does by calling more functions that don’t exist, until we have to implement the real code. Let’s try it out.

# First Steps

First we call new on a transaction, that doesn’t yet exist.

```ruby
transaction = Transaction.new
```

Now we implement it.

```ruby
class Transaction
end
```

Next we call validate on the transaction. We don’t worry about the data the transaction needs, that comes later. We are going to follow the call/implement/call loop all the way to the data we need.

# Adding Validation

```ruby
transaction = Transaction.new
transaction.validate
```

Now, let’s create a validate method.

```
class Transaction
  def validate
  end
end
```

Now that validate exists, we want to call methods that will help us achieve it’s goal. The function should validate the account id and amount. Again, we don’t think about where that data is coming from yet. We want to focus on the high level policy and describing what should happen, not how. So we call the methods we wish existed, validate_amount, and validate_account_id.

```ruby
class Transaction
  def validate 
    validate_amount
    validate_account_id
  end
end
```

Now let’s create them, leaving their implementations blank.

```ruby
class Transaction
  def validate 
    validate_amount
    validate_account_id
  end
  
  # Added
  def validate_amount
  end
  
  def validate_account_id
  end
end
```

How should these methods validate the data? Well, they should raise an exception if the data in an instance variable is nil. Notice the instance variables don’t exist, yet we call them like they are there.

```ruby
class Transaction
  # ...
  def validate_amount
    # Added
    raise Exception, ‘amount is missing’ if amount.nil?
  end
  def validate_account_id
    # Added
    raise Exception, ‘account id is missing’ if account_id.nil?
  end
end
```

Now let’s implement the accessors to those variables, as if they were already in the class. At this point, they are always nil but that is okay. For good measure, we make them private to preserve encapsulation.

```ruby
class Transaction
  # ...
  private
  attr_reader :amount, account_id
end
```

The time has finally come to get the data, let’s assign them like they are passed into the constructor.

```ruby
class Transaction
  def initialize(account_id, amount)
    @account_id = account_id
    @amount = amount
  end
  # ...
end
```

# Building What We Need

Next, we must change the call site of the constructor, that we used earlier. We are going to throw away the code that explicitly called new and create a function instead. This function will be named make_transaction, it takes one parameter, the params data.

```ruby
t̶r̶a̶n̶s̶a̶c̶t̶i̶o̶n̶ ̶=̶ ̶T̶r̶a̶n̶s̶a̶c̶t̶i̶o̶n̶.̶n̶e̶w̶
transaction = make_transaction(params)
transaction.validate
```

We create the make_transaction function.

```ruby
def make_transaction(params)
end
```

Lastly, we have to define how it should make the transaction. It should pull the data from the params Hash and return a new transaction.

```ruby
def make_transaction(params)
  Transaction.new(params[:account_id], params[:amount])  
end
```

We now have a working bit of code that validates and creates a transaction. This loop of call/implement/call allows us to think at the same level of abstraction in each function we write. Only when we can’t delegate anymore do we have to specify a real detail of some behavior.

# Conclusion

If we follow this declare first, implement later strategy, we can end up with some really clean code. It will be easy to read exactly what is happening. Heck, even our PM could read it and we don’t need tons of comments either. Here is a sample of code that can come from this technique.

```ruby
transaction = make_transaction(params)
transaction.validate()
account = find_account(transaction.account_id)
account.add_transaction(transaction)
account.print_balance
```

**If you enjoyed this post, please share it on your social media. Thanks!**

*Photo by Jeremiah Berman on Unsplash*