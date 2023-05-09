<script lang="ts">
  import { goto } from '$app/navigation';
  import { onMount } from 'svelte';

  // get params 'id', 'perm' and 'scope' from query
  let id: string, perm: string, scope: string;
  let loaded = false;
  let rediring = false;
  const getPerm = (perm: string, ours: boolean) =>
    `&${ours ? 'p' : 'permissions'}=${encodeURIComponent(perm)}`;
  const getScope = (scope: string, ours: boolean) =>
    `&${ours ? 's' : 'scope'}=${encodeURIComponent(scope)}`;
  const getLink = (id: string, perm: string, scope: string, ours = false) =>
    `${
      ours
        ? `${window.location.protocol}//${
            document.location.host ?? document.location.hostname
          }/?id`
        : 'https://discord.com/api/oauth2/authorize?client_id'
    }=${encodeURIComponent(id)}${
      ours ? (perm === '8' ? '' : getPerm(perm, ours)) : getPerm(perm, ours)
    }${
      ours
        ? scope === 'bot applications.commands' ||
          scope === 'bot%20applications.commands'
          ? ''
          : getScope(scope, ours)
        : getScope(scope, ours)
    }`;
  let err: string | undefined = undefined;
  onMount(() => {
    const query = new URLSearchParams(window.location.search);
    id = query.get('id') ?? '';
    perm = query.get('perm') ?? query.get('p') ?? '8';
    scope = query.get('scope') ?? query.get('s') ?? 'bot applications.commands';
    if (id && perm && scope) {
      window.location.replace(getLink(id, perm, scope));
      rediring = true;
    }
    loaded = true;
  });
  $: {
    if (!id) err = 'No Client ID';
    else if (isNaN(Number(id))) err = 'Client ID is not a number';
    else if (Number(id) < 1000000000) err = 'Client ID is wayy too small';
    else if (isNaN(Number(perm)) && perm)
      err = 'Client Permissions is not a number';
    else err = undefined;
  }
</script>

<svelte:head>
  <title>bot invite</title>
  <meta name="description" content="bot stuff" />
</svelte:head>

<main style="--yee: {id ? 'rgb(170, 255, 170)' : 'rgba(255, 170, 170, 0.3)'}">
  {#if loaded}
    {#if rediring}
      <h1>Redirecting...</h1>
    {:else}
      <span>Client ID</span>
      <input type="text" bind:value={id} placeholder="1105363089047695328" /><br
      />
      <span>Bot Permissions</span>
      <input type="number" bind:value={perm} placeholder="8" /><br />
      <span>Client Scope</span>
      <input
        type="text"
        bind:value={scope}
        placeholder="bot applications.commands"
      />
      <div style="margin-bottom:8px;" />
      {#if id && !err}
        <a href={getLink(id, perm, scope, true)} target="_blank"
          >{getLink(id, perm, scope, true)}</a
        >
      {:else}
        <p style="display: inline-block;margin: 0 0;color: #faa">
          {err ?? 'Unknown Error'}
        </p>
      {/if}
    {/if}
  {:else}
    <h1>Loading...</h1>
  {/if}
</main>

<style lang="scss">
  main {
    filter: drop-shadow(0px 0px 4px var(--yeee));
    span,
    input {
      width: 202px;
      display: inline-block;
    }
    span {
      text-align: right;
    }
    input {
      color: #fff;
      background: #333;
      border: 1px solid #555;
    }
  }
</style>
