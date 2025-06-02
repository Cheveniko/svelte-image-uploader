<script lang="ts">
  import "../app.css";
  import { injectAnalytics } from "@vercel/analytics/sveltekit";
  import { Toaster } from "$lib/components/ui/sonner/index.js";

  let { children } = $props();
  injectAnalytics();
</script>

<Toaster richColors closeButton />
{@render children()}
