<script lang="ts">
	import Form from '$lib/Form.svelte';
	import Todo, { type TodoType } from '$lib/Todo.svelte';
	import { flip } from 'svelte/animate';
	import { scale } from 'svelte/transition';

	export let todos: TodoType[];
	let pendingTodos: string[] = [];

	$: allTodos =
		pendingTodos.length !== 0
			? [
					...todos,
					...pendingTodos.map<TodoType>((pendingTodo) => ({
						uid: pendingTodo,
						created_at: new Date(),
						done: false,
						disabled: true,
						text: pendingTodo
					}))
			  ]
			: todos;

	let createInput: HTMLInputElement;
</script>

<svelte:head>
	<title>Todos</title>
	<meta name="description" content="A todo list app" />
</svelte:head>

<div class="todos">
	<h1>Todos</h1>

	<Form
		method="post"
		on:submit={({ detail }) => {
			const text = detail.submitted.get('text');
			if (typeof text === 'string') {
				pendingTodos.push(text);
				pendingTodos = pendingTodos;
			}
			detail.form.reset();
			createInput.focus();
		}}
		on:complete={({ detail: { submitted } }) => {
			const text = submitted.get('text');
			console.log(text);

			pendingTodos = pendingTodos.filter((pending) => pending !== text?.toString());
		}}
	>
		<div class="new">
			<input
				name="text"
				aria-label="Add todo"
				placeholder="+ tap to add a todo"
				bind:this={createInput}
			/>
		</div>
	</Form>

	{#each allTodos as todo (todo.uid)}
		<div
			animate:flip={{ duration: 200 }}
			in:scale|local={{ start: 0.7, duration: todo.uid === todo.text ? 200 : 0 }}
			out:scale|local={{ start: 0.7, duration: todo.uid !== todo.text ? 200 : 0 }}
		>
			<Todo {todo} />
		</div>
	{/each}
</div>

<style>
	.todos {
		width: 100%;
		max-width: var(--column-width);
		margin: var(--column-margin-top) auto 0 auto;
		line-height: 1;
	}

	.new {
		margin: 0 0 0.5rem 0;
	}

	input {
		border: 1px solid transparent;
	}

	input:focus-visible {
		box-shadow: inset 1px 1px 6px rgba(0, 0, 0, 0.1);
		border: 1px solid #ff3e00 !important;
		outline: none;
	}

	.new input {
		font-size: 28px;
		width: 100%;
		padding: 0.5em 1em 0.3em 1em;
		box-sizing: border-box;
		background: rgba(255, 255, 255, 0.05);
		border-radius: 8px;
		text-align: center;
	}
</style>
