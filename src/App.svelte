<script>
  import { Router, Route, Link } from 'svelte-routing';
  import Page from './Page.svelte';
  // import HomePage from './HomePage.svelte';
</script>

<Router>
    <Route path="/" component={Page} />
    <!-- <Route path="/page" component={Page} /> -->
</Router>
