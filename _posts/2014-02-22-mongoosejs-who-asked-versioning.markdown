---
layout: post
title: Dealing with unwanted  versioning in mongoosejs
image: versioning.png
date:   2014-02-22 12:00:00
---
<p class="intro"> <a href="http://mongodb.org"><span class="dropcap">M</span>ongodb</a>  and <a href="http://expressjs.com/">expressjs</a> connected via 
<a href="http://mongoosejs.com/">mongoosejs</a>
is the default choice of stack for  many a developers who are building applications on nodejs. I went with the mob and I
have nothing to regret. Having spent a significant part of my life writing untestable jquery code, I knew I did not want
to deal with it any more.I went with the new kid in the block angularjs.</p>

Our [application](https://github.com/himangshuj/sokratik-lab) captures many actions happening on the browser and asynchronously posts them
to the server. Once in a while, the responses from these requests do not come in chronological order. One fine day , while
debugging I found that my document was being reverted to an earlier state seamlessly. This had never happened in my dev
setup but damn the production beta servers. On debugging further I found that mongoose js ends a magical variable **__v**
to my documents. Now this variable is handled magically behind the scenes.

{% highlight javascript %}
    {property1: 'val1',__v: 0}// request sent at t= to   and replied at t=t1`
    {property1: 'val2',__v: 0}// request sent at t= t1   and replied at t=t0`
    {property1: 'val3',__v: 0} //request sent at t=t2 but not saved in the server.
{% endhighlight %}

Thank god to the network tab, I could isolate that __v was the culprit. Here is the fix that works

{% highlight javascript %}
var sanitizeRequestBody = function (object) {
    return _.omit(object,'__v');
};

//before saving
var objToSave = _.extend(originalobj,sanitizeRequestBody(req.body));
{% endhighlight %}
