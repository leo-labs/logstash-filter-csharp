A logstash filter to parse C# stack traces and exception messages.

Installation
============
As this plugin has been shared on [RubyGems](https://rubygems.org) with the
name [logstash-filter-csharp](https://rubygems.org/gems/logstash-filter-csharp)
you can install it using the following command from your Logstash installation
path:

```sh
bin/logstash-plugin install logstash-filter-csharp
```

Usage
=====

Here is an example configuration that reads fields from a JSON-formatted log
and expects stack traces to be in the field `stackTrace` and exceptions to be
in the field `exception. It parses these into fields which are in this example
written to metadata that is then consumed by
[logstash-output-sentry](https://github.com/javiermatos/logstash-output-sentry)

```
filter {
  grok {
    match => { "message" => "\[%{LOGLEVEL:log_type}\] %{TIMESTAMP_ISO8601:timestamp_iso} :: %{GREEDYDATA:json_message}" }
  }

  json {
    source => "json_message"
  }

  if [stackTrace] {
   csharp {
     type => "stacktrace"
     source => [stackTrace]
     target => "[@metadata][sentry][stacktrace]"
     stackframe_path_prefix => ".*\\(?=prod\\)"
     most_recent_call_last => true
   }
 }

 if [exception] {
   csharp {
     type => "exception"
     source => [exception]
     target => "[@metadata][sentry][exception]"
     most_recent_call_last => true
  }
}
```

Running Tests
=============

The tests live in `spec/`. To run them, do something like

```
bundle install
bundle exec rspec
```

If this does not work, there might be a problem with your jruby setup. You
might want to get RVM and run something like this first
```
rvm install jruby
rvm use jruby
gem install bundler
```
