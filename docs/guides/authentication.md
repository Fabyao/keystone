<!--[meta]
section: guides
title: Authentication
subSection: advanced
[meta]-->

# Authentication

_Note on terminology_:

- _Authentication_ refers to a user identifying themselves.
  Within this document, we abbreviate _Authentication_ to _Auth_.
- _Access Control_ refers to the specific actions an authenticated or anonymous
  user can take. Often referred to as _authorization_ elsewhere.
  The specifics of how this is done is outside the scope of this document.
  See [Access Control](/guides/access-control) for more.

## Admin UI

Username / Password authentication can be enabled on the Admin UI.

> NOTE: Admin Authentication will only restrict access to the Admin _UI_.
>
> To also restrict access to the _API_,
> you must setup [Access Control](/guides/access-control) config.

To setup authentication, you must instantiate an _Auth Strategy_, and create a
list used for authentication in `index.js`:

Here, we will setup a `PasswordAuthStrategy` instance:

```javascript
const { Text, Password } = require('@keystonejs/fields');
const { PasswordAuthStrategy } = require('@keystonejs/auth-password');
const { AdminUIApp } = require('@keystonejs/app-admin-ui');

const keystone = keystone.createList('User', {
  // ...
  fields: {
    username: { type: Text },
    password: { type: Password },
  },
});

const authStrategy = keystone.createAuthStrategy({
  type: PasswordAuthStrategy,
  list: 'User',
  config: {
    identityField: 'username', // default: 'email'
    secretField: 'password', // default: 'password'
  },
});

// Enable Admin UI login by adding the authentication strategy
const admin = new AdminUIApp({ authStrategy });
```

Once your KeystoneJS server is restarted, the Admin UI will now enforce
authentication.

### Logging in to the Admin UI

The first time you setup authentication, you may not be able to login. This is
because there are no `User`s who can do the logging in.

First, disable authentication on the Admin UI by removing `authStrategy` from
the `new AdminUIApp()` call:

```diff
- const admin = new AdminUIApp({ authStrategy });
+ const admin = new AdminUIApp({ });
```

Second, disable access control by removing `access` from the 
`keystone.createList('User', ...` call:

```diff
-  access: {
-    read: access.userIsAdminOrOwner,
-    update: access.userIsAdminOrOwner,
-    create: access.userIsAdmin,
-    delete: access.userIsAdmin,
-    auth: true,
-  },
```

Restart your KeystoneJS App, and visit <http://localhost:3000/users> - you should now be able to access the Admin UI without logging in.

Next, create a User (be sure to set both a username and password).

Add the `authStrategy` config back to the `new AdminUIApp()` call

```diff
- const admin = new AdminUIApp({ });
+ const admin = new AdminUIApp({ authStrategy });
```

Restart your KeystoneJS App once more, and try to visit <http://localhost:3000/users>; you will be presented with the login screen.

Finally; login with the newly created `User`'s credentials.

## API Access Control

Adding Authentication as above will only enable login to the Admin UI, it _will
not_ restrict API access.

**To restrict API access, you must setup [Access Control](/guides/access-control).**
