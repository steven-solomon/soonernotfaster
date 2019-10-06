---
title: "How to Test Private Methods in Ruby and Rails"
featured_image: "images/how_to_test_private_methods_in_ruby_and_rails.jpeg"
images: ["images/how_to_test_private_methods_in_ruby_and_rails.jpeg"]
date: 2019-09-22T09:18:08-04:00
draft: true
---

So you have written an implementation for that new feature in your Rails app, but your team won't accept your Pull Request until there are unit tests. What's more, your changes are to private methods of a class. In frustration you shout, "How do I test this code!?"

Testing private methods may seem difficult when not using TDD, but this article gives you two strategies that will make your life easier, so that you can get your feature merged. 

# Test via the public interface 

Imagine you are working on a finance application, and your Product Manager assigns the following story to your iteration:

```gherkin
As an account holder
I want to NOT see pending transactions applied to my balance
So that I know the current amount of money I have in the account
```

There currently is an `Account` class that allows client code to add, and sum transactions. Each `Transaction` has an amount, description, and pending flag. The pending transaction feature requires changes to the private `sum` method, where all the account balance is calculated.

```ruby
class Account
  def initialize
    @transactions = []
  end

  def add(transaction)
    @transactions << transaction
  end

  def balance
    sum(@transactions)
  end

  private

  def sum(transactions)
    transactions
    .map { |t| t.amount }
    .reduce(0) { |acc, amount| acc + amount}
  end
end

class Transaction
  def initialize(amount, description, pending)
    @amount = amount
    @description = description
    @pending = pending
  end

  attr_reader :amount, :description, :pending
end
```

A modification to the private method, filters out the transactions that have the `pending` flag set to a `truthy` value.

```ruby
# Account 
# ...
def sum(transactions)
  transactions
  .reject { |t| t.pending } # filter out pending
  .map { |t| t.amount }
  .reduce(0) { |acc, amount| acc + amount}
end
# ...
```

Now that the feature work is done, how do you test this change?

## Testing strategy

Since public functions ultimately invoke the private ones. We would expect that a change in those private methods, should either result in a change of state, or a change in behavior of the class. These difference will be observable via the public methods of the class. 

In order to test that behavior we need to construct a set of test data that will exercise the private method in the way we want.

Here we do this by passing in only pending transactions to the `Account`, then we check that the balance is `zero`.

```ruby
it 'ignores pending transactions' do
  account = Account.new
  account.add(Transaction.new(10, "Transaction 1", true))
  account.add(Transaction.new(5, "Transaction 2", true))
  account.add(Transaction.new(-5, "Transaction 3", true))

  expect(account.balance).to eq(0)
end
```

# Move methods to a different class

Another story comes into our backlog. This one asks us to calculate the statement balance for an `Account`. 

```gherkin
As an account holder
I want to see the transactions for this statement
So that I know how much I have spent
```

This change also affects our `Account` class. We have added the `statement_balance` method, which looks at the current date and sums the transactions for the current month. This code introduces a new condition around filtering. If the `is_statement` flag is `true` then the month values are selected, else the pending transactions are ignored.

```ruby
class Account
  # ...
  def balance
    sum(@transactions)
  end

  def statement_balance
    sum(@transactions, true)
  end

  private

  def sum(all_transactions, is_statement = false)
    transactions_to_total = is_statement ?
                              all_transactions.select do |t|
                                t.date.month == Date.today.month && t.date.year == Date.today.year
                              end
                              : all_transactions.reject {|t| t.pending}

    transactions_to_total
      .map {|t| t.amount}
      .reduce(0) {|acc, amount| acc + amount}
  end
end
```

## Testing strategy

In order to solve this testing issue, let's look at testing this function by moving it to another class.

The first thing you should notice, is that the user story mentions the idea of a `Statement`. This is a missing domain concept in our system.

Creating a class for the statement concept would provide a more logical place to hang code related to it. With the new class created, we can move the `sum` logic to it. The method will be public, allowing us to test it more easily.

The first step in the move method refactoring, is to not [misapply the DRY Principle](/posts/dont-repeat-yourself-is-misunderstood/) and separate the two unrelated filtering operations from the code. Notice that `sum` and `sum_statement` are separate, and neither contain conditions.

```ruby
class Account
  # ...
  def balance
    sum(@transactions)
  end

  def statement_balance
    sum_statement(@transactions)
  end

  private

  # Removed code
  # def sum(all_transactions, is_statement = false)
  #   transactions_to_total = is_statement ?
  #                             all_transactions.select do |t|
  #                               t.date.month == Date.today.month && t.date.year == Date.today.year
  #                             end
  #                             : all_transactions.reject {|t| t.pending}

  #   transactions_to_total
  #     .map {|t| t.amount}
  #     .reduce(0) {|acc, amount| acc + amount}
  # end

  # Added. Sums all non_pending transactions
  def sum(all_transactions)
    all_transactions
      .reject {|t| t.pending}
      .map {|t| t.amount}
      .reduce(0) {|acc, amount| acc + amount}
  end

  # Added. Sums statements for the current month
  def sum_statement(all_transactions)
    all_transactions
      .select {|t| t.date.month == Date.today.month && t.date.year == Date.today.year}
      .map {|t| t.amount}
      .reduce(0) {|acc, amount| acc + amount}
  end
end
```

Next let's move the code to the `Statement` class, and parameterize the date. The parameter will make it more testable.

```ruby
# Account class
def statement_balance
  Statement.sum(@transactions, Date.today)
end
```

```ruby
# Statement class
class Statement
  def self.sum(all_transactions, today)
    all_transactions
      .select {|t| t.date.month == today.month && t.date.year == today.year}
      .map {|t| t.amount}
      .reduce(0) {|acc, amount| acc + amount}
  end
end
```

Let's write a test around `Statement#sum` that passes in a variety of dates and verifies the sum only contains the transactions for the parameterized month.

```ruby
it 'Statement#sum' do
  transactions = [
    Transaction.new(10, nil, false, last_year),
    Transaction.new(10, nil, false, last_month),
    Transaction.new(-5, nil, false, Date.today),
    Transaction.new(13, nil, false, Date.today),
  ]

  expect(Statement.sum(transactions, Date.today)).to eq(8)
end
```

# Conclusion

Unit Testing can be a frustrating endeavor as it gives you feedback on the design of your code. Your choice can be to respond to or ignore that feedback.

You can act on the feedback of your code being difficult to test in the following ways:

1) Test via the public methods of the class
2) Move the method to another class and make it public

It may be tempting to try and find ways to invoke your private class using the reflection APIs of the language, but this will make your tests brittle, and hard to maintain. Whenever possible, it is better to simplify the design of the code, rather than come up with cunning solutions.

**If you enjoyed this post, please share it on your social media. Thanks!**

*Photo by Micah Williams on Unsplash*