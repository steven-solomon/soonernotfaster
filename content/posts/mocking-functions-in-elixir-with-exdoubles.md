---
title: "Mocking Functions in Elixir With ExDoubles"
date: 2019-12-29T09:39:38-05:00
featured_image: "images/mocking_functions_in_elixir_with_exdoubles.jpeg"
images: ["images/mocking_functions_in_elixir_with_exdoubles.jpeg"]
tags: ['elixir', 'coding', 'testing']
---
So I would have considered this crazy a few years ago, but I wrote my own mock framework in Elixir called [ExDoubles](https://hex.pm/packages/exdoubles). 

I know, I know, there are tons of them already, but hear me out. This one seeks to act like mock libraries in other languages, and put focus on the key element of abstraction in Elixir, the function.

## Why? Why? Why?

I have been working in Elixir for the last six months, after years of Ruby and C#. I am starting to enjoy myself with one exception, I don't care for the existing mocking frameworks. The libraries that are already written either explicitly disallow testing error scenarios in favor of contract consistency, or they look like Erlang.

Contract consistency between your mock and the real implementation is very important. It allows you to have feedback when your test code stops acting like the real code. However, it comes at a cost, testing edge cases is not as simple.  

Let's face it, while Erlang is a very robust and powerful language, it doesn't share syntax with c-style languages. Syntax is one of the strengths of Elixir over Erlang. I prefer when libraries for Elixir, look like Elixir.

With the why out of the way, let me tell you how you can mock and stub functions with ExDoubles.

## Creating mocks

Here is an example that calculates and saves a user's paycheck. We want to mock the save behavior in this test.

```elixir
test "employee with no hours receives zero pay" do
  {:ok, mock_save_fn} = mock(:save_fn_label, 2)

  calculate_employee_pay([], "some_employee_id", mock_save_fn)

  assert verify(:save_fn_label, called_with(["some_employee_id", 0]))
end
```

In order to create a mock, you call the mock function with a `label`, and an `arity`—how many arguments the function takes. This `label` is used to verify expected values, or stub it's return value (more on that shortly).

```elixir
{:ok, mock_save_fn} = mock(:save_fn_label, 2)
```
  
With our mock function created, we can now pass it to the function we are testing. 

```elixir
calculate_employee_pay([], "some_employee_id", mock_save_fn)
```

In order to make sure `calculate_employee_pay/3` calls the mock as expected, we can use ExDoubles' `verify` function with a matcher. 

```elixir
assert verify(:save_fn_label, called_with(["some_employee_id", 0]))
```

We verify that the function associated with our `label` is called with `"some_employee_id"` and `0`. The `called_with` matcher takes an array of the arguments.

## Stubs for profit

Mocking isn't the only thing ExDoubles can do, it can also create stubs. The `when_called` function allows you to specify the return values of a mocked function.

Here we show that the `:stub_value` is returned when the mock is called:

```elixir
test "returns stubbed value from a mock" do
  {:ok, mock_fn} = mock(:mock_label, 0)

  when_called(:mock_label, :stub_value)

  assert :stub_value == mock_fn.()
end
```

If `when_called` is invoked multiple times, each value is returned in the order it was passed.

```elixir
test "returns stubbed values in the order they were passed to `when_called`" do
  {:ok, mock_fn} = mock(:mock_label, 0)

  when_called(:mock_label, :stub_value_1)
  when_called(:mock_label, :stub_value_2)
  when_called(:mock_label, :stub_value_3)

  assert :stub_value_1 == mock_fn.()
  assert :stub_value_2 == mock_fn.()
  assert :stub_value_3 == mock_fn.()
end
```

## What about the trade offs?

Like anything this library has trade offs. You gain the ability to mock functions, but you have to change how you design your system. 

Since ExDoubles only gives the ability to mock functions, you can no longer structure your code so that it can directly calls modules you wish to mock. 

Here is an example with the unmockable functions of `CartRepository.fetch_items/1` and `TaxRepository.fetch_tax/1`:

```elixir
def execute(%User{id: id, zip_code: zip_code}) do
    with {:ok, sundries} <- CartRepository.fetch_items(id),
         {:ok, tax} <- TaxRepository.fetch_tax(zip_code) do
    
      {:ok, total_cost(sundries, tax)}
    else
      error -> error
    end
end
```

Notice that the above code directly references the modules where functions are defined.

If you wish to mock these functions you will have to restructure your code to pass the functions as arguments. Here is a more mock friendly example:

```elixir
def execute(%User{id: id, zip_code: zip_code}, fetch_items, fetch_tax) do
    with {:ok, sundries} <- fetch_items.(id),
         {:ok, tax} <- fetch_tax.(zip_code) do
    
      {:ok, total_cost(sundries, tax)}
    else
      error -> error
    end
end
```

It is now possible to mock/stub `fetch_items/1` and `fetch_tax/1`. This allows us to test both the success scenarios:

```elixir
test "returns total with tax when cart_repository returns populated list" do
    {:ok, mock_cart_repository} = mock(:cart_repository, 1)
    {:ok, mock_tax_repository} = mock(:tax_repository, 1)

    when_called(:cart_repository, {:ok, [%Sundry{id: 1, cost: D.new("1.00"), name: "gloves"}]})
    when_called(:tax_repository, {:ok, D.new("0.8")})

    assert {:ok, D.new("1.800")} == CartCalculator.execute(@user, mock_cart_repository, mock_tax_repository)
end
```

As well as the error scenarios:

```elixir
test "returns error when cart_repository returns an error" do
    {:ok, mock_cart_repository} = mock(:cart_repository, 1)
    
    when_called(:cart_repository, {:error, :some_error_message})
    
    assert {:error, :some_error_message} == CartCalculator.execute(@user, mock_cart_repository, nil)
    
    verify(:cart_repository, called_with([@user.id]))
end

test "returns error when tax_repository returns error" do
    {:ok, mock_cart_repository} = mock(:cart_repository, 1)
    {:ok, mock_tax_repository} = mock(:tax_repository, 1)
    
    when_called(:cart_repository, {:ok, [%Sundry{id: 1, cost: D.new("0.00"), name: "gloves"}]})
    
    when_called(:tax_repository, {:error, :tax_not_found})
    
    assert {:error, :tax_not_found} == CartCalculator.execute(@user, mock_cart_repository, mock_tax_repository)
    
    verify(:tax_repository, called_with([@user.zip_code]))
end
```

## Come on, give it a try

I have been using [ExDoubles](https://hex.pm/packages/exdoubles) day to day on my current project, but that is just one data point. If you are using Elixir, I would love to hear your thoughts. If you aren't using Elixir, maybe this is a reason for you to give it a try.

{{< thank-you >}}

*Photo by Dominik Scythe on Unsplash*