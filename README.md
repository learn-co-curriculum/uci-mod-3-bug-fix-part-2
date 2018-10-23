# Title

## Learning Goals

- Recognize correctness of code as a programming goal
- Recognize readability of code is a programming goal

## Introduction

Correct code can be _bad_.  Incorrect code can be _well-written_

As a programmer, we have to try to create code that is both _correct_ **and**
well-written.

### Broken and Hard to Read

Code that's hard to read **and** incorrect is the worst.

```ruby
class TotalCostCalculator
  attr_reader :description, :cost

  def initialize(description="A sun-drenched paradise", cost=325000, rate=0.02, mortgage_years)
    @description = description
    @cost = cost
    @rate = rate
    @term = 12 * mortgage_years
  end

  def total_loan_cost
    @tlc ||= ((@rate * cost) / 1 - (1 + @rate) ** ( -1 * @term) ) * @term
  end
end

calc = TotalCostCalculator.new("A paradise in Barbados", 250000, 6.5, 5)
puts calc.total_loan_cost
```

Most people would not regard this code as fun to debug. You might be a genius
who sees the bug instantly, but a lot of people find `total_loan_cost`'s logic
difficult to reason about. To fix the bug, the implementation of
`total_loan_cost` should look like:

```ruby
    @tlc ||= ((@rate / 100 / 12 * @cost) / (1 - ( 1 + (@rate  / 100 / 12) )**( @term  * -1)) ) * @term
```

The `@rate` needs to be divided by 100 (to get a decimal) and then again by the
number of months in a year.

This code is now no longer _broken_, but it is not _good code_.  It's _correct_
but it's not maintainable or readable. It's hard to talk about and verify the
sub-calculations in the `total_loan_cost` calculation.

We should aim to **not** write code like the following even though it is _correct_:

```ruby
class MortgageCalculator
  attr_reader :description, :cost

  def initialize(description="A sun-drenched paradise", cost=325000, rate=0.02, mortgage_years)
    @description = description
    @cost = cost
    @rate = rate
    @term = 12 * mortgage_years
  end

  def total_loan_cost
    @tlc ||= ((@rate / 100 / 12 * @cost) / (1 - ( 1 + (@rate  / 100 / 12) )**( @term  * -1)) ) * @term
  end
end

x = MortgageCalculator.new("A paradise in Barbados", 250000, 6.5, 5)
p x.total_loan_cost #=> 293492.22328093013
```

## Broken and Easy to Read

Let's explore what things might have been like if we'd been given
more-clearly-written, but _incorrect_ code.

```ruby
class MortgageCalculator
  attr_reader :description, :cost

  def initialize(description="A sun-drenched paradise", cost=325000, rate=0.02, mortgage_years)
    @description = description
    @cost = cost
    @rate = rate
    @mortgage_years = mortgage_years
  end

  def total_loan_cost
    numerator / denominator * mortgage_months
  end

  private

  def numerator
    @rate * @cost
  end

  def denominator
    1 - ( 1 + @rate )**( mortgage_months * -1)
  end

  def mortgage_months
    @mortgage_years * 12
  end
end

x = MortgageCalculator.new("A paradise in Barbados", 250000, 6.5, 5)
p x.total_loan_cost #=> 97500000.0
````

Using a process (which we'll cover in another lesson) we can see that the
inputs to `total_loan_cost` are `numerator`, `denominator` and,
`mortgage_months`.

Let's validate our inputs. We'll start with the first input, `numerator`. The
code says that we're going to multiply the `@cost` of the property by a
`@rate`, a fraction.

Before we print out that value (or use a debugging tool to stop execution),
let's think about what we *expect* the return value to be.

- What do you expect `numerator` to look like?
- Is it a `Float` or an `Integer`? It should be a rate, so it should be a `Float`.
- Is it big? Small? It should be smaller than the `@cost` since a decimal
  multiplied by a whole number should be less than the original.

Let's use `p` to print out `numerator`'s test value and test our hypothesis.

```ruby
class MortgageCalculator
  attr_reader :description, :cost

  def initialize(description="A sun-drenched paradise", cost=325000, rate=0.02, mortgage_years)
    @description = description
    @cost = cost
    @rate = rate
    @mortgage_years = mortgage_years
  end

  def total_loan_cost
    numerator / denominator * mortgage_months
  end

  private

  def numerator
    p @rate * @cost
  end

  def denominator
    1 - ( 1 + @rate )**( mortgage_months * -1)
  end

  def mortgage_months
    @mortgage_years * 12
  end
end

x = MortgageCalculator.new("A paradise in Barbados", 250000, 6.5, 5)
p x.total_loan_cost #=> 97500000.0
````

The output is:

```text
1,625,000.0 <==== (arrow and commas added for readability)
97500000.0
 => 97500000.0
```

Wait a second! This does not match our expectations! We're way off! Our bug
_has_ to be here.  If we consult the formula page, we see that the expectation
is that the rate is expressed as a percentage, converted to decimal, divided by
the 12 months of the year.

Debugging and fixing well-structured, well-written code is clear.  To correct
it is simple and makes simple code even simpler. We'll change the `@rate`
instance variable to be divided by `(100 * 12)` from the very start.  We'll
also change the variable name so that it's even clearer that we're talking
about a rate per month. Oh yeah, we should also remove our debugging `p`
statement.

## Correct and Easy to Read

```ruby
class MortgageCalculator
  attr_reader :description, :cost

  def initialize(description="A sun-drenched paradise", cost=325000, rate=0.02, mortgage_years)
    @description = description
    @cost = cost
    @int_rate_decimal_per_month = rate / 100 / 12
    @mortgage_years = mortgage_years
  end

  def total_loan_cost
    numerator / denominator * mortgage_months
  end

  private

  def numerator
    @int_rate_decimal_per_month * @cost
  end

  def denominator
    1 - ( 1 + @int_rate_decimal_per_month )**( mortgage_months * -1)
  end

  def mortgage_months
    @mortgage_years * 12
  end
end

x = MortgageCalculator.new("A paradise in Barbados", 250000, 6.5, 5)
p x.total_loan_cost #=> 293492.22328093013
````

## The Big Picture

> As students in Mod 1 we should be writing code that is easy to maintain,
> easy to communicate about, and easy reason about **while also** being correct.

Many beginners mistakenly believe that "getting the answer right" is the goal
programming. Getting the right answer with a code that's intelligble to other
humans, including your future self, is the goal.

## Conclusion

The remainder of this module is designed to help you:

- Write maintainable, readable code that is correct
- Debug code with a repeatable process
- Build confidence with core programming data structures (`Arrays` and `Hash`es and `Enumerables`)
- Practice your new approach with challenging labs
