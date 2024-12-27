<script>
	export const prerender = true;
	import '../lib/styles/reset.css';
	import '../lib/styles/custom.css';
	import '../lib/styles/theme.css';
	let { children } = $props();
	// import 'lib/styles/reset.css';
	// import 'lib/styles/custom.css';
	// import 'lib/styles/theme.css';
</script>

<!-- <slot /> -->
{@render children()}
