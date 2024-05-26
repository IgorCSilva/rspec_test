
## Getting the Words Right

### The Basics

We can use:
- describe
- it

We can pass Ruby class/module/object and string as description, including the combination.

Examples:

```rb
RSpec.describe 'My awesome gardening API' do
end

RSpec.describe Perennials::Rhubarb do
end

RSpec.describe Perennials do
end

RSpec.describe my_favorite_broccoli do
end

RSpec.describe Garden, 'in winter' do
end
```

We can pass metadata to all theses examples:
```rb

RSpec.describe Garden, 'in winter', uses_network: true do
end
```

### Other Ways to Get the Words Right

We can use `context` instead of `describe`.
We can use `example` instead of `it`.

RSpec provides the `specify` alias too.
```rb
RSpec.describe 'Deprecations' do
  specify 'MyGem.config is deprecated in favor of MyGem.configure'
  specify 'MyGem.run is deprecated in favor of MyGem.start'
end
```

### Defining Your Own Names

If you want to pause execution after each example in a group to inspect the data, you can import the Pry gem and call binding.pry inside each `it` block.

```rb
pit '...description...' do

end
```

- .../spec_helper.rb
```rb
RSpec.configure do |rspec|
  rspec.alias_example_group_to :pdescribe, pry: true
  rspec.alias_example_to :pit, pry: true

  rspec.after(:example, pry: true) do |ex|
    require 'pry
    binding.pry
  end
end
```

## Sharing Common Logic

The main three organization tools are let definitions, hooks, and helper methods.

```rb
RSpec.describe 'POST a successful expense' do
  # let definitions
  # obs.: RSpec never runs the block until it's actually needed.
  let(:ledger) { instance_double('ExpenseTracker::Ledger') }
  let(:expense) { 'some' => 'data' }

  # hook
  before do
    allow(ledger).to receive(:record)
      .with(expense)
      .and_return(RecordResult.new(true, 417, nil))
  end

  # helper method
  def parsed_last_response
    JSON.parse(last_response.body)
  end
end
```

#### Hooks

Writing a hook invlves two concepts. The type of hook controls when it runs relative to your examples. The scope controls how often you hook runs.

- Type

There are three types: before, after and around.

The before hook will run before the examples.
The after hook will run after the examples, even if the example fails or the before hook raises an exception.

The around hook put the spec inside the hook, so part of the hook runs before the example and part runs after.

```rb
RSpec.describe MyApp::Configuration do
  around(:example) do |ex|
    original_env = ENV.to_hash
    ex.run
    ENV.replace(original_env)
  end
end
```

We can define the hooks once for you entire suite, in an RSpec.configure block:

- spec/spec_helper.rb or somewhere in spec/support
```rb
RSpec.configure do |config|
  config.around(:example) do |ex|
    original_env = ENV.to_hash
    ex.run
    ENV.replace(original_env)
  end
end
```

You’ve defined these config hooks in one place, but they’ll run for every example in your test suite. Note the trade-offs here:

• Global hooks reduce duplication, but can lead to surprising “action at a distance” effects in your specs.

• Hooks inside example groups are easier to follow, but it’s easy to leave out an important hook by mistake when you’re creating a new spec file.

#### Scope

Most of the hooks you've written are meant to run once per example. Since this behavior is the norm, RSpec sets the scope to :example if you do not provide one.

Sometimes, though, a hook needs to do a really time-intensive operation like creating a bunch database tables or launching a live web browser. Running the hook once per spec would be cost-prohibitive.

For these cases, you can run the hook just once for the entire suite of specs or once per example group. Hooks take a :suite or :context argument to modify the scope.

Here we have a hook that launch a web browser just once for an example group:
```rb
RSpec.describe 'Web interface to my thermostat' do
  before(:context) do
    WebBrowser.launch
  end

  after(:context) do
    WebBrowser.shutdown
  end
end
```

When yu use a :context hook you're responsible for cleaning up any resulting state - othewise, it can caouse other specs to pass or fail incorrectly.

To run a piece of setup code just once, before the first example begins, we use :suite scope:

```rb
require 'fileutils'

RSpec.configure do |config|
  config.before(:suite) do
    # Remove leftover temporary files
    FileUtils.rm_rf('tmp')
  end
end
```

The RSpec.configure is the only place :suite hooks are allowed, because they exist independently of any examples or groups.

You can use `before` and `after` hooks with any of the three scopes(:example, :context, :suite). But `around` hooks only support :example scope.

