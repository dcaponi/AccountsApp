# SvelteKit PocketBase Auth Starter Kit

An insanely simple free plug & play setup for those wanting auth/user management in a SvelteKit app with as little pain as possible.

## Getting Started - Local Development
1. Clone this repo
2. Set `VITE_POCKETBASE_URL=http://127.0.0.1:5555` in `AccountsApp/.env.local`
3. Run `make frontend-up` to bring up the skeleton app and navigate to `/login`
4. Run `make backend-up` to bring up PocketBase
5. Go to the PocketBase UI at `localhost:5555/_/` click "Settings" (tool icon on the far left side)
6. Set "Application name" to whatever you like. "Set Application URL" to `localhost:5173` (we need this for redirect URLs to the SvelteKit pieces later)
7. Follow one or both of the flow setup guides below
8. Create routes in `/routes` following standard SvelteKit developer guidance
   1. If you want them to be *protected* i.e. user is logged in to see the page, add the folder to `const protectedRoutes = ['/protected']` in `AccountsApp/src/routes/+layout.server.ts`
   2. You can also configure the page an un-authed user gets bounced to (in this example its `/login`)

## Optional - Email/Password Flow with Confirmation Email
   1. Go to "Mail Settings" and set the "Verification Template" Action URL to `{APP_URL}/confirm-verification/{TOKEN}`
      1. There's a `confirm-verification` folder in the AccountsApp SvelteKit project that handles flipping the "verified" flag when a user visits the Action URL from their email.
      2. Check "Use SMTP mail server"
      3. If you're cheap like me and don't want to pay for a SendGrid or Mail Gun you can:
         1. Set up a Gmail Account
         2. In the account settings (click the avatar in the top right and click "Manage Your Google Account") and click Security in the Left Menu
         3. Activate 2-step verification
         4. In the search bar on the top left, search "App Passwords"
         5. Create a new App Password and note it somewhere safe
      4. Set the SMTP server host (smtp.gmail.com if youre doing the cheap way)
      5. Set port to whatever your SMTP provider tells you (587)
      6. Leave the auth methods as their defaults
      7. Provide your username and password from your SMTP provider
         1. If you did the cheap option thats the email for your new gmail account and the app password.

## Optional - OAuth Flow
   1. Go to your desired OAuth provider's website and follow their instructions for integrating OAuth2
      1. Set the Redirect URI to `http://localhost:5173/callback` 
         1. there's a `callback` folder in AccountsApp SvelteKit project that handles the auth code response and requests an access token. This is why we set the "Application URL" earlier
      2. This should result in a Client ID and Client Secret
   2. In PocketBase go to the Auth Providers setting in the left menu
   3. Select the sprocket icon for the auth providers you have set up and plug in the Client ID and Secret (and other information you may need. Microsoft is a more complex example)
   4.  Navigate to `http://localhost:5173/login` 
      1. You can try `http://localhost:5173/protected` and see it kicks you back to `/login` and `/unprotected` and see the page
   5. Attempt login with your OAuth provider
   6. You should be redirected to `http://localhost:5173/` and navigating to `/login` brings you to `/` and `/protected` should reveal a secret

⚠️ Unless you really know what you're doing, avoid messing with `hooks.server.ts` or `routes/login/+page.server.ts` as that's where most of the auth magic happens

## Going to Production - Pocketbase
1. Deploy PocketBase to a server per PocketBase instructions. I build and push a docker image with PB on it to ECR and run it on a VM in the cloud (see docker-compose.yml for example)
2. Configure the production PocketBase instance in a similar fashion to local development. The only differences are you'll use your domain instead of localhost `mydomain.com/_` to get into the admin panel for PocketBase and `mydomain.com` for the Application name settings.
3. Add production domains (e.g. mydomain.com/callback) to your OAuth provider callback url list

## Going to Production - Client
1. Deploy your SvelteKit app using your prefered deployment flow. I use Vercel and it deploys off of changes to `main`. 

## You Should Know
1. This uses cookie based JWTs and therefore cookies are only shared if you're using https, and only over http (not accessable via javascript). Therefore ensure your frontend and backend share domains.
2. You can set the cookie samesite setting to something other than lax if you don't want to use OAuth. Lax is required for OAuth since your app will be using cookies while talking to 3rd party services.
3. Pocketbase uses a different signing secret per session, so on the backend when validating the JWT on the cookie, a HTTP call to PocketBase via the SDK or their API is required.
4. The design philosophy behind this was to get decent auth working in a quickly repeatable fashion using the cheapest setup possible. This is in no way suggested for enterprise grade auth flows, but if you have less than 10k daily active users it _should_ be fine. Lets be real, if you have 10k daily active users, go get funding and make scale and enterprise shit someone else's problem 😉
