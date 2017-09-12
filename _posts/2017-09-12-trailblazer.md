---
layout: post
title: Trailblazer, beyond MVC
excerpt_separator: <!-- /excerpt -->
---

Trailblazer provides abstractions for code that doesn't belong in Models, Views, or Controllers.

<!-- /excerpt -->

Previously, we discussed how [dumb templates]({% post_url 2017-03-01-dumb-templates %}) help Mexico Insurance Online provide resellers with hyper-customized versions of our application. Now, we'll discuss a way of compartmentalizing code, which extends this flexibility by compartmentalizing operations with [Trailblazer](http://trailblazer.to/).

The basic constructs in Rails are Models, Views, and Controllers (MVC).  Each of these has a specific purpose which they excel at.  Models are great at storing data in a database.  Controllers are great at routing web requests to ruby code.  Views are great at presenting the data and overall User eXperience (UX).  These constructs from Rails are the basics constructs of any simple CRUD (Create, Read, Update, Delete, the four basic actions taken on data) application.

In business applications, however, we have more complex logic that may not fit into the MVC "buckets".  As an example, consider the following, to charge a credit card (completely made up):

```ruby
def charge_card(cc_info, amount)
  transaction = Braintree.new_transaction(cc_num: cc_info.account_number)
  if transaction.authorize(amount)
    transaction.charge(amount)
  else
    raise "Unable to charge credit card."
  end
end
```

The credit card info will never be saved to the database, used directly inside a view, and is not a part of a web request, so doesn't belong in MVC.  This is where Trailblazer steps in and offers a new abstraction called an Operation.  Each operation encompasses a specific workflow.  As an example, the above code could be written as an operation:

```ruby
class Payment::Charge::CreditCard < Trailblazer::Operation
  step :model!
  step :authorize!
  failure :error!
  step :charge!
  failure :error!

  private

  def model!(options, params:, **)
    options['model'] = params['gateway'].new_transaction(
      cc_num: params['cc_info'].account_number
    )
  end

  def authorize!(options, params:, **)
    options['model'].authorize(params['amount'])
  end

  def charge!(options, params:, **)
    options['model'].charge(params['amount'])
  end

  def error!
    raise "Unable to charge credit card."
  end
end
```

Even writing this out now, I find myself thinking, "That's a lot more code to do the same thing", but let's break the operation down.  The calls to `step` and `failure` outline the process.  Simple and readable definition of the steps required to charge a credit card.  Each step is a method, and by appending `!`, we tell Trailblazer that the return value of the method is important.  A 'falsey' return will cause the Operation to trigger the next `failure` step, stopping the operation.  Also notice that steps are reusable, in the case of `error!`

The operation can be easily called from anywhere in the application with `Payment::Charge::CreditCard.call()`.  This means that if you are creating both a web interface, and an API, the same Operation can be called, and be safe in assuming it will work the same either way.  In fact, Trailblazer doesn't even require Rails, so this same Operation could be used in Rails, Sinatra, or even in a command line tool.  

As icing on the cake, this Operation is also much easier to test compared to the same code in a Controller.  Because the Operation does not rely on Rails (or even ActiveRecord), it can be tested in isolation of the rest of the system, making for faster tests, and easier isolation.

Trailblazer is not a silver bullet for development, but it does add useful abstractions to help keep code DRY, and help maintain the Single Responsibility Principle.