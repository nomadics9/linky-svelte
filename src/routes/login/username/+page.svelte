<script lang="ts">
    import AuthCheck from "../../../lib/components/AuthCheck.svelte";
    import { db, user, userData } from "$lib/firebase";
    import { doc, getDoc, writeBatch} from "firebase/firestore";
  
    let username = "";
    let loading = false;
    let isAvailable = false;
    let isLowercase = false;
    let debounceTimer: NodeJS.Timeout;

    const re = /^(?=[a-zA-Z0-9._]{3,16}$)(?!.*[_.]{2})[^_.].*[^_.]$/;
  
  $: isValid = username.length > 3 && username.length < 16 && re.test(username);
  $: isTouched = username.length > 0;
  $: isTaken = isValid && !isAvailable && !loading
  
    async function checkAvailability() {
      isAvailable = false;
      clearTimeout(debounceTimer);
      if (username = username.toLowerCase())
        isLowercase
  
      loading = true;
  
      debounceTimer = setTimeout(async () => {
          console.log("checking availability of", username);
          
          const ref = doc(db, "usernames", username);
          const exists = await getDoc(ref).then((doc) => doc.exists());
  
          isAvailable = !exists;
          loading = false;
  
      }, 500);
  
    }
  
    async function confirmUsername() {
    console.log("confirming username", username);
    const batch = writeBatch(db);
    batch.set(doc(db, "usernames", username.toLowerCase()), { uid: $user?.uid });
    batch.set(doc(db, "users", $user!.uid), { 
      username, 
      photoURL: $user?.photoURL ?? null,
      published: true,
      bio: 'Bio',
    });

    await batch.commit();
    username = '';
    isAvailable = false;

  }
  
  </script>
  
  
  
  <AuthCheck>
    {#if $userData?.username}
        <p class="text-lg">
            Your Username is <span class="text-xl text-success font-bold">@{$userData.username}</span>
        </p>
        <p class=" text-sm">Username cannot be changed</p>
        <a class="btn btn-primary" href="/login/photo">UPLOAD PROFILE IMAGE</a>
    {:else}
      <h2>Username</h2>
      <form class="w-2/5 lowercase" on:submit|preventDefault={confirmUsername}>
        <input
          type="text"
          placeholder="Username"
          draggable="true"
          class="input w-full"
          bind:value={username}
          on:input={checkAvailability}
          class:input-error={(!isValid && isTouched)}
          class:input-warning={isTaken}
          class:input-success={isAvailable && isValid && !loading}
        />
        <div class="my-4 min-h-16 px-8 w-full">
          {#if loading}
            <p class="text-secondary">Checking availability of @{username}...</p>
          {/if}
      
          {#if !isValid && isTouched}
            <p class="text-error text-sm">
              must be 3-16 characters long, alphanumeric only
            </p>
          {/if}
      
          {#if isValid && !isAvailable && !loading}
            <p class="text-warning text-sm">
              @{username} is not available
            </p>
          {/if}
          {#if isLowercase}
          <p class="text-error text-sm">must be lowercase</p>
          {/if}
      
          {#if isAvailable && isValid && !isLowercase}
            <button class="btn btn-success lowercase">Confirm username @{username} </button>
          {/if}
        </div>
      </form>
      {/if}
  </AuthCheck>