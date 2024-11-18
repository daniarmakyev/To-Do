<script>
    export let placeholder = '';
    export let value = '';
  </script>
  
 <input class="p-2 border-[1px] border-gray-950 rounded-md outline-none" bind:value={value} type="text" placeholder={placeholder}>
  