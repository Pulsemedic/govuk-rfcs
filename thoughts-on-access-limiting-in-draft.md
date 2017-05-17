## **Problem**

Editions in Whitehall have the ability to be marked as "access limited". This means that until they are published, only users in the organisation the edition belongs to, or (for world news articles) associated with the world locations of the organisation, can view the edition in draft. This functionality needs to be replicated in the draft stack so that access-limited content is only viewable by the relevant users before publication.

**Proposal**

&nbsp;boolean to formats/metadata.json so it is added to the base of all edition schemas. (Don't need to provide the actual org or location since they are already provided in links?)

&nbsp;

**Enforcing in draft**

Option 1: in content store

Add gds-sso gem to content-store and create User model (shouldn't need any extra fields).  
When contentitem has access\_limited, check organisation matches.  
Pass through OAuth token and JSON blob when making API request to content-store - how?  
How to redirect to signon if not authenticated?

&nbsp;

Option 2: new authenticator service

Sits between router and the frontend apps, uses gds-sso gem.  
Would need to store data from signon which means having a database. Is there no way to use gds-sso without storing data locally?  
Makes request to content-store to determine if item is access limited. Maybe a new content-store endpoint to just return org slug, to avoid passing the whole content item around twice?  
How to route via authenticator, since router will point to government-frontend?

&nbsp;

Option 3: do it in government-frontend

Use gds-sso here, so would need to add a database&nbsp;(again, is there really no way of doing it without?)  
Simplifies routing and redirection, and would only need to request content once  
Possible duplication if other frontends also need this functionality.

&nbsp;

For the sake of completeness, option 4: in router

Horrible as a) blurs responsibility of router and b) it's go so we would need to re-implement all the sso stuff.&nbsp;  
But, it has a database, it's central, and would avoid problems with routing and redirection.&nbsp;  
Wouldn't solve issue of requesting data twice though, and would need to have a way of rendering 'not for you' page.&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

