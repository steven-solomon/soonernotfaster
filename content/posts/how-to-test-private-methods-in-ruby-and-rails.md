---
title: "How to Test Private Methods in Ruby and Rails"
featured_image: "images/how_to_test_private_methods_in_ruby_and_rails.jpeg"
images: ["images/how_to_test_private_methods_in_ruby_and_rails.jpeg"]
date: 2019-09-22T09:18:08-04:00
draft: true
---

So you have written an implementation in your Rails app, but your team won't accept your Pull Request until there are unit tests. Whats more is that your implementation is primarily changes to the private or protected methods of a class. How do you test this code? We explore your options below.

TK image of private and protected objects

# The problem

Imagine you are working on a finance application. This code includes the Account class below. Notice that the `Account` is currently summing all transactions as part of calculating balance. In the `Account` class there is a private method `:sum` that totals the transactions. 

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

Your story is to filter out pending transactions from the current balance. So you change the private method to filter out the transactions that are pending.

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

But how are you going to test this change?

## Option #1: Test via the public interface 

The public functions of a class ultimately invoke the private ones. So a change in those private methods, should either result in a change of state, or a change in behavior of the class. One or both of these should be observable via the public methods of the class. 

So the most straightforward way to test the private behavior is to call the public method with data that will work on the private method in the way you want.

Here we do this with the public by passing in only pending transactions to the `Account`.

```ruby
it 'ignores pending transactions' do
  account = Account.new
  account.add(Transaction.new(10, "Transaction 1", true))
  account.add(Transaction.new(5, "Transaction 2", true))
  account.add(Transaction.new(-5, "Transaction 3", true))

  expect(account.balance).to eq(0)
end
```

## Option #2: Move methods to a different class

"One class' private method, is another's public method" - TK find author

The simplest thing you can do when trying to test a private method is identify the idea it represents and create a new class that it can be a method on. Once on that new class, the method can be made public. This new public method will be easy to test.

The fact that you want to test it directly in someway is a hint, that it belongs on it's own class.

Another story comes through that calculates the statement balance for an `Account`. The `:statement_balance` method looks at the current date and sums the transactions for that month. This code introduces a new condition around filtering. If the `is_statement` flag is `true` then the month values are selected, else the pending transactions are ignored.

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

The first step here is to not misapply the DRY Principle and remove the conditions on from the code.

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

  # sums all non_pending transactions
  def sum(all_transactions)
    all_transactions
      .reject {|t| t.pending}
      .map {|t| t.amount}
      .reduce(0) {|acc, amount| acc + amount}
  end

  # sums statements for the current month
  def sum_statement(all_transactions)
    all_transactions
      .select {|t| t.date.month == Date.today.month && t.date.year == Date.today.year}
      .map {|t| t.amount}
      .reduce(0) {|acc, amount| acc + amount}
  end
end
```

Once the code is separated out logically, if it is still too difficult to test through the public interface. Then it makes sense to move this code to another class, and parameterize whatever you need to make it testable. In the code below, we parameterize the date.

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

With the `:sum` now a public method on `Statement`, with the hard to control data as a parameter. It should be easier to test.

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

With both as an option, you can test your private methods in one of two ways:

1) Test via the public methods of the class
2) Move the method to another class and make it public

There are other cunning ways to try to invoke private methods, however I view them as bad practice so you will have to look elsewhere to find ways to ignore the design feedback you are recieving ;)

**If you enjoyed this post, please share it on your social media. Thanks!**

*Photo by Micah Williams on Unsplash*