<script lang="ts">
  import UserLink from "$lib/components/UserLink.svelte";
  import type { PageData } from "./$types";
  import { userData, auth } from "$lib/firebase";
  import {
    signInWithPopup,
    GoogleAuthProvider,
    signOut,
  } from "firebase/auth";
    import { goto } from "$app/navigation";
    import { toasts, ToastContainer, FlatToast }  from "svelte-toasts";

  export let data: PageData;
  async function signInWithGoogle() {
    const provider = new GoogleAuthProvider();
    const user = await signInWithPopup(auth, provider);
    console.log(user);
  }
  async function signOutSSR() {
    await signOut(auth)
        goto('/')
  }
  const showToast = () => {
    const toast = toasts.add({
      description: 'Copied!',
      duration: 3000, // 0 or negative to avoid auto-remove
      placement: 'bottom-center',
      theme: 'dark',
      type: 'success',
      onClick: () => {},
      // component: BootstrapToast, // allows to override toast component/template per toast
    });
    };
  function copyURL() {
    const copytext = `https://nmd.mov/${$userData?.username}`
    navigator.clipboard.writeText(copytext)
    showToast()
  }
</script>

<svelte:head>
  <title>@{data.username} Links</title>
  <meta name="description" content="{data.bio}" />
  <meta property="og:image" content="{data.photoURL}" />
  <meta property="og:title" content="{data.username} Links" />
  <meta property="og:description" content="{data.bio}" />
  <meta property="og:url" content="https://nmd.mov/{data.username}" />
</svelte:head>

<main class=" prose text-center mx-auto mt-8">
  <h1 class="text-4xl text-teal-700">
    @{data.username}
  </h1>
  <img
    src={data.photoURL ?? "/user.png"}
    alt="profile"
    width="128"
    class="mx-auto rounded-2xl mt-8"
  />
  <p class="text-xl mx-auto my-8 whitespace-pre-line">
    {data.bio ?? "no bio yet..."}
  </p>
</main>

<div class="mx-auto">
  <ul class=" list-none min-w-96">
    {#each data.links as item}
      <div class="my-2">
        <UserLink {...item} />
      </div>
    {/each}
  </ul>
  <br />
</div>
{#if $userData?.username == data.username}
  <div class=" top-0 right-0 absolute">
    <a href="/{$userData?.username}/edit">
      <button class="btn btn-ghost text-xs btn-xs mx-3 my-3"
        >Edit Profile</button
      >
      <button
        class="btn btn-error text-xs btn-xs mx-3 my-3"
        on:click={signOutSSR}>Sign Out</button
      >
    </a>
  </div>
  <div class="mx-auto">
  <button on:click={copyURL} class="mx-auto btn btn-neutral">share</button>
  <ToastContainer let:data={data}>
		<FlatToast {data}  />
	</ToastContainer>
  </div>

{:else}
  <div class="top-0 right-0 absolute">
    <button
      class="btn btn-success text-xs btn-xs mx-3 my-3"
      on:click={signInWithGoogle}>Sign in</button
    >
  </div>
{/if}
