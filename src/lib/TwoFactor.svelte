<script>
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";
  import { onMount } from "svelte";
  import { AUTH_CONFIG } from "../data/constants";
  import Lock from "./Lock.svelte";

  // Component props with default values
  export let onSuccess;
  export let onError;
  export let validateCode;
  export let inputLength = AUTH_CONFIG.INPUT_LENGTH;
  export let heading = "Easy peasy";
  export let instruction = "Enter 6-digit code from your authentication APP.";

  let code = Array(inputLength).fill("");
  let activeIndex = 0;
  let isError = false;
  let isSuccess = false;
  let glowVisibility = false;
  let inputs = [];

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

  const handleInput = (event, index) => {
    const value = event.target.value;
    const inputEl = inputs[index];

    if (isNaN(value)) {
      inputEl.value = code[index];
      return;
    }

    // when the input contains two values, it is a replacement

    if (value.length === 2) {
      const digits = value.split("");
      const newDigit =
        digits[0] === digits[1]
          ? digits[0]
          : digits.find((d) => d !== code[index]);
      code[index] = inputEl.value = newDigit;
    } else {
      code[index] = value;
    }

    code = [...code];
    if (value && index < inputLength - 1) activeIndex = index + 1;
    if (code.join("").length === inputLength) validateInputCode();
  };

  // Handles keyboard navigation
  const handleKeydown = (event, index) => {
    const { key, shiftKey } = event;

    const navigationMap = {
      Backspace: () => {
        if (!code[index] && index > 0) {
          activeIndex = index - 1;
          code[activeIndex] = "";
        } else {
          code[index] = "";
        }
        code = [...code];
        isSuccess = isError = false;
      },
      // when user reaches last input and keeps pressing navigation again, focus shifts to the other end
      // shift + left sets focus to first input
      ArrowLeft: () => {
        event.preventDefault();
        activeIndex = shiftKey
          ? 0
          : activeIndex === null
            ? index
            : (index - 1 + inputLength) % inputLength;
      },
      ArrowRight: () => {
        event.preventDefault();
        activeIndex = shiftKey ? inputLength - 1 : (index + 1) % inputLength;
      },
    };

    navigationMap[key]?.();
  };

  const handlePaste = (event, index) => {
    // only handle paste when it is the first input and no digits are filled
    if (index === 0 && !code.some((digit) => digit)) {
      event.preventDefault();

      const pastedData = event.clipboardData.getData("text").trim();

      if (isNaN(pastedData)) return;

      // take only first inputLength digits if pasted text is longer
      const digits = pastedData.slice(0, inputLength).split("");

      // fill the code array with new digits
      code = Array(inputLength)
        .fill("")
        .map((_, i) => digits[i] || "");

      // update inputs
      code.forEach((digit, i) => {
        if (inputs[i] && digit) {
          inputs[i].value = digit;
        }
      });

      if (code.join("").length === inputLength) {
        setTimeout(() => {
          validateInputCode();
        }, 500);
      }
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
            bind:this={inputs[index]}
            value={code[index]}
            id={`input-${index}`}
            class={`input${isError ? " error" : isSuccess ? " success" : ""}`}
            type="text"
            inputmode="numeric"
            pattern="[0-9]*"
            maxlength="2"
            autocomplete="one-time-code"
            aria-required="true"
            aria-label={`Digit ${index + 1} of ${inputLength}`}
            aria-invalid={isError}
            class:filled={!isError && code[index]}
            on:input={(event) => handleInput(event, index)}
            on:keydown={(event) => handleKeydown(event, index)}
            on:paste={(event) => handlePaste(event, index)}
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
