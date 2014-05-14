---
layout: post
title: Testing subdomains using Capybara
---

I've been using the excellent Capybara gem to run integration tests in the app I'm currently working on. This app requires custom user-chosen subdomains, which the Capybara wiki admits are hard to test using Capybara:

> Domain names (including subdomains) don’t work under rack-test. Since it’s a pain to set up subdomains for the other drivers anyway, you should consider an alternate solution.

The main difficulty is getting your app to be accessible through non-default hosts and managing the differences between drivers. As I was on a bit of a testing binge I refused to believe this so attempted to hack my way through this problem. Note, this solution requires you to alter your hosts file (or otherwise point requests to the right place) so if that's not for you I can't help.

The first thing I wanted to do is to be able to define the host Capybara uses to access the app, both to set the subdomain and set the length of the TLD to match my subdomain_fu configuration over different drivers (rack-test tests under example.org, others under localhost). So I added a manual_host configuration option to the Capybara Server class:

``` ruby

class Capybara::Server
  def self.manual_host=(value)
    @manual_host = value
  end
  def self.manual_host
    @manual_host ||= 'localhost'
  end

  def url(path)
    if path =~ /^http/
      path
    else
      (Capybara.app_host || "http://#{Capybara::Server.manual_host}:#{port}") + path.to_s
    end
  end
end

```

This defaults to localhost, retaining previous functionality if not used. It also overwrites the url generation method to use the manually set host instead of localhost. We can then choose to send requests using the modified host using tags:

``` ruby
# env.rb
Before ('@subdomain') do
  Capybara::Server.manual_host = 'subdomain.myapp.local'
  Capybara.default_host = 'subdomain.example.org'
end

After('@subdomain') do
  Capybara.default_host = "example.org"
  Capybara::Server.manual_host = "myapp.local"
end
```

So I can tag any features I wish to run under a subdomain with @subdomain. After the feature is run the hosts are reset to the defaults. The last step is to add the line "127.0.0.1    myapp.local" and "127.0.0.1    subdomain.myapp.local" to your hosts file and make sure all other subdomains that are generated in your tests are also in your hosts file.

It may well be that you can set example.org to point locally in your hosts file or rack-test can use other domains that can be used across non-rack-test drivers. It's not a flexible solution but it works for me!

