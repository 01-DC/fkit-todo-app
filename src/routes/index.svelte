<script>
	import { firebaseApp, authProvider, db, auth } from "$lib/firebase"
	import {
		collection,
		onSnapshot,
		doc,
		updateDoc,
		deleteDoc,
		addDoc,
		getDoc,
		getDocs,
		setDoc
	} from "firebase/firestore"
	import { signInWithPopup, signOut } from "firebase/auth"
	import { isLoggedIn, user, colRef } from "$lib/stores"
	import { browser } from "$app/env"
	import Todos from "../components/Todos.svelte"

	// const firebaseApp =
	// 	browser && getApps().length <= 1
	// 		? initializeApp(firebaseConfig)
	// 		: getApp()

	$colRef = browser && collection(db, "users/cJ5V3hgAaUSvpGKxVYpF/todos")

	let todos = []

	const loadTodos = async () => {
		onSnapshot($colRef, (querySnapshot) => {
			let fbTodos = []
			querySnapshot.forEach((doc) => {
				fbTodos.push({ ...doc.data(), id: doc.id })
			})
			todos = fbTodos
			console.table(todos)
		})
	}
	console.log({ firebaseApp, db })

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
</style>
