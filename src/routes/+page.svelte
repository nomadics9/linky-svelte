<script lang="ts">
    import { userData, user, auth} from "$lib/firebase";
    import { goto } from '$app/navigation';
    import { onMount } from "svelte";

    function getLogged() {
        if ($userData?.username != undefined) {
            goto(`/${$userData?.username}`)
            console.log(user)
        }
        else {
            goto(`/login`)
        }

    }
    
   onMount(() => {
            setTimeout(getLogged, 2000)
        })

</script>
<head>
  <title>Linky</title>
  <meta name="description" content="Your web tree!" />
  <meta property="og:image" content="https://nmd.mov/favicon.png" />
  <meta property="og:title" content="Linky" />
  <meta property="og:description" content="Your web tree!" />
  <meta property="og:url" content="https://nmd.mov" />
</head>

{#if $userData == undefined}
<div class="mx-auto my-auto">
<a href="/login" class="btn btn-outline">/Start</a>
</div>
{/if}

{#if $userData != undefined}
<div class="m-auto">
<span class="loading loading-spinner text-success"></span>
</div>
{/if}





