<!-- HomePage.svelte -->
<!-- Do not touch this page. it's the animation page, limit your activities to Page.svelte by adjusting the telegram values as necessary-->
<script>
  import { onMount } from 'svelte';
  import { navigate } from 'svelte-routing';

  let step = 0;
  let interval;

  onMount(() => {
    // Animation interval: cycles steps every 1 second
    interval = setInterval(() => {
      step = (step + 1) % 3;
    }, 1000);

    // Navigate to /page after 5 seconds
    const timeout = setTimeout(() => {
      clearInterval(interval);
      navigate('/page');
    }, 5000);

    // Cleanup on destroy
    return () => {
      clearInterval(interval);
      clearTimeout(timeout);
    };
  });
</script>

<style>
  .loader-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
  }

  .rect-track {
    display: flex;
    gap: 20px;
  }

  .rect {
    width: 60px;
    height: 120px;
    background-color: grey;
    opacity: 0.3;
    transition: opacity 0.5s ease, background-color 0.5s ease;
    border-radius: 4px;
  }

  .rect.active {
    opacity: 1;
    background-color: #003770;
  }
</style>

<div class="loader-container">
  <div class="rect-track">
    <div class="rect {step === 0 ? 'active' : ''}"></div>
    <div class="rect {step === 1 ? 'active' : ''}"></div>
    <div class="rect {step === 2 ? 'active' : ''}"></div>
  </div>
</div>
