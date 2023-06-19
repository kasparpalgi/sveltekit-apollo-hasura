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
	const USER_ID = 1; // Change it to your actual user id

	const cache = new InMemoryCache({
		typePolicies: {
			Subscription: {
				fields: {
					chat_heads: {
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

	const GET_CHAT_MESSAGES_SUBSCRIPTION = gql`
		subscription GetChatHeadsStreamingSubscription($user: Int!) {
			chat_heads(
				where: {
					_or: [
						{ sender_id: { _eq: $user } }
						{ receiver_id: { _eq: $user } }
						{ group_id: { _eq: $user } }
					]
				}
				order_by: { date_created: desc }
			) {
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
	`;

	let chatMessages = [];

	const subscription = client
		.subscribe({ query: GET_CHAT_MESSAGES_SUBSCRIPTION, variables: { user: USER_ID } })
		.subscribe({
			next: ({ data }) => data && (chatMessages = data.chat_heads),
			error: (err) => console.error('Subscription error', err)
		});

	onDestroy(() => subscription.unsubscribe());
</script>

<h1>Chat heads</h1>
<div class="chat-container">
	{#each chatMessages as message (message.id)}
		<div class="chat-message">
			<p><strong>Sender:</strong> {message.userBySenderId?.first_name}</p>
			{#if message.user?.first_name}
				<p><strong>Receiver:</strong> {message.user?.first_name}</p>
			{/if}
			{#if message.group?.group_name}
				<p><strong>Group:</strong> {message.group?.group_name}</p>
			{/if}
			<p><strong>Message:</strong> {message.message_text}</p>
			<p><strong>Date created:</strong> {message.date_created}</p>
			<p><strong>Date updated:</strong> {message.date_updated}</p>
		</div>
	{/each}
</div>
