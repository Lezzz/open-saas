---
title: Authentication
---

Setting up your app's authentication is easy with Wasp. In fact, it's aready set up for your in the `main.wasp` file: 

```tsx title="main.wasp" " 
  auth: {
    userEntity: User,
    methods: {
      usernameAndPassword: {}, // works out-of-the-box!
      // more auth methods can be added here
    },
    onAuthFailedRedirectTo: "/",
  },
```

The great part is, by defining your auth config in the `main.wasp` file, Wasp manages most of the Auth process for you, including the auth-related databse entities for user credentials and sessions, as well as auto-generated client components for your app on the fly (aka AuthUI -- you can see them in the `src/client/auth` folder).

## Migrating to a different Auth method

We've set up the template to get you started with Wasp's simplest auth method, `usernameAndPassword`, but we suggest you only use it to get your app developlment going. 

**In production, you should opt for one or more of Wasp's more secure Auth methods**:
- `email` (email verified Auth, with forgotten password and reset options),
- `google`,
- `gitHub`, 

If you want to use one or a combination of these Auth methods, you can easily do so by changing the `auth.methods` object in the `main.wasp` file.

### Email Verified Auth

The `email` method, with it's use of an Email Sending provider to verify a user's email, is preferrable to `usernameAndPassword` because it's more secure and allows for password reset options. 

🚫 **Note that the `email` and `usernameAndPassword` methods can not be used together.** 

We've pre-configured the `email` auth method for you in a number of different files but commented out the code in case you'd like to quickly implement it in your app. To do so, you'll first need to fill in your Email Sending provider's API keys. We chose [SendGrid](https://sendgrid.com) as the provider, but Wasp can also handle [MailGun](https://mailgun.com), or SMTP. 

After you've signed up for a Sendgrid account, perform the following steps:

1. Add your `SENDGRID_API_KEY` to the `.env.server` file
2. Remove `usernameAndPassword`and uncomment the `email` properties from the `auth.methods` object in `main.wasp`:
    ```ts title="main.wasp" del={4} ins={5-7}
    auth: {
      //...
      methods: {
        usernameAndPassword: {}, 
        email: {
          //...
        }, 

    ```
2. Uncomment `RequestPasswordResetRoute`, `ResetPasswordRoute`, `EmailVerificationRoute` in the `main.wasp` file
   ```ts title="main.wasp"
      route RequestPasswordResetRoute { path: "/request-password-reset", to: RequestPasswordResetPage }
      page RequestPasswordResetPage {
        component: import { RequestPasswordReset } from "@client/auth/RequestPasswordReset",
      }

      route PasswordResetRoute { path: "/password-reset", to: PasswordResetPage }
      page PasswordResetPage {
        component: import { PasswordReset } from "@client/auth/PasswordReset",
      }

      route EmailVerificationRoute { path: "/email-verification", to: EmailVerificationPage }
      page EmailVerificationPage {
        component: import { EmailVerification } from "@client/auth/EmailVerification",
      }
   ```
3. Uncomment out the above routes's respective code in the `/src/client/auth` folder:
    - `EmailVerification.tsx`
    - `PasswordReset.tsx` 
    - `RequestPasswordReset.tsx`
4. Uncomment out the code in `app/src/server/auth/email.ts` as well.
5. Modify the `User` entity in `main.wasp` so that `username`, `email`, and `password` are all optinal fields. 
    ```tsx title="main.wasp" ins="?"
      entity User {=psl
        id                        Int             @id @default(autoincrement())
        email                     String?         @unique
        username                  String?         @unique
        password                  String?
    ```
6. Finally, uncomment the `username` property in `app/src/server/auth/setAdditionalUserFields.ts`:
    ```ts ins={2-4}
      export default defineAdditionalSignupFields({
        username: (data) => {
          return data.email as string
        },
        isAdmin: (data) => {
          if (!data.email) {
            return false;
          }
          const adminEmails = process.env.ADMIN_EMAILS?.split(',') || [];
          return adminEmails.includes(data.email as string);
        },
      });
    ```

And that's it. Wasp will take care of the rest and update your AuthUI components accordingly.

Check out the  [Wasp Auth docs](https://wasp-lang.dev/docs/auth/overview) for more info.

## Google Auth

We've also customized and pre-built the Google auth flow for you. To start using it you just need to uncomment out the `google` method in `main.wasp` file and fill in your API keys in the `.env.server` file. 

To get your Google API keys, follow the instructions in Wasp's [Google Auth docs](https://wasp-lang.dev/docs/auth/social-auth/google#3-creating-a-google-oauth-app).

Again, Wasp will take care of the rest and update your AuthUI components accordingly.

## GitHub Auth

To easily add GitHub as a login option, you can follow the instructions in Wasp's [GitHub Auth docs](https://wasp-lang.dev/docs/auth/social-auth/github#3-creating-a-github-oauth-app).