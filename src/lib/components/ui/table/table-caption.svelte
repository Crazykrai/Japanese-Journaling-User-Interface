<script>
	import { cn } from "$lib/utils.js";
	let className = undefined;
	export { className as class };
</script>

<caption class={cn("font-medium mt-4 text-sm", className)} {...$$restProps}>
	<slot />
</caption>
