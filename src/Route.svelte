<script>
  import { onDestroy, getContext } from 'svelte';
  import { CTX_ROUTER } from './utils';

  export let key = null;
  export let path = '';
  export let props = null;
  export let exact = undefined;
  export let fallback = undefined;
  export let component = undefined;
  export let condition = undefined;
  export let redirect = undefined;

  const { assignRoute, unassignRoute, routeInfo } = getContext(CTX_ROUTER);

  let activeRouter = null;
  let activeProps = {};
  let fullpath;

  function getProps(given, required) {
    const { props, ...others } = given;

    // Fix(#62):https://github.com/kazzkiq/svero/issues/62
    // Err: required.ForEach is not a function
    required = typeof required === 'object' ? Object.keys(required) : required;

    // prune all declared props from this component
    required.forEach(k => {
      delete others[k];
    });

    return {
      ...props,
      ...others,
    };
  }

  [key, fullpath] = assignRoute(key, path, { condition, redirect, fallback, exact });

  $: {
    activeRouter = $routeInfo[key];
    activeProps = getProps($$props, arguments[0]['$$'].props);
  }

  onDestroy(() => {
    unassignRoute(fullpath);
  });
</script>

{#if activeRouter}
  {#if component}
    <svelte:component this={component} router={activeRouter} {...activeProps} />
  {:else}
    <slot router={activeRouter} props={activeProps} />
  {/if}
{/if}
