

5 Jan 2026

# Project Specification: "Pocketlink" URL Shortener

The "Pocketlink" service provides users with the ability to shorten, manage, and track long URLs, simplifying link sharing for marketing, social media, and personal use.

## 1. User Personas

To understand the customer journeys, we'll define primary user types:

**The Registered User:** A "power user" (like a marketer, creator, or developer) who needs to create multiple links, manage them, customize them, and track their performance.

Originally I thought That we could have guest users, but this opens us up to abuse. So we will avoid it. 

## 2. Journey Flows

Here are the different flows for what a customer can do in the app.

- Landing page, create link and sign up
- Sign in
- Forgot Password
- Create  quick link
- Create new link
- View all links
- View a link's details
- Edit a link's details
- View account activity data
- Export account activity data
- Change email address
- Change password
- Delete account

When accessing a link created by a user a single action can be done:
- Navigate to a short link


### Landing page, create link and sign up

- User starts on landing page and will enter their long link in the form and click get your link CTA. 
- User is navigated to the Create Your Account page with URL encoded long link. 
- User fills in create your account form with email and password or will log in using social authentication.
- Account is created for user and they are navigated to the dashboard
- A modal for successful link creation is shown to the user on top of the dashboard. 
- User can just click create account and not create a link upon sign up if they wish. 

### Sign in

- User starts on sign in page. They will enter their email and password and hit login CTA. 
- If user is successfully logged in, they are sent to dashboard. 

### Forgot Password

- User will start on sign in page and click forgot password link. 
- User is navigated to forgot password page where they enter the email they used to register and click send reset link CTA. 
- A user is sent to a Check Your Inbox page confirming an email was sent to them. 
- User will need to click reset link in email to be navigated to set a new password page. 
- Upon successfully creating a new password, user is shown password changed success page with a CTA to take them to the login page. 

### Create  quick link

- User starts on dashboard. Paste their long link into the create a new pocket link form and click create your pocket link CTA. 
- User is shown, link created successfully, modal upon successful link creation. 
- If user clicks view link details, they are navigated to the link details page. 

### View all links

- User can view all of their links by navigating to my links page via dashboard. 

### Create new link

- User starts on My Links page. They click Create New Link CTA. 
- User is navigated to create a new link page. They fill in the create link form and click create your link CTA. 
- User is shown, link created successfully, modal upon successful link creation. The model is shown on top of the My Links page, which the user will have been navigated back to. 
- If user clicks view link details, they are navigated to the link details page.

### View a link's details

- User starts on my links page. They click on a link they wish to view the details for. 
- They are navigated to the link details page to see details and analytics for the link in question. 

### Edit a link's details

- User starts on my links page. They click the edit button on a link they wish to edit. 
- User is navigated to edit link page where they edit the details of the link in the form and click save or cancel. 
- User can also access the edit page by clicking edit from the link details page. 

### View account activity data
### Export account activity data
### Change email address
### Change password
### Delete account

# Core business operations / questions

The key problems in the application we need to solve are:
- What validation do we want to perform on URLs that are given to us? 
- How will we actually generate the short link slugs? 
- How do we handle short link slug collisions, especially if they happen concurrently? 
- How will we redirect quests to their destination? 
- How do we track the analytics for each click? 
- What counts as a visit for analytics purposes? 
- What exactly do we track in the analytics? 
- What happens to analytics data when a link is deleted? 
- How far back does analytics data go? (store forever, or rolling window?)
- Do we allow users to provide a customized short link alias? 
- Do we need any caching and how will we implement this? 
- Do we need or want any rate limiting? 
- How are we going to handle authentication? 
- What happened when a user edits the destination URL of a link? 
- What happens when a user deletes a link? 
- What happens when a user deletes their account?
- What happens when someone tries to visit a deleted/expired short link?
- How do we handle URLs that are already short links from your own service? (prevent loops?)
- Do you need any concept of "public" vs "private" links?

## What validation do we want to perform on destination URLs that are given to us?

- We should ensure that destination URLs are valid. 
- We will only support URLs with the HTTPS protocol. 
- We will prevent users providing destination links that are links to other pocket link links to avoid looping.
- We will Follow any redirects the destination link does up to five hops and validate the last in the chain. 
- We will pass provided domain names through a content filter to check their suitability. 
  - We will reject destination links to certain content. (adult / dark web / scams)
  - We will use the Google WebRisk API to categorize submitted URLs.

Logic:
-  We check the URL is valid Based on URL specification.
-  We check it has a maximum length of 2000 characters. 
-  We check that the URL is using the HTTPS protocol.
-  We check that the link isn't on one of the domains we own (a pocket link domain).
-  We make a HEAD requests to the destination link to see if it redirects. If it does, we do the following:
   -  Follow the redirect hops up to a maximum of 5 times.
   -  Check that none of the redirects hop link back to an earlier step in the chain. (Prevent redirect loops)
   -  Check that none of the redirects violate any of the earlier validation points. (protocol, length, valid URL, not a pocket link domain etc)
   -  Set a timeout for the redirect hops of 5 seconds
   -  Take the last item in the chain and validate that against the web risk API.
-  We make an API call to the Google WebRisk API to check the status of the URL / domain.
   -  We should consider caching the result of this for future use, especially if we can do checks at the domain level. 
-  If any of these validation points fail, we reject URL as invalid.
   -  We provide rejection reasons to the user:
      -  "Sorry our systems determined the destination link is risky, we can't create a short link for it."
      -  "We only support   HTTPS links."
      -  "The URL is invalid: <detail here if possible>
      -  "We only support URLS with a maximum length of 2000 characters."
      -  "The destination URL contains a redirect loop."
      -  "The destination URL took too long to validate."


