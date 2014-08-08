---
layout: post
title:  "Ember Data with nested REST API"
date:   2014-08-08 10:01:54
tags: ember rest-api resources
---

So it all started with a small project with a few friends to help out one of the local hospitals in Dublin. We chose [Python](https://www.python.org/) and [Flask](http://flask.pocoo.org/) to write the API and the UI to begin with. 1 week in we've decided to separate out the API from the frontend - a very sane decision. We debated for a few hours and took [Ember.js](http://emberjs.com/) as our frontend piece. Me who was learning Python already with this, thought: great, another thing under my belt when we finish. Little did I know about the melting pot of joy and frustration Ember.js provides...

# The problem -  REST API with nested resource

We included Emebr.Data from the beginning to connect to our API and hopefully reduce the code writing efforts when it comes to CRUD operations. And this is when the frustration really begun. I was aiming to get a (I would have thought) simple thing done - use and change the resources from a nested API resource:

{% highlight javascript %}
GET /appointments/1
{
  "appointments": {
    "id": 1,
    "tags": [1],
    ...
  }
}

GET /appointments/1/tags
{
  "tags":[
    {
      "id": 1,
      "name": "Important"
    },
    {
      "id": 2,
      "name": "Not so important"
    },
    ...
  ]
}

GET /appointments/1/tags/1
{
  "tags": {
    "id": 1,
    "name": "Important"
  }
}
{% endhighlight %}

My goal was to:

* List appointments
* List tags associated to an appointment
* Tag (POST /appointments/1/tags) and untag (DELETE /appointments/1/tags/1) appointments easily


{% highlight javascript linenos %}
App.Tag = Ember.Data.extend({
  id: DS.attr(),
  name: DS.attr()
});

App.Appointment = Ember.Data.extend({
  id: DS.attr(),
  tags: DS.hasMany('Tag'),
  ...
});

{% endhighlight %}

*I presumed* that Ember Data will handle nested resources in the background - given that it deals with them [so easily in the routes](http://emberjs.com/guides/routing/defining-your-routes/#toc_resources). Well, 3 days of endless Ember guide, Stackoverflow, Ember forum searches I was not getting closer to the solution. I've tried everything under the sun to get it working, embedded the resources in the appointments API response, side loaded them (which kind of worked) but then my relations got all messed up and handling them became problematic in the Ember app. In the end found a few crumbs of information and solutions and thought I put this together to show how I've solved it.

# Use links to load the relations

You can load the nested resources by modifying slightly your API responses and provide a list of IDs and a links hash at the root of the response:

{% highlight javascript %}
GET /appointments/1
{
  "appointments": {
    "id": 1,
    "tag_ids": [1],
    ...
  },
  "links": {
    "tags": "appointments/1/tags",
    ...
  }
}

{% endhighlight %}

Ember when seeing this combo will go and fetch that data while working on the main promise of the object. You can include multiple links and list of ids, it will work just fine with the model setup above.

   NOTE `tag_ids` won't be present on the ember object, only `tags`!

So that's it, first bit is done with loading, no problems there. All there and dandy, with a simple solution, <s>almost impossible to</s> you can find on the guides. On to the next item: modify those appointment tags.

# RESTAdapter to the rescue

After another day of trial-and-error with resources, model definitions and various API changes reverted everything and decided to write an intermerdiate model to handle this with a custom [RESTAdapeter](http://emberjs.com/guides/models/the-rest-adapter/). But first I needed to define that model:

{% highlight javascript linenos %}
App.AppointmentTag = DS.Model.extend({
    appointment_id: DS.attr(),
    tag_id: DS.attr()
});
{% endhighlight %}

And the next bit will be surpisingly easy. I envisioned (after thinking all my googling skills left me) this will be horrible to write and test. I needed two operations on the Adapter: `createRecord` to add a tag to the appointment and `destroyRecord` for untagging.

{% highlight javascript linenos %}
App.AppointmentTagAdapter = App.ApplicationAdapter.extend({
  createRecord: function(store, type, record){
    var data = {id: record.get('tag_id')};
    return this.ajax(
      "%@/%@".fmt(this.get('host'), this.baseUri(record.get('appointment_id'))),
      "POST",
      { data: {tag: data }}
    );
  },
  destroyRecord: function(store, type, record){
    var url = "%@/%@/%@".fmt(
      this.get('host'),
      this.baseUri(record.get('appointment_id')),
      record.get('tag_id')
    );
    return this.ajax(url, "DELETE", { data: {}});
  },
  baseUri: function(apt_id) {
    return "appointments/%@/tags".fmt(apt_id);
  }
});
{% endhighlight %}

And here comes the joy, as with very little extra code I managed to achieve what I was originally aiming for, all I needed is a small change on the controller actions to handle these, example for adding a tag:

{% highlight javascript linenos %}
App.AppointmentController = Ember.ObjectController.extend({
  ...
  actions: {
    tag: function(tag_id) {
      var self = this;
      var new_tag = self.store.getById('tag', parseInt(tag_id));
      var apt_tag = self.store.createRecord('appointment.tag', {
        tag_id: tag_id,
        appointment_id: self.get('model').get('id')
      });
      apt_tag.save(); // Calls adapter.createRecord
    },
    ...
  }
});
{% endhighlight %}

All-in-all have to say I was happy enough with it, although in the middle of all the frustration had to remind myself everytime: it's ok, Ember.Data is in Beta, things are not working fully yet, and the docs won't be up-to-date. If you accept that, it's easier to manage the rage:)


