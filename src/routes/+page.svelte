<script>
	import { ApolloClient, InMemoryCache, HttpLink, gql } from '@apollo/client/core';
	import { setClient, query } from 'svelte-apollo';

	const hasuraAdminSecret = 'your_hasura_admin_secret'; // Place this in ENV
	let chatMessages;

	$: {
		if ($messages.data && $messages.data.chat_messages) {
			chatMessages = $messages.data.chat_messages;
		}
	}

	function createApolloClient() {
		const link = new HttpLink({
			uri: 'https://api.yourdomain.com/v1/graphql', // Add your Hasura endpoint
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
			chat_messages(
				where: { _or: { receiver_id: { _eq: 1 }, sender_id: { _eq: 1 } } }
				order_by: { date_created: desc }
			) {
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

<h1>Messages</h1>
<div class="chat-container">
	{#if $messages.loading}
		<p>Loading...</p>
	{:else if chatMessages}
		{#each chatMessages as message (message.date_created)}
			<div class="chat-message">
				<p>
					<strong>{message.user.first_name}:</strong>
					<span>{message.message_text}</span>
				</p>
				<p>
					<small>Created at: {message.date_created}</small>
					<small>Updated at: {message.date_updated}</small>
				</p>
			</div>
		{/each}
	{:else}
		<p>No messages found</p>
	{/if}
	{#if $messages.error}
		<p>Something went wrong: {$messages.error.message}</p>
	{/if}
</div>
