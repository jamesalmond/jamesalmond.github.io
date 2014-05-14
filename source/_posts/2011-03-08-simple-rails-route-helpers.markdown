---
title: Simple Rails route helpers
layout: post
---

Sometimes you end up with route configurations that repeat through your
routes.rb file. Taking inspiration from [Devise](https://github.com/plataformatec/devise) 
I added some little route helpers to the app I'm working on.

Before:

``` ruby
MyApp::Application.routes.draw do
  resources :pages do
    get :delete, :on => :member
    resources :images
    resources :tags
  end
  resources :people do
    get :delete, :on => :member
    resources :images
    resources :tags
  end
  resources :places do
    get :delete, :on => :member
    resources :images
    resources :tags
  end
end
```

If we add a simple helper to the routes:

``` ruby
module MyApp::RoutesHelpers
  def default_routes_for(*models)
    models.each do |model|
      resources model do
        get :delete, :on => :member
        resources :images
        resources :tags
      end
    end
  end
end

ActionDispatch::Routing::Mapper.send(:include, MyApp::RoutesHelpers)
```

Then we can use the following rotues:


``` ruby
MyApp::Application.routes.draw do
  default_routes_for(:pages, :people, :places)
end
```

A trivial example, maybe, but a simple little tidy up!
