<script lang="ts">
  import { onMount, onDestroy } from "svelte";

  export let count: number;
  let message: string = null;

  function handleMessageEvent(request, _, sendResponse) {
    if (request.type === "count_updated") {
      sendResponse({ message: `from ${count} to ${request.count}` });
      count = request.count;
    }
  }
  
  const increment = () => (count += 1);
  const decrement = () => (count -= 1);

  const handleSave = async () => {
    await chrome.storage.sync.set({ count });
    message = "Updated!";

    chrome.runtime.sendMessage({count, type: "count_updated"}, (res) => {
      const lastError = chrome.runtime.lastError;

      // This conditional check is important to remove the error
      if (lastError) {
        // Handle error here
        console.log("TODO:", lastError.message);
        return;
      }
      message += ` ${res.message}`;
    });
  
    setTimeout(() => {
      message = null;
    }, 2000);

    onMount(() => {
      chrome.runtime.onMessage.addListener(handleMessageEvent);

      // For demo purposes only
      setTimeout(() => {
        chrome.runtime.onMessage.removeListener(handleMessageEvent);
      }, 4000);

      return () => {
        chrome.runtime.onMessage.removeListener(handleMessageEvent);
      };
    });

    onDestroy(() => {
      chrome.runtime.onMessage.removeListener(handleMessageEvent);
    });
  };
</script>

<div class=" bg-blue-50 min-w-[20rem] p-4 flex flex-col gap-4">
  <p class="text-blue-800 text-xl">
      Current count: <span class="font-extrabold">{count}</span>
  </p>
  <div class="flex gap-2">
    <button on:click={decrement}>-</button>
    <button on:click={increment}>+</button>
    <button class="ml-auto" on:click={handleSave}>Save</button>
    {#if message}<span class="font-bold text-blue-800">{message}</span>{/if}
  </div>
</div>

<style scoped>
  button {
    color: theme('colors.blue.700');
    padding: theme('spacing.2') theme('spacing.4');
    font-size: theme('fontSize.base');
    border: 1px solid theme('borderColor.blue.400');
    box-shadow: theme('boxShadow.lg');
    background-color: theme('backgroundColor.blue.50');
  }
  
  button:hover,
  button:focus {
    background-color: theme('colors.blue.800');
    color: theme('colors.blue.50');
  }
</style>