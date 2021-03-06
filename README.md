# LexisNexisInstantAuthenticate

Generate & Score Instant Authenicate quizes using Lexis Nexis SOAP API.

## Notes

This gem originally started as a simple Savon.rb service, however, it was quickly made clear that the LN SOAP API has some rough edges that require some specialized changes.
The main patch was around authentication. This gem monkey patches the `Akami::WSSE#wsse_username_token` method to include the cleartext password along with the digest auth.

This gem also does not leverage the WSDL from LN, even though it does import it into Savon. Savon has problems parsing the WSDL correctly and as such does not provide a reliable interface.


## Installation

Add this line to your application's Gemfile:

```ruby
gem 'lexis_nexis_instant_authenticate'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install lexis_nexis_instant_authenticate

## Usage

###Create a client.
```ruby
@client = LexisNexisInstantAuthenticate::Client.new(username: ENV['LEXIS_UN'], password: ENV['LEXIS_PW'], flow: "LEXIS_FLOW", transaction_id: SecureRandom.uuid)
```

Options:
 * `username:` Lexis Nexis supplied username. Typically, "#{account_id}/#{username}"
 * `password:` Lexis Nexis supplied password.
 * `flow:` Name of workflow.
 * `transaction_id:` Optional GUID. This must be the same string between quiz creation & quiz scoring requests. If you do not pass one in on quiz creation, `SecureRandom.uuid` will be used and returned with the generated quiz. That same string must be sent when scoring quiz.

###Generate a quiz.
```ruby
  response = @client.create_quiz({first_name: "Joe", middle_name: "L", last_name: "Smith", ssn: "123456789", dob: {day: "01", month: "01", year: "1950"}})
  if response.success?
    return response
  else
    return response.status
  end
```

Response will have:
* `response.questions`
* `response.id`
* `response.transaction_id`

###Scoring Responses
The `score_quiz` method expects the following arguments:
* `transaction_id` This is the GUID that you called the `create_quiz` method with or the GUID that was generated for you.
* `responses` This is an array of hashes in the form of `{question_id: 1234, choice_id: 4321}`

```ruby
  response = @client.score_quiz(transaction_id, Array(@responses).map(&:with_indifferent_access))
  if response.success? && response.pass?
    return true
  else
    return response.status
  end
```

Response will have:
* `response.success?`
* `response.pass?`
* `response.score`
* `response.proofing_response`

## To-Do
* Write Tests
* Transition to actual WSDL from LN.
* Refactor `Response` Object.

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).


## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/RepPro/lexis_nexis_instant_authenticate.

