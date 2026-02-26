<script setup lang="ts">
import { ref } from 'vue';
import Vue3OtpInput from '.';

const bindValue = ref<string[]>([]);

const handleComplete = (value: string) => {
  console.log('OTP completed: ', value);
  console.log('OTP v-model:value: ', bindValue.value);
};

const handleChange = (value: string) => {
  console.log('OTP changed: ', value);
  console.log('OTP v-model:value: ', bindValue.value);
};

const clear = () => {
  bindValue.value = [];
};

const fill = () => {
  bindValue.value = '12ee99'.split('');
};
</script>

<template>
  <div style="display: flex; flex-direction: row">
    <button @click="clear">clear</button>
    <button @click="fill">fill</button>
    <Vue3OtpInput
      ref="otpInput"
      v-model:value="bindValue"
      class="otp-input-container"
      input-class="otp-input"
      :conditional-class="['one', 'two', 'three', 'four']"
      separator="-"
      input-type="letter-numeric"
      :count="4"
      auto-focus
      force-ordering
      :placeholder="['*', '*', '*', '*']"
      @change="handleChange"
      @complete="handleComplete"
      @update:value="bindValue = $event"
    />
  </div>
</template>

<style>
.otp-input-container {
  display: flex;
}
.otp-input {
  width: 40px;
  height: 40px;
  padding: 5px;
  margin: 0 10px;
  font-size: 20px;
  border-radius: 4px;
  border: 1px solid rgba(0, 0, 0, 0.3);
  text-align: center;
}
.otp-input.is-complete {
  background-color: #e4e4e4;
}
.otp-input.error {
  border: 1px solid red !important;
}
.otp-input::-webkit-inner-spin-button,
.otp-input::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
input::placeholder {
  font-size: 15px;
  text-align: center;
  font-weight: 600;
}
</style>
