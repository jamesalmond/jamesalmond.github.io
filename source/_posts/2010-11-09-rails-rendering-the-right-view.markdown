---
title: 'Rails: rendering the right view'
layout: post
---

I've recently seen an article on Rails best practices that encouraged the use of a 'magic' controller gem to abstract all the controller logic away from the user. Now, I'm all for skinny controllers (I'll come to the fat models at a later date) and I'm sure these gems work great for basic CRUD apps but who's ever created 'just a basic CRUD app'?  These kind of gems, along with most of the tutorials I've seen, reinforce the idea that a controller can only redirect or render a view that matches the name of the method. Want to render something based on the state of the model then you just use a partial, right?

Well, no. I believe that the view should be responsible for taking an object (or multiple objects) and rendering the required output; not taking some objects and choosing which views to render based on their state. It's the job of the controller to choose what to render. The offending code generally looks something like this:

``` ruby
#some_object_controller.rb
class SomeObjectController < ApplicationController
  def search
    @search_results = SomeClass.find(params[:search_term])
  end
end

# search.html.haml
- if @search_results.count > 0
  - @search_results.each do |result|
    #...
- else
  %p Sorry, you have no results!

```

This can be refactored to something like the following:

``` ruby
#some_object_controller.rb
class SomeObjectController < ApplicationController

  def search
    @search_results = SomeClass.find(params[:search_term])
    if @search_results.count > 0
      render :search_results
    else
      render :no_search_results
    end
  end
  
end

# search_results.html.haml
- @search_results.each do |result|
  #...

# no_search_results.html.haml
%p Sorry, you have no results!

```

This clearly separates the responsibility of each view. You can even do something like this:

``` ruby
class SomeObjectController < ApplicationController
  def show
    @object = SomeObject.find(params[:id])
    render @object.status
  end
end
```

With the corresponding view file for each of the (finite) states of the object.

"But you shouldn't put logic in your controllers!". Yes, you should.  You should put controller logic in your controllers. Agreed, you shouldn't put business logic or the logic of your storage layer in a controller; but the logic about what should be done with the results of a query (or similar) is definitely suited to the controller.

How do we benefit:

* Although controllers aren't the easiest thing to test, views are even more laborious. Moving the logic from the view to the controller makes it easier to test.
* The view now has a single responsibility. One view is responsible for rendering no results, one for rendering a sequence of results. This makes it less fragile: you're less likely to break another part of the system when changing the no results page.
* As each view has an individual responsibility it becomes easier to find the view when it needs changing. As long as you use understandable file names.
* The view no longer has logic in it. Great for anyone that writes HTML/HAML but is scared of teh Rubies!

This small refactor has the effect of making your intentions more easily testable, your views clearers and your code more flexible and less fragile.


