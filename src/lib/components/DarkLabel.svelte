<script lang="ts">
  export let htmlFor: string | undefined = undefined;
</script>

<label
  for={htmlFor}
  class="block text-sm font-medium text-[#FF6500] mb-1 {$$props.class}"
>
  <slot />
</label> 