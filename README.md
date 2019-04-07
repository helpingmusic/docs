# HOME Tech

The following is meant to be an overview of different components of the HOME system.

The main components that make up HOME are `platform`, `web`, and `admin`. 

The `platform` repository is the api to all tools within HOME. It handles all business logic and 3rd party SaaS services used by HOME.

The `web` repository is the web app for HOME members to access.

The `admin` repository is the HOME internal tools for managing users.

#### Services Used
##### Stripe
Stripe is used for subscriptions and payment processing. 

Subscriptions are needed to sign up with the service and the plan determines the level of access to the platform a user has. 

A user can be charged outside of the subscription by booking studio time. They can also pay with credits that are given within the platform (see `CreditTransaction` and `AllowanceTransaction`). 

Stripe is also used for coupon generation. When a user signs up, a referral coupon code is generated for them that will give the user $5 off and the referrer $5 of credits.

##### Mailchimp
Mailchimp is used for transactional emails. Templates can be found under `src/core/email/email-templates.enum.ts`.

##### Sentry
Sentry is used for error reporting. Exceptions thrown on both the client and server will be reported there.

##### Algolia
Algolia is used for indexing all users for search. When a user is created/updated their information is sent to algolia for indexing. The client app will use the algolia client to ingest the information in the search directory. Only members with active accounts are included in the search directory.

##### Prismic
Prismic is a headless CMS used to manage content in HOME. At the moment, only admin announcements are managed via Prismic. But this can be extended to more media and copy around the site.

##### AWS
###### S3
S3 is used for the static files that are uploaded the HOME. Namely avatar/banner images and audio files.
###### Codebuild
Codebuild is used to test/lint/verify builds pushed to github. Failing builds will block pull requests in Github.

---

## helpingmusic/platform 

Platform is meant to be the central piece of HOME Tech. It includes the API and all business logic for HOME.

#### Built with:
- Typescript
- Node.JS
- [Nest.JS](https://nestjs.com) -- Server Framework
- MongoDB
- [Mongoose](https://mongoosejs.com/) -- ORM

#### API
Run `npm run start` and go to `http://localhost:3000/api` to view auto-generated swagger documentation.

#### General Patterns
###### Services
Business logic is broken into service classes. These are distinctly different than controllers. Being that controllers handle http requests. Moving logic to protocol agnostic class allows easier unit testing and leaves open the migration to graphql in the future.

###### Command Query Responsibility Segregation
The app utilizes the [Nest CQRS module](https://docs.nestjs.com/recipes/cqrs) quite a bit. This is mostly to seperate logic that is required before giving an api response, and side effects. For example, queuing an email after a password reset token has been created.

###### Output Objects
Controller functions with `Output` decorator add middleware to the endpoint. They will run the returned object through `class-transformer`, the purpose being to filter out unneeded/private properties from the reponse object.

###### Cron Jobs
All cron jobs can be added to the `worker` module. At the moment this runs on the same machine as the server, but structuring as a seperate module allows adjusting in the future. 

--------

## helpingmusic/web
Web is the web app for HOME. It’s the main portal for members to engage with. 

#### Built with:
- Typescript
- [Angular](https://angular.io/) -- Web Framework
- [Angular/material](https://material.angular.io/) -- Styling & ease of use
    - Angular/material is used to improve programming productiviy but not having to write components like modals, menus, form fields, and other non-business logic related items.
- [ngrx/store](https://ngrx.io/) -- State Management
    - ngrx/store is used to manage all data on the front end. It implements the [flux](http://facebook.github.io/flux/docs/in-depth-overview.html#content)/redux pattern but using [rxjs](http://reactivex.io/).

#### Pages/Components:
###### Login
Login w/ email and password. This page used to use oauth for facebook and google logins, but has been temporarily removed, awaiting better testing.

###### Registration
Registration is made up of several components. The first is basic information to start users into the registration tunnel. After the intial signup, they are in walkthrough to fill out billing information, profile type, bios/interests, and profile image.

###### Home
This is main landing page of the site and will contain announcements from HOME. Announcements are pulled from a 3rd party headless CMS, [Prismic](https://prismic.io/). 

###### Newsfeed
The newfeed is where any user can post something. The newsfeed is meant to act as a forum. Posts can be/edited and deleted. They can also be commented on.

###### Member Content
The member content page is media/videos posted by HOME. These are currently pulled from the api, but will need to be migrated to the CMS in the future.

###### Directory
Searching community members is major feature of the app. This page should show all members and allow them to be searched by name and other text tied to their profile. Search indexing is done by a 3rd party, [Algolia](https://algolia.com). The app uses the algolia client library to search indexed profiles.

###### Discounts
This page pulls from the API to list out local benefits/discounts available to HOME members. 

###### Session Booking
Users can book studio time through the app. On the "My Sessions" page, users can find their upcoming and past studio sessions. Each session listed give the date/time of the session, how long it is, and the amount charged to book it. Users can also cancel their upcoming sessions.

From there, if they click "Book A Session", they can see all available studios (denoted as a `Bookable` in most of the code). Once clicking on a certain studio, they can see information on the studio, the availability for the week, and fill out a form to book a session.

If they book a session, a checkout component will slide out. This component handles payment and functions as a confirmation.

###### Notifications Sidebar
When a user clicks on the notifications icon in the navbar, a panel slides out on the right of the screen. The panel has 2 tabs, notifications and notification settings. The first will list all notifications the user has had and will link to respective pages. The notification settings tab allows users to choose what they want to be notified about, and whether they want to be notified by browser notifications or by email, or both. Notifications include home announcements, comments on their posts, reviews on their profiles, and updates to their account.

###### Member Profiles
Each member has their own public profile. Profiles will be hidden if the user's profile is not finished or if their subscription is not active. The member profile page will include an avatar and banner image that they can upload. The "About" tab will show their contact, bio, and other details filled out during their registration. The "Posts" tab will list all of the posts the user has made on the community newsfeed. The "Reviews" tab is where other members can leave a review on their account, which is a rating out of 5 stars, and a comment about the person. The "Music" tab will list songs that the user has uploaded to the system. When playing a song, a music player will pop up that will stay active throughout the app.

###### Account Settings
This is where a user can edit their account info. The settings are broken into multiple collapsable panels. They can edit all of the same info that they filled out during registration.

###### Account Billing
The billing page is where the user can see their membership tier and status of their subscription. They can also set their default payment method here. The bottom of the page will list all past invoices of the user.

----

## helpingmusic/admin 

This is the internal tool for managing subscriptions.

#### Built with:
- Typescript
- [Angular](https://angular.io/) -- Web Framework
- [Angular/material](https://material.angular.io/) -- Styling & ease of use
    - Angular/material is used to improve programming productiviy but not having to write components like modals, menus, form fields, and other non-business logic related items.
- [ngrx/store](https://ngrx.io/) -- State Management
    - ngrx/store is used to manage all data on the front end. It implements the [flux](http://facebook.github.io/flux/docs/in-depth-overview.html#content)/redux pattern but using [rxjs](http://reactivex.io/).



