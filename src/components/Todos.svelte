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

<div>
	<h1>Todos 📗</h1>
	<input type="text" bind:value={task} placeholder="Add a task" />
	<button on:click={addTodo}>Add</button>

	<ol>
		{#each todos as todo}
			<li class:complete={todo.isComplete}>
				<span>
					{todo.task}
				</span>
				<span>
					<button on:click={() => markTodoAsComplete(todo)}>✓</button>
					<button on:click={() => deleteTodo(todo.id)}>✗</button>
				</span>
			</li>
		{:else}
			<p>No todos found</p>
		{/each}
		<p class="error">{error}</p>
	</ol>
</div>
<svelte:window on:keydown={keyIsPressed} />

<style>
	.complete {
		text-decoration: line-through;
	}
	.error {
		color: red;
	}
	input {
		padding: 8px;
	}
</style>
