---
title: Resourceful routing with Rails
layout: post
---

Resource routing in Rails is most often used for nesting related ActiveRecord objects, such as a post having many comments:

``` ruby
class Post < ActiveRecord::Base
  has_many :comments
end

class Comment < ActiveRecord::Base
  belongs_to :post
end

#routes
resources :posts do
  resources :comments
end
```
The resource method of the router hooks up the "RESTful" urls with a set of standard set of controller methods and HTTP verb:

{% highlight text%}
    post_comments GET    /posts/:post_id/comments(.:format)          {:action=>"index", :controller=>"comments"}
                  POST   /posts/:post_id/comments(.:format)          {:action=>"create", :controller=>"comments"}
 new_post_comment GET    /posts/:post_id/comments/new(.:format)      {:action=>"new", :controller=>"comments"}
edit_post_comment GET    /posts/:post_id/comments/:id/edit(.:format) {:action=>"edit", :controller=>"comments"}
     post_comment GET    /posts/:post_id/comments/:id(.:format)      {:action=>"show", :controller=>"comments"}
                  PUT    /posts/:post_id/comments/:id(.:format)      {:action=>"update", :controller=>"comments"}
                  DELETE /posts/:post_id/comments/:id(.:format)      {:action=>"destroy", :controller=>"comments"}
            posts GET    /posts(.:format)                            {:action=>"index", :controller=>"posts"}
                  POST   /posts(.:format)                            {:action=>"create", :controller=>"posts"}
         new_post GET    /posts/new(.:format)                        {:action=>"new", :controller=>"posts"}
        edit_post GET    /posts/:id/edit(.:format)                   {:action=>"edit", :controller=>"posts"}
             post GET    /posts/:id(.:format)                        {:action=>"show", :controller=>"posts"}
                  PUT    /posts/:id(.:format)                        {:action=>"update", :controller=>"posts"}
                  DELETE /posts/:id(.:format)                        {:action=>"destroy", :controller=>"posts"}

```

These structures are map nice and clearly to the default CRUD actions of your AR objects and how to use this functionality is covered in other tutorials.

Once you step outside of the default CRUD actions it's not immediately clear how to add further functionality to your routes structure. One of the options is to add extra methods to the resource's collection or methods as a member of each object. Let's take the concept of setting the status of a blog post from "draft" to "published", with the status of the post being stored as a string in the post object. We'll need a page to view the current status and a URL to post to that updates the status.


``` ruby
resources :posts do
  member do 
    get :status
    put :update_status
  end
end

```

which gives us the following URLs:

{% highlight text%}
       post_status GET    /posts/:post_id/status(.:format)        {:action=>"status", :controller=>"posts"}
post_update_status PUT    /posts/:post_id/update_status(.:format) {:action=>"update_status", :controller=>"posts"}
             posts GET    /posts(.:format)                        {:action=>"index", :controller=>"posts"}
                   POST   /posts(.:format)                        {:action=>"create", :controller=>"posts"}
          new_post GET    /posts/new(.:format)                    {:action=>"new", :controller=>"posts"}
         edit_post GET    /posts/:id/edit(.:format)               {:action=>"edit", :controller=>"posts"}
              post GET    /posts/:id(.:format)                    {:action=>"show", :controller=>"posts"}
                   PUT    /posts/:id(.:format)                    {:action=>"update", :controller=>"posts"}
                   DELETE /posts/:id(.:format)                    {:action=>"destroy", :controller=>"posts"}

```

In this example I've chosen not to use the Post's update URL to set the status. By using a separate controller method we can also use a separate model method (rather than the default 'update_attributes'). This means that when we set a new status we can handle any business logic around a status change within that specific method and not within a pile of spaghetti callback methods. I'll save the 'ActiveRecord spaghetti callback methods are evil' post for another day!

The following code sets up the controller actions that handle the new routes:

``` ruby
class PostsController < ApplicationController

  # with corresponding posts/status.html.haml
  def status
    @post = Post.find(params[:id])
  end
  
  def update_status
    post = Post.find(params[:id])
    post.update_status(params[:new_status])
    redirect_to post_path(post)
  end
end

```

