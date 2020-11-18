---
layout: post
title:      "Transforming Variable Names in a Rails API"
date:       2020-11-18 18:13:58 +0000
permalink:  transforming_variable_names_in_a_rails_api
---


When working on projects for Flatiron School that have a Rails API backend and a JavaScript front end, I came across the challenge of handling variable names. It is convention in Ruby to name variables with snake case (like_this) and it is convention in Javascript to name variable in snake case (likeThis). When sending data back and forth, these differences need to be accounted for.


### This is how I handled transforming the variable names of my data using DRY code in the Rails API backend of my Flatiron projects


## From JavaScript to Rails

To handle camel case to snake case, I utiilized the [JSON API Serializer](https://github.com/jsonapi-serializer/jsonapi-serializer). 

This serializer comes with a macro that can transform variable names.

```
set_key_transform :camel_lower
```

Simply include this macro in the serializer file, and it will transform camel case variables from JavaScript into snake case. Keep in mind that this line of code needs to above other code in the serializer.


## From Rails to JavaScript 

To handle snake case to camel case, I utilized a `before_action` and the built-in Ruby method `deep_transform_keys!`. From the docs, this method will: "Destructively convert all keys by using the block operation." In this case, we want the block operation to add an underscore.

In my application_controller, I defined a helper method that I call `snake_case_params`.

```
  def snake_case_params
    request.parameters.deep_transform_keys!(&:underscore)
  end
```

I then used this method in a before_action on my application controller so that it applied to every controller in my application (as they all inherit from the application controller).

```
before_action :snake_case_params
```


