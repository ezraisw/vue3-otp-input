<script lang="ts" setup>
import { computed, onMounted, ref, watchEffect } from 'vue';
import SingleOtpInput from './single-otp-input.vue';

type Props = {
  count?: number;
  separator?: string;
  inputClass?: string | string[];
  conditionalClass?: string[];
  inputType?: 'number' | 'tel' | 'letter-numeric' | 'password';
  inputmode?: 'none' | 'text' | 'tel' | 'url' | 'email' | 'numeric' | 'decimal' | 'search';
  autoFocus?: boolean;
  continuous?: boolean;
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
  continuous: false,
  disabled: false,
  placeholder: () => [],
});

const emit = defineEmits<{
  change: [codeStr: string];
  complete: [codeStr: string];
}>();

const code = defineModel<string[]>('value', { default: [] });

const complete = computed(() => code.value.length > 0 && code.value.length === props.count && code.value.every((c) => !!c));

const setCodeAt = (index: number, c: string) => {
  if (index < 0 && index >= code.value.length) {
    return;
  }
  const newCode = [...code.value];
  newCode[index] = c;
  code.value = newCode;
};

const insertCodeAt = (index: number, c: string) => {
  if (index < 0 && index >= code.value.length) {
    return;
  }
  const newCode = [...code.value];
  newCode.splice(index, 0, c);
  newCode.pop();
  code.value = newCode;
};

const shiftCodeAt = (index: number) => {
  if (index < 0 && index >= code.value.length) {
    return;
  }
  const newCode = [...code.value];
  newCode.splice(index, 1);
  // Fill leftover
  code.value = newCode.concat('');
};

const activeIndex = ref(-1);

watchEffect(() => {
  emit('change', code.value.join(''));

  // The input is complete if every slot is filled and the length is correct
  if (complete.value) {
    emit('complete', code.value.join(''));
  }

  // Sanitize bad string elements
  if (code.value.some((c) => c.length > 1)) {
    code.value = code.value.map((c) => c.substring(0, 1));
  }

  // Sanitize bad array length
  if (code.value.length < props.count) {
    code.value = code.value.concat(...new Array<string>(props.count - code.value.length).fill(''));
  } else if (code.value.length > props.count) {
    code.value = code.value.slice(0, props.count);
  }
});

onMounted(() => {
  if (props.autoFocus) {
    activeIndex.value = 0;
  }
});

const handleFocus = (index: number) => {
  const firstEmptyIndex = code.value.findIndex((c) => !c);
  if (props.continuous && firstEmptyIndex !== -1) {
    // Only the next empty input or the current input is focusable
    focusInput(Math.min(firstEmptyIndex, index));
  } else {
    // Every input is focusable
    focusInput(index);
  }
};

const handleBlur = () => {
  activeIndex.value = -1;
};

// Focus on input by index
const focusInput = (index: number) => {
  if (activeIndex.value === index) {
    return;
  }
  activeIndex.value = Math.max(Math.min(props.count - 1, index), 0);
};

// Focus on next input
const focusNextInput = () => {
  focusInput(activeIndex.value + 1);
};

// Focus on previous input
const focusPrevInput = () => {
  focusInput(activeIndex.value - 1);
};

const distributeMultiCharStringAt = (index: number, value: string) => {
  let sanitizedValue = value;
  // Remove non-numeric or non-alphanumeric characters depending on the input type
  switch (props.inputType) {
    case 'number':
      sanitizedValue = sanitizedValue.replace(/[^\d]/g, '');
      break;
    case 'letter-numeric':
      sanitizedValue = sanitizedValue.replace(/[^\w]/g, '');
      break;
  }

  // Paste data from focused input onwards and clamp to the number of count
  const codeWithPastedData = code.value.slice(0, index).concat(...sanitizedValue.split(''));
  const slicedOtp = codeWithPastedData.slice(0, props.count);

  code.value = slicedOtp;

  // Move focus to the end of the newly inserted data
  focusInput(slicedOtp.length);
};

// Handle pasted OTP
const handlePaste = (index: number, event: ClipboardEvent) => {
  event.preventDefault();

  // Clamp the pasted data length to at most the number of count
  const pastedData = event.clipboardData?.getData('text/plain').substring(0, props.count - index);
  if (pastedData === undefined) {
    return;
  }

  distributeMultiCharStringAt(index, pastedData);
};

// Handle autofilled OTP (different from paste)
const handleAutofill = (index: number, value: string) => {
  distributeMultiCharStringAt(index, value);
};

const handleValueUpdate = (index: number, value: string) => {
  if (props.continuous) {
    // For continuous, since it is guaranteed to be filled left to right, we can check the last character
    // Do not try to insert when it is filled
    if (code.value[code.value.length - 1]) {
      return;
    }
    insertCodeAt(index, value);
  } else {
    setCodeAt(index, value);
  }
  // Only move to next input if we're not at the last input
  if (activeIndex.value < props.count - 1) {
    focusNextInput();
  }
};

// Handle cases of backspace, delete, left arrow, right arrow
const handleKeydown = (index: number, event: KeyboardEvent) => {
  switch (event.key) {
    case 'Backspace':
      event.preventDefault();
      if (props.continuous) {
        // Continuous input mimics normal continuous input behavior
        // User cannot select fields to the right of the first empty field
        // Backspace deletes the character BEHIND the cursor if the current field is empty
        if (index === props.count - 1 && code.value[index]) {
          // Do not move the focus when there's value at the current field
          setCodeAt(index, '');
        } else {
          shiftCodeAt(code.value[index] ? index : index - 1);
          focusPrevInput();
        }
      } else {
        if (code.value[index]) {
          // Do not move the focus when there's value at the current field
          setCodeAt(index, '');
        } else {
          setCodeAt(index - 1, '');
          focusPrevInput();
        }
      }
      break;
    case 'Delete':
      event.preventDefault();
      if (props.continuous) {
        // Shift left once but don't move the focus to mimic usual input behavior
        shiftCodeAt(index);
      } else {
        setCodeAt(index, '');
      }
      break;
    case 'ArrowLeft':
      event.preventDefault();
      focusPrevInput();
      break;
    case 'ArrowRight':
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
        :focus="activeIndex === i"
        :value="code[i]"
        :input-type="inputType"
        :inputmode="inputmode"
        :class="[...(inputClass ? [inputClass] : []), ...(conditionalClass?.[i] ? [conditionalClass?.[i]] : [])]"
        :last-child="i === count - 1"
        :replace="!continuous"
        :placeholder="placeholder?.[i]"
        :fake-disabled="continuous && complete"
        :disabled="disabled"
        @update:value="(value) => handleValueUpdate(i, value)"
        @keydown="(event) => handleKeydown(i, event)"
        @paste="(event) => handlePaste(i, event)"
        @autofill="(value) => handleAutofill(i, value)"
        @focus="handleFocus(i)"
        @blur="handleBlur"
      />
      <span v-if="separator && i !== count - 1">
        <span>{{ separator }}</span>
      </span>
    </div>
  </div>
</template>
