<script>
import HomeScreenIcon from "./HomeScreenIcon";
import ShareIcon from "./ShareIcon";

export default {
  components: {
    HomeScreenIcon,
    ShareIcon,
  },
  props: {
    delay: { type: Number, default: 1000 },
    copyTitle: { type: String, required: true },
    copyBody: { type: String, required: true },
    copyAddHomeButtonLabel: { type: String, required: true },
    copyShareButtonLabel: { type: String, required: true },
    copyClosePrompt: { type: String, required: true },
    permanentlyHideOnDismiss: { type: Boolean, default: true },
    promptData: { type: Object, default: () => ({}) },
    maxVisits: { type: Number, default: null },
    onClose: { type: Function, default: () => {} },
    debug: { type: Boolean, default: false },
  },

  data() {
    return {
      isVisible: false,
    };
  },

  created() {
    // eslint-disable-next-line no-extra-boolean-cast
    this.isVisible = !Boolean(this.delay);

    if (this.delay) {
      setTimeout(() => {
        // Prevent keyboard appearing over the prompt if a text input has autofocus set
        if (document.activeElement) {
          document.activeElement.blur();
        }

        this.isVisible = true;
      }, this.delay);
    }
  },

  computed: {
    isiOS13AndUp() {
      return /OS (13|14)/.test(window.navigator.userAgent);
    },
    visibilityClass() {
      return this.isVisible ? "visible" : "hidden";
    },
    iOSClass() {
      return this.isiOS13AndUp ? "modern" : "legacy";
    },
  },

  methods: {
    dismissPrompt(evt) {
      document.body.classList.remove("noScroll");
      this.isVisible = false;

      if (this.permanentlyHideOnDismiss) {
        localStorage.setItem(
          "iosPwaPrompt",
          JSON.stringify({
            ...this.promptData,
            visits: this.maxVisits,
          })
        );
      }

      if (typeof this.onClose === "function") {
        this.onClose(evt);
      }
    },

    onTransitionOut(evt) {
      if (!this.isVisible) {
        evt.currentTarget.style.display = "none";
      }
    },
  },
};
</script>

<template>
  <fragment>
    <div
      :class="['pwaPromptOverlay', visibilityClass, iOSClass, 'iOSPWA-overlay']"
      aria-label="Close"
      role="button"
      @click="dismissPrompt"
      @transitionEnd="onTransitionOut"
    />
    <div
      :class="['pwaPrompt', visibilityClass, iOSClass, 'iOSPWA-container']"
      aria-describedby="pwa-prompt-description"
      aria-labelledby="pwa-prompt-title"
      role="dialog"
      @transitionEnd="onTransitionOut"
    >
      <div :class="['pwaPromptHeader', 'iOSPWA-header']">
        <p id="pwa-prompt-title" :class="['pwaPromptTitle', 'iOSPWA-title']">
          {{ copyTitle }}
        </p>
        <button
          :class="['pwaPromptCancel', 'iOSPWA-cancel']"
          @click="dismissPrompt"
        >
          {{ copyClosePrompt }}
        </button>
      </div>
      <div :class="['pwaPromptBody', 'iOSPWA-body']">
        <div :class="['pwaPromptDescription', 'iOSPWA-description']">
          <p
            id="pwa-prompt-description"
            :class="['pwaPromptCopy', 'iOSPWA-description-copy']"
          >
            {{ copyBody }}
          </p>
        </div>
      </div>
      <div :class="['pwaPromptInstruction', 'iOSPWA-steps']">
        <div :class="['pwaPromptInstructionStep', 'iOSPWA-step1']">
          <ShareIcon
            :class="['pwaPromptShareIcon', 'iOSPWA-step1-icon']"
            :modern="isiOS13AndUp"
          />
          <p :class="['pwaPromptCopy', 'bold', 'iOSPWA-step1-copy']">
            {{ copyShareButtonLabel }}
          </p>
        </div>
        <div :class="['pwaPromptInstructionStep', 'iOSPWA-step2']">
          <HomeScreenIcon
            :class="['pwaPromptHomeIcon', 'iOSPWA-step2-icon']"
            :modern="isiOS13AndUp"
          />
          <p :class="['pwaPromptCopy', 'bold', 'iOSPWA-step2-copy']">
            {{ copyAddHomeButtonLabel }}
          </p>
        </div>
      </div>
    </div>
  </fragment>
</template>

<style lang="scss" scoped>
$overlay-color-legacy: rgba(0, 0, 0, 0.8);
$overlay-color-modern-light: rgba(10, 10, 10, 0.5);
$overlay-color-modern-dark: rgba(10, 10, 10, 0.5);

$bg-color-legacy: rgba(250, 250, 250, 0.8);
$bg-color-modern-light: rgba(255, 255, 255, 0.6);
$bg-color-modern-dark: rgba(65, 65, 65, 0.7);

$border-color-legacy: rgba(0, 0, 0, 0.1);
$border-color-modern-light: rgba(60, 60, 67, 0.29);
$border-color-modern-dark: rgba(140, 140, 140, 0.7);

$font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", Roboto,
  "Helvetica Neue", Arial, sans-serif;

$title-color-legacy: rgb(51, 51, 51);
$title-color-modern-light: rgba(0, 0, 0, 1);
$title-color-modern-dark: rgba(255, 255, 255, 1);