Hooray! Minus some of the AR implementation, we've now got status updates.
Whilst this example is fairly simple, when we start adding more functionality to posts both our controller and routes start to get a bit clumsy and bloated. Let's add some search functionality to our posts. Iwant a page that has the search box on that posts its options to the app, the app will then convert those options into some URL parameters that we can access as a get method (allowing our users to bookmark a search and refresh without issue):


``` ruby
resources :posts do
  member do 
    get  :status
    put  :update_status
  end
  collection do
    get  :search
    post  :new_search
    get  :show_search
  end
end

```
{% highlight text%}
       status_post GET    /posts/:id/status(.:format)        {:action=>"status", :controller=>"posts"}
update_status_post PUT    /posts/:id/update_status(.:format) {:action=>"update_status", :controller=>"posts"}
      search_posts GET    /posts/search(.:format)            {:action=>"search", :controller=>"posts"}
  new_search_posts POST   /posts/new_search(.:format)        {:action=>"new_search", :controller=>"posts"}
 show_search_posts GET    /posts/show_search(.:format)       {:action=>"show_search", :controller=>"posts"}
             posts GET    /posts(.:format)                   {:action=>"index", :controller=>"posts"}
                   POST   /posts(.:format)                   {:action=>"create", :controller=>"posts"}
          new_post GET    /posts/new(.:format)               {:action=>"new", :controller=>"posts"}
         edit_post GET    /posts/:id/edit(.:format)          {:action=>"edit", :controller=>"posts"}
              post GET    /posts/:id(.:format)               {:action=>"show", :controller=>"posts"}
                   PUT    /posts/:id(.:format)               {:action=>"update", :controller=>"posts"}
                   DELETE /posts/:id(.:format)               {:action=>"destroy", :controller=>"posts"}
```

And our Posts controller also needs some more methods:

``` ruby
class PostsController < ApplicationController

  # other Post CRUD methods
  
  # with corresponding posts/status.html.haml
  def status
    @post = Post.find(params[:id])
    
  end
  def update_status
    post = Post.find(params[:id])
    post.update_status(params[:new_status])
    redirect_to post_path(post)
  end

  def search
    # renders the search form
  end
  def new_search
    redirect_to posts_show_search_path Post.process_search_params(params[:search])
  end
  def show_search
    @posts = Post.search_from_params(params)
  end
end
```

We now have search! But there's also a few things that make this example less than ideal. The more functionality we add to Posts the more routes we have to define manually and the more methods we have in our Posts controller. This results in controllers that are hard to navigate due to size, routes that are defined by how the developer decided to name them that day (e.g. new_search could be create_search etc.) and the Posts controller is now responsible for much more than it should ! So, how can we fix this? We need some uniform system of representing concepts in our system.

## Re-thinking resources

Resources in the Rails routing world are often mapped directly to persistence or ActiveRecord objects. Let's redefine the use of resource to mean a individual 'concept' within the system, something that has its own behaviour or affects another part of the system. Our persistence objects still map to resources as before but other concepts such as actions on those objects or specific behaviours of those objects can also map to resources. Get back to the example will probably make it a bit clearer.
Within our example a Post still remains a resource as it was previously. We can use the methods and routes created by the definition of Post as a resource to view, create and alter the definition of a post. For example, changing the basic attributes such as title or body.
But, given our new definition of resource we can now start to consider other things resources. As the status attribute doesn't really define the post but defines the state the post is in it's a good candidate for having the functionality to change it being moved from the Posts controller. This allows us to deal with the implications of changing the state separately from the mundane actions of simply updating attributes; a division of responsibility from the previous example. We now have the methods we need to show the current status (GET show) and update the status (PUT update).
We can also consider the search as a resource. Now, this one maps a little more closely to the idea of a resource as we previously defined, but doesn't map directly to a persistence model. We have and action that shows the form (GET new), we have a method that accepts posted parameters (POST create) and we have a method that displays a specific search given an identifier (the search term) (GET show).
We can now tidy up our routes a bit:

``` ruby
resources :posts do
  resource :status, :controller => :posts_status_controller
  resources :searches, :controller => :posts_searches_controller
