<script>
  import { quintOut } from "svelte/easing";
  import { crossfade } from "svelte/transition";
  import { flip } from "svelte/animate";
  import Icon from "svelte-awesome";
  import { times, plus } from "svelte-awesome/icons";

  const [send, receive] = crossfade({
    duration: d => Math.sqrt(d * 200),

    fallback(node, params) {
      const style = getComputedStyle(node);
      const transform = style.transform === "none" ? "" : style.transform;

      return {
        duration: 600,
        easing: quintOut,
        css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
      };
    }
  });

  function add(e) {
    e.preventDefault();
    const todo = {
      id: uid++,
      done: false,
      description: inputValue
    };

    todos = [todo, ...todos];
    inputValue = "";
  }

  function remove(todo) {
    todos = todos.filter(t => t !== todo);
  }

  function mark(todo, done) {
    const index = todos.indexOf(todo);
    todos[index].done = done;
  }

  let inputValue = "";
  let uid = 1;
  let todos = [];

  // load initial state from localStorage (if any)
  if (localStorage.todos) {
    todos = JSON.parse(localStorage.todos) || [];
    uid = todos.reduce((p, c) => (c.id > p ? c.id : p), 0) + 1;
  }

  // keep localStorage synced with current state
  $: {
    localStorage.todos = JSON.stringify(todos);
  }
</script>

<style>
  :global(:root) {
    --todo-input-text: black;
    --todo-input-bg: white;
    --todo-input-border: 1px solid #ccc;
    --todo-item-text: #333;
    --todo-item-bg: hsl(240, 8%, 93%);
    --todo-item-bg-hover: white;
    --todo-item-border: 1px solid hsl(240, 8%, 70%);
    --todo-item-done-bg: hsl(240, 8%, 98%);
    --todo-item-done-border: 1px solid hsl(240, 8%, 90%);
    --todo-font-family: "Roboto", "Segoe UI", Tahoma, Geneva, Verdana,
      sans-serif;
    --todo-remove-bg: black;
    --todo-remove-text: white;
  }
  .svelte-todo {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 1em;
    max-width: 36em;
    margin: 0 auto;
    font-family: var(--todo-font-family);
  }
  label {
    display: block;
  }
  input,
  button {
    font-family: inherit;
    font-size: inherit;
    padding: 0.4em;
    margin: 0 0 0.5em 0;
    box-sizing: border-box;
  }
  button {
    user-select: none;
  }
  input {
    border: var(--todo-input-border);
    color: var(--todo-input-text);
    background-color: var(--todo-input-bg);
  }
  .svelte-todo > form {
    font-size: 1.4em;
    grid-column: 1/3;
    display: flex;
  }
  .svelte-todo > form > button {
    flex-grow: 1;
  }
  .svelte-todo > form > input {
    flex-grow: 5;
  }
  h2 {
    font-size: 2em;
    font-weight: 200;
    user-select: none;
    margin: 0 0 0.5em 0;
  }
  .todo-item {
    position: relative;
    line-height: 1.2;
    padding: 0.5em 2.5em 0.5em 2em;
    margin: 0 0 0.5em 0;
    border-radius: 2px;
    user-select: none;
    border: var(--todo-item-border);
    background-color: var(--todo-item-bg);
    color: var(--todo-item-text);
  }
  .todo-item:hover {
    background-color: var(--todo-item-bg-hover);
  }
  .todo-item input[type="checkbox"] {
    position: absolute;
    left: 0.5em;
    top: 0.7em;
    margin: 0;
  }
  .done {
    border: var(--todo-item-done-border);
    background-color: var(--todo-item-done-bg);
  }
  .removeButton {
    position: absolute;
    top: 0.25rem;
    right: 0.45rem;
    width: 1.6rem;
    height: 1.6rem;
    color: var(--todo-remove-text);
    background: var(--todo-remove-bg);
    border-radius: 5rem;
    border: none;
    opacity: 0.5;
    transition: opacity 0.2s;
    cursor: pointer;
    font-size: 0.7em;
  }

  .removeButton:hover,
  .removeButton:focus {
    opacity: 1;
  }
</style>

<div class="svelte-todo">
  <form on:submit={add}>
    <!-- svelte-ignore a11y-autofocus -->
    <input
      autofocus
      class="svelte-todo-input"
      placeholder="what needs to be done?"
      bind:value={inputValue}
      aria-label="what needs to be done?" />
    <button disabled={!inputValue}>
      <Icon data={plus} />
      add
    </button>
  </form>
  {#if todos.length}
    <div class="left">
      <h2>todo</h2>
      {#each todos.filter(t => !t.done) as todo (todo.id)}
        <label
          class="todo-item"
          in:receive={{ key: todo.id }}
          out:send={{ key: todo.id }}
          animate:flip={{ duration: 200 }}>
          <input type="checkbox" on:change={() => mark(todo, true)} />
          {todo.description}
          <button
            class="removeButton"
            on:click={() => remove(todo)}
            aria-label="Remove">
            <Icon data={times} />
          </button>
        </label>
      {/each}
    </div>
    <div class="right">
      <h2>done</h2>
      {#each todos.filter(t => t.done) as todo (todo.id)}
        <label
          class="todo-item done"
          in:receive={{ key: todo.id }}
          out:send={{ key: todo.id }}
          animate:flip={{ duration: 200 }}>
          <input type="checkbox" checked on:change={() => mark(todo, false)} />
          {todo.description}
          <button
            class="removeButton"
            on:click={() => remove(todo)}
            aria-label="Remove">
            <Icon data={times} />
          </button>
        </label>
      {/each}
    </div>
  {/if}
</div>
