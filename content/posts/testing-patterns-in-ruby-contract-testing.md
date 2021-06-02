---
title: "Testing Patterns in Ruby: Contract Testing"
featured_image: "images/testing_patterns_in_ruby_contract_testing.jpeg"
images: ["images/testing_patterns_in_ruby_contract_testing.jpeg"]
date: 2018-08-20T07:58:57-04:00
tags: ['ruby', 'coding', 'testing']
---

*Maintain shared behavior of multiple implementations by making them pass the same tests.*

# Motivation

When building a system, there may emerge a subsystem or class that has to share a baseline set of behavior with other implementations, while providing other non-functional requirements. The shared behavior can be shown to exist by using a contract test.

# Example

Test doubles are a great way to reduce overall speed of the test suite by only running tests against in-memory versions of external services. We can verify a test double behaves like the real implementation, by running the same contract test against both. Though, we will likely run the slower test less often.

Here we have a VideoGameServiceContract that verifies all implementations of the VideoGameService duck-type (or interface for those using static languages).

![Contract Testing Diagram](/images/contract_testing.png)

In Ruby the VideoGameServiceContract can be implemented by using an RSpec shared example. Our VideoGameService should be able to query all games and add a game that will show up in subsequent queries.

Notice we only test the service via the public interface, this is crucial to contract testing as utilizing implementation details will make the tests coupled to specific implementations.

```ruby
shared_examples 'VideoGameService Contract' do
  it 'should have no video games by default' do
    expect(video_game_service.all_video_games.size).to eq(0)
  end

  it 'should save video game' do
    video_game_service.save(VideoGame.new('E.T. for NES'))

    expect(video_game_service.all_video_games.size).to eq(1)
    expect(video_game_service.all_video_games.first).to eq(VideoGame.new('E.T. for NES'))
  end
end
```

This contract is utilized by using the it_behaves_like method from RSpec. Allowing the tests to be included inside the describe block for InMemoryVideoGameService.

```ruby
describe InMemoryVideoGameService do
  it_behaves_like 'VideoGameService Contract'

  let(:video_game_service) {InMemoryVideoGameService.new}
end
```

Theses same shared specs can be applied to the HttpVideoGameService.

```ruby
describe HttpVideoGameService do
  let(:video_game_service) {HttpVideoGameService.new}

  it_behaves_like 'VideoGameService Contract'
end
```

Though both implementations are wildly different, both pass the tests. If certain implementations of the contract are slow they can be run less often as their own build in your favorite CI application.

Here is the code for both implementations. *The only thing I wish to highlight is that both implementations are different but pass the contract.*

```ruby
class InMemoryVideoGameService
  def initialize
    @video_games = []
  end

  def all_video_games
    @video_games
  end

  def save(video_game)
    @video_games << video_game
  end
end
class HttpVideoGameService
  def all_video_games
    JSON.parse(
      HTTParty.get('http://localhost:4567/all_video_games').body
    ).map do |d|
      VideoGame.new(d['name'])
    end
  end

  def save(video_game)
    HTTParty.post(
      'http://localhost:4567/save_video_game',
      {
        body: { name: video_game.name }.to_json,
        headers: { 'Content-Type' => 'application/json' }
      }
    )
  end
end
```

# Conclusion

Contract tests are a great way to verify that multiple subsystems act the same way. Verifying test doubles are a particularly valuable way to use contract tests. Contract tests can also be used against a set of different implementations that share only baseline behavior and add extra behavior like more efficient memory usage or faster data structure traversal.

Source code for this [contract testing example](https://github.com/steven-solomon/video_game_example/tree/contract-test-blog)

{{< thank-you >}}

*Photo by Tuân Nguyễn Minh on Unsplash*