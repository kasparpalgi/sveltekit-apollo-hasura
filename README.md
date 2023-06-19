# SvelteKit & Hasura subscriptions with Apollo

- https://youtu.be/8SCgCkHfZyQ
- https://hasura.io/learn/graphql/svelte-apollo

Add to existing SvelteKit project: `npm install --save isomorphic-ws ws @apollo/client/core graphql-ws svelte-apollo`

or

Just clone and `npm install` and then add your Hasura API endpoint and admin secret to `/src/routes/susbscriptions/+page.svelte` and then `npm run dev` and subscriptions can be seen at: http://localhost:5174/subscriptions

TODO:

- We have at the moment just all messages - want to see first the list of users I have messages with and groups I belong and when click on it then will open messages just with that user or messages in that group
  - ~~Create table "chat_heads" and relationships~~
  - Subscribe to "chat_heads" (query below)
  - Make chat heads clickable
- Send message & optimistic UI updates after mutations
- Update lastSeen

## "chat_heads" subscription

```
subscription GetChatHeadsStreamingSubscription($user: Int!) {
  chat_heads(where: {_or: [{sender_id: {_eq: $user}}, {receiver_id: {_eq: $user}}, {group_id: {_eq: $user}}]}, order_by: {date_created: desc}) {
    id
    message_text
    date_created
    date_updated
    user {
      id
      first_name
      profile_pic
    }
    userBySenderId {
      id
      first_name
      profile_pic
    }
    group {
      id
      group_name
      pic_url
    }
  }
}
```
