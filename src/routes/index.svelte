<script>
	import { authProvider, db, auth } from "$lib/firebase"
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
		})
	}

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
		<h1>
			A todo app built using <span style:color="#ff3434">FKIT</span> stack.
		</h1>
		<h1 style:color="#ffcc30">
			<span style:color="#ff3434">F</span>irebase
			<span style:color="white">&amp;</span>
			Svelte<span style:color="#ff3434">Kit</span>
		</h1>
		<h2>Please login to add todos</h2>
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
		margin-top: 20px;
		transition: all 300ms ease-out;
		cursor: pointer;
	}

	.login-button:hover {
		background-color: #ffcc30;
		transform: translateY(-8px);
	}

	.logout-button {
		color: #ffcc30;
		font-family: inherit;
		text-decoration: underline;
		background-color: #000;
		font-size: 1rem;
		cursor: pointer;
		margin-top: 16px;
	}
</style>
