<script>
    import { ApolloClient, InMemoryCache, HttpLink, gql } from '@apollo/client/core';
    import { setClient, query } from 'svelte-apollo';
  
    const hasuraAdminSecret = 'woofi2023#hasura';
  
    function createApolloClient() {
      const link = new HttpLink({
        uri: 'https://api.woofi.ai/v1/graphql',
        headers: {
          'X-Hasura-Admin-Secret': hasuraAdminSecret
        }
      });
  
      const cache = new InMemoryCache();
  
      return new ApolloClient({
        link,
        cache
      });
    }
  
    const GET_MESSAGES = gql`
    query MyMessages {
      chat_messages(where: {_or: {receiver_id: {_eq: 1}, sender_id: {_eq: 1}}}, order_by: {date_created: desc}) {
        user {
          first_name
        }
        userBySenderId {
          first_name
        }
        group {
          group_name
        }
        message_text
        date_created
        date_updated
      }
    }
    `;
  
    const client = createApolloClient();
    setClient(client);
    const messages = query(GET_MESSAGES);
  
    $: console.log($messages);
  </script>
  
  


<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>
