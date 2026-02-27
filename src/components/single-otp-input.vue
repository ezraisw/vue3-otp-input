<script setup lang="ts">
import { computed, ref, watch } from 'vue';

type Props = {
  inputType?: 'number' | 'tel' | 'letter-numeric' | 'password';
  inputmode?: 'none' | 'text' | 'tel' | 'url' | 'email' | 'numeric' | 'decimal' | 'search';
  focus?: boolean;
  lastChild?: boolean;
  replace?: boolean;
  placeholder?: string;
  fakeDisabled?: boolean;
  disabled?: boolean;
};

const props = withDefaults(defineProps<Props>(), {
  inputType: 'tel',
  inputmode: 'numeric',
  focus: false,
  lastChild: false,
  replace: false,
  placeholder: '',
  fakeDisabled: false,
  disabled: false,
});

const emit = defineEmits<{
  keydown: [event: KeyboardEvent];
  paste: [event: ClipboardEvent];
  autofill: [value: string];
  focus: [];
  blur: [];
}>();

const value = defineModel<string>('value', { default: '' });
const input = ref<HTMLInputElement | null>(null);

const handleChange = (event: Event) => {
  const inputEvent = event as InputEvent;
  const inputElement = event.target as HTMLInputElement;
  const newValue = inputElement.value;

  // If fake disabled, do not allow any changes
  if (props.fakeDisabled) {
    inputElement.value = value.value;
    return;
  }

  // Multi character insertion means that it is an autofill
  // E.g., if input was "4" and user typed "5", insertedData is "5".
  // E.g., if iOS autofilled "1234", insertedData is "1234".
  const insertedValue = inputEvent.data;
  if (insertedValue && insertedValue.length > 1) {
    emit('autofill', insertedValue);
    inputElement.value = value.value;
    return;
  }

  // Fallback for older browsers where .data might be null,
  // checking if the length jumped by more than 1 character.
  if (!insertedValue && newValue.length - value.value.length > 1) {
    emit('autofill', newValue.trim());
    inputElement.value = value.value;
    return;
  }

  // Accidental multi character insertion on the same input
  if (newValue.length > 1) {
    const usedValue = props.replace ? (insertedValue ?? newValue.slice(-1)) : newValue.charAt(0);

    value.value = usedValue;
    // Force DOM sync
    inputElement.value = usedValue;
    return;
  }

  // Normal case
  value.value = newValue;
};

const allowedFunctionalKeys = [
  'Backspace',
  'Tab',
  'Enter',
  'ArrowLeft',
  'ArrowRight',
  'Delete',
];

const handleKeydown = (event: KeyboardEvent) => {
  if (props.disabled) {
    event.preventDefault();
    return;
  }

  // Only allow characters 0-9, DEL, Backspace, Enter, Right and Left Arrows, and pasting
  const isNumber = /^[0-9]$/.test(event.key);
  const isLetter = /^[a-zA-Z]$/.test(event.key);
  const isModifier = event.ctrlKey || event.metaKey;
  const isAllowedType = isNumber || (props.inputType === 'letter-numeric' && isLetter);

  if (isAllowedType || allowedFunctionalKeys.includes(event.key) || isModifier) {
    emit('keydown', event);
  } else {
    event.preventDefault();
  }
};

const handlePaste = (event: ClipboardEvent) => {
  emit('paste', event);
};

const handleFocus = () => {
  // Select the value so it can be immediately deleted using Backspace/Delete
  input.value?.select();
  emit('focus');
};

const handleBlur = () => {
  emit('blur');
};

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
