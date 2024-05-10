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