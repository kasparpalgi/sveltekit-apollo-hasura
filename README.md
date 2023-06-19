# SvelteKit & Hasura subscriptions with Apollo

* https://youtu.be/8SCgCkHfZyQ
* https://hasura.io/learn/graphql/svelte-apollo

Add to existing SvelteKit project: `npm install --save isomorphic-ws ws @apollo/client/core graphql-ws svelte-apollo`

or 

Just clone and `npm install` and then add your Hasura API endpoint and admin secret to `/src/routes/susbscriptions/+page.svelte` and then `npm run dev` and subscriptions can be seen at: http://localhost:5174/subscriptions

TODO:
* Fix: see only messages I have sent / received
* Send message & optimistic UI updates after mutations
* Update lastSeen
* Add Hasura migrations/metadata

deprecated subscriptions-transport-ws@0.11.0: The `subscriptions-transport-ws` package is no longer maintained. We recommend you use `graphql-ws` instead. For help migrating Apollo software to `graphql-ws`, see https://www.apollographql.com/docs/apollo-server/data/subscriptions/#switching-from-subscriptions-transport-ws    For general help using `graphql-ws`, see https://github.com/enisdenjo/graphql-ws/blob/master/README.md

# create-svelte

Everything you need to build a Svelte project, powered by [`create-svelte`](https://github.com/sveltejs/kit/tree/master/packages/create-svelte).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
npm create svelte@latest

# create a new project in my-app
npm create svelte@latest my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.
