<script>
// @ts-nocheck
    import GUN from "gun";
    import SEA from "gun";
    import { onMount } from "svelte";
    import Login from "./Login.svelte";
    import ChatMessage from "./ChatMessage.svelte";
    import {user, username} from "./user.js";
    const db = GUN();

    let newMessage;
    let messages = [];

    onMount(() => {
        // Get Messages
        db.get("chat")
            .map()
            .once(async (data, id) => {
                if (data) {
                    const key = "#foo";

                    var message = {
                        // transform the data
                        who: await db.user(data).get("alias"),
                        what: (await SEA.SEA.decrypt(data.what, key)) + "",
                        when: GUN.state.is(data, "what"),
                    };
                    if (message.what) {
                        messages = [...messages.slice(-100), message];
                    }
                }
            });
    });

    async function sendMessage() {
        const secret = await SEA.SEA.encrypt(newMessage, '#foo');
        const message = user.get('all').set({what: secret});
        const index = new Date().toISOString();
        db.get('chat').get(index).put(message);
        newMessage = '';
    }
</script>

<style>
    .odd-message-container {
        background-color:rgb(78, 78, 78) 
    }

    .even-message-container {
        background-color: rgb(179, 179, 179);
    }
</style>

<div class="container">
    {#if $username}
        <main>
            {#each messages as message, idx (message.when)}
            <div class={idx % 2 == 0 ? 'odd-message-container' : 'even-message-container'}>
                <ChatMessage {message} sender={$username}/>
            </div>
            {/each}
        </main>

        <form on:submit|preventDefault={sendMessage}>
            <input type="text" placeholder="Type a message..." bind:value={newMessage} maxlength="100" />

            <button type="submit" disabled={!newMessage}>send</button>
        </form>
    {:else}
        <main>
            <Login />
        </main>
    {/if}
</div>
