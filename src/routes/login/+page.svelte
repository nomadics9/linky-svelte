<script lang="ts">
  import { auth} from "$lib/firebase";
  import {
    GoogleAuthProvider,
    signOut,
    signInWithPopup,
  } from "firebase/auth";
  import { user } from "$lib/firebase";


  async function signInWithGoogle() {
    const provider = new GoogleAuthProvider();
    const user = await signInWithPopup(auth, provider);
    }
    console.log(user);
</script>

<!---<h2 class=" card-title stat-title">Login</h2>-->

{#if $user}
  <h2 class="card-title">Welcome, {$user.displayName}</h2>
  <p class="text-center text-success">You are logged in</p>
 <!--- <button class="btn btn-warning" on:click={() => signOut(auth)}
    >Sign out</button
  > -->
  <a href="login/username">
    <button class="btn btn-primary btn-success">Next</button>
  </a>
{:else}
  <button class="btn btn-primary" on:click={signInWithGoogle}
    >Sign in with Google</button
  >
{/if}
