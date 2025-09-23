<script>
import AppContainer from "./AppContainer.svelte";
import AppFooter from "./AppFooter.svelte";
</script>

<style scoped>
.modal-form-page {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  min-height: 100vh;
  width: 100%;
  box-sizing: border-box;
  padding: 0;
  margin: 0;
  background: #0088C1;
}

/* Center AppContainer vertically using a wrapper */
.content-wrapper {
  flex-grow: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
}

/* AppFooter should not shrink and stay at the bottom */
.modal-form-page> :last-child {
  flex-shrink: 0;
  width: 100%;
}
</style>


 <div class="modal-form-page">
    <!-- <AppNavbar /> -->
    <div class="content-wrapper">
      <AppContainer />
    </div>
    <AppFooter />
  </div>