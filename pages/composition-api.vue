<template>
  <main>
    <h1>With Composition API <small>*Experimental</small></h1>
    <h2>data ()</h2>
    <p>
      {{ message }}
    </p>
    <h2>computed ()</h2>
    <p>
      {{ computedMessage }}
    </p>
    <h2>asyncData ()</h2>
    <p>
      {{ asyncMessage }}
    </p>
    <h2>fetch ()</h2>
    <p v-if="fetchState.pending || !fetchState.timestamp">
      Fetching from frontend...
    </p>
    <ul v-else>
      <li
        v-for="todo in fetchedTodos.slice(0, 5)"
        :key="todo.id"
      >
        {{ todo.completed ? '&#9989;' : '&#11036;' }} {{ todo.id }}: {{ todo.title }}
      </li>
    </ul>
    <h2>Vuex Store</h2>
    <ul>
      <li>On root store: {{ descriptionOnStore }}</li>
      <li>On setting store (namespaced): dark mode is <strong>{{ isDarkMode ? 'enabled' : 'disabled' }}</strong></li>
      <a
        href
        @click.prevent="toggleDarkMode"
      >
        Toggle dark mode
      </a>
    </ul>
    <h2>Nuxt Middleware</h2>
    <p>{{ userAgent }}</p>
    <h2>Nuxt Plugin</h2>
    <p>{{ $truncate(userAgent || '') }}</p>
  </main>
</template>

<script lang="ts">
import {
  computed,
  defineComponent,
  ref,
  onBeforeMount,
  onMounted,
  onBeforeUnmount,
  onUnmounted,
  onBeforeUpdate,
  onUpdated,
  useAsync,
  useContext,
  useFetch,
  useMeta
} from '@nuxtjs/composition-api'

import type { RootState } from '~/store'
import { namespace as settingStoreNamespace, SettingState, actionType } from '~/store/setting'

// types required for data
interface ToDo {
  userId: number
  id: number
  title: string
  completed: boolean
}

export default defineComponent({
  fetchOnServer: false,
  middleware: 'user-agent',

  props: {
    // Basic type check
    //  (`null` and `undefined` values will allow any type)
    propA: Number,
    // Multiple possible types
    propB: [String, Number],
    // Required string
    propC: {
      type: String,
      required: true
    }
  },

  setup (props, context) {
    // data
    const message = ref("I'm defined on data()")
    const fetchedTodos = ref<ToDo[]>([])

    //lifecycle hooks
    onBeforeMount(() => {}) // When this hook is called, the component has finished setting up its reactive state, but no DOM nodes have been created yet. It is about to execute its DOM render effect for the first time.
    onMounted(() => {}) // All of its synchronous child components have been mounted and its own DOM tree has been created and inserted into the parent container.
    onBeforeUnmount(() => {}) // When this hook is called, the component instance is still fully functional.
    onUnmounted(() => {}) // Use this hook to clean up manually created side effects such as timers, DOM event listeners or server connections.

    onBeforeUpdate(() => {}) // This hook can be used to access the DOM state before Vue updates the DOM. It is also safe to modify component state inside this hook.
    onUpdated(() => {}) // This hook is called after any DOM update of the component, which can be caused by different state changes. If you need to access the updated DOM after a specific state change, use nextTick() instead.


    // computed
    const descriptionOnStore = computed(() => (context.root.$store.state as RootState).description)
    const computedMessage = computed(() => message.value.replace('data()', 'computed()'))
    const isDarkMode = computed(() => (context.root.$store.state.setting as SettingState).darkMode)

    // methods
    const toggleDarkMode = (): void => {
      context.root.$store.dispatch(`${settingStoreNamespace}/${actionType.TOGGLE_DARK_MODE}`)
    }

    const asyncMessage = useAsync(() => "I'm defined on asyncData()")
    // Note that `route`, `query`, `from` and `params` are reactive refs (accessed with `.value`), but the rest of the context is not.
    // To smooth your upgrade to Nuxt 3, it is recommended not to access `route`, `query`, `from` and `params` from `useContext` but rather to use the `useRoute` helper function.
    const userAgent = useAsync(() => useContext().userAgent)

    // `useFetch` must be called synchronously within `setup()`. Any changes made to component data - that is, to properties _returned_ from `setup()` - will be sent to the client and directly loaded. Other side-effects of `useFetch` hook will not be persisted.
    // `useFetch` should be used with `ref`s and not `ssrRef`s because state serialization and hydration is already covered by `useFetch`. Else, the state would be sent from server to client "twice", via the `ssrRef` and via `useFetch`
    // `$fetch` and `$fetchState` will already be defined on the instance - so no need to return `fetch` or `fetchState` from setup.
    const { fetchState } = useFetch(() => {
      return window.fetch('https://jsonplaceholder.typicode.com/todos')
        .then(response => response.json())
        .then((data: ToDo[]) => { fetchedTodos.value = data })
    })

    // set some meta tags
    useMeta(() => ({
      title: 'Composition API Demo',
      meta: [{
        name: 'message',
        content: computedMessage.value
      }]
    }))

    // allow access after setup to these
    return {
      message,
      asyncMessage,
      userAgent,
      fetchState,
      fetchedTodos,
      toggleDarkMode,
      descriptionOnStore,
      computedMessage,
      isDarkMode
    }
  },

  head: {}
})
</script>
