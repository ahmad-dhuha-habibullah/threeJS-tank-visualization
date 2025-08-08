# 3D Svelte Liquid Tank Visualization

A Svelte component that displays a realistic, real-time 3D visualization of a liquid tank, designed for easy integration into larger applications and connection to live data sources.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Setup and Installation](#setup-and-installation)
- [Key Files](#key-files)
- [Connecting to a Live Database](#connecting-to-a-live-database)
- [License](#license)

## Prerequisites

- **Node.js**: Ensure Node.js (which includes npm) is installed on your machine. Download it from [nodejs.org](https://nodejs.org) if needed.

## Setup and Installation

1. **Clone the Repository**  
   Open your terminal and run:

   ```bash
   git clone https://github.com/ahmad-dhuha-habibullah/threeJS-tank-visualization.git
   ```

2. **Navigate to the Project Directory**  
   ```bash
   cd threeJS-tank-visualization
   ```

3. **Install Dependencies**  
   Run the following command to install all required packages, including SvelteKit and Three.js:

   ```bash
   npm install
   ```

4. **Run the Development Server**  
   Start the development server with:

   ```bash
   npm run dev
   ```

5. **View the App**  
   Open your web browser and navigate to `http://localhost:5173`. You should see the 3D tank visualization.

## Key Files

- **`src/lib/TankVisualization.svelte`**:  
  The core 3D component that renders the tank based on the `level` property. This is a self-contained module and typically does not require edits.

- **`src/routes/+page.svelte`**:  
  The main page that hosts and tests the `TankVisualization` component. Edit this file to connect to your real database and pass the `waterLevel` variable to the component's `level` prop.

## Connecting to a Live Database

The `TankVisualization` component is designed to integrate with a live data source. All connection logic should be implemented in the parent component (`src/routes/+page.svelte`).

In `src/routes/+page.svelte`, locate the `<script>` section:

```svelte
<script>
  import TankVisualization from '$lib/TankVisualization.svelte';
  import { onMount } from 'svelte';

  let waterLevel = 0; // Controls the tank's level

  // --- CONNECT TO YOUR DATABASE HERE ---
  onMount(() => {
    // Example: Fetch data from a Django API
    /*
    async function getTankLevel() {
      const response = await fetch('https://your-api.com/tank/level');
      const data = await response.json();
      waterLevel = data.level; // Update with API data
    }
    getTankLevel();
    */

    // Temporary manual setting for demonstration
    waterLevel = 65;
  });
</script>
```

### Steps to Connect to a Live Data Source

1. **For Initial Data**:  
   Replace `waterLevel = 65;` with a `fetch` call to your Django API endpoint inside the `onMount` function.

2. **For Live Updates**:  
   If using WebSockets or another real-time technology, set up a listener inside `onMount`. When new data arrives, update the `waterLevel` variable. Svelte's reactivity will automatically refresh the 3D visualization.

Example with WebSocket:

```svelte
onMount(() => {
  const socket = new WebSocket('wss://your-api.com/tank/updates');
  socket.onmessage = (event) => {
    const data = JSON.parse(event.data);
    waterLevel = data.level; // Update with WebSocket data
  };
  return () => socket.close(); // Cleanup on component unmount
});
```

## License

This project is licensed under the [MIT License](LICENSE). See the `LICENSE` file for details.
