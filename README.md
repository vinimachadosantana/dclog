# DcLog

## About

This gem provides you a garantee of logging stdout and stderr in both Rails and Sidekiq.

It creates a friendly interface to both Rails and Sidekiq loggers, trying to throw in the called function and considerably reducing code line size.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'dclog'
```

And then execute:

    $ bundle install

By configuring environment.rb, you can create a standard log to all environments. You can also set it up only in production.rb, but we recomemend that you create a standard in order to avoid overwriting environments.

```ruby
# config/environment.rb

require_relative 'application'

Rails.application.configure do
  ...

  logger = ActiveSupport::Logger.new($stdout)
  logger.formatter = proc do |severity, datetime, progname, message|
    msg_regex = message.match(Dclog::LOG_REGEX)
    request_id = msg_regex.nil? ? nil : msg_regex[1]
    msg = msg_regex.nil? ? message : msg_regex[2]

    "#{JSON.dump(
      severity: severity,
      date: datetime.strftime('%Y-%m-%d %H:%M:%S'),
      caller: progname,
      request_id: request_id,
      message: msg
    )}\n"
  end
  config.logger    = ActiveSupport::TaggedLogging.new(logger)
  level            = ENV.fetch('LOG_LEVEL', 'info')
  config.log_level = Rails.env.test? ? :warn : level.underscore.to_sym
  config.log_tags  = [:request_id]

  ...
end

Rails.application.initialize!
```

## Usage

Call gem + severity level and it automatically searches the methods name.

```ruby
# info severity
Dclog.info('Information is key')

# warn severity
Dclog.warn('Beware of warning')

# error severity
Dclog.error('Another one bites the dust')

# fatal severity
Dclog.fatal('FATALITY')

# debug severity
Dclog.debug('Rubber duck time')
```

You should consider the possibility of replacing all log calls to only use Dclog in order to create a standard and avoid confusion.

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and the created tag, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/[USERNAME]/dclog.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
