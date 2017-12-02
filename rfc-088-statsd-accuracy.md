# Statsd Accuracy

## Summary

An abstract, tl;dr or executive summary of your RFC.

## Problem

A problem has been identified with the integration between a number of our
applications and their integration with statsd. It affects applications that
are run on multiple hosts and send statistics with a key that is not unique
to that host. The consequence of the problem is that only one instance of
statistics is aggregated.

### How it happens

To understand how this problem occurs it is useful to understand how [statsd][]
and [Graphite][] work.

With how we have [configured][statsd-configuration] Graphite for statsd we
effectively have a database where there is an entry for every 5s of time. This
is populated by statsd. If statsd sends data to graphite at a rate of more than
once every 5 seconds only the last entry will be stored in Graphite.

Our statsd configuration involves statsd running on each host in our
infrastructure. Our applications will send requests to their local statsd each
time a call is executed (which is sent as a UDP packet). This local statsd is
configured to build up statistics for 5 seconds and then send the aggregated
data to Graphite.

This configuration works if each statistic sent to Graphite has something to
distinguish which host it is being sent from. However if it does not have
anything to distinguish the host then only the last host which sent a statistic
will be recorded.

For example:

If we have email-alert-api running on backend-1 and backend-2 sending an
increment value with a key of `stats.govuk.app.email_alert_api.test_increment`
and both have something happen at a rate of 10 per second. Both of these will
send a value to Graphite of 10 and Graphite will just store the last one
received, effectively discarding half of the statistics.

Whereas if we have a key of
`stats.govuk.app.email_alert_api.{hostname}.test_increment`
and both have something happen at a rate of 10 per second. Both of these will
send a value to Graphite of 10 and Graphite will store both of them. We can
then produce a graph that sums the data so we have an accurate value of 20
per second.

### How to replicate

It's relatively trivial to replicate this issue on the integration environment.

On backend-1 and backend-2 run:
```
$ govuk_app_console email-alert-api
irb(main):001:0> loop { EmailAlertAPI.statsd.increment("testing_concurrency"); sleep(0.1) }
```

And checking Graphite:
```
$ curl https://graphite.integration.publishing.service.gov.uk/render/\?target\=stats.govuk.app.email-alert-api.testing_concurrency\&format\=csv\&from=-30seconds
stats.govuk.app.email-alert-api.testing_concurrency,2017-11-28 18:34:25,10.0
stats.govuk.app.email-alert-api.testing_concurrency,2017-11-28 18:34:30,9.8
stats.govuk.app.email-alert-api.testing_concurrency,2017-11-28 18:34:35,10.0
stats.govuk.app.email-alert-api.testing_concurrency,2017-11-28 18:34:40,10.0
stats.govuk.app.email-alert-api.testing_concurrency,2017-11-28 18:34:45,10.0
stats.govuk.app.email-alert-api.testing_concurrency,2017-11-28 18:34:50,10.0
```

Whereas if we include the hostname:

```
irb(main):002:0> loop { EmailAlertAPI.statsd.increment("#{Socket.gethostname}.testing_concurrency"); sleep(0.1) }
```

We can sum the output:

```
$ curl https://graphite.integration.publishing.service.gov.uk/render/\?target\=sumSeries\(stats.govuk.app.email-alert-api.\*.testing_concurrency\)\&format\=csv\&from=-30seconds
sumSeries(stats.govuk.app.email-alert-api.*.testing_concurrency),2017-11-28 18:39:50,20.0
sumSeries(stats.govuk.app.email-alert-api.*.testing_concurrency),2017-11-28 18:39:55,19.8
sumSeries(stats.govuk.app.email-alert-api.*.testing_concurrency),2017-11-28 18:40:00,20.0
sumSeries(stats.govuk.app.email-alert-api.*.testing_concurrency),2017-11-28 18:40:05,19.8
sumSeries(stats.govuk.app.email-alert-api.*.testing_concurrency),2017-11-28 18:40:10,20.0
sumSeries(stats.govuk.app.email-alert-api.*.testing_concurrency),2017-11-28 18:40:15,20.0
```

### Impact

The impact of this problem seems to be really quite low since it looks to have
been a problem that has existed for ~5 years without it coming to the attention
of anyone. Most of our statistics that are used for monitoring regarding
application health seem to be done on a host basis so they will not have been
affected.

However it is possible that some of the statistics we have used for application
decisions/understanding could have been incorrect. Some examples of these
could be:

- @TODO

[statsd]: https://github.com/etsy/statsd
[Graphite]: https://graphiteapp.org/
[statsd-configuration]: https://github.com/alphagov/govuk-puppet/blob/9ea4b8753a0a8779eaa8ebd7f19dd6fbc3cd578c/modules/govuk/files/node/s_graphite/storage-schemas.conf#L8-L10

## Proposal

There are a number of approaches we can take to resolve/mitigate this issue,
which have varying degrees of impact and effort involved to utilise them. It's
also possible that a solution may compromise aspects of the various options.

### Switching from statsd to an alternative system

Switching system would be a way to re-structure statistics from the ground up
and for this issue to be considered. Popular systems for this would be
[Prometheus](https://prometheus.io) and [InfluxDB](https://www.influxdata.com/).

From talking with the infrastructure team there are longer scale plans (or
could still be at the idea stage) change the statsd/Graphite/Icinga based
system to a Prometheus based one. There is also work being undertaken by the
Reliability Engineering team at GDS to investigate usage of Prometheus across
GDS.

However, given the scale of changes required to switch from our statsd/Graphite
structure, this work is considered outside the scope of this RFC. It is noted
though that our current set-up is something that would ideally be replaced
in the not too distant future.

### Utilise repeating within statsd

statsd comes bundled with the ability to repeat statistics, this could be used
to send statistics from each individual host to a centralised statsd instance
which aggregates the statistics. This does however introduce a bunch of
network complications:

- The amount of traffic of every one of our stats boxes to a central one may
  well overwhelm a central statsd
- There isn't really much point having statsd running on individual hosts if
  everything is repeated, but if we change that we run a higher risk of packet
  loss

An alternative to a simple repeating utility would be to use a statsd smart
repeater plugin. This approach has been [spiked][statsd-repeater] for
integration into GOV.UK infrastructure.

Some of the concerns of this are:

- Trusting a relatively unmaintained, not widely used plugin
- How to integrate this without causing a large amount of disruption (either
  by changing everything in one-go, or by setting up lots of additional
  databases to switch over to)

### Do aggregation in carbon aggregator

Graphite has a daemon, [carbon aggregator][], which can be used to aggregate
the data coming into graphite. This can be used to aggregate the input into
graphite.

The challenge for this would be to set this up in a way where it could be used
to aggregate lots of different data series without requiring masses of
configuration (or configuration each time a new data series is added).

If we could determine a few simple wildcard based rules, this would likely be
the easiest option to fix the incorrect stats without requiring changes to
each application.

### Always specify a hostname when sending statsd

This is the approach where





[statsd-repeater]: https://github.com/alphagov/govuk-puppet/pull/6834
[carbon aggregator]: http://graphite.readthedocs.io/en/latest/carbon-daemons.html#carbon-aggregator-py
Describe your proposal, with a focus on clarity of meaning. You MAY use [RFC2119-style](https://www.ietf.org/rfc/rfc2119.txt) MUST, SHOULD and MAY language to help clarify your intentions.
