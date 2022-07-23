<script>
	export let todos = undefined
	import { db } from "$lib/firebase"
	import { doc, updateDoc, deleteDoc, addDoc } from "firebase/firestore"
	import { isLoggedIn, user, colRef } from "$lib/stores"

	let task = ""
	let error = ""

	const addTodo = async () => {
		if (task !== "") {
			await addDoc($colRef, {
				task: task,
				isComplete: false,
				createdAt: new Date()
			})
			error = ""
		} else error = "Task is empty"
		task = ""
	}

	const markTodoAsComplete = async (item) => {
		await updateDoc(doc(db, `users/${$user.uid}/todos`, item.id), {
			isComplete: !item.isComplete
		})
	}

	const deleteTodo = async (id) => {
		await deleteDoc(doc(db, `users/${$user.uid}/todos`, id))
	}

	const keyIsPressed = (event) => {
		if (event.key === "Enter" && $isLoggedIn) addTodo()
	}
</script>

<div class="todo-wrapper">
	<h1>Todos 📗</h1>
	<div class="input-wrapper">
		<input type="text" bind:value={task} placeholder="Add a task" />
		<button on:click={addTodo} class="add-button">Add</button>
	</div>

	<div class="todos-list">
		{#each todos as todo}
			<div class="list-item">
				<div>
					<button
						class="list-okay"
						on:click={() => markTodoAsComplete(todo)}>✓</button
					>
					<span class:complete={todo.isComplete}>
						{todo.task}
					</span>
				</div>
				<span>
					<button
						class="list-del"
						on:click={() => deleteTodo(todo.id)}>✗</button
					>
				</span>
			</div>
		{:else}
			<p>No todos found</p>
		{/each}

		{#if error.length > 0}
			<p class="error">{error}</p>
		{/if}
	</div>
</div>
<svelte:window on:keydown={keyIsPressed} />

<style>
	.todo-wrapper {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	.add-button {
		padding: 11px;
		font-family: inherit;
		font-size: 1rem;
		background-color: white;
		cursor: pointer;
		border: none;
		transition: all 300ms ease-out;
	}
	.add-button:hover {
		color: white;
		background-color: #ff3434;
		transform: translateY(-4px);
	}
	.todos-list {
		max-width: 500px;
		text-align: left;
		margin: 16px 0;
	}
	.list-item {
		display: flex;
		margin: 10px 0;
		font-size: 1.1rem;
		justify-content: space-between;
		width: 100%;
	}

	.list-del {
		background-color: black;
		color: #ff3434;
		font-size: 1rem;
		cursor: pointer;
		border: 1px solid #ff3434;
		margin-left: 10px;
	}
	.list-okay {
		background-color: black;
		color: #34ff34;
		font-size: 1rem;
		cursor: pointer;
		border: 1px solid #34ff34;
	}

	.complete {
		text-decoration: line-through;
		color: gray;
	}

	.error {
		color: red;
		border: 1px solid red;
		width: fit-content;
		padding: 8px;
		width: 100%;
		text-align: center;
	}

	input {
		padding: 10px;
		outline: none;
		width: 250px;
		font-family: inherit;
		font-size: 1rem;
	}

	input:focus {
		outline: 2px solid #ff3434;
	}
</style>
