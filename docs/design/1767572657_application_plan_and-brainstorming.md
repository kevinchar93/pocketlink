

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
