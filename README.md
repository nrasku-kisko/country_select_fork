# Rails – Country Select
[![Gem Version](https://badge.fury.io/rb/country_select.svg)](https://badge.fury.io/rb/country_select) [![build](https://github.com/countries/country_select/actions/workflows/test.yml/badge.svg)](https://github.com/countries/country_select/actions/workflows/test.yml)
[![Code Climate](https://codeclimate.com/github/countries/country_select.svg)](https://codeclimate.com/github/countries/country_select)
[![CodeQL](https://github.com/countries/country_select/actions/workflows/codeql-analysis.yml/badge.svg)](https://github.com/countries/country_select/actions/workflows/codeql-analysis.yml)


Provides a simple helper to get an HTML select list of countries using the
[ISO 3166-1 standard](https://en.wikipedia.org/wiki/ISO_3166-1).

While the ISO 3166 standard is a relatively neutral source of country
names, it may still offend some users. Developers are strongly advised
to evaluate the suitability of this list given their user base.

## UPGRADING

[**An important message about upgrading from 1.x**](UPGRADING.md)

## Reporting issues

Open an issue on the [issue tracker](https://github.com/countries/country_select/issues/new). Ideally provide versions used, and code example that demonstrates the issue.

## Installation

Install as a gem using

```shell
gem install country_select
```
Or put the following in your Gemfile

```ruby
gem 'country_select', '~> 8.0'
```

## Usage

Within `form_for` you can use this select like other form elements:

```ruby
<%= form_for User.new, url: root_url do |f| %>
  <%= f.country_select :country_code %>
<% end %>
```

Simple use supplying model and attribute as parameters:

```ruby
country_select("user", "country")
```

Supplying priority countries to be placed at the top of the list:

```ruby
country_select("user", "country", priority_countries: ["GB", "FR", "DE"]) # Countries will be sorted by name according to the current locale
# or
country_select("user", "country", priority_countries: ["GB", "FR", "DE"], sort_provided: false) # Countries will be displayed is the provided order
```

Supplying only certain countries:

```ruby
country_select("user", "country", only: ["GB", "FR", "DE"]) # Countries will be sorted by name according to the current locale
# or
country_select("user", "country", only: ["GB", "FR", "DE"], sort_provided: false) # Countries will be displayed is the provided order
```

Discarding certain countries:

```ruby
country_select("user", "country", except: ["GB", "FR", "DE"])
```

Pre-selecting a particular country:

```ruby
country_select("user", "country", selected: "GB")
```

Using existing `select` options:
```ruby
country_select("user", "country", include_blank: true)
country_select("user", "country", { include_blank: 'Select a country' }, { class: 'country-select-box' })
```

Supplying additional html options:

```ruby
country_select("user", "country", { priority_countries: ["GB", "FR"], selected: "GB" }, { class: 'form-control', data: { attribute: "value" } })
```

### Using a custom formatter

You can define a custom formatter which will receive an
[`ISO3166::Country`](https://github.com/countries/countries/blob/master/lib/countries/country.rb)
```ruby
# config/initializers/country_select.rb

# Return a string to customize the text in the <option> tag, `value` attribute will remain unchanged
CountrySelect::FORMATS[:with_alpha2] = lambda do |country|
  "#{country.iso_short_name} (#{country.alpha2})"
end

# Return an array to customize <option> text, `value` and other HTML attributes
CountrySelect::FORMATS[:with_data_attrs] = lambda do |country|
  [
    country.iso_short_name,
    country.alpha2,
    {
      'data-country-code' => country.country_code,
      'data-alpha3' => country.alpha3
    }
  ]
end
```

```ruby
country_select("user", "country", format: :with_alpha2)
country_select("user", "country", format: :with_data_attrs)
```

### Using customized defaults

You can configure overridable defaults for `except`, `format`, `locale`,
`only` and `priority_countries` in an initializer.

````ruby
# config/initializers/country_select.rb

CountrySelect::DEFAULTS[:except] = [ "ZZ" ]
````

The example would exclude "ZZ" from every `country_select` tag, where no `except` option is given.

### ISO 3166-1 alpha-2 codes
The `option` tags use ISO 3166-1 alpha-2 codes as values and the country
names as display strings. For example, the United States would appear as
`<option value="US">United States of America</option>`

Country names are automatically localized based on the value of
`I18n.locale` thanks to the wonderful
[countries gem](https://github.com/countries/countries/).

Current translations include:

  * en
  * de
  * es
  * fr
  * it
  * ja
  * nl

In the event a translation is not available, it will revert to the
globally assigned locale (by default, "en").

This is the only way to use `country_select` as of version `2.0`. It
is the recommended way to store your country data since it will be
resistant to country names changing.

The locale can be overridden locally:

```ruby
country_select("user", "country_code", locale: 'es')
```

#### Getting the Country Name from the countries gem

```ruby
class User < ActiveRecord::Base

  # Assuming country_select is used with User attribute `country_code`
  # This will attempt to translate the country name and use the default
  # (usually English) name if no translation is available
  def country_name
    country = ISO3166::Country[country_code]
    country.translations[I18n.locale.to_s] || country.common_name || country.iso_short_name
  end

end
```

## Example Application

An example Rails application demonstrating the different options is
available at [countries/country_select_demo](https://github.com/countries/country_select_demo) and deployed to [country-select-demo.onrender.com](https://country-select-demo.onrender.com).
The relevant view files live [here](https://github.com/countries/country_select_demo/tree/master/app/views/demo).

## Contributing

### Tests

```shell
bundle
bundle exec rake
```

### Updating gemfiles
The default rake task will run the tests against multiple versions of
Rails. That means the gemfiles need occasional updating, especially when
changing the dependencies in the gemspec.

```shell
for i in gemfiles/*.gemfile
do
BUNDLE_GEMFILE=$i bundle install --local
done
```

Copyright (c) 2008 Michael Koziarski, released under the MIT license
