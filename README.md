# Internet Identity (II) Alternative Origins

When signing in with II into an app, the principal you get depends on the current URL (e.g. `https://canister-id.icp0.io`) you're accessing the site from.
If you use another URL to access your site, like `https://canister-id.icp1.io`, or a custom domain like `https://example.com`, you will get a different principal from II when signing in.

To make sure users get the same principal independent of whether they access your app from e.g., `https://example.com` or from `https://canister-id.icp0.io`, you can leverage the derivation origin together with the alternative domains feature.

In short, you can tell II to always pretend you're signing in from `https://canister-id.icp0.io`, even if the user is on `https://example.com`. 
To do that, all you need to do is in the `authClient`, set the `derivationOrigin` to `https://canister-id.icp0.io`.

In the following, make sure to replace the canister IDs / URLs with the ones you wish to use.
To test, you can use `https://canister-id.icp0.io` and `https://canister-id.icp1.io`.
Make sure to deploy your app once to see which canister ID you have, and fill it in accordingly.

For example, in this app, we set in `frontend/src/App.jsx` on line 47:

```js
await state.authClient.login({
  derivationOrigin: "https://brj6x-syaaa-aaaaf-qaqjq-cai.icp0.io", // Always use this to derive canister IDs!
  identityProvider,
  onSuccess: updateActor
});
```

But that is only part of the story. 
To make this work, when users log in to your app on `https://example.com`, II needs to know that it is actually allowed to pretend you're signing in from `https://canister-id.icp0.io`.
To show II that this is okay, you need to set a file in `.well-known/ii-alternative-origins`.
In there, we specify which domains are allowed to login on behalf of what we specified in the `derivationOrigin` above, namely on behalf of `https://canister-id.icp0.io`.

So, we need to set the file like this:

```json
{ 
  "alternativeOrigins": ["https://example.com"]
}
```

Now, when you log in from `https://example.com`, it will use the derivation origin `https://brj6x-syaaa-aaaaf-qaqjq-cai.icp0.io` that we set in the `authClient` above.
It will also check if it is actually allowed to use that derivation origin by looking if the current URL you're accessing the site from (`https://example.com`) is in the list of alternative origins!
Since that is the case, it will now happily sign you in as if you were on `https://brj6x-syaaa-aaaaf-qaqjq-cai.icp0.io`, and give you the same principal.

There is one more file we need for this to work, though.
We need a `.ic-assets.json5` file like this to make sure the `.well-known` is not ignored and sends the correct headers when requested:

```json
[
  {
    "match": ".well-known",
    "ignore": false
  },
  {
    "match": ".well-known/ii-alternative-origins",
    "headers": {
      "Access-Control-Allow-Origin": "*",
      "Content-Type": "application/json"
    },
    "ignore": false
  }
]
```

With that, you can sign in from multiple URLs and get the same principal on each of them!

More info on alternative origins [here](https://internetcomputer.org/docs/building-apps/authentication/alternative-origins), and on the derivation origin [here](https://internetcomputer.org/docs/references/ii-spec#client-authentication-protocol).


## Deploying from ICP Ninja

When viewing this project in ICP Ninja, you can deploy it directly to the mainnet for free by clicking "Run" in the upper right corner. Open this project in ICP Ninja:

[![](https://icp.ninja/assets/open.svg)](https://icp.ninja/i?g=https://github.com/gwapupward-hub/Internet-Identity-Alternative-Origins)

## Build and deploy from the command-line

### 1. [Download and install the IC SDK.](https://internetcomputer.org/docs/building-apps/getting-started/install)

### 2. Download your project from ICP Ninja using the 'Download files' button on the upper left corner, or [clone the GitHub examples repository.](https://github.com/dfinity/examples/)

### 3. Navigate into the project's directory.

### 4. Deploy the project to your local environment:

```
dfx start --background --clean && dfx deploy
```

## Security considerations and best practices

If you base your application on this example, it is recommended that you familiarize yourself with and adhere to the [security best practices](https://internetcomputer.org/docs/building-apps/security/overview) for developing on ICP. This example may not implement all the best practices.
