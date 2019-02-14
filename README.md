# Auth0 Research Project

## Purpose

- Determine and define how we might use auth for:

  - User Authentication
  - Setting up Users, Roles, Permissions
  - Mapping permissions to ACL

- See how we can replicate what we have in the users service with:

  - Superusers
  - Biba Users
  - Sales People
  - Managers

- Switch to Casbin potentially instead of the validation middleware

- Try to create a user in Auth0 and log in using a js library - does it come with a JWT that has everything we need?

## Adding Login to App

### How it Works

1. User clicks login
2. User is authenticated
3. App requested the usersID token
4. Auth0 responds with users ID token

Same kinda gist depending on if you're using an SPA, regular web page or mobile app

### Implementation

1. Configure sign-in methods: user, password (can store user in Auth0 or our db), social, etc
2. Customize sign-in UI
3. User auth0 SDK to trigger flow

## Calling the API

### How it Works

1. If user not authenticated, redirects to Auth0 Auth server
2. User is authenticated
3. App requests an ID Tokem Access Token, and Refresh token
4. Auth0 responds with tokens
5. Access Token can be used to call API and retrieve requested data

###Implementation
