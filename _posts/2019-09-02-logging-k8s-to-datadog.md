---
layout: post
title: Logging Kubernetes to Datadog
excerpt_separator: <!-- /excerpt -->
---

Logging 40 pods, with three different server technologies, in one cluster.
Sound daunting?  Datadog made it surprisingly easy.
<!-- /excerpt -->
Kubernetes has allowed me to increase the cloud usage density, lowering costs, and increasing productivity.  Logging all of those pods running on the same servers, could be taxing, but Datadog and Kubernetes made it easy.

Datadog provides wonderful [directions](https://docs.datadoghq.com/integrations/kubernetes/) to get you started.  Shortly after getting the logging working, I found an issue revolving around Kubernetes status checks.  Status checks produce a lot of logs if you're not ready for it.  Datadog was ingesting over 3 million logs per day, before the systems hit production, at an estimated $177/mo.

## Solution

To limit the logs produced as a side effect of Kubernetes, I modified each systems to suppress logging from Kubernetes.  To determine the log entries from Kubernetes, I took two approaches:

1. Ignore requests from Kubernetes' User-Agents (`kube-probe/1.13+` in my case).
2. Tell Kubernetes to check a different URL, and ignore requests to that URL.

### Apache

Apache was fairly easy to suppress logs:

    SetEnvIf User-Agent "^kube-probe" dontlog
    SetEnvIf Request_URI "\.(jpg|js|png|ico|css|woff)$" dontlog
    CustomLog /proc/self/fd/1 central env=!dontlog

In this case, I turn of logging of those events where the User-Agent matches the regular expression.  I also added a few other lines to ignore directories and static file types, also lowering the number of log entries.

### NGINX

NGINX was also easy.  First, Kubernetes needed to be configured to use `/status.txt` as it's liveness endpoint.  If you're hosting an application within NGINX (such as PHP), you may want to use a path which ensures the application is alive (`/status.php`, for example).

Then, configure NGINX to ingore those requests:

    location /status.txt {
      access_log off;
    }

### Ruby on Rails

Ruby on Rails is where things got more challenging.  By default, Rails' logging is rather verbose.

    200 OK in 159ms (Views: 158.3ms | ActiveRecord: 0.0ms)
    Rendered root/index.html.haml within layouts/application (24.8ms)
    Parameters: {"locale"=>"en-US"}
    Processing by QuotesController#new as HTML
    Started GET "/" for <sic> at 2019-08-23 15:59:56 +0000

Step one is to shore this up.  I personally like [Lograge](https://github.com/roidrage/lograge) for this application.  It gets your logs down to a parsable format.  But what about ignore Kubernetes requests?  That requires some additional setup.  In your `ApplicationController` class, you need to add the User-Agent to the payload using a method provided by Lograge:

    def append_info_to_payload(payload)
      super
      payload[:user_agent] = request.user_agent
    end

Now, in `initializer/lograge.rb`, filter out requests based on the `User-Agent`:

    IGNORE_AGENTS = Regexp.union([
      /^kube-probe/i,
    ])
    Rails.application.configure do
      config.lograge.enabled = true
      config.lograge.ignore_custom = lambda do |event|
        event.payload[:user_agent].match(IGNORE_AGENTS).present?
      end
    end

I created a regular expression here, so that additional ignored user-agents can be easily specified later.

### Conclusion

Datadog logging is a beautiful thing, and worth looking into.  Be sure to monitor your usage during the free trial, though.  I also faced challenges in other applications, but those are too specialized to be worth listing here.

