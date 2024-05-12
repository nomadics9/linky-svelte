<script lang="ts">
  import { page } from "$app/stores";
  import { db, user, userData } from "$lib/firebase";
  import {
    arrayRemove,
    arrayUnion,
    doc,
    setDoc,
    updateDoc,
  } from "firebase/firestore";
  import { writable } from "svelte/store";
  import SortableList from "$lib/components/SortableList.svelte";
  //import { SortableList } from "@jhubbardsf/svelte-sortablejs"
  import UserLink from "$lib/components/UserLink.svelte";
  import * as Avatar from "$lib/components/ui/avatar";



  const icons = ["X", "Youtube", "TikTok", "LinkedIn", "GitHub", "Instagram", "Custom"];

  const formDefaults = {
    icon: "custom",
    title: "",
    url: "https://",
  };

  const formData = writable(formDefaults);
  let showForm = false;
  var bio = writable("");

  $: urlIsValid = $formData.url.match(/^(ftp|http|https):\/\/[^ "]+$/);
  $: titleIsValid = $formData.title.length < 20 && $formData.title.length > 0;
  $: formIsValid = urlIsValid && titleIsValid;

  async function addLink(e: SubmitEvent) {
    const userRef = doc(db, "users", $user!.uid);

    await updateDoc(userRef, {
      links: arrayUnion({
        ...$formData,
        id: Date.now().toString(),
      }),
    });

    formData.set({
      icon: "",
      title: "",
      url: "",
    });

    showForm = false;
  }

  async function deleteLink(item: any) {
    const userRef = doc(db, "users", $user!.uid);
    await updateDoc(userRef, {
      links: arrayRemove(item),
    });
  }

   let editing = false
  function editLink(item: any) {
    const userRef = doc(db, "users", $user!.uid)
    showForm = true
    editing = true
    deleteLink(item)
    formData.set(item)
  }

  async function submitEdit () {
    const userRef = doc(db, "users", $user!.uid);
    editing = false
  }

  function cancelLink() {
    formData.set(formDefaults);
    showForm = false;
  }

  function sortList(e: CustomEvent) {
    const newList = e.detail;
    const userRef = doc(db, "users", $user!.uid);
    setDoc(userRef, { links: newList }, { merge: true });
    console.log(newList)
  }

  function updateBio(e: string) {
    const userRef = doc(db, "users", $user!.uid);
    updateDoc(userRef, {
      bio: e,
    });
  }

  let publish = $userData?.published
  function togglePublish() {
    publish = !$userData?.published
    const userRef = doc(db, "users", $user!.uid);
    updateDoc(userRef, {
      published: publish
    });
  }
  publish == $userData?.published
  

  function moveUp(index: any) {
    const userRef = doc(db, "users", $user!.uid)
    const oldlist = $userData?.links
    const newList = [...oldlist!];
    newList[index] = [newList[index-1], (newList[index-1] = newList[index])][0];

    setDoc(userRef, { links: newList }, {merge: true});
  }

    function moveDown(index: any) {
    const oldlist = $userData?.links
    const newList = [...oldlist!];
    newList[index] = [newList[index+1], (newList[index+1] = newList[index])][0];

    const userRef = doc(db, "users", $user!.uid)
    setDoc(userRef, { links: newList }, {merge: true});

  }

  //const loggedin = $userData?.username == $page.params.username;
  const loggedin = $userData?.username == $userData?.username;

</script>


<main class="mx-auto min-w-96 max-w-xl">
  {#if loggedin}
    <div class="relative mx-auto">
    <h1 class="text-3xl font-bold mt-8 mb-4 text-center">
      Edit your Profile
    </h1>
      <a href="/login/photo" class="absolute -top-4 transition-all">
        <Avatar.Root class="size-14">
          <Avatar.Image src="{$userData?.photoURL}" alt="profile photo" />
          <Avatar.Fallback>Profile</Avatar.Fallback>
        </Avatar.Root>
      </a>
    </div>
    <SortableList list={$userData?.links} on:sort={sortList} let:item let:index >
      <li class="group relative mx-auto max-w-80" >
        <UserLink {...item} />
        <button
          on:click={() => deleteLink(item)}
          class="btn btn-xs btn-error  group-hover:visible transition-all absolute -right-6 bottom-10"
          >Delete</button
        >
        <button
          on:click={() => editLink(item)}
        class="btn btn-xs btn-primary  group-hover:visible transition-all absolute -right-6 top-10 min-w-14"
        >Edit</button>
        <button
        on:click={() => moveUp(index)}
      class="btn btn-xs btn-accent  group-hover:visible transition-all absolute -left-6 bottom-10"
      >☝️</button>
      <button
      on:click={() => moveDown(index)}
    class="btn btn-xs btn-error  group-hover:visible transition-all absolute -left-6 top-10 z-50"
    >👇</button>
      </li>
    </SortableList>

    {#if showForm}
      <form
        on:submit|preventDefault={addLink}
        class="bg-base-200 p-10 w-full mx-auto rounded-xl"
      >
        <select
          name="icon"
          class="select select-sm"
          bind:value={$formData.icon}
        >
          {#each icons as icon}
            <option value={icon.toLowerCase()}>{icon}</option>
          {/each}
        </select>
        <input
          name="title"
          type="text"
          placeholder="Title"
          class="input input-sm"
          bind:value={$formData.title}
        />
        <div class=" my-2">
        <input
          name="url"
          type="text"
          placeholder="URL"
          class="input input-sm w-full"
          bind:value={$formData.url}
        />
        </div>
        <div class="my-4">
          {#if !titleIsValid}
            <p class="text-error text-xs">Must have valid title</p>
          {/if}
          {#if !urlIsValid}
            <p class="text-error text-xs">Must have a valid URL</p>
          {/if}
          {#if formIsValid}
            <p class="text-success text-xs">Looks good!</p>
          {/if}
        </div>
        {#if editing}
        <button
          disabled={!formIsValid}
          type="submit"
          on:click={submitEdit}
          class="btn btn-success block"
          >Confirm
        </button>
        <button type="button" class="btn btn-xs my-4 btn-disabled" on:click={cancelLink}
        >Cancel</button>
        {:else}
        <button
        disabled={!formIsValid}
        type="submit"
        class="btn btn-success block"
        >Add Link
      </button>
      <button type="button" class="btn btn-xs my-4" on:click={cancelLink}
      >Cancel</button>
      {/if}
      </form>
    {:else}
      <button
        on:click={() => (showForm = true)}
        class="btn btn-outline btn-info block mx-auto my-4"
      >
        Add a Link
      </button>
      <div class="form-control">
        <label class="label cursor-pointer">
          <span class="label-text">Published</span> 
          <input type="checkbox" class="toggle" bind:value={publish} on:change={togglePublish}/>
        </label>
      </div>
    {/if}
  {/if}

  {#if !showForm && loggedin}
    <div class="mx-auto group relative content-center">
      <textarea
        placeholder={$userData?.bio}
        class="textarea textarea-bordered text-center mx-auto w-full min-h-40 border whitespace-pre-line"
        bind:value={$bio}
        on:input={updateBio($bio)}
      />
      <button
        on:click={() => updateBio($bio)}
        class="btn btn-xs btn-success invisible group-hover:visible transition-all absolute -right-6 bottom-10"
        >Save</button
      >
    </div>
    <a href="/{$userData?.username}/">
      <button class="btn btn-success my-4 block mx-auto w-3/4">Update</button>
    </a>
  {/if}
  {#if !loggedin}
    <p class="text-error mx-auto my-auto">You Must Be Signed In</p>
    <a href="/login">
      <button class="btn btn-primary my-4 block mx-auto w-3/4">Login</button>
    </a>
  {/if}
</main>
