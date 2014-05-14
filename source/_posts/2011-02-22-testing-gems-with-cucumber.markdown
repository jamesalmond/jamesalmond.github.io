---
title: Cucumber for gem documentation 
layout: post
---

I released my first gem, [Dozuki](http://github.com/jamesalmond/dozuki), a few weeks ago (work in progress!), both because I thought the functionality I was extracting could be useful to others but also as a bit of a test-bed for trying some new things with testing.

Alongside playing with some different ways of writing mocked and stubbed tests, I wanted to have a go at writing some Cucumber features that could also be used as [Relish](http://relishapp.com)-style documentation.

My experience of Cucumber in the Rails world is using it for customer-facing acceptance/integration tests with the steps driving the interface of a web app using something like [Capybara](https://github.com/jnicklas/capybara).

With Dozuki I wanted the Cucumber to be a way of clearly specifying what a feature should do, a way of integration testing the code and, to avoid duplicating effort, a set of documentation on how to use the code. Not too far from the intent of my normal features but the difference here is the way the steps are expressed. Let's have a look at extracting an integer from an XML document:

``` cucumber

Feature: Getting integers from the document
  In order to provide simpler way of getting integers from a node
  As a traverser
  I want to access nodes using the int method and an xpath
  
  Scenario: getting the int of a single node
    When I parse the XML:
      """
        <root>
          <name>St. George's Arms</name>
          <average_price>20.32</average_price>
          <number_of_beers>2</number_of_beers>
        </root>
      """
    And I call "int('/root/number_of_beers')" on the document
    Then the result should be 2

```

For a standard web app I'd try to not couple my steps too closely to the
implementation, making the features a little less brittle (e.g. using
"When I add the post" instead of the more explicit "When press 'Add Post'").
Here I want the exact opposite. I want my features to be coupled directly to
the implementation so I can demonstrate how to use it. To help with that I've
put the XML explicitly in each step. This stops the reader from having to
look in background steps for each scenario and allows you to change the XML
between scenarios. Secondly, I've used an actual code snippet on the penultimate
line. But you'll notice that the amount of code I've used is minimal. Too keep
the feature readable, I've only used the amount of code needed to demonstrate
the feature I'm currently testing and documenting; we're going to need another
feature and set of scenarios for the creation of a document.

Let's have a look at the steps behind the features:

``` ruby

When /^I parse the XML:$/ do |string|
  @doc = Dozuki::XML.parse(string)
end

When /^I call "([^"]*)" on the document$/ do |code|
  @result = @doc.instance_eval(code)
end

Then /^the result should be (\d+)$/ do |int|
  @result.should == int.to_i
end

```

The only noteworthy step here is the second, allowing us to execute the
code we've specified in the scenario against the document created in the
previous step by simply evaluating the code against the instance. The other two should be self-explanatory.

In another example we might want to assert that a certain piece of code
raises an error:

``` cucumber

Scenario: getting the int of a non-existent node
  When I parse the XML:
    """
      <root>
        <name>St. George's Arms</name>
        <average_price>20.32</average_price>
        <number_of_beers>2</number_of_beers>
      </root>
    """
  Then calling "int('//something/missing')" on the document should raise a "NotFound" error
```

Here we've got to do something a bit cleverer than the previous step as
the raised error has to be caught:

``` ruby

Then /^calling "([^"]*)" on the document should raise a "([^"]*)" error$/ do |code, error|
  begin
    @doc.instance_eval(code)
    fail "Expected error #{error}, nothing raised"
  rescue Dozuki::XML.const_get(error) => e
    @error = e
  end
end

```

Here the code is being evaluated against the instance as before, but if
no error is raised then we force a failure. The step only matches
against the error we've defined by getting the Exception constant from
the gem's scope using the name set in the scenario. The error is also
assinged to an instance variable in case we need to inspect it in
further steps.

These steps and features work well and remain fairly simple but what
about methods that take a block as an argument? How do we write feature
for those without the implementation details getting in the way? An
example in Dozuki is a simplified XPath each method:

``` cucumber
Scenario: using each to traverse a document and getting the integer elements
  When I parse the XML:
    """
      <root>
        <name>St. George's Arms</name>
        <average_price>20.32</average_price>
        <number_of_beers>2</number_of_beers>
        <rooms>
          <room>5</room>
          <room>7</room>
        </rooms>
      </root>
    """
  And I call "each('/root/rooms/room').as_int" on the document and collect the results
  Then the results should contain 5
  And the results should contain 5
```

To avoid having to define the code which handles the called block in the
scenario it's been hidden with "and collect the results". If we put the
block in the scenario we'd also have to deal with the assertion on the results or collecting them into some container in the scenario too. I figured this would be a bit messy. The implementation for these steps gets a bit hairy:

``` ruby

When /^I call "([^"]*)" on the document and collect the results$/ do |code|
  @results = @doc.instance_eval("results = [];" + code + "{|res| results << res}; results;")
end

```

Help! What's happening here? OK, well we're defining an array, executing
the code from the scenario and adding a block to the function call that
appends items passed to the block to an array... all in a string
executed against the instance. Riiighhht..

There's more features you can browse in the [Dozuki GitHub
repository](https://github.com/jamesalmond/dozuki/tree/master/features).
Take a look!

## Conclusion

Well, that's my first attempt at Cucumber as documentation. What do you
think?


The first obstacle I came across was finding a balance between the amount of code in
the document and the amount of natural language. Too much natural
language you lose the benefit of knowing how to run the code, too much
code and you lose the 'running commentary' and it's less expressive. My
intention was to use enough code to cover that 'feature' of the gem and
not cover other features.

Secondly, and I didn't spend too long thinking about this, the block
examples result in some funky steps. The less code in the example means
a bit more work behind the scenes but I think it's worth it and
hopefully the intent is understood. I also
tried to keep the steps as simple and readable as possible so anyone who
wanted to do some digging might find out a bit more. I think the block
one might be stretching it a bit..!

Hopefully I've covered enough examples to give other people an idea of how to start having a go at this.
Overall, I think it's something I'd pursue. The simple examples are easy to follow, easy to maintain, document the code and function as tests. Surely worth something?
