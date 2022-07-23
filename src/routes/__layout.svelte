<!-- <h1>Layout</h1> -->
<div id="svelte-app">
	<slot />
	<footer>
		Made by <span style:color="#ff3434">DC</span> with
		<span style:color="#ff3434">&lt;3</span>
	</footer>
</div>

<style>
	#svelte-app {
		font-family: "Menlo", "Consolas", "Monaco", "Liberation Mono",
			"Lucida Console", monospace;
		padding: 0;
		margin: 0;
		box-sizing: border-box;
		min-height: 100vh;
		background-color: #000;
		text-align: center;
		display: grid;
		color: white;
	}

	footer {
		margin-top: auto;
		margin-bottom: 10vh;
	}
</style>
