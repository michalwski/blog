# MongooseIM 2.1.0beta1 what happens when you give your team freedom

My name is Michał Piotrowski, and I'm MongooseIM's platform tech lead.
With this blog post I'd like to announce and describe our latest release.
MongooseIM 2.1.0beta1 is one of the strongest release I was ever involved.
It comes with new features on the XMPP level and also contains much more internal, very technical changes.

For this release we created a [roadmap](http://mongooseim.readthedocs.io/en/2.1.0beta1/Roadmap) and gave the team the freedom to implement it.
The end result is not only implementation of most of the items from road map but also many corresponding changes resulting in improved code quality or performance.

## Push Notifications

### For mobile devices

These days most of instant messaging applications are mobile first or mobile only.
To deliver great product for mobile devices you have to be able to send push notifications to offline users in one way or another.
This was possible so far with the [mod_http_notification](http://mongooseim.readthedocs.io/en/2.1.0beta1/modules/mod_http_notification/) module.
The problem with this approach is that you need to create your own HTTP service talking to `APNS` or `FCM`.


In this version we added support for push notifications as described in [XEP-0357: Push Notifications](https://xmpp.org/extensions/xep-0357.html).
Also we new service to our platform [MongoosePush](https://github.com/esl/MongoosePush) which can be used to communicate with both `APNS` and `FCM`.
Configuration of all the required components to enable push notifications may not be trivial.
That's why we create a detailed [tutorial](http://mongooseim.readthedocs.io/en/latest/user-guide/Push-notifications/) on this topic.

Thanks to this functionality you can enable push notifications in your application based on a standard way.
There is no need to create ad-hoc, custom solutions.

### For other services

In many cases you may want to get some of the events from MongooseIM in your other services.
For example to apply some business logic or implement a custom archive.
Again, this was possible with MongooseIM so far by aforementioned `mod_http_notification` whit the same limitations as described above.

MongooseIM 2.1.0beta1 comes with one another, more flexible way to integrate with other services in the whole infrastructure.
Thanks to [mod_aws_sns](http://mongooseim.readthedocs.io/en/2.1.0beta1/modules/mod_aws_sns/) you can configure MongooseIM to push certain events to your Amazon SNS queues.

Now only the sky is the limit in applying any business logic you may want on all the delivered events.

## Full text search for MAM

Most of modern instant messaging applications needs full text search on the messages from archive.
While not officially described in any XEP it's possible to add custom queries to MAM requests, for instant full test search.

This is exactly what we did in MongooseIM 2.1.0beta1.
With this version you can use custom field `full-text-search` in your MAM query to get all messages matching provided pattern.
Complete example can be found in [the PR](https://github.com/esl/MongooseIM/pull/1136) implementing this functionality.
This has its limitation:
* works only with MySQL, Postgres or Riak backends.
* MySQL and Postgres implementation is not the most efficient as it's based on SQL's full text search.

Please take a look at [mod_mam](http://mongooseim.readthedocs.io/en/2.1.0beta1/modules/mod_mam/) documentation to see how to enable this feature.

## Continuous load testing

Developing MongooseIM platform we are always focused on quality of our product.
That's why we have integrate MongooseIM with services like [travis-ci](https://travis-ci.org/esl/MongooseIM),
[coveralls.io](https://coveralls.io/github/esl/MongooseIM) and [gadget-ci](http://gadget.inakalabs.com) long time ago.
Thanks to these, every single change is properly tested.
Code quality and test coverage are also monitored to be sure we deliver the best possible quality.

For the past few months we've been working on another service which helps us deliver better products.
[Tide](http://tide.erlang-solutions.com) is our own service integrated with MongooseIM build pipeline.
Thanks to this service we are able to continuously load test our pull requests to see what's the performance impact of prosed changes.

## XMPP pipelining

The usual way of connecting to a XMPP server is based on a request response patter.
Clients sends a stanza, waits for the reply from the server and then sends another stanza.
This is required when the client connects to a server for the first time.
Many things about the server capabilities is not known to the client so it needs to wait for the response to proceed.
In case of a communication with well know server, where the clients know about all the features provided by it, almost all initial requests can be combined in on request.

This improves connection or re-connection times significantly.
Following diagrams shows re-connection time (with stream resumption) with and without pipelining.

TODO get diagram from Konrad

As you can see the reconnection time with pipelining takes **at most** less then **50ms** while without pipelining it's at least **150ms**.

This improvement doesn't come for free though. This changes increases MongooseIM's memory consumption.
Tests on our Tide platform before and after the change was merged shows that MongooseIM consumes now around 200Mb more memory on a load test with 50K users.o

## Erlang distribution over TLS

Many of our customers where asking if this is possible to encrypt the communications between Erlang nodes in the cluster.
They need this for various reasons, and improved security is always an important factor when deciding weather to encrypt Erlang traffic or not.

Our answer to this problem is yes, it's possible.
Magnus Henoch already described how to do this for Erlang in general in his [blog post](https://www.erlang-solutions.com/blog/erlang-distribution-over-tls.html).
Starting from there we extended MongooseIM to simplify such encrypted setup and add it to 2.1.0beta1.
Detailed documentation on how to encrypt Erlang traffic for your MongooseIM cluster you can read in the [Distribution over TLS](http://mongooseim.readthedocs.io/en/2.1.0beta1/operation-and-maintenance/tls-distribution/) guide.

You can expect a dedicated blog post with load test results on this topic in the near future.


## Changelog

Please feel free to read the detailed [changelog](https://github.com/esl/MongooseIM/releases/tag/2.0.1), where you can follow links and find the code changes.

## Special thanks to our contributors!

Special thanks to our contributors: [@astro](https://github.com/astro), [@aszlig](https://github.com/aszlig)!

## Next?

### From beta1 to final release through beta2

These release is a beta which means we are still adding a lot of changes before final release.
Having in mind that we put quality of our product so high with every change I strongly recommend giving this beta a try.
This also means that you can influence the final 2.1.0 release by giving your feedback to us.

### Peer to peer or mediated binary streaming

We will deliver an ICE/STUN/TURN server, coded in Elixir, to allow peer to peer and one to one media streaming, like voice, video, screen sharing, and whiteboarding. The signalling part, named Jingle, is already available.

## Call for action

You can:
* Star our repo: [github.com/esl/MongooseIM](https://github.com/esl/MongooseIM)
* Follow us on Twitter: [twitter.com/MongooseIM](https://twitter.com/MongooseIM/)
