# Problem

Our applications at the moment are more tightly coupled to the infrastructure that is necessary or good. This is going to make transitioning to a containerised setup harder.

# Proposal

This is therefore a proposal for how we should configure our Rails apps to use ideas from [The Twelve-Factor App](http://12factor.net/) to reduce this coupling. This details how Rails apps should behave because most of our apps are Rails, but these proposals can easily be applied to apps using other technologies.

## Configuration

Any config details that are specific to the deployment environment should be&nbsp;passed to the app using environment variables. This includes any credentials,&nbsp;locations of database servers etc. More details - [http://12factor.net/config](http://12factor.net/config)

Many of the default generated Rails config files include code to read these&nbsp;values from the environment in production (eg [secrets.yml](https://github.com/rails/rails/blob/4-2-stable/railties/lib/rails/generators/rails/app/templates/config/secrets.yml)).&nbsp;We should use these environment variable names where they exist.

These environment variables will be set by whatever mechanism is responsible for&nbsp;starting the app. At present, this is handled by the `govuk_setenv` script that&nbsp;reads environment variables from files managed by puppet. In future this&nbsp;mechanism may change, but the important point is that the applications&nbsp;themselves won't need to be updated to reflect this change, they'll continue to&nbsp;read the same environment variables.

&nbsp;

&nbsp;

&nbsp;

