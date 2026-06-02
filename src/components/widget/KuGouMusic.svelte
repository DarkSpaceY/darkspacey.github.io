<script>
  import { onMount } from "svelte";

  const userId = "1716967559";

  let playlists = [];
  let loading = true;
  let error = false;
  let userName = "DarkSpace";

  onMount(async () => {
    try {
      const res = await fetch(
        `https://api-hot.ikutuga.com/kugou/user/playlist?userid=${userId}&page=1&pagesize=6`,
        { signal: AbortSignal.timeout(5000) },
      );
      if (!res.ok) throw new Error("API error");
      const data = await res.json();

      if (data?.data?.info) {
        playlists = data.data.info.map((p) => ({
          name: p.specialname || "未命名歌单",
          songCount: p.songcount || 0,
          playCount: p.playcount || 0,
          img: p.imgurl?.replace("{size}", "400") || "",
          id: p.specialid,
        }));
      }
    } catch {
      try {
        const res2 = await fetch(
          `https://api.i-meto.com/meting/api?server=kugou&type=playlist&id=${userId}`,
          { signal: AbortSignal.timeout(5000) },
        );
        if (res2.ok) {
          const data2 = await res2.json();
          if (Array.isArray(data2)) {
            playlists = data2.slice(0, 6).map((s) => ({
              name: s.title || "未知歌曲",
              author: s.author || "",
              cover: s.pic || "",
              id: s.id,
            }));
          }
        } else {
          throw new Error("Meting API failed");
        }
      } catch {
        error = true;
      }
    }
    loading = false;
  });
</script>

<div class="card-base p-3 onload-animation" style="animation-delay: 50ms">
  <div class="text-sm font-bold text-75 mb-2 flex items-center gap-1.5">
    <svg class="w-4 h-4" viewBox="0 0 24 24" fill="currentColor">
      <path d="M12 3v10.55c-.59-.34-1.27-.55-2-.55-2.21 0-4 1.79-4 4s1.79 4 4 4 4-1.79 4-4V7h4V3h-6z"/>
    </svg>
    最近在听
  </div>

  {#if loading}
    <div class="flex items-center justify-center py-6">
      <div class="w-5 h-5 border-2 border-[var(--primary)] border-t-transparent rounded-full animate-spin"></div>
    </div>
  {:else if error}
    <div class="text-center py-4 text-xs text-50">
      <p>正在加载音乐数据...</p>
      <p class="mt-1">酷狗音乐 · DarkSpace</p>
    </div>
  {:else if playlists.length > 0 && playlists[0].cover !== undefined}
    <div class="flex flex-col gap-1.5">
      {#each playlists.slice(0, 5) as song}
        <div class="flex items-center gap-2 bg-[var(--btn-regular-bg)] rounded-lg p-1.5">
          {#if song.cover}
            <img src={song.cover} alt={song.name} class="w-8 h-8 rounded object-cover flex-shrink-0" loading="lazy" />
          {/if}
          <div class="min-w-0 flex-1">
            <div class="text-xs truncate text-90">{song.name}</div>
            {#if song.author}
              <div class="text-[10px] text-50 truncate">{song.author}</div>
            {/if}
          </div>
        </div>
      {/each}
    </div>
  {:else if playlists.length > 0}
    <div class="grid grid-cols-2 gap-2">
      {#each playlists.slice(0, 4) as pl}
        <div class="bg-[var(--btn-regular-bg)] rounded-lg overflow-hidden">
          {#if pl.img}
            <img src={pl.img} alt={pl.name} class="w-full aspect-square object-cover" loading="lazy" />
          {:else}
            <div class="w-full aspect-square bg-[var(--btn-regular-bg-hover)] flex items-center justify-center text-30">
              <svg class="w-8 h-8" viewBox="0 0 24 24" fill="currentColor"><path d="M12 3v10.55c-.59-.34-1.27-.55-2-.55-2.21 0-4 1.79-4 4s1.79 4 4 4 4-1.79 4-4V7h4V3h-6z"/></svg>
            </div>
          {/if}
          <div class="p-1.5">
            <div class="text-[10px] truncate text-90">{pl.name}</div>
            <div class="text-[9px] text-50">{pl.songCount} 首</div>
          </div>
        </div>
      {/each}
    </div>
  {:else}
    <div class="text-center py-4 text-xs text-50">
      <p>🎵 酷狗音乐 · DarkSpace</p>
    </div>
  {/if}

  <div class="mt-2 text-[10px] text-center text-50">
    数据来自酷狗音乐
  </div>
</div>
