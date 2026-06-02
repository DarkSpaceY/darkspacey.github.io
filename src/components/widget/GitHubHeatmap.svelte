<script lang="ts">
  import { onMount } from "svelte";

  const username = "DarkSpaceY";

  interface ContributionDay {
    date: string;
    count: number;
    level: number;
  }

  let contributions = $state<ContributionDay[]>([]);
  let totalContributions = $state(0);
  let loading = $state(true);
  let error = $state(false);

  onMount(() => {
    fetchContributions();
  });

  async function fetchContributions() {
    try {
      const res = await fetch(
        `https://github-contributions-api.deno.dev/${username}.json`,
        { signal: AbortSignal.timeout(8000) },
      );
      if (!res.ok) throw new Error("API failed");
      const data = await res.json();
      contributions = data.contributions || [];
      totalContributions = data.total?.lastYear || contributions.reduce((s, d) => s + d.count, 0);
    } catch {
      error = true;
    }
    loading = false;
  }

  const levelColors = [
    "bg-[var(--btn-regular-bg)]",
    "bg-[color-mix(in_srgb,var(--primary)_25%,var(--card-bg))]",
    "bg-[color-mix(in_srgb,var(--primary)_50%,var(--card-bg))]",
    "bg-[color-mix(in_srgb,var(--primary)_75%,var(--card-bg))]",
    "bg-[var(--primary)]",
  ];

  function cellLabel(count: number, date: string): string {
    return count === 0 ? `No contributions on ${date}` : `${count} contribution${count > 1 ? "s" : ""} on ${date}`;
  }

  function getTooltipStyle(count: number): string {
    return count === 0 ? "text-50" : "text-[var(--primary)]";
  }

  function getDate(dateStr: string): string {
    const d = new Date(dateStr);
    return d.toLocaleDateString("en-US", { month: "short", day: "numeric", year: "numeric" });
  }
</script>

<div class="card-base p-4 onload-animation" style="animation-delay: 200ms">
  <div class="flex items-center justify-between mb-3">
    <div class="text-sm font-bold text-75 flex items-center gap-1.5">
      <svg class="w-4 h-4" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/></svg>
      GitHub Contributions
    </div>
    {#if !loading && !error}
      <span class="text-xs text-50">{totalContributions.toLocaleString()} contributions in the last year</span>
    {/if}
  </div>

  {#if loading}
    <div class="flex items-center justify-center py-8">
      <div class="w-5 h-5 border-2 border-[var(--primary)] border-t-transparent rounded-full animate-spin"></div>
    </div>
  {:else if error}
    <div class="text-xs text-50 py-4 text-center">
      <a href={`https://github.com/${username}`} target="_blank" rel="noopener noreferrer" class="hover:text-[var(--primary)] transition-colors">
        View contributions on GitHub →
      </a>
    </div>
  {:else}
    <div class="overflow-x-auto" style="-webkit-overflow-scrolling: touch;">
      <div class="flex gap-0.5" style="min-width: fit-content;">
        {#each contributions as day}
          <div
            class="w-[11px] h-[11px] rounded-sm {levelColors[day.level] || levelColors[0]} flex-shrink-0"
            role="img"
            aria-label={cellLabel(day.count, day.date)}
            title={cellLabel(day.count, day.date)}
          ></div>
        {/each}
      </div>
    </div>
    <div class="flex items-center justify-between mt-3 text-[10px]">
      <div class="flex items-center gap-1.5 text-50">
        <span>Less</span>
        {#each [0, 1, 2, 3, 4] as l}
          <div class="w-2.5 h-2.5 rounded-sm {levelColors[l]}"></div>
        {/each}
        <span>More</span>
      </div>
      <a href={`https://github.com/${username}`} target="_blank" rel="noopener noreferrer" class="text-50 hover:text-[var(--primary)] transition-colors">
        @{username}
      </a>
    </div>
  {/if}
</div>
