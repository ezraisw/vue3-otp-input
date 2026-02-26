<script setup lang="ts">
import { computed, ref, watch } from 'vue';

type Props = {
  inputType?: 'number' | 'tel' | 'letter-numeric' | 'password';
  inputmode?: 'none' | 'text' | 'tel' | 'url' | 'email' | 'numeric' | 'decimal' | 'search';
  focus?: boolean;
  lastChild?: boolean;
  placeholder?: string;
  disabled?: boolean;
};

const props = withDefaults(defineProps<Props>(), {
  inputType: 'tel',
  inputmode: 'numeric',
  focus: false,
  lastChild: false,
  placeholder: '',
  disabled: false,
});

const emit = defineEmits<{
  keydown: [event: KeyboardEvent];
  paste: [event: ClipboardEvent];
  focus: [];
  blur: [];
}>();

const value = defineModel<string>('value', { default: '' });
const input = ref<HTMLInputElement | null>(null);

const handleChange = (e: Event) => {
  const newValue = (e.target as HTMLInputElement).value;
  if (newValue && newValue.trim().length > 1) {
    // This is a workaround for dealing IOS does not fire onPaste event from sms auto-populate
    // The `:maxlength="lastChild ? 1 : undefined"` on the input is also needed for this workaround
    (e as unknown as { clipboardData: { getData: () => string } }).clipboardData = {
      getData: () => newValue.trim(),
    };
    emit('paste', e as ClipboardEvent);
    return;
  }
  value.value = newValue;
};

const isCodeLetter = (charCode: number) => charCode >= 65 && charCode <= 90;
const isCodeNumeric = (charCode: number) => (charCode >= 48 && charCode <= 57) || (charCode >= 96 && charCode <= 105);

const handleKeydown = (event: KeyboardEvent) => {
  if (props.disabled) {
    event.preventDefault();
  }

  // Only allow characters 0-9, DEL, Backspace, Enter, Right and Left Arrows, and pasting
  const keyEvent = event || window.event;
  const charCode = keyEvent.which ? keyEvent.which : keyEvent.keyCode;
  // Allow any input on iOS
  if (/iPhone|iPad|iPod/.test(navigator.userAgent)) {
    emit('keydown', event);
    return;
  }

  if (
    isCodeNumeric(charCode) ||
    (props.inputType === 'letter-numeric' && isCodeLetter(charCode)) ||
    [8, 9, 13, 37, 39, 46, 86].includes(charCode)
  ) {
    emit('keydown', event);
  } else {
    keyEvent.preventDefault();
  }
};

const handlePaste = (event: ClipboardEvent) => emit('paste', event);

const handleFocus = () => {
  // Select the value so it can be immediately deleted using Backspace/Delete
  input.value?.select();
  emit('focus');
};

const handleBlur = () => emit('blur');

const actualInputType = computed(() => (['letter-numeric', 'number'].includes(props.inputType) ? 'text' : props.inputType));

watch(
  () => props.focus,
  (newFocus, oldFocus) => {
    // Check if focus has changed
    // Prevent any focusing if "focus" changes
    if (oldFocus === newFocus || !input.value) {
      return;
    }

    if (newFocus) {
      input.value.focus();
      input.value.select();
    } else {
      input.value.blur();
    }
  },
);
</script>

<template>
  <input
    ref="input"
    data-test="single-input"
    :type="actualInputType"
    :inputmode="inputmode"
    :placeholder="placeholder"
    :disabled="disabled"
    autocomplete="one-time-code"
    min="0"
    max="9"
    :maxlength="lastChild ? 1 : undefined"
    pattern="[0-9]"
    :value="value"
    :class="{ 'is-complete': !!value }"
    @input="handleChange"
    @keydown="handleKeydown"
    @paste="handlePaste"
    @focus="handleFocus"
    @blur="handleBlur"
  />
</template>
