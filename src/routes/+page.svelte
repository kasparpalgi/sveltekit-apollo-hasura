<script>
	import { ApolloClient, InMemoryCache, HttpLink, gql } from '@apollo/client/core';
	import { setClient, query, mutation } from 'svelte-apollo';

	const hasuraAdminSecret = 'your_hasura_admin_secret'; // Place this in ENV
	let chatMessages;
	let messageInput = '';
	let senderId = 1;
	let receiverId = 8;

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

	const ADD_MESSAGE = gql`
		mutation AddMessage(
			$sender_id: Int!
			$receiver_id: Int
			$group_id: Int
			$message_text: String!
		) {
			insert_chat_messages_one(
				object: {
					sender_id: $sender_id
					receiver_id: $receiver_id
					group_id: $group_id
					message_text: $message_text
				}
			) {
				id
			}
		}
	`;

	const client = createApolloClient();
	setClient(client);
	const messages = query(GET_MESSAGES);
	const addMessageMutation = mutation(ADD_MESSAGE);

	async function addMessage() {
		try {
			await addMessageMutation({
				variables: { sender_id: senderId, receiver_id: receiverId, message_text: messageInput },
			});
			console.log('Message sent');
			messageInput = '';
		} catch (error) {
			console.log('Message send error');
		}
	}

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
					<strong>{message.user.first_name} said:</strong>
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
	<form class="formInput" on:submit|preventDefault={addMessage}>
		<input class="input" placeholder="Type your message…" bind:value={messageInput} />
		<i class="inputMarker fa fa-angle-right" />
		<button type="submit">Send</button>
	</form>
</div>
