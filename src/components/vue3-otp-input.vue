<script lang="ts" setup>
import { onMounted, ref, watchEffect } from 'vue';
import SingleOtpInput from './single-otp-input.vue';

// keyCode constants
const BACKSPACE = 8;
const LEFT_ARROW = 37;
const RIGHT_ARROW = 39;
const DELETE = 46;

type Props = {
  count?: number;
  separator?: string;
  inputClass?: string | string[];
  conditionalClass?: string[];
  inputType?: 'number' | 'tel' | 'letter-numeric' | 'password';
  inputmode?: 'none' | 'text' | 'tel' | 'url' | 'email' | 'numeric' | 'decimal' | 'search';
  autoFocus?: boolean;
  forceOrdering?: boolean;
  placeholder?: string[];
  disabled?: boolean;
};

const props = withDefaults(defineProps<Props>(), {
  count: 4,
  separator: '',
  inputClass: '',
  conditionalClass: () => [],
  inputType: 'letter-numeric',
  inputmode: 'text',
  autoFocus: false,
  forceOrdering: false,
  disabled: false,
  placeholder: () => [],
});

const emit = defineEmits<{
  change: [codeStr: string];
  complete: [codeStr: string];
}>();

const code = defineModel<string[]>('value', { default: [] });

const setCodeAt = (index: number, c: string) => {
  if (index < 0) {
    return;
  }
  const newCode = [...code.value];
  newCode[index] = c;
  code.value = newCode;
};

const activeInput = ref(-1);

watchEffect(() => {
  emit('change', code.value.join(''));

  // Check if every slot is filled
  if (code.value.every((c) => !!c)) {
    emit('complete', code.value.join(''));
  }
});

// Separate the watchEffect to keep clear dependency
watchEffect(() => {
  if (code.value.length === props.count) {
    return;
  }

  // If bad code array is set
  // Keep the array length
  if (code.value.length < props.count) {
    code.value = code.value.concat(...new Array<string>(props.count - code.value.length).fill(''));
  } else {
    code.value = code.value.slice(0, props.count);
  }
});

onMounted(() => {
  if (props.autoFocus) {
    activeInput.value = 0;
  }
});

const handleFocus = (index: number) => {
  if (props.forceOrdering) {
    const firstEmptyIndex = code.value.findIndex((c) => !c);
    if (firstEmptyIndex === -1) {
      focusInput(props.count - 1);
    } else {
      focusInput(firstEmptyIndex);
    }
  } else {
    focusInput(index);
  }
};

const handleBlur = () => {
  activeInput.value = -1;
};

// Focus on input by index
const focusInput = (input: number) => {
  activeInput.value = Math.max(Math.min(props.count - 1, input), 0);
};

// Focus on next input
const focusNextInput = () => {
  focusInput(activeInput.value + 1);
};

// Focus on previous input
const focusPrevInput = () => {
  focusInput(activeInput.value - 1);
};

// Handle pasted OTP
const handlePaste = (index: number, event: ClipboardEvent) => {
  event.preventDefault();

  // Clamp the pasted data length to at most the number of count
  let pastedData = event.clipboardData?.getData('text/plain').substring(0, props.count - index);
  if (pastedData === undefined) {
    return;
  }

  // Remove non-numeric or non-alphanumeric characters depending on the input type
  switch (props.inputType) {
    case 'number':
      pastedData = pastedData.replace(/[^\d]/g, '');
      break;
    case 'letter-numeric':
      pastedData = pastedData.replace(/[^\w]/g, '');
      break;
  }

  // Paste data from focused input onwards and clamp to the number of count
  const codeWithPastedData = code.value.slice(0, index).concat(...pastedData.split(''));
  const slicedOtp = codeWithPastedData.slice(0, props.count);

  code.value = slicedOtp;

  focusInput(slicedOtp.length);
};

const handleValueUpdate = (index: number, value: string) => {
  setCodeAt(index, value);
  // Only move to next input if we're not at the last input
  if (activeInput.value < props.count - 1) {
    focusNextInput();
  }
};

// Handle cases of backspace, delete, left arrow, right arrow
const handleKeydown = (index: number, event: KeyboardEvent) => {
  switch (event.keyCode) {
    case BACKSPACE:
      event.preventDefault();
      // Do not move the focus when there's a value at last input
      // This is to maintain consistent cursor location
      if (index === props.count - 1 && code.value[index]) {
        setCodeAt(index, '');
      } else {
        setCodeAt(index - 1, '');
        focusPrevInput();
      }
      break;
    case DELETE:
      event.preventDefault();
      setCodeAt(index, '');
      break;
    case LEFT_ARROW:
      event.preventDefault();
      focusPrevInput();
      break;
    case RIGHT_ARROW:
      event.preventDefault();
      focusNextInput();
      break;
  }
};
</script>

<template>
  <div>
    <!-- To turn off autocomplete when input-type is password -->
    <input
      v-if="inputType === 'password'"
      autocomplete="off"
      name="hidden"
      type="text"
      style="display: none"
    />
    <div
      v-for="(_, i) in count"
      :key="i"
      style="display: flex; align-items: center"
    >
      <SingleOtpInput
        :focus="activeInput === i"
        :value="code[i]"
        :separator="separator"
        :input-type="inputType"
        :inputmode="inputmode"
        :class="[...(inputClass ? [inputClass] : []), ...(conditionalClass?.[i] ? [conditionalClass?.[i]] : [])]"
        :last-child="i === count - 1"
        :placeholder="placeholder?.[i]"
        :disabled="disabled"
        @update:value="(value) => handleValueUpdate(i, value)"
        @keydown="(event) => handleKeydown(i, event)"
        @paste="(event) => handlePaste(i, event)"
        @focus="handleFocus(i)"
        @blur="handleBlur"
      />
      <span v-if="separator && i !== count - 1">
        <span>{{ separator }}</span>
      </span>
    </div>
  </div>
</template>
