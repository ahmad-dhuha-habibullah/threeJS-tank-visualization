3D Svelte Liquid Tank Visualization
This project is a Svelte component that displays a realistic, real-time 3D visualization of a liquid tank. It is designed to be easily integrated into a larger application and connected to a live data source.

How to Test the Code
Follow these steps to run the project on your local machine.

Prerequisites
You must have Node.js installed, which includes npm.

Setup and Installation
Clone the Repository Open your terminal and run the following command to clone the project files from GitHub:

git clone https://github.com/ahmad-dhuha-habibullah/threeJS-tank-visualization.git

Navigate to Project Directory

cd svelte-tank-app

Install Dependencies
This will download all the necessary packages, including SvelteKit and three.js.

npm install

Run the Development Server

npm run dev

View the App
Open your web browser and navigate to http://localhost:5173. You should see the 3D tank visualization running.

Key Files
There are two main files you need to be aware of:

src/lib/TankVisualization.svelte: This is the core 3D component. It is a self-contained module that only cares about one thing: rendering the tank based on the level property it receives. You should not need to edit this file.

src/routes/+page.svelte: This is the main page that hosts and tests the visualization component. This is the file you will edit to connect to your real database. It shows how to pass the waterLevel variable to the component's level prop.

Connecting to a Live Database
The component is ready to be connected to a live data source. All the connection logic should be handled in the parent component (src/routes/+page.svelte).

Open src/routes/+page.svelte. You will see this section in the <script> tag:

<script>
  import TankVisualization from '$lib/TankVisualization.svelte';
  import { onMount } from 'svelte';

  let waterLevel = 0; // The variable that controls the tank's level.

  // --- THIS IS WHERE YOU CONNECT TO THE DATABASE ---
  onMount(() => {
    // To connect to your Django API, you would use a `fetch` call here.
    // For example:
    /*
    async function getTankLevel() {
      const response = await fetch('https://your-api.com/tank/level');
      const data = await response.json();
      waterLevel = data.level; // Update the variable with data from the API
    }
    getTankLevel();
    */

    // For now, it's set manually for demonstration:
    waterLevel = 65;
  });
</script>

To connect your real-time data:

For initial data: Replace the manual waterLevel = 65; line with a fetch call to your Django API endpoint inside the onMount function.

For live updates: If you are using WebSockets or another real-time technology, you will set up the listener inside onMount. When a new message arrives with a new level, simply update the waterLevel variable. Svelte's reactivity will automatically update the 3D visualization.
