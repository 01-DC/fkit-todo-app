<script>
	import { firebaseApp, authProvider, db, authoriser } from "$lib/firebase"
	import {
		collection,
		onSnapshot,
		doc,
		updateDoc,
		deleteDoc,
		addDoc
	} from "firebase/firestore"
	import { signInWithPopup } from "firebase/auth"

	import { browser } from "$app/env"

	// const firebaseApp =
	// 	browser && getApps().length <= 1
	// 		? initializeApp(firebaseConfig)
	// 		: getApp()

	const colRef = browser && collection(db, "todos")
	let todos = []

	const unsubscribe =
		browser &&
		onSnapshot(colRef, (querySnapshot) => {
			let fbTodos = []
			querySnapshot.forEach((doc) => {
				fbTodos.push({ ...doc.data(), id: doc.id })
			})
			todos = fbTodos
			console.table(todos)
		})

	console.log({ firebaseApp, db })

	let task = ""
	let error = ""

	const addTodo = async () => {
		if (task !== "") {
			await addDoc(colRef, {
				task: task,
				isComplete: false,
				createdAt: new Date()
			})
			error = ""
		} else error = "Task is empty"
		task = ""
	}

	const markTodoAsComplete = async (item) => {
		await updateDoc(doc(db, "todos", item.id), {
			isComplete: !item.isComplete
		})
	}

	const deleteTodo = async (id) => {
		await deleteDoc(doc(db, "todos", id))
	}

	const keyIsPressed = (event) => {
		if (event.key === "Enter") addTodo()
	}

	const login = async () => {
		try {
			const res = await signInWithPopup(authoriser, authProvider)
			console.log(res.user)
		} catch (error) {
			console.log(error.code)
			console.log(error.message)
		}
	}
</script>

<button on:click={login}>Login</button>
<h1>Todos</h1>
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

<svelte:window on:keydown={keyIsPressed} />

<style>
	.complete {
		text-decoration: line-through;
	}
	.error {
		color: red;
	}
</style>
