
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