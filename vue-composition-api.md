---
title: Composition API
---

## Composables

* Function to encapsulate and reuse stateful logic
* Leverages the Vue Composition API

----

Mouse composable sample

<iframe title="can i use cors" frameborder="0" width="100%" height="500pt" src="https://stackblitz.com/edit/vue-zdd1nk?ctl=1&embed=1&file=src/composables/mouse.js&hideNavigation=1&view=editor"></iframe>

----

Written in Vue 2

<iframe title="can i use cors" frameborder="0" width="100%" height="500pt" src="https://stackblitz.com/edit/vue-dbmxrf?ctl=1&embed=1&file=src/mixins/mouse.js&hideNavigation=1&view=editor"></iframe>

----

Composables may call other composables

<iframe title="can i use cors" frameborder="0" width="100%" height="500pt" src="https://stackblitz.com/edit/vue-gb2tr9?ctl=1&embed=1&file=src/composables/mouse.js&hideNavigation=1&view=editor"></iframe>

---

## Flexible arguments

Arguments might be treated as refs...

```js [3]
// When we need to use a ref in the composable
export default useMyComposable(input) {
  const ref = ref(input);
}
```

----

...or as primitives

```js [3]
// When we need to use a raw value in the composable
export default useMyComposable(input) {
  const rawValue = unref(input);
}
```

----

### Note

* `ref` [creates a new one](https://github.com/vuejs/core/blob/main/packages/reactivity/src/ref.ts#L81) from a `primitive` or returns it if it's a `ref`.
* `unref` [returns the `ref` value](https://github.com/vuejs/core/blob/main/packages/reactivity/src/ref.ts#L138) or the bare `primitive` if it's not a `ref`.

---

## Async State (fetch)

```js
// fetch.js
import { ref } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)

  fetch(url)
    .then((res) => res.json())
    .then((json) => (data.value = json))
    .catch((err) => (error.value = err))

  return { data, error }
}
```

----

### Problem with this composable

* Static URL as input
* Fetch only once
* How to re-fetch when the URL changes?

----

### Solution: `watchEffect` and `refs`

#### watchEffect()

> Runs a function immediately while reactively tracking its dependencies and re-runs it whenever the dependencies are changed.

[[1]](#/references)
----

```js [2|8-13,17|18-26]
// fetch.js
import { ref, isRef, unref, watchEffect } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)

  function doFetch() {
    // reset state before fetching..
    data.value = null
    error.value = null
    // unref() unwraps potential refs
    fetch(unref(url))
      .then((res) => res.json())
      .then((json) => (data.value = json))
      .catch((err) => (error.value = err))
  }

  if (isRef(url)) {
    // setup reactive re-fetch if input URL is a ref
    watchEffect(doFetch)
  } else {
    // otherwise, just fetch once
    // and avoid the overhead of a watcher
    doFetch()
  }

  return { data, error }
}
```

----

Fetch sample

<iframe title="can i use cors" frameborder="0" width="100%" height="500pt" src="https://stackblitz.com/edit/vue-kkecf3?ctl=1&embed=1&file=src/composables/useFetch.js&hideNavigation=1&view=editor"></iframe>

---

## Dynamic return values

* Pattern used ib [`VueUse`](https://vueuse.org/) library
* Allows to return single or multiple values

----

```js
export default useComposable(input, options) {
// 1. Add in the `controls` option
const { controls = false } = options;

...

// 2. Either return a single value or an object
if (controls) {
    return { singleValue, anotherValue, andAnother };
  } else {
    return singleValue;
  }
}
```

---

<!-- .slide: id="references" -->

## References

* [Vue 3 documentation: Composables](https://vuejs.org/guide/reusability/composables.html#composables)
* [[1] watchEffect](https://vuejs.org/api/reactivity-core.html#watcheffect)
