<script>
  import TwoFactor from "./lib/TwoFactor.svelte";
  import { AUTH_CONFIG } from "./data/constants";

  const handleSuccess = (code) => {
    // console.log("Success:", code);
  };

  const handleError = (code) => {
    // console.log("Error:", code);
  };

  const validateCode = (code) => code === AUTH_CONFIG.CORRECT_CODE;
</script>

<main>
  <TwoFactor {validateCode} onSuccess={handleSuccess} onError={handleError} />
</main>

<style>
  main {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background-color: var(--main-bg, #e4f4ff);
  }
</style>
