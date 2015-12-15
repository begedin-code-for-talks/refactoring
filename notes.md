# 1. Extract temp to query

If there's a temp var, it often makes sense to extract it into a method.

_Methods longer than a line are a code smell_

When we see code inline, we read it. When it's in its own method, we see just the method name and blackbox the behavior automatically. We now know more about the code flow and it's easier to understand it.

# 2. Tell, don't ask

We're asking `Order` about itself from within `OrdersReport`. It's better to tell the object what to do than ask it about itself.

`Order#placed_between? (start_date, end_date)`

# 3. Data clump

We have a set of arguments that we always pass along together. It means those arguments should've been an object - `DateRange`

Most OO programmers are too reluctant to extract classes.

Our class doesn't even have behavior, but it's still an improvement, because we have a name for something.

Also, it reduced coupling.


# A. Parameter coupling example

If we pass `nil` into `print_to_console`, it will blow up. `to_sentence` is not defined.

# 4. Move date logic into DateRange

Define `include?` method.

Code smell here is called "feature envy". One class concerns itself too much with the internals of another.

# B. Make the code read as you would say it.

"Total sales within date range are the total sales of the orders withing range"
- define a total_sales method with takes an array of orders and adds them up

# 5. null_object.rb - The null object pattern

Rather than have nil stand in place, create a `NullObject` to stand in place for it (a class with same properties and default values). In our case, `NullContact`

Now, we can delete code!

Also, we now have "tell, don't ask"

## There's a downside

We need to keep the apis in sync. New method on `Contact` means new method on `NullContact`. It's a cost, but it's usually worth it.

How many times do we have `if current_user # bla bla bla`?

# 6 Depend upon abstractions

Most programmers are aware of this idea. We use linq or something instead of writing SQL. We don't plug bytes into a socket, we use an http library.

We want the whole to be simpler than the sum of its parts.

If we have a bunch of classes that work together, wrap them up into an API of some sort.

We have our SomeGem, but we could abstract it further.

We make a Gateway class.

Now, when we want to change our payment system, one file needs to change.

It helps with tests to. We had to stub the gem's methods to write tests. Those are 3rd party methods. What if the gem updates and those methods go away? Our tests stub them, so we have no idea this is happening.

Now, we spec our PaymentGateway directly, and then stub its methods in other specs.

# What to refactor

1. God objects
- something that everything seems to interact with
- current user in a large app
- they likely violate the rule of single responsibility
- first rule of classes - they should be really small
- second rule of classes - they should be smaller than that
2. High-churn files - something that changes a lot
- if a file is changing all the time, it's because we don't understand it
- churn gem
3. File where bugs often appear
- bugs love company
- if bug occur there, the code is too complicated for us to see it

* Read Clean Code: A Handbook of Agile Software
* Read Refactoring: Improving the Design of Existing Code
* Read Growing Object-Oriented Software Guided by Tests