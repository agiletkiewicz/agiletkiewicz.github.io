---
layout: post
title:      "Object-Oriented JavaScript: using an adapter class"
date:       2020-10-19 16:41:24 +0000
permalink:  object-oriented_javascript_using_an_adapter_class
---


*This post orignally appeared on my [Medium Blog](https://medium.com/agi-coding-bootcamp/object-oriented-javascript-using-an-adapter-class-for-fetch-requests-and-responses-72c28fa6c7a1) on Sept 21, 2020.*


![](https://miro.medium.com/max/700/1*L98UqS0AM3sJq_Mt5Yk7sw.jpeg)


An important part of my required [JavaScript project](https://github.com/agiletkiewicz/virtual-swap-frontend) for Flatiron School was using objected-oriented Javascript to encapsulate related data and behavior. Fortunately, ES6 has introduced a new syntax for declaring a class using the class keyword. Using this convenient syntactic sugar, I could focus on how best to apply object-programming in my project, a single-page application with an HTML, CSS, and JavaScript frontend with a Rails API backend.

A quick snippet from MDN web docs on what object-oriented programming is:

> The basic idea of OOP is that we use objects to model real world things that we want to represent inside our programs, and/or provide a simple way to access functionality that would otherwise be hard or impossible to make use of.

Coming from the world of Ruby on Rails I was very used to thinking about using OOP to model real world ideas, like goals and tasks. In my JavaScript project I challenged myself to think more about the latter half of MDN’s definition: what functionality would be useful to encapsulate inside of an object package, making it easier to access in my single-page application?

**One of the ways that I did that was to leverage an Adapter class to handle all of the fetch requests to my Rails API backend. **

[This video](https://www.youtube.com/watch?v=seCqoRBCq9U&feature=youtu.be) got me started.

**Benefits of using an Adapter class:**
* When I deploy my app, or when the URL changes for any reason, I only have to change it in one place
* The code in my index.js file is DRY, making it easier to read and maintain

**Declaring the class:**


```
class Adapter {
    constructor(path) {
        this.path = "http://localhost:3000/api/v1" + path
    }
getRequest() {
        return fetch(this.path).then(res => res.json())
    }
postRequest(bodyData) {
        
        const configObj = {
            method: "POST",
            headers: {"Content-Type": "application/json"},
            body: JSON.stringify(bodyData)
        }
return fetch(this.path, configObj).then(res => res.json())
    }
deleteRequest() {
        
        const configObj = {
            method: "DELETE",
            headers: {"Content-Type": "application/json"},
        }
return fetch(this.path, configObj).then(res => res.json())
    }
}
```

**Creating a new instance of the class:**

Here is one example of where I use the Adapter class to create a new item.

```
function createItemFetch(title, size, notes, image_url) {
  const bodyData = {title, size, notes, image_url, user_id: User._current.id}
new Adapter("/items").postRequest(bodyData)
  .then(event => {
    if (event.errors) {
      const error = document.querySelector("#create-item-error");
      error.innerText = ""
      for (const element of event.errors) {
        error.innerText += element;
        const br = document.createElement('br');
        error.appendChild(br);
      }
    } else {
      const item = new ItemFromForm(event.id, event);
      document.querySelector("#create-item-form").reset();
      item.renderItemCardFromDb();
    }
  })
  .catch(error => {
    console.error(error);
    document.querySelector("#create-item-error").innerText = "Something went wrong. Please try again.";
  })
}
```

In creating a new item using a fetch request to my Rails backend, I know I have to handle a few different responses:
* A server error that prevents it from fulfilling the request
* An error in saving the item because of a validation implemented to prevent bad data from being persisted to the database
* A successful save to the database, therefore needing to update the DOM to show the new item

By using the Adapter class (and also an Item class) I was able to focus more on handling these different responses and less the fetch and subsequent DOM manipulation which is repeated elsewhere. I encapsulated the fetch request and response in the Adapter class, and the rendering of a new item in the Item class.
