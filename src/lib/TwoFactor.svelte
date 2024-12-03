<script>
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";
  import { onMount } from "svelte";
  import { AUTH_CONFIG } from "../data/constants";
  import Lock from "./Lock.svelte";

  // Component props with default values
  export let onSuccess = () => {};
  export let onError = () => {};
  export let validateCode = () => {};
  export let inputLength = AUTH_CONFIG.INPUT_LENGTH;
  export let heading = "Easy peasy";
  export let instruction = "Enter 6-digit code from your authentication APP.";

  let code = Array(inputLength).fill("");
  let activeIndex = 0;
  let isError = false;
  let isSuccess = false;
  let glowVisibility = false;

  // Animation configuration for the glow effect
  const glowSize = tweened(AUTH_CONFIG.GLOW_SIZE.MIN, {
    duration: AUTH_CONFIG.ANIMATION_DURATION,
    easing: cubicOut,
  });

  // Helper function to focus specific input field
  const focusInput = (index) => {
    const inputElement = document.getElementById(`input-${index}`);
    if (inputElement) {
      inputElement.focus();
    }
  };

  // Focus first input on component mount
  onMount(() => focusInput(activeIndex));

  // Reactive statement to handle input focus
  $: if (activeIndex < inputLength) {
    focusInput(activeIndex);
  }

  // Handles input changes and validation
  const handleInput = (event, index) => {
    const value = event.target.value.trim();

    // Validate numeric input
    if (isNaN(value) || value.length > 1) {
      code[index] = "";
      code = [...code];
      return;
    }

    // Update code array and manage focus
    code[index] = value;
    code = [...code];

    if (value && index < inputLength - 1) {
      activeIndex = index + 1;
    }

    // Validate complete code
    if (code.join("").length === inputLength) {
      activeIndex = null;
      validateInputCode();
    }
  };

  // Handles backspace and state reset
  const handleKeydown = (event, index) => {
    if (event.key === "Backspace") {
      if (!code[index] && index > 0) {
        activeIndex = index - 1;
        code[activeIndex] = "";
      } else {
        code[index] = "";
      }
      code = [...code];
      isSuccess = false;
      isError = false;
    }
  };

  // Validates entered code and triggers callbacks
  const validateInputCode = () => {
    const enteredCode = code.join("");

    if (validateCode(enteredCode)) {
      isSuccess = true;
      isError = false;
      onSuccess(enteredCode);
    } else {
      isError = true;
      isSuccess = false;
      onError(enteredCode);
    }

    glowVisibility = true;
    triggerGlowAnimation();
  };

  // Manages glow animation timing
  const triggerGlowAnimation = () => {
    glowSize.set(AUTH_CONFIG.GLOW_SIZE.MAX);
    setTimeout(() => {
      glowSize.set(AUTH_CONFIG.GLOW_SIZE.MIN);
      glowVisibility = false;
    }, AUTH_CONFIG.ANIMATION_DURATION);
  };
</script>

<div class="container" role="form" aria-label="Two-factor authentication">
  <div class="lock-container" aria-live="polite" aria-atomic="true">
    <div
      class="glow {isSuccess ? 'green' : isError ? 'red' : ''} {glowVisibility
        ? ''
        : 'hide'}"
      style="width: {$glowSize}px; height: {$glowSize}px;"
      aria-hidden="true"
    ></div>
    <div
      class="lock"
      role="status"
      aria-label={isSuccess
        ? "Authentication successful"
        : isError
          ? "Authentication failed"
          : "Enter authentication code"}
    >
      <Lock {isSuccess} {isError} />
    </div>
    <span
      class="lock--stable-bg {isSuccess ? 'success' : isError ? 'error' : ''}"
      aria-hidden="true"
    ></span>
  </div>

  <h1 class="heading">{heading}</h1>
  <p class="instruction-text" id="instruction">
    {instruction}
  </p>

  <div
    class="action-container {isError ? 'error' : ''}"
    role="group"
    aria-labelledby="instruction"
  >
    <div class="input-container">
      {#each code as _, index}
        <span
          class={`input-wrapper ${activeIndex === index ? "active" : "inactive"} ${index === 2 ? " extra-space" : ""}`}
        >
          <input
            id={`input-${index}`}
            class={`input${isError ? " error" : isSuccess ? " success" : ""}`}
            type="text"
            inputmode="numeric"
            pattern="[0-9]*"
            maxlength="1"
            autocomplete="one-time-code"
            aria-required="true"
            aria-label={`Digit ${index + 1} of ${inputLength}`}
            aria-invalid={isError}
            class:filled={!isError && code[index]}
            bind:value={code[index]}
            on:input={(event) => handleInput(event, index)}
            on:keydown={(event) => handleKeydown(event, index)}
          />
        </span>
      {/each}
    </div>

    <div class="button-container">
      <button
        type="button"
        class="button {isSuccess ? 'success' : isError ? 'error' : ''}"
        aria-disabled={!isSuccess}
        disabled={!isSuccess}
        aria-live="polite"
      >
        <div class="button-fill" aria-hidden="true"></div>
        <span class="button-text">
          {#if isSuccess}
            Let's Go!
          {:else if isError}
            Wrong Code
          {:else}
            {inputLength - code.join("").length} digits left
          {/if}
        </span>
      </button>
    </div>
  </div>
</div>
