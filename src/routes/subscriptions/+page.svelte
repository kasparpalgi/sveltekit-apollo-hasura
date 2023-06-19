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

	const hasuraAdminSecret = 'your_hasura_admin_secret';
	let subscription;

	const cache = new InMemoryCache({
		typePolicies: {
			Subscription: {
				fields: {
					chat_messages_stream: {
						merge(existing, incoming) {
							return existing ? [...existing, ...incoming] : [...incoming];
						}
					}
				}
			}
		}
	});

	const wsClient = createClient({
		url: 'wss://api.yourdomain.com/v1/graphql',
		webSocketImpl: WebSocket,
		connectionParams: () => ({
			headers: {
				'X-Hasura-Admin-Secret': hasuraAdminSecret
			}
		})
	});

	const wsLink = new ApolloLink(
		(operation) =>
			new Observable((observer) => {
				// Ensure that the query is a string
				const { query, ...others } = operation;
				const newOperation = {
					query: print(query),
					...others
				};

				const unsubscribe = wsClient.subscribe(newOperation, {
					next: observer.next.bind(observer),
					error: observer.error.bind(observer),
					complete: observer.complete.bind(observer)
				});

				return () => {
					if (unsubscribe) {
						unsubscribe();
					}
				};
			})
	);

	const httpLink = new HttpLink({ uri: 'https://api.woofi.ai/v1/graphql' });

	const link = split(
		({ query }) => {
			const definition = getMainDefinition(query);
			return definition.kind === 'OperationDefinition' && definition.operation === 'subscription';
		},
		wsLink,
		httpLink
	);

	const client = new ApolloClient({
		link,
		cache
	});

	const GET_CHAT_MESSAGES_SUBSCRIPTION = gql`
		subscription GetChatMessagesStreamingSubscription {
			chat_messages_stream(batch_size: 10, cursor: { initial_value: { id: 0 } }) {
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

	const observable = client.subscribe({ query: GET_CHAT_MESSAGES_SUBSCRIPTION });

	subscription = observable.subscribe({
		next({ data }) {
			if (data) {
				chatMessages = data.chat_messages_stream;
			}
		},
		error(err) {
			console.error('Subscription error', err);
		}
	});

	onDestroy(() => {
		if (subscription) {
			subscription.unsubscribe();
		}
	});
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