Potential issues:
- What should we do if the web risk API is down?
  - for now What we will do is just display an appropriate message saying the short link cannot be created due to our Risk scanning service being down.
  - I think a solution for this in the long term is to create the link, but give the user a warning that we were unable to check it. The link will not be accessible until it can be validated. 
    - We could create a background job that will automatically Attempt to validate the link later. 
    - or we can put a notification on the user's dashboard that they have unvalidated links due to an outage and that they need to manually click a CTA to attempt revalidation later down the line. 
- Is there any kind of rate limiting from the web risk API we have to be concerned with?
  - For now, users will have a hard limit of 20 links that they can create. So I don't think that number is high enough to warrant any explicit rate limiting within the application. 
  - We can look to configure rate limiting in any load balancer service that we use with the cloud provider that the application is deployed to. 


## How will we actually generate the short link slugs?

Thinking
- We want to generate a slug that is short.
- We can use a nano ID with the alphabet below.
  - `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`
- The nano id will have length 7 chars
  - with this 
- **Question** I'm not sure if we want to generate a slug that is random because it will be harder to index it.
- Perhaps what we can do is:
  - generate a nano ID (that is random)
  - write it to our DB table
  - then because we're using a cache anyway, put the mapping in the cache at this point
  - we can also ensure the cache is pre-populated when it's started up
- This way I should be able to insert without having to generate an index with the random ID (slower writes).
- When we come to look up the short link, we will look for it initially in the cache where it should be populated. 
- If it can't be found in the cache, we'll look it up from the table and insert it into the cache for next time. 
- **Question** What happens if we fail a put the mapping in the cache
  - This should be fine as long as it's written to the database. As when we look it up, if it's not in the cache, we'll look for it in the DB, then populate the cache with the mapping. 
**Question** How do we handle short link slug collisions, especially if they happen concurrently? 
  - Should be handled for us by our database schema. We've put a constraint that the domain & the back half of short link in the short_links table must be unique - the database will enforce the unique property, even if two slugs are generated at the same time and attempt to write to the DB at the same time.
  - The second slug inserted will create error upon insertion about violating the unique constraint.
  - When this error happens, we should be okay to simply regenerate a new ID without a collision - we can try this up to 10 times, And if we still have collisions, we increment the length of the nano ID by one and try again. 

Logic:
- generate a nano id with default length of 7 using alphabet `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`
- Check if the nano ID exists in the database. (collision check)
  - If collision occurs, generate new ID & check again
  - ID generation up to 5 times if we still have a collision, increment the length of the nano ID by one & repeat
  - Fail after a maximum of 15 attempts.
- Use the result as the short link slug.
- The slug should be inserted in the cache and then into the database as a transaction when being used. 

Possible implementation details:
- Redis supports a bulk loading, taking a text file, containing entries in the Redis protocol.
- We can create an admin script that will generate a text file to populate the cache from the tables containing the short link mappings.
- We can create a script that uses this nightly to make sure that the cache is properly populated. 
- We can also use it to create a script to pre-populate the cache if we need to turn it off for upgrades, etc. 
- https://redis.io/docs/latest/develop/clients/patterns/bulk-loading/

## How will we redirect quests to their destination? 

- Pocket link will be hosted on the domain `pocketlink.co.uk`
- Short links will have the domain `pok.ad` , example like `pok.ad/sfhSHc9`
- Requests from both domains will be routed to the web application.
- We inspect the domain and route the request to the right controller. 
- The application will look in its cache, find the destination link, 
  -  if found return a response. 
-  If it's not in the cache, the application will do a database lookup. 
  -  if found return a response. 
-  If it can't be found in the cache or in a database, a 404 response will be returned. 
-  When the response returned to the user, the cache control header will be set to only cache in the browser for 5 minutes.
   -  TTL is 5 min so we strike a good balance between performance and today analytics. 


## How do we track the analytics for each click?

- We will use a NoSQL database to track link analytics (mongoDB)
- If we've successfully looked up and returned a response for a short link we will write a document to the collection `link_visit`
- We will query this collection to create our analytics visualizations.  

## What counts as a visit for analytics purposes?

- not planning on putting any client side injected code into the redirect
- a visit will be counted when a request hits the application server

## What exactly do we track in the analytics? 

- the link
- link owner
- time of the event
- ip address (hashed)
- user agent
- referrer

see doc: 1767571394_document_db_model_for_storing_events.md

##



## What happens to analytics data when a link is deleted? 


## How far back does analytics data go? (store forever, or rolling window?)

## Do we allow users to provide a customized short link alias?

No, we don't want to introduce this complexity, That said, the current design would allow for it in the future.

## Do we need any caching and how will we implement this? 

## Do we need or want any rate limiting? 

## How are we going to handle authentication? 

## What happened when a user edits the destination URL of a link?

## What happens when a user deletes a link?

The short link will be soft deleted by setting a time on the deleted_at field.

We will create an hourly cron job that will hard delete the short links and archive the back_half by inserting it into the archived_short_links table, we'll be able to use this table to look up existing slugs when generating slugs for new short links.

## What happens when a user deletes their account?

We will delete all of the links associated with their account. 

We will do this as a batch job instead of doing it upon the user deleting their account (using soft delete).

## What happens when someone tries to visit a deleted/expired short link?



## How do we handle URLs that are already short links from your own service? (prevent loops?)

We will handle this in the validation step that is run when a user creates new links. We will prevent them from providing destination links that link to a pocket link URL. 

## Do you need any concept of "public" vs "private" links?

No, we will not be offering any facility for creating private links. All short links created on the website will be publicly accessible. 
