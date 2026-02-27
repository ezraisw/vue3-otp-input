# vue3-otp-input

> Vue 3 OTP Input is a 5.0 KB fully customizable OTP (one-time password) input component for OTPs, transaction pins, and passwords built with Vue 3.x and Vue Composition API.


## Documentation

<a href="https://deepwiki.com/ejirocodes/vue3-otp-input" target="_blank">Link to doc</a>


## 📹 Demo

![Gifphy](https://media.giphy.com/media/Db04PkC7vMzqksOpq6/giphy.gif)

## ⚙️ Installation

### Install as package

You can install `vue3-otp-input` via the terminal.

```sh
pnpm i vue3-otp-input
```

OR

### Install via CDN

```sh
<script src="https://unpkg.com/vue3-otp-input"></script>

<script src="https://cdn.jsdelivr.net/npm/vue3-otp-input"></script>

```

## 📖 Usage [Demo](https://github.com/ejirocodes/vue3-otp-input/blob/main/src/App.vue)

### 1/3. Register as a Vue component locally OR

```ts
<script setup lang="ts">
import { ref } from 'vue';
import VOtpInput from 'vue3-otp-input';

const bindValue = ref<string[]>([]);

const handleComplete = (value: string) => {
  console.log('OTP completed: ', value);
};

const handleChange = (value: string) => {
  console.log('OTP changed: ', value);
};

const clear = () => {
  bindValue.value = [];
};

const fill = (value: string) => {
  bindValue.value = value.split('');
};
</script>
```

### 1/3. Register as a Vue component globally

```js
//  main.js or main.ts
import { createApp } from 'vue'
import App from './App.vue'

import VOtpInput from "vue3-otp-input";

const app = createApp(App)

app.component('v-otp-input', VOtpInput).mount('#app')

```

### 2/3. Use the registered component in your Vue template

```html
<template>
    <div style="display: flex; flex-direction: row">
      <VOtpInput
        v-model:value="bindValue"
        input-class="otp-input"
        :conditional-class="['one', 'two', 'three', 'four']"
        separator="-"
        input-type="letter-numeric"
        :count="4"
        auto-focus
        continuous
        :placeholder="['*', '*', '*', '*']"
        @change="handleChange"
        @complete="handleComplete"
      />
    </div>
    <button @click="clear()">Clear Input</button>
    <button @click="fill('2929')">Fill Input</button>
</template>
```

### 3/3 [Optional]. Some basic styling options

#### This css has to go into a `<style>` tag which does not have scoped attributed

```css
<style>
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
/* Background colour of an input field with value */
.otp-input.is-complete {
  background-color: #e4e4e4;
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
```

## 🍔 Props

<table>
  <tr>
    <th>Name<br></th>
    <th>Type</th>
    <th>Required</th>
    <th>Default</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>value</td>
    <td>string[]</td>
    <td>false</td>
    <td>[]</td>
    <td>v-model:value for two-way binding. Each element represents one input's value.</td>
  </tr>
  <tr>
    <td>count</td>
    <td>number</td>
    <td>false</td>
    <td>4</td>
    <td>Number of OTP inputs to be rendered.</td>
  </tr>
  <tr>
    <td>separator</td>
    <td>string</td>
    <td>false</td>
    <td>""</td>
    <td>Provide a custom separator between inputs. For instance, <code>separator="-"</code> would add <code>-</code> between each input.</td>
  </tr>
  <tr>
    <td>input-class</td>
    <td>string | string[]</td>
    <td>false</td>
    <td>""</td>
    <td>Style applied or class passed to each input.</td>
  </tr>
  <tr>
      <td>input-type</td>
      <td>string</td>
      <td>false</td>
      <td>"letter-numeric"</td>
      <td>Input type. Optional value: "password", "number", "tel", "letter-numeric".</td>
  </tr>
  <tr>
      <td>inputmode</td>
      <td>string</td>
      <td>false</td>
      <td>"text"</td>
      <td>This allows a browser to display an appropriate virtual keyboard. Optional value: "numeric", "text", "tel". <a href="https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/inputmode">Learn More</a></td>
  </tr>
  <tr>
    <td>auto-focus</td>
    <td>boolean</td>
    <td>false</td>
    <td>false</td>
    <td>Auto focuses input on initial page load.</td>
  </tr>
  <tr>
    <td>continuous</td>
    <td>boolean</td>
    <td>false</td>
    <td>false</td>
    <td>Makes the inputs behave like a single continuous text field: characters insert and shift rather than replacing in-place, Backspace/Delete shift remaining characters, and focus is constrained sequentially. See note below.</td>
  </tr>
  <tr>
    <td>placeholder</td>
    <td>string[]</td>
    <td>false</td>
    <td>[]</td>
    <td>Specify an expected value for each input. Example: <code>:placeholder="['*', '*', '*', '*']"</code>. The length of this array should be equal to <code>count</code>.</td>
  </tr>
  <tr>
    <td>conditional-class</td>
    <td>string[]</td>
    <td>false</td>
    <td>[]</td>
    <td>Specify a class to be applied to each input based on its index. Example: <code>:conditional-class="['one', 'two', 'three', 'four']"</code>. The length of this array should be equal to <code>count</code>.</td>
  </tr>
  <tr>
    <td>disabled</td>
    <td>boolean</td>
    <td>false</td>
    <td>false</td>
    <td>Disables all the input fields.</td>
  </tr>
</table>

## 🐴 Events

<table>
  <tr>
    <th>Name<br></th>
    <th>Description</th>
  </tr>
  <tr>
     <td>change</td>
     <td>Emitted when the OTP value changes. Returns the joined OTP string.</td>
    </tr>
  <tr>
    <td>complete</td>
    <td>Emitted when all inputs are filled. Returns the joined OTP string.</td>
  </tr>
</table>

> **Note:** `clearInput()` and `fillInput()` methods have been removed. Use `v-model:value` to clear or fill inputs directly (e.g., `bindValue = []` to clear, `bindValue = '1234'.split('')` to fill).

> **About `continuous` mode:** This mode makes the OTP input behave more like a standard text field split across boxes — characters shift on insert and delete, and you cannot skip ahead to arbitrary inputs. However, this may feel unintuitive for OTP entry, where users typically expect each box to be an independent slot they can click and edit freely. Use this mode only if your UX specifically calls for sequential, typewriter-style input.

## 🤟🏽 License

[MIT](https://choosealicense.com/licenses/mit/)

## 📓 Contributing

Contributions are always welcome!

See `contributing.md` for ways to get started.

Please adhere to this project's `code of conduct`.

## 🧸 Appendix

This component is a rewite of [vue-otp-input](https://github.com/bachdgvn/vue-otp-input) component to support Vue 3.x. Feel free to use it in your project, report bugs and make PR 👏🏽.