$font-color-legacy: rgb(123, 123, 122);
$font-color-modern-light: rgba(60, 60, 67, 0.6);
$font-color-modern-dark: rgba(235, 235, 245, 0.6);

$blue-color-legacy: rgb(45, 124, 246);
$blue-color-modern-light: rgba(0, 85, 179, 1);
$blue-color-modern-dark: rgba(9, 132, 255, 1);

.noScroll {
  overflow: hidden;
}

.pwaPromptOverlay {
  background-color: $overlay-color-legacy;
  left: 0;
  min-height: 100vh;
  min-height: -webkit-fill-available;
  opacity: 0;
  position: fixed;
  top: 0;
  transition: opacity 0.2s ease-in;
  width: 100vw;
  z-index: 999999;

  &.visible {
    opacity: 1;
    display: block;
  }

  &.hidden {
    pointer-events: none;
    touch-action: none;
  }

  &.modern {
    @media (prefers-color-scheme: dark) {
      & {
        background: $overlay-color-modern-dark;
        color: $font-color-modern-dark;
      }
    }
  }
}

.pwaPrompt {
  -webkit-backdrop-filter: blur(10px);
  backdrop-filter: blur(10px);
  background-color: $bg-color-legacy;
  border-radius: 10px;
  bottom: 0;
  color: black;
  filter: brightness(1.1);
  left: 0;
  margin: 0 8px 10px;
  overflow: hidden;
  position: fixed;
  transform: translateY(calc(100% + 10px));
  transition: transform 0.4s cubic-bezier(0.4, 0.24, 0.3, 1);
  width: calc(100vw - 16px);
  z-index: 999999;

  &.visible {
    transform: translateY(0);
    display: block;
  }

  &.hidden {
    pointer-events: none;
    touch-action: none;
  }

  &.modern {
    background: $bg-color-modern-light;
    filter: brightness(1.6);

    @media (prefers-color-scheme: dark) {
      & {
        background: $bg-color-modern-dark;
        filter: brightness(1.1);
      }
    }
  }
}

.pwaPromptHeader {
  align-items: center;
  border-bottom: 1px solid $border-color-legacy;
  border-top: 0px;
  border-left: 0px;
  border-right: 0px;
  border-width: 0.5px;
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-between;
  padding: 13px 16px;

  .modern & {
    border-color: $border-color-modern-light;

    @media (prefers-color-scheme: dark) {
      & {
        border-color: $border-color-modern-dark;
      }
    }
  }

  .pwaPromptTitle {
    color: $title-color-legacy;
    font-family: $font-family;
    font-size: 18px;
    font-weight: 500;
    line-height: 1.125;
    margin: 0;
    padding: 0;

    .modern & {
      color: $title-color-modern-light;

      @media (prefers-color-scheme: dark) {
        & {
          color: $title-color-modern-dark;
        }
      }
    }
  }

  .pwaPromptCancel {
    color: $blue-color-legacy;
    font-size: 16px;
    padding: 0;
    margin: 0;
    border: 0;
    background: transparent;

    .modern & {
      color: $blue-color-modern-light;

      @media (prefers-color-scheme: dark) {
        & {
          color: $blue-color-modern-dark;
        }
      }
    }
  }
}

.pwaPromptBody {
  display: flex;
  width: 100%;

  .pwaPromptDescription {
    border-bottom: 1px solid $border-color-legacy;
    border-top: 0px;
    border-left: 0px;
    border-right: 0px;
    border-width: 0.5px;
    color: inherit;
    margin: 0 16px;
    padding: 16px;
    width: 100%;

    .modern & {
      border-color: $border-color-modern-light;

      @media (prefers-color-scheme: dark) {
        & {
          border-color: $border-color-modern-dark;
        }
      }
    }
  }
}

.pwaPromptCopy {
  color: $font-color-legacy;
  font-family: $font-family;
  font-size: 13px;
  line-height: 17px;
  margin: 0;
  padding: 0;

  &.bold {
    font-weight: 600;
  }

  .modern & {
    color: $font-color-modern-light;

    @media (prefers-color-scheme: dark) {
      & {
        border-color: $font-color-modern-dark;
        color: $font-color-modern-dark;
      }
    }
  }
}

.pwaPromptInstruction {
  color: inherit;
  margin: 0 16px;
  padding: 16px;

  .pwaPromptInstructionStep {
    align-items: center;
    display: flex;
    flex-flow: row nowrap;
    justify-content: flex-start;
    text-align: left;
    margin-bottom: 16px;

    &:last-of-type {
      margin-bottom: 0;
    }
  }

  .pwaPromptShareIcon,
  .pwaPromptHomeIcon {
    flex: 0 0 auto;
    height: 30px;
    margin-right: 32px;
    width: 25px;
  }

  .pwaPromptHomeIcon {
    color: $blue-color-legacy;

    .modern & {
      color: black;
      fill: black;

      @media (prefers-color-scheme: dark) {
        & {
          color: white;
          fill: white;
        }
      }
    }
  }

  .pwaPromptShareIcon {
    color: $blue-color-legacy;
    fill: $blue-color-legacy;

    .modern & {
      color: $blue-color-modern-light;
      fill: $blue-color-modern-light;

      @media (prefers-color-scheme: dark) {
        & {
          color: $blue-color-modern-dark;
          fill: $blue-color-modern-dark;
        }
      }
    }
  }
}
</style>
