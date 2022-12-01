# Handling promises

![](https://user-images.githubusercontent.com/5574267/130862554-652397ed-1b1e-40d4-a250-c38734ec8e5d.png)

## toast.loading

If you want to take care of each step yourself you can use `toast.loading` and update the notification yourself.

::: sandbox
```vue App.vue [active]
<script setup>
import { toast } from 'vue3-toastify';
import Msg from './Msg.vue';
import 'vue3-toastify/dist/index.css';

const notify = () => {
  const id = toast.loading('Please wait...');

  setTimeout(() => {
    toast.update(id, {
      render: Msg,
      autoClose: true,
      closeOnClick: true,
      closeButton: true,
      type: 'success',
      isLoading: false,
    });

    setTimeout(() => {
      // done
      toast.done(id);
    }, 1000);
  }, 2000);
};
</script>

<template>
  <div>
    <button @click="notify">toast</button>
  </div>
</template>
```

```vue /src/Msg.vue
<script>
import { ToastProps } from 'vue3-toastify';
import { PropType } from 'vue';

export default {
  name: 'Msg',
  props: {
    closeToast: Function,
    toastProps: Object,
    // for ts
    // closeToast: Function as PropType<(e?: MouseEvent) => void>,
    // toastProps: Object as PropType<ToastProps>,
  },
};
</script>

<template>
  <div>
    <p>I am a vue component</p>
    <p>Position: {{ toastProps?.position }}</p>
    <button
      @click="($event) => { closeToast && closeToast($event) }"
    >
      Click me to close toast
    </button>
  </div>
</template>
```
:::

## toast.promise

The library exposes a `toast.promise` function. Supply a promise or a function that return a promise and the notification will be updated if it resolves or fails. When the promise is pending a spinner is displayed.

Let's start with a simple example

:::sandbox
```vue App.vue
<script setup>
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';

const displayPromise = () => {
  const resolveAfter3Sec = new Promise(resolve => setTimeout(resolve, 3000));
  toast.promise(
    resolveAfter3Sec,
    {
      pending: 'Promise is pending',
      success: 'Promise resolved 👌',
      error: 'Promise rejected 🤯',
    },
  );

  const functionThatReturnPromise = () => new Promise((resolve, reject) => setTimeout(reject, 3000));
  toast.promise(
    functionThatReturnPromise,
    {
      pending: 'Promise is pending',
      success: 'Promise resolved 👌',
      error: 'Promise rejected 🤯',
    },
    {
      position: toast.POSITION.BOTTOM_CENTER,
    },
  );
};
</script>

<template>
  <div>
    <button @click="displayPromise">display promise</button>
  </div>
</template>
```
:::

:::tip
`toast.promise` return the provided promise so you can chain it
:::

```js
const response = await toast.promise(
  fetch('A_URL'),
  {
    pending: 'Promise is pending',
    success: 'Promise resolved 👌',
    error: 'Promise rejected 🤯',
  },
);
console.log(response);
```

Displaying a simple message is what you would want to do in 90% of cases. But what if the message you want to display depends on the promise response, what if you want to change some options for the error notification? Rest assured, under the hood, the library uses `toast.update`. Thanks to this, you have full control over each notification.


:::sandbox
```vue App.vue
<script setup>
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';

const displayPromise = () => {
  const resolveWithSomeData = new Promise((resolve, reject) => setTimeout(() => resolve({ message: 'world' }), 3000));
  toast.promise(
    resolveWithSomeData,
    {
      pending: {
        render() {
          return "I'm loading";
        },
        // other options
        icon: false,
      },
      success: {
        render(res) {
          return 'Hello ' + res.data.message;
        },
        // other options
        icon: '🟢',
      },
      error: {
        render(err) {
          // When the promise reject, data will contains the error
          return h('div', 'Err: ' + err.data.message);
          // return 'Err: ' + err.data.message;
        },
        // render: 'just text',
        // render: h('div', 'error'),
      },
    },
  );
};
</script>

<template>
  <div>
    <button @click="displayPromise">display promise</button>
  </div>
</template>
```
:::
