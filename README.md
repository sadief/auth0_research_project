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

### Implementation

1. Configure API
2. Get Access Token
3. Call your API
4. Refresh your access token

## Protecting Your API

### How it Works

1. App autheticates user
2. Auth0 responds with users ID Token and Access Token
3. App calls API passing along Access Token
4. API responds with requested info

### Implementation

1. Configure your API
2. Use JWT validation library to validate tokens
3. Respond to the request

## Manage Users & User Profiles

### How it Works

1. Auth0 creates a user profile for each unique user
2. Auth0 fills the profile with information provided by user or from connection
3. If information is sourced from a connection Auth0 refreshed the data each time user authenticates
4. There might be some differences in how providers refer to attributes to Auth- applies a Normalized Use Profile which sorts that out

### Manage user identities and profile information

- Scopes: auth flows include a parameter that lets you specify a scope. This controls the claims in the JWT.
- Management Dashboard: Administrators can manually edit
- Management API: Provide access to read, update, and delete user profiles
- Custom Database scripts: If a custom db is used as the connection you can write scripts to implement lifecycle events like create, login, verify, delete and change password
- Rules: rules execute after user has been authenticated.

## Define and Maintain Custom User Data

### How it Works

Two kinds of metadata in Auth0

- `user_metadata`: stores user attributes (preferences) that don't impact users core functionality. Authenticated user can edit this.
- `app_metadata`: stores information like security roles and ACL groups that impact users core functionality. A user can't modify these.

These look like your basic json object and can be accessed as such:

```json
{
  "emails": "jane.doe@example.com",
  "user_metadata": {
    "hobby": "surfing"
  },
  "app_metadata": {
    "plan": "full"
  }
}
```

```javascript
console.log(user.email); // "jane.doe@example.com"
console.log(user.user_metadata.hobby); // "surfing"
console.log(user.app_metadata.plan); // "full"
```

### Customize and maintain user data

- Use [rules](##rules), which execute after user has been authenticated to change user profile during authentication transaction
- Use `GET/userinfo` endpoint to get a user's `user-metadata`, but first you have to write a Rule to copy metadata properties to ID Token
- If you have db connection use Authentication API with Signup endpoint to set `user-metadata`
- Use Management API to create, retrieve, or update `user-metadata` and `app-metadata`

## Rules

Rules are Javasript functions that execute when a user authenticates to the app. They run once the auth process is complete. They also run during the token refresh flow

1. App initiates an auth request to Auth0
2. Auth0 routes request to IDentity PRovider through configured connection
3. User authenticates
4. ID Token and/or Access Token is passed through Rules pipeline then sent to the app

### Syntax

Rule is a function with these arguments:

- `user` user object as it comes from identity provider
- `context` an object containing contextual information
- `callback` function to send potentially modified tokens back to Auth0, or an error. (always call this or the script will timeout)

Rules exectue in the order shown on the Auth0 Dashboard. If a rule depends on the execution of another rule, move the dependant rule lower in the list.

### Accounts and Tenants

When you create an account, you then create a Tenant which is a `logical isolation unit` and seems to be most often separated by like 'development, staging, production

### Domains

Then you have to pick a name for your Tenant which (appended with auth0.com) will be your Auth0 Domain. Custom domains cost.

### Application

Then you have to register each application which is either:

- Native
- Single Page Web App
- Regular Web App
- Machine to Machine App

Each app gets a Client ID and a Client Secret

## Auth0 Management API

### Intro

The API used to perform administrative tasks

### Authentication

Need to get an API token

Auth0 Management API uses JWT's to authenticat requests

To make calls to the API send then token in the good old fashioned way:

````bash
 curl -H "Authorization: Bearer eyJhb..."
    https://@@TENANT@@.auth0.com/api/v2/users
    ```
````

## Authorization Extension

Auth extension provides support for user auth via Groups, Roles, and Permissions,

**_Roles and Permissions are set on a per-application basis. If you need same roles or permissions on another application you'll have to create them separately_**

### Auth0 My App is requesting access to your tenant

https://community.auth0.com/t/how-do-i-skip-the-consent-page-for-my-api-authorization-flow/6035/13
https://community.auth0.com/t/how-can-i-automatically-grant-access-to-a-client-for-a-user-without-prompting-the-user/7128/7

UI Cannot be customized, however should only pop up for third party. It hates localhost and will always pop up, but I believe it _shouldn't_ pop up when in production

### Are users application specific

No - All users associated with a single Auth0 tenant are shared between the tenants application and have access to the applications.
To keep users separate and restrict their access it's recommended to create an additional tenant.

### Can we have anonymous users?

I don't _think_ so: https://community.auth0.com/t/manage-anonymous-user-flow/8966
https://community.auth0.com/t/jwt-token-for-guest-anonymous-unauthenticated-users/15653

### Is it possible to use a rule to block access to an application based on someone's role or group?

It is possible to do this based on their Role - I haven't been able to find information about it reagarding Groups though?
You can just create a custom rule for this
https://auth0.com/docs/api-auth/restrict-requests-for-scopes
