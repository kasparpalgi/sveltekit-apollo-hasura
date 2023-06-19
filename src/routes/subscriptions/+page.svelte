<script>
	import { createClient } from 'graphql-ws';
	import {
		gql,
		ApolloClient,
		split,
		InMemoryCache,
		HttpLink,
		ApolloLink,
		Observable
	} from '@apollo/client/core';
	import { getMainDefinition } from '@apollo/client/utilities';
	import { onDestroy } from 'svelte';
	import { print } from 'graphql';
	import WebSocket from 'isomorphic-ws';

	const SECRET = 'your_hasura_admin_secret';
	const GRAPHQL_URI = 'https://api.yourdomain.com/v1/graphql';
	const WS_URI = 'wss://api.yourdomain.com/v1/graphql';
	const BATCH_SIZE = 10;

	const cache = new InMemoryCache({
		typePolicies: {
			Subscription: {
				fields: {
					chat_messages_stream: {
						merge: (existing = [], incoming) => [...existing, ...incoming]
					}
				}
			}
		}
	});

	const wsClient = createClient({
		url: WS_URI,
		webSocketImpl: WebSocket,
		connectionParams: {
			headers: { 'X-Hasura-Admin-Secret': SECRET }
		}
	});

	const wsLink = new ApolloLink(
		(operation) =>
			new Observable((observer) => {
				const { query, ...others } = operation;
				const unsubscribe = wsClient.subscribe({ query: print(query), ...others }, observer);
				return unsubscribe;
			})
	);

	const httpLink = new HttpLink({ uri: GRAPHQL_URI });
	const link = split(
		({ query }) => {
			const { kind, operation } = getMainDefinition(query);
			return kind === 'OperationDefinition' && operation === 'subscription';
		},
		wsLink,
		httpLink
	);

	const client = new ApolloClient({ link, cache });
	const query = gql`
		subscription GetChatMessagesStreamingSubscription {
			chat_messages_stream(batch_size: ${BATCH_SIZE}, cursor: { initial_value: { id: 0 } }) {
				group_id
				id
				receiver_id
				sender_id
				message_text
				date_created
				date_updated
			}
		}
	`;

	let chatMessages = [];
	const subscription = client.subscribe({ query }).subscribe({
		next: ({ data }) => data && (chatMessages = data.chat_messages_stream),
		error: (err) => console.error('Subscription error', err)
	});

	onDestroy(() => subscription.unsubscribe());
</script>

<h1>Messages</h1>
<div class="chat-container">
	{#each chatMessages as message (message.id)}
		<div class="chat-message">
			<p><strong>Sender:</strong> {message.sender_id}</p>
			<p><strong>Receiver:</strong> {message.receiver_id}</p>
			<p><strong>Message:</strong> {message.message_text}</p>
			<p><strong>Date created:</strong> {message.date_created}</p>
			<p><strong>Date updated:</strong> {message.date_updated}</p>
		</div>
	{/each}
</div>
