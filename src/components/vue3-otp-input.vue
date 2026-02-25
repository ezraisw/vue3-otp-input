<script lang="ts" setup>
import { ref, watch } from "vue";
import SingleOtpInput from "./single-otp-input.vue";

// keyCode constants
const BACKSPACE = 8;
const LEFT_ARROW = 37;
const RIGHT_ARROW = 39;
const DELETE = 46;

type Props = {
  numInputs?: number;
  separator?: string;
  inputClasses?: string | string[];
  conditionalClass?: string[];
  inputType?: "number" | "tel" | "letter-numeric" | "password";
  inputmode?:
    | "none"
    | "text"
    | "tel"
    | "url"
    | "email"
    | "numeric"
    | "decimal"
    | "search";
  shouldAutoFocus?: boolean;
  placeholder?: string[];
  isDisabled?: boolean;
  shouldFocusOrder?: boolean;
};

const props = withDefaults(defineProps<Props>(), {
  numInputs: 4,
  separator: "",
  inputClasses: "",
  inputmode: "text",
  shouldAutoFocus: false,
  isDisabled: false,
  placeholder: () => [],
  conditionalClass: () => [],
});

const emit = defineEmits<{
  (e: "on-change", value: string): void;
  (e: "on-complete", value: string): void;
}>();

const otp = defineModel<string>("value", { default: "" });

/**
 * This represents the currently active input index.
 * It can be decoupled from the length of the current value because users
 * might choose to change from a specific index.
 */
const activeInput = ref<number>(0);

watch(otp, (value, oldValue) => {
  if (value === oldValue) {
    return;
  }

  if (value === "") {
    activeInput.value = 0;
  }

  emit("on-change", value);

  if (value.length === props.numInputs) {
    emit("on-complete", value);
  }
}, {
  immediate: true,
});

const setOtpAt = (index: number, value: string) => {
  if (index < 0) {
    return;
  }
  // Immutable string edit
  let newValue = otp.value.substring(0, index) + value + otp.value.substring(index + 1);
  newValue = newValue.trim();
  otp.value = newValue;
};

const handleOnFocus = (index: number) => {
  activeInput.value = index;
};
const handleOnBlur = () => {
  // Don't reset activeInput if we're at the last input
  if (activeInput.value !== props.numInputs - 1) {
    activeInput.value = -1;
  }
};

// Focus on input by index
const focusInput = (input: number) => {
  activeInput.value = Math.max(Math.min(props.numInputs - 1, input), 0);
};
// Focus on next input
const focusNextInput = () => {
  focusInput(activeInput.value + 1);
};
// Focus on previous input
const focusPrevInput = () => {
  focusInput(activeInput.value - 1);
};

// Change OTP value at focused input
const changeCodeAtFocus = (value: number | string) => {
  setOtpAt(activeInput.value, value.toString());
};

// Handle pasted OTP
const handleOnPaste = (event: ClipboardEvent) => {
  event.preventDefault();

  // Clamp the pasted data length to at most the number of inputs
  const pastedData = event.clipboardData
    ?.getData("text/plain")
    .slice(0, props.numInputs - activeInput.value);

  if (pastedData === undefined) {
    return "Invalid pasted data";
  }

  if (props.inputType === "number" && !pastedData.match(/^\d+$/)) {
    return "Invalid pasted data";
  }

  if (props.inputType === "letter-numeric" && !pastedData.match(/^\w+$/)) {
    return "Invalid pasted data";
  }

  // Paste data from focused input onwards and clamp to the number of inputs
  const otpWithPastedData = otp.value + pastedData;
  const slicedOtp = otpWithPastedData.slice(0, props.numInputs);

  otp.value = slicedOtp;

  focusInput(slicedOtp.length);
};

const handleOnChange = (value: number | string) => {
  changeCodeAtFocus(value);
  // Only move to next input if we're not at the last input
  if (activeInput.value < props.numInputs - 1) {
    focusNextInput();
  }
};
const clearInput = () => {
  otp.value = "";
};

const fillInput = (value: string) => {
  otp.value = value;
};

// Handle cases of backspace, delete, left arrow, right arrow
const handleOnKeyDown = (event: KeyboardEvent, index: number) => {
  switch (event.keyCode) {
    case BACKSPACE:
      event.preventDefault();
      changeCodeAtFocus("");
      focusPrevInput();
      break;
    case DELETE:
      event.preventDefault();
      changeCodeAtFocus("");
      break;
    case LEFT_ARROW:
      event.preventDefault();
      focusPrevInput();
      break;
    case RIGHT_ARROW:
      event.preventDefault();
      focusNextInput();
      break;
    default:
      focusOrder(index);
      break;
  }
};

/**
 *
 * @param currentIndex - index of the input
 * @description - This function is used to focus the input in the order of the input index
 *
 * @example
 * 1. If the user is entering the OTP in the order of the input index, then the input will be focused in the order of the input index
 * 2. If the user is entering the OTP in the reverse order of the input index, then the input will be focused in the reverse order of the input index
 */
const focusOrder = (currentIndex: number) => {
  if (props.shouldFocusOrder) {
    setTimeout(() => {
      if (currentIndex >= otp.value.length) {
        focusInput(otp.value.length);
        setOtpAt(currentIndex, "");
      }
    }, 100);
  }
};

defineExpose({
  clearInput,
  fillInput,
});
</script>

<template>
  <div style="display: flex" class="otp-input-container">
    <!--    To turn off autocomplete when otp-input is password-->
    <input
      v-if="inputType === 'password'"
      autocomplete="off"
      name="hidden"
      type="text"
      style="display: none"
    />
    <SingleOtpInput
      v-for="(_, i) in numInputs"
      :key="i"
      :focus="activeInput === i"
      :value="otp[i]"
      :separator="separator"
      :input-type="inputType"
      :inputmode="inputmode"
      :input-classes="inputClasses"
      :conditionalClass="conditionalClass?.[i]"
      :is-last-child="i === numInputs - 1"
      :should-auto-focus="shouldAutoFocus"
      :placeholder="placeholder?.[i]"
      :is-disabled="isDisabled"
      @on-change="handleOnChange"
      @on-keydown="handleOnKeyDown($event, i)"
      @on-paste="handleOnPaste"
      @on-focus="handleOnFocus(i)"
      @on-blur="handleOnBlur"
    />
  </div>
</template>

<style scoped></style>
