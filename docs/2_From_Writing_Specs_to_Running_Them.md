
## Cusotmizing Your Specs' Output

### The Progress Formatter

Create a new test file.

- spec/coffee_spec.rb
```ruby

class Coffee

  def ingredients()
    @ingredients ||= []
  end

  def add(ingredient)
    ingredients << ingredient
  end

  def price()
    1.00
  end
end

RSpec.describe 'A cup of coffee' do
  let(:coffee) { Coffee.new }

  it 'costs $1' do
    expect(coffee.price).to eq(1.00)
  end

  context 'with milk' do
    before { coffee.add :milk }

    it 'costs $1.25' do
      expect(coffee.price).to eq(1.25)
    end
  end
end
```

`context` is just an alias for `describe`.

Now run the tests.
`rspec`

You'll see one dot for each completed example, with failures and exceptions called out with letters.

A formatter receives events from RSpec and then reports the results. Formatters can write data in any sormat, and send the output anywhere (such as to the console, a file, or over a network).

### The Documentation Formatter

To see the output in documentation format, pass --format documentation (or just -fd) to rspec.

`rspec --format documentation`

Now you'll see an additional log in terminal:
```
A cup of coffee
  costs $1
  with milk
    costs $1.25 (FAILED - 1)

An ideal sandwich
  is delicious
  lets me add toppings

...
```

### Syntax Highlighting

Add a dependency in gemspec.
```ruby
...
  spec.add_dependency "coderay", "~> 1.1.1"
...
```

Stop the container and run again with `docker-compose up --build`.
(or run `bundle install`)

Run the tests.
`rspec -fd`

Note the line `expect(coffee.price).to eq(1.25)` has Ruby syntax highlighting.

## Identifying Slow Examples

Create the test file.

- spec/slow_spec.rb
```ruby
RSpec.describe 'The sleep() method' do
  it('can sleep for 0.1 second') { sleep(0.1) }
  it('can sleep for 0.2 second') { sleep(0.2) }
  it('can sleep for 0.3 second') { sleep(0.3) }
  it('can sleep for 0.4 second') { sleep(0.4) }
  it('can sleep for 0.5 second') { sleep(0.5) }
end
```

Run with --profile option.
`rspec --profile 2`

The output:
```
Top 2 slowest examples (0.90096 seconds, 59.3% of total time):
  The sleep() method can sleep for 0.5 second
    0.50049 seconds ./spec/slow_spec.rb:7
  The sleep() method can sleep for 0.4 second
    0.40048 seconds ./spec/slow_spec.rb:6

Top 2 slowest example groups:
  The sleep() method
    0.30066 seconds average (1.5 seconds / 5 examples) ./spec/slow_spec.rb:2
  A cup of coffee
    0.00529 seconds average (0.01059 seconds / 2 examples) ./spec/coffee_spec.rb:17
```

## Running Just What You Need

You can run all tests in a directory.
example: `rspec spec/unit`

Or a specific test.
example: `rspec spec/unit/specific_spec.rb`.

Load more than one directory.
example: `rspec spec/unit spec/products`

And mix.
example: `rspec spec/unit spec/foo_spec.rb`

### Running Examples by Name

Running example by name using --example or -e option.

`rspec -e milk -fd`

This command will run the examples that contains the work milk (case sensitive)

### Running Specific Failures

You can specify the example by line.
`rspec ./spec/coffee_spec.rb:25`

### Running Everything that Failed

It's needed to specify a file to save the tests to rerun.

Add configuration at the top of the file.

- spec/coffee_spec.rb
```ruby
RSpec.configure do |config|
  config.example_status_persistence_file_path = 'spec/examples.txt'
end

...
```

Now, run the examples.
`rspec`

and after run only the failed examples.
`rspec --only-failures`

You should to see only the failed example running.

Let's fix it updating the price function.

- spec/coffee_spec.rb
```ruby
...
  def price()
    1.00 + ingredients().size * 0.25
  end
...
```

And run `rspec --only-failures`.

We can use the option `--next-failure` too, to walk through the failed examples.

### Focusing Specific Examples

You can mark examples as focused just adding the f to the beggining of the RSpec method name.

- context -> fcontext
- it -> fit
- describe -> fdescribe

Change context to fcontext.

- spec/coffee_spec.rb
```ruby
  
  fcontext 'with milk' do
    ...
  end
```

Now, configure the RSpec to run just the focused examples.

- spec/coffee_spec.rb
```ruby
  
RSpec.configure do |config|
  config.filter_run_when_matching(focus: true)
  ...
end
```

Running `rspec` only he focused examples must be executed.

To finish, remove the f from fcontext.

### Tag Filtering

The line `fcontext 'with milk' do` is a shorthand for `context 'with mild', focus: true do`.

We can do this to `it` and `describe` too.

Use tag this way:
`rspec --tag last_run_status:failed`.

## Marking Work in Progress

### Starting With the Description

Add empty examples inside the `with milk` context.

```ruby
it 'is light in color'
it 'is cooler than 200 degrees Fahrenheit'
```

Run tests.
`rspec spec/coffee_spec.rb`

You should see:
```
..**

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) A cup of coffee with milk is light in color
     # Not yet implemented
     # ./spec/coffee_spec.rb:36

  2) A cup of coffee with milk is cooler than 200 degrees Fahrenheit
     # Not yet implemented
     # ./spec/coffee_spec.rb:37


Finished in 0.00191 seconds (files took 0.08156 seconds to load)
4 examples, 0 failures, 2 pending
```

### Marking Incomplete Work

Mark spec as pending adding `pending` with a description.

- spec/coffee_spec.rb
```ruby
...
    it 'is light in color' do
      pending 'Color not implemented yet'
      expect(coffee.color).to be(:light)
    end

    it 'is cooler than 200 degrees Fahrenheit' do
      pending 'Temperature not implemented yet'
      expect(coffee.temperature).to be < 200.0
    end
...
```

Run the tests.

The result is shown below.
```
..**

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) A cup of coffee with milk is light in color
     # Color not implemented yet
     Failure/Error: expect(coffee.color).to be(:light)
     
     NoMethodError:
       undefined method `color' for an instance of Coffee
     # ./spec/coffee_spec.rb:38:in `block (3 levels) in <top (required)>'

  2) A cup of coffee with milk is cooler than 200 degrees Fahrenheit
     # Temperature not implemented yet
     Failure/Error: expect(coffee.temperature).to be < 200.0
     
     NoMethodError:
       undefined method `temperature' for an instance of Coffee
     # ./spec/coffee_spec.rb:43:in `block (3 levels) in <top (required)>'

Finished in 0.00206 seconds (files took 0.07988 seconds to load)
4 examples, 0 failures, 2 pending
```

The tests won't be marked as failed.

### Completing Work in Progress

Impleent the color and temperature methods inside Coffee class.

```ruby
...
  def color()
    ingredients.include?(:milk) ? :light : :dark
  end

  def temperature()
    ingredients.include?(:milk) ? 190.0 : 205.0
  end
...
```

Run the tests.

The tests will be marked as failed, because the pending still exists. Remove them and run tests again.

The tests will pass.

If you don't want run the test, you can use `skip` instead of `pending`.