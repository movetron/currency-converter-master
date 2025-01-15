<script lang="ts">
  export let currencies: [string],
    defaults: Defaults,
    name: string,
    storage: Storage;
  import { BASE, MASTER } from "./stores";
  import { onMount } from "svelte";
  import numeral from "numeral";
  import { slide } from "svelte/transition";

  interface Defaults {
    BASE: string;
    MASTER: number;
  }

  interface Storage {
    currency: string;
    value: number;
  }

  let error;
  const stor = {
    cur: window.localStorage.getItem(storage.currency),
    val: window.localStorage.getItem(storage.value.toString()),
  };

  // Update storage when currency changes
  $: window.localStorage.setItem(storage.currency, $BASE);
  $: window.localStorage.setItem(storage.value, $MASTER.toString());

  async function getExchangeRates(base: string) {
    const key: string = "cd08a31a4b50cbf89c21208b";
    try {
      const response = await fetch(
        `https://v6.exchangerate-api.com/v6/${key}/latest/${base}`
      );
      return await response.json();
    } catch (e) {
      console.error(e);
      return (error = e);
    }
  }

  function reset() {
    BASE.set(defaults.BASE);
    MASTER.set(defaults.MASTER);
    return console.log("Reset");
  }

  onMount(async () => {
    if (stor.cur) {
      // Initialize currency default from storage
      BASE.set(stor.cur);
    }
    if (stor.val) {
      MASTER.set(Number(stor.val));
    }
  });
</script>

<header>
  <h1>{name}</h1>
</header>
<main>
  <div id="master-controls">
    <input type="number" bind:value={$MASTER} />
    <select bind:value={$BASE}>
      <option value="" selected disabled>Currency</option>
      {#each currencies as currency}
        <option value={currency}>{currency}</option>
      {/each}
    </select>
    <button on:click={reset}>reset</button>
  </div>
  {#if error}
    <p>Error: {error}</p>
  {/if}
  {#await getExchangeRates($BASE)}
    <p id="loading">loading...</p>
  {:then data}
    <div id="conversions">
      {#each currencies as currency}
        <form transition:slide>
          <input
            type="text"
            name={currency}
            class={data.conversion_rates[currency] > 1 ? "good" : "bad"}
            value={numeral(data.conversion_rates[currency] * $MASTER).format(
              "0,0.00"
            )}
            readonly
          />
          <label for={currency}>{currency}</label>
        </form>
      {/each}
    </div>
  {/await}
</main>

<style type="text/scss">
  header {
    text-align: center;
    padding: 1rem 0;
  }

  main {
    #master-controls {
      display: flex;
      justify-content: center;
      padding: 1rem 0;
      * + * {
        margin-left: 1rem;
      }
    }
    #conversions {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-around;
      form {
        display: flex;
        flex-direction: column;
        justify-content: stretch;
        padding: 0.5rem;
        input {
          border: none;
        }
        label {
          border-top: 1px solid var(--primary);
        }
      }
    }
    #loading {
      text-align: center;
    }
  }
</style>
