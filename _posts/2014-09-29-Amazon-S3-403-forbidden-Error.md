---
layout: post
title: Amazon S3 403 forbidden Error
categories:
- Web Development
- Django
- AWS
comments: true
---


On the startup I work we use many Amazon Web Services for our website infrastructure.
One of them is Amazon's Cloud Storage or [Amazon S3](http://aws.amazon.com/s3/) as they have named it.
With S3 Amazon provides storage access through standarized web service interfaces (e.g REST).<br/>
So that's about S3, more information you can find on their [site](http://aws.amazon.com/s3/).

To use Amazon S3 service with our django setup we have used the excellent
[django-storages](http://django-storages.readthedocs.org/en/latest/).

After a couple of months with no issues at all, I tried to upload a file to S3 via the admin interface of django when the following
error occured:

<!--more-->

{% highlight python %}
File "../lib/python2.7/site-packages/boto/s3/bucket.py", line 169, in get_key
    key, resp = self._get_key_internal(key_name, headers, query_args_l)
  File "../lib/python2.7/site-packages/boto/s3/bucket.py", line 211, in _get_key_internal
    response.status, response.reason, '')
S3ResponseError: S3ResponseError: 403 Forbidden
{% endhighlight %}

First, I tried to check if something has changed or was wrong with our AWS credentials.
I checked if we have read/write access to the corresponding S3 bucket:
{% highlight python %}
>>> import boto
>>> s3 = boto.connect_s3('<access_key>', '<secret_key>')
>>> bucket = s3.lookup('s3_bucket_name')
>>> key = bucket.new_key('testkey')
>>> key.set_contents_from_string('This is a read/write test')
>>> key.delete()
{% endhighlight %}

All went as it should. So our credentials were correct.

After a couple of hours of
trial and error <img class="right" src="{{ site.baseurl }}public/images/writing_process.gif">
I found the cause:
Our server's misconfigured TIME. It was wrong, and AWS respond with a <i>403 FORBIDDEN</i>.

In order to update the time to the correct value I just ran:
{% highlight bash %}
sudo ntpdate 0.pool.ntp.org
{% endhighlight %}
After that I have checked that ntp is set to auto update by set /etc/cron.daily/ntpdate to be executable:
{% highlight bash %}
sudo chmod 755 /etc/cron.daily/ntpdate
{% endhighlight %}

So that was all!
<img class="center" src="{{ site.baseurl }}public/images/code-35.gif">