The old way to specify :example and :context hooks is using :each and :all scopes, but because :all is confuse suggesting all the examples in the entire suite is recommended use :each and :context instead.

### Helper Methods

```rb
RSpec.describe BerlinTransitTicket do
  let(:ticket) { BerlinTransitTicket.new }

  before do
    # These values depend on `let` definitions
    # defined in the nested contexts below!
    #
    ticket.starting_station = starting_station
    ticket.ending_station = ending_station
  end

  let(:fare) { ticket.fare }

  context 'when starting in zone A' do
    let(:starting_station) { 'Bundestag' }

    context 'and ending in zone B' do
      let(:ending_station) { 'Leopoldplatz' }

      it 'costs € 2.70' do
        expect(fare).to eq 2.7
      end
    end

    context 'and ending in zone C' do
      let(:ending_station) { 'Birkenwerder' }

      it 'costs € 3.30' do
        expect(fare).to eq 3.3
      end
    end
  end
end
```

To get a better representation of what behavior we're testing, we can use this structure:

```rb
RSpec.describe BerlinTransitTicket do
  def fare_for(starting_station, ending_station)
    ticket = BerlinTransitTicket.new
    ticket.starting_station = starting_station
    ticket.ending_station = ending_station
    ticket.fare
  end

  context 'when starting in zone A and ending in zone B' do
    it 'costs € 2.70' do
      expect(fare_for('Bundestag', 'Leopoldplatz')).to eq 2.7
    end
  end

  context 'when starting in zone A and ending in zone C' do
    it 'costs € 3.30' do
      expect(fare_for('Bundestag', 'Birkenwerder')).to eq 3.3
    end
  end
end
```

#### Including Modules
- In specific files:
```rb
RSpec.describe 'Expense Tracker API', :db do
  include Rack::Test::Methods
  
  def app
    ExpenseTracker::API.new
  end
  # ...
end
```

- In all test files:
```rb
RSpec.configure do |config|
  config.include APIHelpers
end
```

Most of the time, you probably want to load using a conditional way.

## Sharing Examples Groups

### Sharing Contexts

Context definition to share:
```rb
RSpec.shared_context 'API helpers' do
  include Rack::Test::Methods

  def app
    ExpenseTracker::API.new
  end

  before do
    basic_authorize 'test_user', 'test_password'
  end
end
```

You can include this way:
```rb
RSpec.describe 'Expense Tracker API', :db do
  include_context 'API helpers'
  # ...
end
```

Or this way, to all tests:
```rb
RSpec.configure do |config|
  config.include_context 'API helpers'
end
```

### Sharing Examples

If you want to test a HashKeyValueStore and a FileKeyValueStore that have the same interface, we can do the following:

```rb
RSpec.shared_examples 'KV store' do |kv_store_class|
  let(:kv_store) { kv_store_class.new }

  it 'allows you to fetch previously stored values' do
    kv_store.store(:language, 'Ruby')
    kv_store.store(:os, 'linux')

    expect(kv_store.fetch(:language)).to eq 'Ruby'
    expect(kv_store.fetch(:os)).to eq 'linux'
  end

  it 'raises a KeyError when you fetch an unknown key' do
    expect { kv_store.fetch(:foo) }.to raise_error(KeyError)
  end
end
```

Then use it like:
```rb
require 'hash_kv_store'
require 'support/kv_store_shared_examples'

RSpec.describe HashKVStore do
  it_behaves_like 'KV store', HashKVStore
end
```

```rb
require 'file_kv_store'
require 'support/kv_store_shared_examples'

RSpec.describe FileKVStore do
  it_behaves_like 'KV store', FileKVStore
end
```

By convention, shared examples go in spec/support.

We can share a block of code using it_behaves_like too:

```rb
RSpec.shared_examples 'KV store' do
  it 'allows you to fetch previously stored values' do
    kv_store.store(:language, 'Ruby')
    kv_store.store(:os, 'linux')

    expect(kv_store.fetch(:language)).to eq 'Ruby'
    expect(kv_store.fetch(:os)).to eq 'linux'
  end

  it 'raises a KeyError when you fetch an unknown key' do
    expect { kv_store.fetch(:foo) }.to raise_error(KeyError)
  end
end
```
```rb
require 'tempfile'

RSpec.describe FileKVStore do
  it_behaves_like 'KV store' do
    let(:tempfile) { Tempfile.new('kv.store') }
    let(:kv_store) { FileKVStore.new(tempfile.path) }
  end
end
```