end
```

{% highlight text%}
     post_status POST   /posts/:post_id/status(.:format)            {:action=>"create", :controller=>"posts_status_controller"}
 new_post_status GET    /posts/:post_id/status/new(.:format)        {:action=>"new", :controller=>"posts_status_controller"}
edit_post_status GET    /posts/:post_id/status/edit(.:format)       {:action=>"edit", :controller=>"posts_status_controller"}
                 GET    /posts/:post_id/status(.:format)            {:action=>"show", :controller=>"posts_status_controller"}
                 PUT    /posts/:post_id/status(.:format)            {:action=>"update", :controller=>"posts_status_controller"}
                 DELETE /posts/:post_id/status(.:format)            {:action=>"destroy", :controller=>"posts_status_controller"}
   post_searches GET    /posts/:post_id/searches(.:format)          {:action=>"index", :controller=>"posts_searches_controller"}
                 POST   /posts/:post_id/searches(.:format)          {:action=>"create", :controller=>"posts_searches_controller"}
 new_post_search GET    /posts/:post_id/searches/new(.:format)      {:action=>"new", :controller=>"posts_searches_controller"}
edit_post_search GET    /posts/:post_id/searches/:id/edit(.:format) {:action=>"edit", :controller=>"posts_searches_controller"}
     post_search GET    /posts/:post_id/searches/:id(.:format)      {:action=>"show", :controller=>"posts_searches_controller"}
                 PUT    /posts/:post_id/searches/:id(.:format)      {:action=>"update", :controller=>"posts_searches_controller"}
                 DELETE /posts/:post_id/searches/:id(.:format)      {:action=>"destroy", :controller=>"posts_searches_controller"}
           posts GET    /posts(.:format)                            {:action=>"index", :controller=>"posts"}
                 POST   /posts(.:format)                            {:action=>"create", :controller=>"posts"}
        new_post GET    /posts/new(.:format)                        {:action=>"new", :controller=>"posts"}
       edit_post GET    /posts/:id/edit(.:format)                   {:action=>"edit", :controller=>"posts"}
            post GET    /posts/:id(.:format)                        {:action=>"show", :controller=>"posts"}
                 PUT    /posts/:id(.:format)                        {:action=>"update", :controller=>"posts"}
                 DELETE /posts/:id(.:format)                        {:action=>"destroy", :controller=>"posts"}
```

There's two things to note with this example. Firstly the status is defined as a singular resource as we don't have a collection of statuses, so we don't need the URLs that have the unique IDs.
It's routes like /posts/1/status/edit or PUTting to /posts/1/status to update a post that seem to make perfect sense to me.
Secondly, we've specified the names of the controllers. If we only had one search in the system and one status concept we wouldn't have to specify the names of our controllers and we could just name them to match the name of the resource. But while we're at this dividing responsibility game, let's make it clear what the responsibility of the controller is by naming it explicitly. We can also move the actions of from the previous single controller into the new controllers.


``` ruby
class PostsStatusController < ApplicationController
  def show
    @post = Post.find(params[:id])
  end
  def update
    post = Post.find(params[:id])
    post.update_status(params[:new_status])
    redirect_to post_path(post)
  end
end

class PostsSearchController < ApplicationController
  def new
    # renders the search form
  end
  def create
    redirect_to posts_show_search_path Post.process_search_params(params[:search])
  end
  def show
    @posts = Post.search_from_params(params)
  end
end
```

Starting to look a little cleaner, right? And also, we haven't defined any methods outside of the normal (create, show, edit, update, delete) which makes all our controllers consistent and predictable. We've also got some nice routes that look reasonably RESTful (I've tried to avoid using the term RESTful as I don't want to get into the semantics of REST, but I'm feeling reckless today!). We can now also move our views out of the posts folder into the separate posts_searches and posts_status folders; much less clutter.
Hopefully this (rather long!) post has given you an idea of how to structure your controllers and routes in a manageable, consistent and predictable way.

## Summary (TL;DR)

Resource routing in Rails can be used outside of persistence or AR objects to provide predictable, clean and RESTful routes for your application's behaviour. This practice encourages the separation of concerns by creating new controllers, methods and views which are responsible for specific behaviours of the system making the code easier to understand, navigate and test. 

## Another example

``` ruby
resources :bookings do
  resource :cancellation, :controller => :booking_cancellation_controller
end

# Cancellation page: /bookings/1/cancellation/new (BookingsCancellationController GET new)
# Process the cancellation: /bookings/1/cancellation (BookingsCancellationController POST create)
```

