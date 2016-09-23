# The Ruby One Time Password Library

[![Build Status](https://secure.travis-ci.org/mdp/rotp.png)](https://travis-ci.org/mdp/rotp)
[![Gem Version](https://badge.fury.io/rb/rotp.svg)](https://rubygems.org/gems/rotp)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://github.com/mdp/rotp/blob/master/LICENSE)

A ruby library for generating and validating one time passwords (HOTP & TOTP) according to [RFC 4226](http://tools.ietf.org/html/rfc4226) and [RFC 6238](http://tools.ietf.org/html/rfc6238).

ROTP is compatible with the [Google Authenticator](https://github.com/google/google-authenticator) available for [Android](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2) and [iPhone](https://itunes.apple.com/en/app/google-authenticator/id388497605).

Many websites use this for [multi-factor authentication](https://www.youtube.com/watch?v=17rykTIX_HY), such as GMail, Facebook, Amazon EC2, WordPress, and Salesforce. You can find the whole [list here](https://en.wikipedia.org/wiki/Google_Authenticator#Usage).

## Dependencies

* OpenSSL
* Ruby 1.9.3 or higher

## Installation

```bash
gem install rotp
```

## Library Usage

### Time based OTP's

```ruby
totp = ROTP::TOTP.new("base32secret3232")
totp.now # => "492039"

# OTP verified for current time
totp.verify("492039") # => 1474590700
sleep 30
totp.verify("492039") # => nil
```

Optionally, you can provide an issuer which will be used as a title in Google Authenticator.

```ruby
totp = ROTP::TOTP.new("base32secret3232", issuer: "My Service")
totp.provisioning_uri("alice@google.com")
```

### Counter based OTP's

```ruby
hotp = ROTP::HOTP.new("base32secretkey3232")
hotp.at(0) # => "260182"
hotp.at(1) # => "055283"
hotp.at(1401) # => "316439"

# OTP verified with a counter
hotp.verify("316439", 1401) # => 1401
hotp.verify("316439", 1402) # => nil
```

### Verifying a Time based OTP with drift

Some users devices may be slightly behind or ahead of the actual time. ROTP allows users to verify
an OTP code with an specific amount of 'drift'

```ruby
totp = ROTP::TOTP.new("base32secret3232")
totp.now # => "492039"

# OTP verified for current time with ±60 seconds (120 total) allowed drift
totp.verify("492039", drift: 60, at: Time.now - 30) # => 1474590700
totp.verify("492039", drift: 60, at: Time.now - 90) # => nil
```

### Preventing reuse of Time based OTP's

In order to prevent reuse of time based tokens within the interval window (default 30 seconds)
it is necessary to store the last time an OTP was used. The following is an example of this in action:

```ruby
User.find(someUserID)
totp = ROTP::TOTP.new(user.otp_secret)
totp.now # => "492039"

user.last_otp_at # => 1472145530

# Verify the OTP
last_otp_at = totp.verify("492039", after: user.last_otp_at) #=> 1472145760
# Store this on the user's account
user.update(last_otp_at: last_otp_at)
last_otp_at = totp.verify("492039", after: user.last_otp_at) #=> nil
```

### Generating a Base32 Secret key

```ruby
ROTP::Base32.random_base32  # returns a 16 character base32 secret. Compatible with Google Authenticator
```

Note: The Base32 format conforms to [RFC 4648 Base32](http://en.wikipedia.org/wiki/Base32#RFC_4648_Base32_alphabet)

### Google Authenticator Compatible URI's

Provisioning URI's generated by ROTP are compatible with the Google Authenticator App
to be scanned with the in-built QR Code scanner.

```ruby
totp.provisioning_uri("alice@google.com") # => 'otpauth://totp/issuer:alice@google.com?secret=JBSWY3DPEHPK3PXP'
hotp.provisioning_uri("alice@google.com", 0) # => 'otpauth://hotp/issuer:alice@google.com?secret=JBSWY3DPEHPK3PXP&counter=0'
```

This can then be rendered as a QR Code which can then be scanned and added to the users
list of OTP credentials.

#### Working example

Scan the following barcode with your phone, using Google Authenticator

![QR Code for OTP](http://chart.apis.google.com/chart?cht=qr&chs=250x250&chl=otpauth%3A%2F%2Ftotp%2Falice%40google.com%3Fsecret%3DJBSWY3DPEHPK3PXP)

Now run the following and compare the output

```ruby
require 'rubygems'
require 'rotp'
totp = ROTP::TOTP.new("JBSWY3DPEHPK3PXP")
p "Current OTP: #{totp.now}"
```

### Testing

```bash
bundle install
bundle exec rspec
```

## Executable Usage

Once the rotp rubygem is installed on your system, you should be able to run the `rotp` executable
(if not, you might find trouble-shooting help [at this stackoverflow question](http://stackoverflow.com/a/909980)).

```bash
# Try this to get an overview of the commands
rotp --help

# Examples
rotp --secret p4ssword                       # Generates a time-based one-time password
rotp --hmac --secret p4ssword --counter 42   # Generates a counter-based one-time password
```

## Contributors

Have a look at the [contributors graph](https://github.com/mdp/rotp/graphs/contributors) on Github.

## License

MIT Copyright (C) 2016 by Mark Percival, see [LICENSE](https://github.com/mdp/rotp/blob/master/LICENSE) for details.

## Other implementations

A list can be found at [Wikipedia](https://en.wikipedia.org/wiki/Google_Authenticator#Implementations).
