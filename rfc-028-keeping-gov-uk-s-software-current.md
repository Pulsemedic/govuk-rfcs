One of our core values is to use secure and up to date software. This document lays out the recommendations for keeping our Ruby on Rails software current.

# Introduction

We run a lot of Rails applications. This means that we have dependencies on both Rails and Ruby versions.

# Upgrading Rails

It's very important that we're running a currently supported version of Rails for all applications, otherwise we **aren't covered** &nbsp;by [security fixes](http://rubyonrails.org/security/). We should:

- Be running on the current major version - this currently means `4.y.z`
- Maintain our applications at the latest current bugfix release for the minor version we're on (expressed in Gemfile syntax as: `~> X.Y.Z`) - this currently means `4.1.8` and `4.2.3`
- Keep abreast of breaking changes for the next major version (`5.y.z`), and have a plan to migrate our apps before `4.2.x` is deprecated

We have several applications still running `3.y.z` - these applications should have their upgrade path captured in backlogs and be prioritised for the next time major work is planned in those apps, or within the quarter: whichever is first.

# Upgrading Ruby

New versions of Ruby bring us improved performance and nicer syntax for certain things, but also can cause issues with the libraries etc. we use. We should:

- Be running on the current major version - this currently means `2.y.z`
- Maintain our applications at the current or next-to-current minor version - this means `2.2.z` or `2.1.z`, depending on your app's dependencies

We have several applications still running the `1.9.3`&nbsp;family (this version and `2.y.z`&nbsp;are now obsolete) - these applications should have their upgrade path captured in backlogs and prioritised for the next time an application is worked on, or within the quarter: whichever is first.

# Current state

The current state of the Ruby and Rails versions is [listed in this versions spreadsheet](https://docs.google.com/spreadsheets/d/1FJmr39c9eXgpA-qHUU6GAbbJrnenc0P7JcyY2NB9PgU/edit#gid=1480786499) by&nbsp;alext.&nbsp;

&nbsp;

&nbsp;

