# Related RFCs

- 
- 

# How links work now

The publishing API allows us to represent links between content items by posting a links hash containing arrays of content ids.

In our current model, a link encodes stores 3 pieces of information about the relationship between the two items:

We have many link types, which are used in different ways by the front end:

- active\_top\_level\_browse\_page
- children
- content\_owners
- document\_collections
- documents
- email\_alert\_signup
- lead\_organisations
- linked\_items
- mainstream\_browse\_pages
- manual
- ministers
- organisations
- parent
- parent\_taxons
- people
- policy\_areas
- press\_releases
- related
- related\_guides
- related\_mainstream
- related\_policies
- related\_statistical\_data\_sets
- related\_topics
- second\_level\_browse\_pages
- sections
- service\_manual\_topics
- supporting\_organisations
- taxons
- topical\_events
- topics
- top\_level\_browse\_pages
- working\_groups
- world\_locations
- worldwide\_organisations
- worldwide\_priorities

Links can be updated as part of the publishing workflow, or they can updated separately (for example through content tagger). When we change the links originating from a content item, we make a PATCH request to its links URL, with a JSON object mapping link types to arrays of content ids.&nbsp;For example:

Content items aren't required to have an HTML representation, and not everything has a base path (for example contacts can be base-path-less).

The ordering of items within the arrays is arbitrary in these requests: the publishing API deliberately ignores the ordering.

# The case for publisher-controlled ordering

As we have migrated more things to the new publishing platform, we've become aware of multiple use cases where ordering of links matters, and have had to implement error-prone workarounds.

[https://www.gov.uk/browse/childcare-parenting](https://www.gov.uk/browse/childcare-parenting)&nbsp;is an example of a browse page that has been manually ordered in collections publisher. It uses the&nbsp; **_second\_level\_browse\_pages_** link type. There is a natural ordering around age of the child, which helps the user identify the section they need more quickly.

Without the manual override, the sections would be sorted alphabetically, which is generally not very useful unless the user knows the exact name of the section they need. In the absence of more content-specific metadata to order by, manual ordering gets the job done.

Detailed guides and policies can link to _ **organisations** _ where one or more organisations is displayed prominently, followed by some less important organisations, depending on level of involvement with the topic. Example: [https://www.gov.uk/government/policies/2012-olympic-and-paralympic-legacy](https://www.gov.uk/government/policies/2012-olympic-and-paralympic-legacy)&nbsp;

_ **Related** _ links are another set of links that can be manually ordered at the moment, for example on&nbsp;[https://www.gov.uk/set-up-business](https://www.gov.uk/set-up-business)&nbsp;the links are manually ordered by relevance rather than A-Z.

In the future, we may want to change the way we represent the stages in a service (for example&nbsp;[https://www.gov.uk/browse/business/setting-up](https://www.gov.uk/browse/business/setting-up)) to make it easier for service designers to draw a path through the content without having to update all the individual pages separately. Ordered lists might be one way to represent this information.

# Proposal&nbsp;

## Semantics

# Alternatives rejected by this proposal

## Making all link arrays ordered

Essentially reverting&nbsp;

- Fuzzy semantics - unclear if the ordering is intended or not

## Changing the format of the links hash to include additional metadata

&nbsp;Previously rejected in&nbsp;

- More flexible
- Difficult to make backwards compatible
- Lots of publishing apps would need to change
- Lots of frontend apps would need to change

## Keeping links unordered and adding a new container for ordered links

- 

Apps are forced to handle both cases separately

---
status: "DRAFT"
notes: "Not yet complete"
---

## Problem

## Proposal

