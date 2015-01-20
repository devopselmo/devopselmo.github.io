---
layout: post
title:      Use rspec-puppet To Organize Your Work
date:       2013-08-09 12:35:24 -0700
tags: code testing puppet ruby
---

Test-Driven Development for your Puppet code can feel intimidating if you're just starting out. I had been writing Puppet for years before I answered the clarion call of [rspec-puppet](http://rspec-puppet.com). If you keep at it, though, I can promise you it's worth it. I've found that even the simplest manifests can often benefit from testing, if for no other reason because it can help you organize your thoughts as you're getting started.

Recently I needed to puppetize an old internal app. Needing to braindump what I knew of the app's existing system requirements, I decided to use test-writing as my tasklist right from the jump.

<!-- Before You Start
===
If you're going to follow along, we'll assume you have already bootstrapped [rspec-puppet](http://rspec-puppet.com) for your local development environment. If not, go ahead and grab [the accompanying git repo for this post](https://github.com/justinclayton/devopselmo_examplecode_1.git). -->

Let's Do It
===

The class I would be creating would be `role::xyz_app_server`, so I went into our role module repo and created some files:

{% highlight bash %}
touch manifests/xyz_app_server.pp
touch spec/classes/xyz_app_server_spec.rb
{% endhighlight %}

Then, before writing any Puppet code, I started my braindump in `xyz_app_server_spec.rb`:

{% highlight ruby %}
# spec/classes/xyz_app_server_spec.rb
require 'spec_helper'

describe 'role::xyz_app_server', :type => :class do
  it 'should create the app user'
  it 'should give the app user sudo access'
  it 'should install some sort of java'
  it 'should mount the /data nfs share'
  it 'should install a bunch of language packs and fonts, i guess'
  it 'should ulimit for open files 90000'
  it 'might need to do some MTA config'
  it 'should have that special imagemagick im going to have to build into an rpm'
  it 'needs epel but i dont know why yet'
end
{% endhighlight %}

Now when we run `rake spec`, we should see this:

{% highlight bash %}
rake spec
...
  should create the app user (PENDING: Not yet implemented)
  should give the app user sudo access (PENDING: Not yet implemented)
  should install some sort of java (PENDING: Not yet implemented)
  should mount the /data nfs share (PENDING: Not yet implemented)
  should install a bunch of language packs and fonts, i guess (PENDING: Not yet implemented)
  should ulimit for open files 90000 (PENDING: Not yet implemented)
  might need to do some MTA config (PENDING: Not yet implemented)
  should have that special imagemagick im going to have to build into an rpm (PENDING: Not yet implemented)
  needs epel but i dont know why yet (PENDING: Not yet implemented)
...
Finished in 0.00102 seconds (files took 0.5452 seconds to load)
9 examples, 0 failures, 9 pending
{% endhighlight %}

This is taking advantage of a feature of rspec where not creating the `it` block at all puts the tests into a `pending` state. This can be useful, as just creating an empty `it` block makes it pass, and that doesn't help you know if you've finished writing all your tests yet.

Now you can start peeling these off one at a time:

{% highlight ruby %}
# spec/classes/xyz_app_server_spec.rb
require 'spec_helper'

describe 'role::xyz_app_server', :type => :class do
  it 'should create the app user' do
    should contain_user('app').with({
      :uid    => '555',
      :groups => ['wheel'],
    })
  end
  it 'should give the app user sudo access'
  it 'should install some sort of java'
  it 'should mount the /data nfs share'
  it 'should install a bunch of language packs and fonts, i guess'
  it 'should ulimit for open files 90000'
  it 'might need to do some MTA config'
  it 'should have that special imagemagick im going to have to build into an rpm'
  it 'needs epel but i dont know why yet'
end
{% endhighlight %}

{% highlight puppet %}
# manifests/xyz_app_server.pp
class role::xyz_app_server {
  user { 'app':
    uid    => '555',
    groups => ['wheel'],
  }
}
{% endhighlight %}

{% highlight bash %}
$ rake spec
...
  should create the app user
  should give the app user sudo access (PENDING: Not yet implemented)
  should install some sort of java (PENDING: Not yet implemented)
  should mount the /data nfs share (PENDING: Not yet implemented)
  should install a bunch of language packs and fonts, i guess (PENDING: Not yet implemented)
  should ulimit for open files 90000 (PENDING: Not yet implemented)
  might need to do some MTA config (PENDING: Not yet implemented)
  should have that special imagemagick im going to have to build into an rpm (PENDING: Not yet implemented)
  needs epel but i dont know why yet (PENDING: Not yet implemented)
...
Finished in 2.31 seconds (files took 0.5161 seconds to load)
9 examples, 0 failures, 8 pending
{% endhighlight %}

It passed! One down, 8 to go.

Conclusion
===
Part of deving all the ops is applying the same sort of best practices that software devs have been formulating for several years now. Testing is a great example of this. Don't fight it; it really does make things easier in the long run!