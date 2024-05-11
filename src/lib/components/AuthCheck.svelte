<script lang="ts">
    import { user } from "$lib/firebase";
</script>
  
{#if $user}
  <slot />
{:else}
    <main class="card card-body bg-neutral mx-auto">
    <p class="text-error mx-auto">
        You must be signed in to view this page.
    </p>
    <a class="btn btn-primary mt-2" href="/login">Sign in</a>
    </main>
{/if}