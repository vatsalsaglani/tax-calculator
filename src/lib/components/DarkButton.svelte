<script lang="ts">
  export let variant: "default" | "primary" | "secondary" = "default";
  
  const baseStyles = "inline-flex items-center justify-center rounded-md px-4 py-2 text-sm font-medium transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-[#0B192C] shadow-lg hover:scale-[1.02]";
  
  $: variantStyles = {
    primary: "bg-gradient-to-r from-[#FF6500] to-[#FF8500] text-white hover:from-[#FF7500] hover:to-[#FF9500] focus:ring-[#FF6500]",
    secondary: "bg-gradient-to-r from-[#1E3E62] to-[#2A4E78] text-gray-100 hover:from-[#234569] hover:to-[#2F5785] focus:ring-[#1E3E62]",
    default: "bg-gradient-to-r from-[#0B192C] to-[#152A45] text-gray-200 border border-[#1E3E62] hover:border-[#FF6500]/50 focus:ring-[#1E3E62]"
  }[variant];
</script>

<button class="{baseStyles} {variantStyles} {$$props.class}" on:click>
  <slot />
</button> 