# kemal-session-redis-engine

[![CI](https://github.com/crystal-garage/kemal-session-redis-engine/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/crystal-garage/kemal-session-redis-engine/actions/workflows/ci.yml)

Redis session store for [kemal-session](https://github.com/kemalcr/kemal-session) implemented with [jgaskins/redis](https://github.com/jgaskins/redis) client.

## Installation

Add this to your application's `shard.yml`:

```yaml
dependencies:
  kemal-session-redis-engine:
    github: crystal-garage/kemal-session-redis-engine
```

## Usage

```crystal
require "kemal"
require "kemal-session"
require "kemal-session-redis-engine"

Kemal::Session.config do |config|
  config.cookie_name = "redis_test"
  config.secret = "a_secret"
  config.engine = Kemal::Session::RedisEngine.new(
    "redis://localhost:6379/0?initial_pool_size=1&max_pool_size=10&checkout_timeout=10&retry_attempts=2&retry_delay=0.5&max_idle_pool_size=50",
    key_prefix: "my_app:session:"
  )
  config.timeout = Time::Span.new(1, 0, 0)
end

get "/" do
  puts "Hello World"
end

post "/sign_in" do |context|
  context.session.int("see-it-works", 1)
end

Kemal.run
```

The engine comes with a number of configuration options:

| Option     | Description                                                                                    |
| ---------- | ---------------------------------------------------------------------------------------------- |
| redis_url  | where your redis instance lives. defaults to `redis://localhost:6379/0`                        |
| key_prefix | when saving sessions to redis, how should the keys be namespaced. defaults to `kemal:session:` |

If no options are passed the `RedisEngine` will try to connect to a Redis using
default settings.

## Best Practices

### Creating a Client

It's very easy for client code to leak Redis connections and you should
pass a pool of connections that's used throughout Kemal and the
session engine.

### Session Administration Performance

`Kemal::Session.all` and `Kemal::Session.each` perform a bit differently under the hood. If
`Kemal::Session.all` is used, the `RedisEngine` will use the `SCAN` command in Redis
and page through all of the sessions, hydrating the Session object and returing
an array of all sessions. If session storage has a large number of sessions this
could have performance implications. `Kemal::Session.each` also uses the `SCAN` command
in Redis but instead of creating one large array and enumerating through it,
`Kemal::Session.each` will only hydrate and yield the keys returned from the current
cursor. Once that block of sessions has been yielded, RedisEngine will retrieve
the next block of sessions.

## Development

Redis must be running on localhost and bound to the default port to run
specs.

## Contributing

1. Fork it ( https://github.com/crystal-garage/kemal-session-redis-engine/fork )
2. Create your feature branch (git checkout -b my-new-feature)
3. Commit your changes (git commit -am 'Add some feature')
4. Push to the branch (git push origin my-new-feature)
5. Create a new Pull Request

## Contributors

- [[mamantoha](https://github.com/mamantoha)] Anton Maminov - maintainer
- [[neovintage](https://github.com/neovintage)] Rimas Silkaitis - creator, maintainer
- [[crisward](https://github.com/crisward)] Cris Ward
- [[fdocr](https://github.com/fdocr)] Fernando Valverde
