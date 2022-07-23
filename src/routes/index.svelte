<script>
	import { firebaseApp, authProvider, db, auth } from "$lib/firebase"
	import { collection, onSnapshot } from "firebase/firestore"
	import { signInWithPopup, signOut } from "firebase/auth"
	import { isLoggedIn, user, colRef } from "$lib/stores"
	import Todos from "../components/Todos.svelte"

	// const firebaseApp =
	// 	browser && getApps().length <= 1
	// 		? initializeApp(firebaseConfig)
	// 		: getApp()

	let todos = []

	const loadTodos = async () => {
		onSnapshot($colRef, (querySnapshot) => {
			let fbTodos = []
			querySnapshot.forEach((doc) => {
				fbTodos.push({ ...doc.data(), id: doc.id })
			})
			todos = fbTodos
			// console.table(todos)
		})
	}
	// console.log({ firebaseApp, db })

	const login = async () => {
		try {
			const res = await signInWithPopup(auth, authProvider)
			$isLoggedIn = true
			$user = res.user

			$colRef = collection(db, `users/${$user.uid}/todos`)
			await loadTodos()
		} catch (error) {
			console.log(error.code)
			console.log(error.message)
		}
	}

	const logout = async () => {
		try {
			const res = await signOut(auth)
			$isLoggedIn = false
			$user = {}
			todos = []
		} catch (error) {
			console.log(error.code)
			console.log(error.message)
		}
	}
</script>

{#if $isLoggedIn}
	<div>
		<Todos {todos} />
		<button on:click={logout} class="logout-button">Logout</button>
	</div>
{:else}
	<div class="login-wrapper">
		<h1>Please login to add todos</h1>
		<button on:click={login} class="login-button">Google Login</button>
	</div>
{/if}

<style>
	.login-wrapper {
		margin: auto;
	}

	.login-button {
		padding: 8px;
		font-family: inherit;
		font-size: 1.3rem;
		border: none;
		background: #fff;
		margin-top: 16px;
		transition: all 300ms ease-out;
		cursor: pointer;
	}

	.login-button:hover {
		background-color: rgb(255, 52, 52);
		color: white;
		transform: translateY(-8px);
	}

	.logout-button {
		color: rgb(255, 52, 52);
		font-family: inherit;
		text-decoration: underline;
		background-color: #000;
		font-size: 1rem;
		cursor: pointer;
		margin-top: 16px;
	}
</style>
