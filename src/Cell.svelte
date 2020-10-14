<script>
  import Spinner from 'svelte-spinner';
  export let promise;
</script>

{#await promise}<Spinner />{:then response}
  <slot {response} />
{:catch error}<b>{error}</b>{/await}