<script context="module" lang="ts">
  import { enhance } from '$lib/form';
  import Example from '$lib/Example.svx';
	import type { Load } from '@sveltejs/kit';

  async function fetchTodoList({ fetch }){
		const res = await fetch('/todos.json');

		if (res.ok) {
			const todos = await res.json();

			return {
				props: { todos }
			};
		}

		const { message } = await res.json();

		return {
			error: new Error(message)
		};
	}

	// see https://kit.svelte.dev/docs#loading
  export const load: Load = fetchTodoList;
</script>

<script lang="ts">
  import type { TodoT } from '../../components/Todo.d';
  
  export let todos: TodoT[];

  import Todo from '../../components/Todo.svelte';

  async function addTodo(res: Response, form: HTMLFormElement){
		const created = await res.json();
		todos = [...todos, created]

		form.reset();
	}

	async function patch(res: Response) {
		const newTodo = await res.json();

		todos = todos.map(oldTodo => {
			if (oldTodo.uid === newTodo.uid) return newTodo;
			return oldTodo;
		});
  }

  const afterDelete = (todoUid: string) => {
    todos = todos.filter(t => t.uid !== todoUid);
  }
</script>


<svelte:head>
	<title>Todos</title>
</svelte:head>

<Example />

<div class="todos">
	<h1>Todos</h1>

	<form class="new" action="/todos.json" method="post" use:enhance={{
		result: addTodo
	}}>
		<input name="text" aria-label="Add todo" placeholder="+ tap to add a todo">
	</form>

  {#each todos as todo (todo.uid)}
    <Todo todo={todo} patch={patch} afterDelete={afterDelete}/>
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
		box-shadow: inset 1px 1px 6px rgba(0,0,0,0.1);
		border: 1px solid #ff3e00 !important;
		outline: none;
	}

	.new input {
		font-size: 28px;
		width: 100%;
		padding: 0.5em 1em 0.3em 1em;
		box-sizing: border-box;
		background: rgba(255,255,255,0.05);
		border-radius: 8px;
		text-align: center;
	}

</style>
