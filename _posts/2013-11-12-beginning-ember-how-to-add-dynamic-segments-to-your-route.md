---
layout: post
title: "Beginning Ember: How to add dynamic segments to your route"
date: 2013-11-12 20:00:00 -0700
---

[Ember.js](http://emberjs.com) is an incredibly powerful JavaScript framework for creating ambitious web applications. Unfortunately, it can be quite daunting to learn. Over the past year, I have had my fair share of struggles while picking up this framework, and I am going to publish a series of articles on common tasks you will need to do as a beginner (and as an experienced ember dev). This first post details how to add a dynamic segment to a route.

A dynamic segment is a section of the path for a route which will change based on the content of a page. An example of this could be a user profile page. You may have a route with a path of /profile/rickharrison. The first thing you need to do is add the following segment to your route definition:

```javascript
App.Router.map(function() {
    this.resource('profile', { path: '/profile/:username' });
});
```

Dynamic segments are made up of a `:` followed by an identifier.

In order to show the correct content for the specified user, you will need to use the `model` hook of the route. A params object which contains information from the URL, will be the first parameter passed to the hook.

```javascript
App.ProfileRoute = Ember.Route.extend({
    model: function(params, transition) {
        return this.get('store').find('user', params.username);
    }
});
```

The above example uses Ember Data. To demonstrate the concept without Ember Data, see the simple snippet below:

```javascript
App.ProfileRoute = Ember.Route.extend({
    model: function(params, transition) {
        return { username: params.username };
    }
});
```

In order to correctly construct the url, you will need to implement the serialize hook. Your model will be passed in, and you are expected to return an object with the dynamic segment as the key with its value.

```javascript
App.ProfileRoute = Ember.Route.extend({
    model: function(params, transition) {
        return { username: params.username };
    },

    serialize: function(model) {
        return { username: model.get('username'); }
    }
});
```

After your route is fully configured, you can start linking to it from your templates. Just include a simple `link-to` with information for the segment. The first parameter is the route and the second is the value for the segment.

{% raw %}
```javascript
{{#link-to 'profile' 'rickharrison'}}View rickharrison's profile{{/link-to}}
```
{% endraw %}

A complete copy of the sample code can be viewed at [http://jsbin.com/AGetubu/1/edit](http://jsbin.com/AGetubu/1/edit)
