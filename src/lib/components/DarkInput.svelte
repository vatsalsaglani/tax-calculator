<script lang="ts">
  export let value: string;
  export let type: string = "text";
  export let id: string | undefined = undefined;
  export let placeholder: string | undefined = undefined;
</script>

<input
  {type}
  {id}
  {placeholder}
  bind:value
  class="block w-full rounded-md border border-[#1E3E62] bg-[#0B192C] px-3 py-2 text-sm text-gray-100 placeholder-gray-500 shadow-sm transition-all duration-200 focus:border-[#FF6500] focus:outline-none focus:ring-1 focus:ring-[#FF6500]/50 hover:border-[#FF6500]/50 {$$props.class}"
/> 