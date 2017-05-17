# Problem

Our applications at the moment are more tightly coupled to the infrastructure than is necessary or good. This is going to make transitioning to a containerised setup harder.

# Proposal

This is therefore a proposal for how we should configure our Rails apps to use ideas from [The Twelve-Factor App](http://12factor.net/) to reduce this coupling. This details how Rails apps should behave because most of our apps are Rails, but these proposals can easily be applied to apps using other technologies.

## Configuration

Any config details that are specific to the deployment environment should be&nbsp;passed to the app using environment variables. This includes any credentials,&nbsp;locations of database servers etc. More details - [http://12factor.net/config](http://12factor.net/config)

Many of the default generated Rails config files include code to read these&nbsp;values from the environment in production (eg [secrets.yml](https://github.com/rails/rails/blob/4-2-stable/railties/lib/rails/generators/rails/app/templates/config/secrets.yml)).&nbsp;We should use these environment variable names where they exist.

These environment variables will be set by whatever mechanism is responsible for&nbsp;starting the app. At present, this is handled by the `govuk_setenv`&nbsp;script that&nbsp;reads environment variables from files managed by puppet. In future this&nbsp;mechanism may change, but the important point is that the applications&nbsp;themselves won't need to be updated to reflect this change, they'll continue to&nbsp;read the same environment variables.

## Logging

Applications should not deal with opening logfiles etc. Instead they should log&nbsp;to `STDOUT`, and `STDERR`. The OS should deal with capturing these streams and&nbsp;storing them as appropriate. Details - [http://12factor.net/logs](http://12factor.net/logs)

- General logging SHOULD be sent to `STDERR`.
-  as JSON lines suitable for logstash.
- Apps MAY send additional log lines to `STDOUT` providing they are JSON formatted.
- Apps MUST NOT send any non-JSON logging to `STDOUT`.

I've created an example app, and configured it to log as described -&nbsp;[https://github.com/alext/twelve-factor-rails/pull/1](https://github.com/alext/twelve-factor-rails/pull/1)

### Known issues

-  by default. gds-sso will need to be updated to&nbsp;configure it to log elsewhere.  
Alternatively it may be easier and safer to redirect `STDOUT` to `STDERR`, so that only known things will log to the real `STDOUT`&nbsp;

## Asset serving

Twelve-factor recommends that:

> "The twelve-factor app is completely self-contained&nbsp;and does not rely on runtime injection of a webserver into the execution environment to create a web-facing service." -&nbsp;[reference](http://12factor.net/port-binding)

This is at odds with the way we currently serve static assets (nginx is configured to serve everything from the public directory). Some thought needs to be given as to whether this is an acceptable deviation for the efficiency benefits.

The alternative would be to have these assets served by the application process using some rack middleware. We'd need to ensure that this was as efficient as possible (ensure it could be multi-threaded so as not to block application requests), and set appropriate cache headers.

## Dependencies

A twelve-factor app should "Explicitly declare and isolate dependencies" ([http://12factor.net/dependencies](http://12factor.net/dependencies)). Rails apps mostly have this covered through the use of Bundler and Gemfiles.

One area that's not so well covered is any non-gem dependencies provided by the&nbsp;OS. This includes things like external programs (imagemagick, tika etc...), and&nbsp;any libraries required by gems with native extensions (eg libxml), and the&nbsp;compilers necessary to build them. There's no obvious way to resolve this with&nbsp;our current infrastructure, we therefore recommend that a decision on how to&nbsp;resolve this is deferred until we migrate to a containerised setup.

## Separate the build and release stages

Our current deploy process doesn't map onto the process described by twelve-factor ([http://12factor.net/build-release-run](http://12factor.net/build-release-run)).

We're currently using a Capistrano deploy style which does most of the building on the app servers at deploy time. Given ruby is a non-compiled language, there isn't much building to do - it mostly comes down to building assets, and bundling.

We should investigate how to build a single artefact that can be simply deployed and run on servers taking all the necessary config from the environment. &nbsp;This is probably another point that should be deferred until we are transitioning to a containerised setup.

# 12-factor principles

For reference these are all the twelve-factor principles:

[I. Codebase](http://12factor.net/codebase)&nbsp;-&nbsp;One codebase tracked in revision control, many deploys

We already do this.

[II. Dependencies](http://12factor.net/dependencies)&nbsp;-&nbsp;Explicitly declare and isolate dependencies

See above...

[III. Config](http://12factor.net/config)&nbsp;-&nbsp;Store config in the environment

See above...

[IV. Backing Services](http://12factor.net/backing-services)&nbsp;-&nbsp;Treat backing services as attached resources

We already do this (when combined with the Config approach above).

[V. Build, release, run](http://12factor.net/build-release-run)&nbsp;-&nbsp;Strictly separate build and run stages

See above...

[VI. Processes](http://12factor.net/processes)&nbsp;-&nbsp;Execute the app as one or more stateless processes

We already do this

[VII. Port binding](http://12factor.net/port-binding)&nbsp;-&nbsp;Export services via port binding

We already do this

[VIII. Concurrency](http://12factor.net/concurrency)&nbsp;-&nbsp;Scale out via the process model

We already do this

[IX. Disposability](http://12factor.net/disposability)&nbsp;-&nbsp;Maximize robustness with fast startup and graceful shutdown

Unicorn gives us this feature.

[X. Dev/prod parity](http://12factor.net/dev-prod-parity)&nbsp;-&nbsp;Keep development, staging, and production as similar as possible

We already do this

[XI. Logs](http://12factor.net/logs)&nbsp;-&nbsp;Treat logs as event streams

See above...

[XII. Admin processes](http://12factor.net/admin-processes)&nbsp;-&nbsp;Run admin/management tasks as one-off processes

Rake tasks give us this.

&nbsp;

&nbsp;

