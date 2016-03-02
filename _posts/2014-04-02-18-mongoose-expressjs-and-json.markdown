---
layout: post
title: How I solved my first server crash @ lab.sokratik.com
image: expressmongoose.jpg
date:   2014-02-18 12:00:00
categories: posts
---

Our first beta product [lab.sokratik.com] (http://lab.sokratik.com) uses two nodejs servers, one of them is a binaryjs
socket which is a wrapper around amazon s3, the other powers all the html you see on your browser. This is an expressjs
application sitting behind nginx and talking to browser and mongolab. The server architecture is based on [mean]
(http://mean.io) . If you are writing a new application, I would strongly recommend building your code on top of mean.
Having spent considerable time writing javascript, I zeroed in on angularjs for my frontend framework. Without going into
merits of why chose this technology over other, I can say that any angularjs application talks in the language of json.
My experience with server crashes also dictated that I use [forever] (https://github.com/nodejitsu/forever).
When a server goes down even more important than finding
why it went down is to bring the server up.

Essentially the flow of data in our architecture looks like this

1. Browser talks to expressjs server
2. server gets data from mongolab via mongodb
3. browser manipulates the json and sends it to expressjs
4. expressjs sends it back to mongoose.

Since, my application has a few nested json documents. communication from browser to webserver to db server involves a lot
of JSON.parse and JSON.stringify.

A typical document for our application looks like
{% highlight javascript %}
    {property1 : String
     property2: [String],
     property3: [{}]}
{% endhighlight %}

The pain-point for me was **property3**. Since,in  our application lots of asynchronous manipulation in the browser.  A lot of
times(very rarely) **garbage data(null)** creeps in.So we might have a object like

{% highlight javascript %}

    {property1 : 'himangshu'
     property2: ['himangshu','aniket','arnav'],
     property3: [{college:'IITKGP'},{college:'IITB'},null,{college:'IITKGP'}]}
{% endhighlight %}

This works well for the loop between server and browser, data gets corrupted but server is safe.
 But the moment we hit mongoose, our
expressjs server breaks down and node-forever does not even try to revive the server.
 As a self respecting start up techie, I have not tried to fix the root cause but hacked my
way out of trouble. All you have to do is to avoid passing nulls where you expect a javascript object from going to json.
The savior code is given below. It is based on underscorejs.

{% highlight javascript %}

var sanitizeRequestBody = function (object) {
    var property3 = _.without(object.property3, null);
    return _.chain(object).omit('__v').extend({property3:property3}).value();
};

{% endhighlight %}

The  **omit('__v')** is kind of compulsory for mongoosejs and expressjs, I learnt it the hard way and will write about it later.


