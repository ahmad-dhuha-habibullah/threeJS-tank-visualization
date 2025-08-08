<!-- src/routes/+page.svelte -->
<script>
  import TankVisualization from '$lib/TankVisualization.svelte';
  import { onMount } from 'svelte';

  let waterLevel = 0; // Start with an initial level

  // --- THIS IS WHERE YOUR TEAMMATE CONNECTS TO THE DATABASE ---
  onMount(() => {
    // Example 1: Set the level manually
    waterLevel = 65; // Manually set to 65%

    /*
    // Example 2: Simulate fetching data from a database API
    // Your teammate would replace this with a real 'fetch' call to your Django API.
    setTimeout(() => {
      const fetchedLevel = 88.4; // Imagine this value came from the database
      waterLevel = fetchedLevel;
    }, 2000); // Simulate a 2-second delay
    */
  });

</script>

<main>
  <div class="tank-container">
    <TankVisualization level={waterLevel} />
  </div>
</main>

<style>
  main {
    width: 100vw;
    height: 100vh;
  }
  .tank-container {
    width: 100%;
    height: 100%;
  }
</style>
