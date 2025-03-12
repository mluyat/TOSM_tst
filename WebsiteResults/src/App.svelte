<script>
	import Plot from './Plot.svelte';
  </script>
  
  <main>
	<h1>How to combine Machine Learning and ecodesign to find the best structures</h1>
	<Plot />
  </main>

<style>
	 @import url('https://fonts.googleapis.com/css2?family=Aptos+Narrow:wght@700&display=swap');
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
        color: #020202;
        text-transform: uppercase;
        font-size: 3em;
        font-weight: bold;
        font-family: 'Aptos Narrow', sans-serif;

    }
	

	

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>