# Create project

- Create project with bundle:
`bundle gem rspec_test`

- Add Dockerfile:

- Dockerfile
```dockerfile
# Use the official Ruby image from Docker Hub
FROM ruby:3.3.1

# Set the working directory inside the container
WORKDIR /app

# Copy the Gemfile and Gemfile.lock into the container
# COPY Gemfile* *.gemspec lib ./
COPY . .

# Install dependencies using Bundler
RUN bundle install

# Copy the rest of the application code into the container
# COPY . .

# Keeps the container available.
CMD ["tail", "-f", "/dev/null"]
```

Add docker-compose.

- docker-compose.yaml
```yaml
version: '3'

services:
  spec_test:
    build:
      context: .
    container_name: rspec_test
    volumes:
      - .:/app

```

Now execute the application.
`docker-compose up --build`

# Install RSpec

1) Add dependency (recommended)
- rspec_test.gemspec
```ruby
...

spec.add_dependency "rspec", "~> 3.6.0"

...
```

2) Install library

Inside the container install rspec library.

`docker exec -it rspec_test bash`
`gem install rspec -v 3.6.0`
<!-- Successfully installed rspec-support-3.6.0
Successfully installed diff-lcs-1.5.1
Successfully installed rspec-mocks-3.6.0
Successfully installed rspec-expectations-3.6.0
Successfully installed rspec-core-3.6.0
Successfully installed rspec-3.6.0
6 gems installed -->

`rspec --version`
<!-- RSpec 3.6
  - rspec-core 3.6.0
  - rspec-expectations 3.6.0
  - rspec-mocks 3.6.0
  - rspec-support 3.6.0 -->


# Tests

## Creating the frist test

Now, create a test file.
- spec/sandwich_spec.rb:
```ruby
RSpec.describe 'An ideal sandwich' do
  it 'is delicious' do
    sandwich = Sandwich.new('delicious', [])

    taste = sandwich.taste

    expect(taste).to eq('delicious')
  end
end
```

To run all tests just execute `rspec`. RSpec will look inside the folder for files named <<something>>_spec.rb.

```bash
rspec

# F

# Failures:

#   1) An ideal sandwich is delicious
#      Failure/Error: sandwich = Sandwich.new('delicious', [])
     
#      NameError:
#        uninitialized constant Sandwich
#      # ./spec/sandwich_spec.rb:4:in `block (2 levels) in <top (required)>'

# Finished in 0.00133 seconds (files took 0.06932 seconds to load)
# 1 example, 1 failure

# Failed examples:

# rspec ./spec/sandwich_spec.rb:3 # An ideal sandwich is delicious

```

To see the test pass, add the Sandwich implementation in spec file.

- spec/sandwich_spec.rb:
```ruby
Sandwich = Struct.new(:taste, :toppings)

RSpec.describe 'An ideal sandwich' do
...

```

And run the test again, it will pass.
```bash
rspec

# Finished in 0.00154 seconds (files took 0.06852 seconds to load)
# 1 example, 0 failures
```

## Sharing Setup

Add a new test.
- spec/sandwich_spec.rb
```ruby
  it 'lets me add toppings' do
    sandwich = Sandwich.new('delicious', [])
    sandwich.toppings << 'cheese'
    toppings = sandwich.toppings

    expect(toppings).not_to be_empty
  end
```

Run the tests and all should pass.
`rspec`

### Hooks

The 'before' hook will run before each example.
Change code to use it.

- spec/sandwich_spec.rb
```ruby
RSpec.describe 'An ideal sandwich' do

  before { @sandwich = Sandwich.new('delicious', []) }

  it 'is delicious' do
    taste = @sandwich.taste

    expect(taste).to eq('delicious')
  end

  it 'lets me add toppings' do
    @sandwich.toppings << 'cheese'
    toppings = @sandwich.toppings

    expect(toppings).not_to be_empty
  end
end
```

All tests should pass.

If you need to clear out a test database before each example, a hook is a great place to do so.

It is a way to reduce duplication, but not an efficient way. Let's use helper methods.

### Helper Methods

```ruby

Sandwich = Struct.new(:taste, :toppings)

RSpec.describe 'An ideal sandwich' do

  def sandwich()
    @sandwich ||= Sandwich.new('delicious', [])
  end
  
  it 'is delicious' do
    taste = sandwich().taste

    expect(taste).to eq('delicious')
  end

  it 'lets me add toppings' do
    sandwich().toppings << 'cheese'
    toppings = sandwich().toppings

    expect(toppings).not_to be_empty
  end
end
```

### Sharing Objects With let

Only replace the method with let.

```ruby
...

let(:sandwich) { Sandwich.new('delicious', []) }

...
```