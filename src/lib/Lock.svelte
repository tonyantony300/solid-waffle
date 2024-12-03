<script>
  import { onMount } from "svelte";
  import lottie from "lottie-web";
  import animationData from "../data/lockAnimation.json";

  let animatedContainer;
  export let isSuccess = false;
  export let isError = false;
  let lockAnimation;

  // Creates new animation instance with specified color based on state
  function createAnimationWithColor(isUnlocked, isError) {
    if (lockAnimation) {
      lockAnimation.destroy();
    }

    // Deep clone animation data to avoid mutations
    const animationDataCopy = JSON.parse(JSON.stringify(animationData));

    const newColor = isUnlocked
      ? [0.196, 0.804, 0.196, 1]
      : isError
        ? [1, 0.294, 0.392, 1]
        : [0.172, 0.592, 0.91, 1];

    // Update animation color
    animationDataCopy.layers.forEach((layer) => {
      if (layer.ty === 4) {
        layer.shapes.forEach((shapeGroup) => {
          if (shapeGroup.ty === "gr") {
            shapeGroup.it.forEach((item) => {
              if (item.ty === "fl") {
                item.c.k = newColor;
              }
            });
          }
        });
      }
    });

    // Initialize new animation
    lockAnimation = lottie.loadAnimation({
      container: animatedContainer,
      animationData: animationDataCopy,
      loop: false,
      autoplay: false,
      renderer: "svg",
      rendererSettings: {
        preserveAspectRatio: "xMidYMid slice",
      },
    });

    lockAnimation.goToAndStop(0, true);

    return lockAnimation;
  }

  // Manages animation playback based on success state
  function animationManager(isSuccess) {
    if (!lockAnimation) return;
    const totalFrames = lockAnimation.getDuration(true);
    const lastFrame = totalFrames - 1;

    if (isSuccess) {
      const speed = totalFrames / (30 * 1);
      lockAnimation.setSpeed(speed);
      lockAnimation.playSegments([0, lastFrame], true);
    } else {
      lockAnimation.setSpeed(1);
      lockAnimation.goToAndStop(0, true);
    }
  }

  // Initialize animation on mount
  onMount(() => {
    createAnimationWithColor(isSuccess);

    return () => {
      if (lockAnimation) {
        lockAnimation.destroy();
      }
    };
  });

  // React to state changes
  $: {
    if (isSuccess !== undefined || isError != undefined) {
      createAnimationWithColor(isSuccess, isError);
      animationManager(isSuccess);
    }
  }
</script>

<div
  role="img"
  aria-label={isSuccess ? "Animated unlocked lock" : "Animated lock"}
  bind:this={animatedContainer}
  class="animated--card"
/>

<style>
  .animated--card {
    margin: 0px auto;
  }
</style>
