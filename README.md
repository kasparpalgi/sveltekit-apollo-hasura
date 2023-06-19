# SvelteKit & Hasura subscriptions with Apollo

- https://youtu.be/8SCgCkHfZyQ
- https://hasura.io/learn/graphql/svelte-apollo

Add to existing SvelteKit project: `npm install --save isomorphic-ws ws @apollo/client/core graphql-ws svelte-apollo`

or

Just clone and `npm install` and then add your Hasura API endpoint and admin secret to `/src/routes/susbscriptions/+page.svelte` and then `npm run dev` and live query subscriptions for "chat_heads" can be seen at: http://localhost:5174/ and [streaming subscriptions](https://hasura.io/docs/latest/subscriptions/postgres/streaming/index/) can be seen at: http://localhost:5174/subscriptions and at https://github.com/crewnew-git/query-mutation I just tested out first a simple query and mutation with Apollo before I implemented the susbscriptions.

TODO:

- We have at the moment just all messages - want to see first the list of users I have messages with and groups I belong and when click on it then will open messages just with that user or messages in that group
  - ~~Create table "chat_heads" and relationships~~
  - ~~Subscribe to "chat_heads" (query below)~~
  - Make chat heads clickable to open the actual chat head
- Send message & optimistic UI updates after mutations
- Set up DB triggers to update "chat_heads" table
- Update lastSeen