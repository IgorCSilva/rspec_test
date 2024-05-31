
## Defining Metadata

### Metadata Defined By RSpec

Create the file:
- spec/metadata_spec.rb
```rb
require 'pp'

RSpec.describe Hash do
  it 'is used by RSpec for metadata' do |example|
    pp example.metadata
  end
end
```

We can access many info about the example this way.

### Custom Metadata

Update the code..
```rb
...
  it 'is used by RSpec for metadata', fast: true do |example|
...
```
It's the same as:

```rb
...
  it 'is used by RSpec for metadata', :fast do |example|
...
```

Finally, when you set custom metadata on an example group, the contained examples and nested groups will inherit it.

### Derived Metadata

You can run only specific tests marking they with metadata running:
`rspec --tag fast`

This command will run only examples with metadata fast.

If you want set some metadata to many example you can do this at helper file:
- spec_helper.rb
```rb
RSpec.configure do |config|
  config.define_derived_metadata(file_path: /spec\/unit/) do |meta|
    meta[:fast] = true
  end
end
```
This code will add the :fast metadata to every example in the spec/unit folder.

To filter examples by some tag, we use something like this:
```rb
RSpec.configure do |config|
  config.define_derived_metadata(type: :model) do |meta|
    # ...
  end
end
```

### Default Metadata
You can check if some tag already is defined.
```rb
RSpec.configure do |config|
  config.define_derived_metadata do |meta|
    meta[:aggregate_failures] = true unless meta.key?(:aggregate_failures)
  end
end
```

The code above defines aggregate_failures to all examples that not already define it.

## Reading Metadata
We can access metadata like below:
```rb
it 'is used by RSpec for metadata' do |example|
  pp example.metadata
end
```

or

```rb
RSpec.configure do |config|
  config.around(:example) do |example|
    pp example.metadata
  end
end
```

or 

```rb
RSpec.describe 'Music storage' do
  let(:s3_client) do |example|
    S3Client.for(example.metadata[:s3_adapter])
  end
  
  it 'stores music on the real S3', s3_adapter: :real do
    # ...
  end

  it 'stores music on an in-memory S3', s3_adapter: :memory do
    # ...
  end
end
```

## Selecting Which Specs to Run

### The command Line
Run all tests with tag fast: true (or :fast).
`rspec --tag fast`

Run all tests with no tag fast: true (or :fast).
`rspec --tag ~fast`
or 
`rspec --tag '~fast'`

## Changing How Your Specs Run

- :aggregate_failures: Changes how RSpec reacts to failure so that each example runs to completion (instead of stopping at the first failed expectation);
- :pending: Indicates that you expect the example to fail; RSpec will run it and report it as pending if it did fail, or report it as a failure if it passed;
- :skip: Tells RSpec to skip the example entirely but still list the example in the output (unlike with filtering, which omits the example from the output);
- :order: Sets the order in which RSpec runs your specs (can be the same order as they’re defined, random order, or a custom order => :defined, :random)