[![Build Status](https://travis-ci.org/tubedude/finance.svg?branch=working)](https://travis-ci.org/tubedude/finance)
# FINANCE

a library for financial modelling in Ruby.

## INSTALL

    $ sudo gem install finance

## IMPORTANT CHANGES

Contributions by [@thadd](https://github.com/thadd) and
[@bramswenson](https://github.com/bramswenson) have made the `finance`
library fully compatible with rails, at the cost of the `#years` and
`#months` convenience methods on `Integer`, as well as the `#to_d` method for
converting `Numerics` into `DecNums`.  These methods have been removed, due to
conflicts with existing rails methods.

Correspondingly, `finance` has been bumped up to version 2.0.

## OVERVIEW

### GETTING STARTED

    >> require 'finance'

*Note:* As of version 1.0.0, the entire library is contained under the
Finance namespace.  Existing code will not work unless you add:

    >> include Finance

for all of the examples below, we'll assume that you have done this.

### CONFIGURATION

In `config/initializers/finance.rb` Finance allows to set tolerance (eps) and default guess for IRR and XIRR calculations, such as:

```ruby
Finance.configure do |config|
  config.eps = '1.0e-12'
  config.guess = 0.5
end
```

### AMORTIZATION

You are interested in borrowing $250,000 under a 30 year, fixed-rate
loan with a 4.25% APR.

    >> rate = Rate.new(0.0425, :apr, :duration => (30 * 12))
    >> amortization = Amortization.new(250000, rate)

Find the standard monthly payment:

    >> amortization.payment
    => DecNum('-1229.91')

Find the total cost of the loan:

    >> amortization.payments.sum
    => DecNum('-442766.55')

How much will you pay in interest?

    >> amortization.interest.sum
    => DecNum('192766.55')

How much interest in the first six months?

    >> amortization.interest[0,6].sum
    => DecNum('5294.62')

If your loan has an adjustable rate, no problem.  You can pass an
arbitrary number of rates, and they will be used in the amortization.
For example, we can look at an amortization of $250000, where the APR
starts at 4.25%, and increases by 1% every five years.

    >> values = %w{ 0.0425 0.0525 0.0625 0.0725 0.0825 0.0925 }
    >> rates = values.collect { |value| Rate.new( value, :apr, :duration => (5  * 12) }
    >> arm = Amortization.new(250000, *rates)

Since we are looking at an ARM, there is no longer a single "payment" value.

    >> arm.payment
    => nil

But we can look at the different payments over time.

    >> arm.payments.uniq
    => [DecNum('-1229.85'), DecNum('-1360.41'), DecNum('-1475.65'), DecNum('-1571.07'), ... snipped ... ]

The other methods previously discussed can be accessed in the same way:

    >> arm.interest.sum
    => DecNum('287515.45')
    >> arm.payments.sum
    => DecNum('-537515.45')

Last, but not least, you may pass a block when creating an Amortization
which returns a modified monthly payment.  For example, to increase your
payment by $150, do:

    >> rate = Rate.new(0.0425, :apr, :duration => (30 * 12))
    >> extra_payments = 250000.amortize(rate){ |period| period.payment - 150 }

Disregarding the block, we have used the same parameters as the first
example.  Notice the difference in the results:

    >> amortization.payments.sum
    => DecNum('-442745.98')
    >> extra_payments.payments.sum
    => DecNum('-400566.24')
    >> amortization.interest.sum
    => DecNum('192745.98')
    >> extra_payments.interest.sum
    => DecNum('150566.24')

You can also increase your payment to a specific amount:

    >> extra_payments_2 = 250000.amortize(rate){ -1500 }

### IRR and XIRR

```ruby
guess = 0.1
transactions = []
transactions << Transaction.new(-10000, date: '2010-01-01'.to_time(:utc))
transactions << Transaction.new(123000, date: '2012-01-01'.to_time(:utc))
transactions.xirr(guess)
#  => Rate.new(2.507136, :apr)
```


## ABOUT

I started developing `finance` while analyzing mortgages as a personal
project.  Spreadsheets have convenient formulas for doing this type of
work, until you want to do something semi-complex (like ARMs or extra
payments), at which point you need to create your own amortization
table.  I thought I could create a better interface for this type of
work in Ruby, and since I couldn't find an existing resource for these
tools, I am hoping to save other folks some time by releasing what I
have as a gem.

More broadly, I believe there are many calculations that are necessary
for the effective management of personal finances, but are difficult
(or impossible) to do with spreadsheets or other existing open source
tools.  My hope is that the `finance` library will grow to provide a set
of open, tested tools to fill this gap.

If you have used `finance` and find it useful, I would enjoy hearing
about it!

## FEATURES

Currently implemented features include:

* Uses the [flt](http://flt.rubyforge.org/) library to ensure precision decimal arithmetic in all calculations.
* Fixed-rate mortgage amortization (30/360).
* Interest rates
* Various cash flow computations, such as NPV and IRR.
* Adjustable rate mortgage amortization.
* Payment modifications (i.e., how does paying an additional $75 per month affect the amortization?)

## RESOURCES

* [RubyGems Page](https://rubygems.org/gems/finance)
* [Source Code](http://github.com/wkranec/finance)
* [Bug Tracker](https://github.com/wkranec/finance/issues)
* [Google Group](http://groups.google.com/group/finance-gem/topics?pli=1)

## COPYRIGHT

This library is released under the terms of the LGPL license.

Copyright (c) 2011, William Kranec.
All rights reserved.

This program is free software: you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by the
Free Software Foundation, either version 3 of the License, or (at your
option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
General Public License for more details.

You should have received a copy of the GNU General Public License along
with this program.  If not, see <http://www.gnu.org/licenses/>.
