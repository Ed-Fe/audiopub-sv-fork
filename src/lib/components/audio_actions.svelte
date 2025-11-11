<!-- Shared action buttons for audio items: share, download, favorite, follow/unfollow -->
<script lang="ts">
  import { enhance } from "$app/forms";
  import type { ClientsideUser } from "$lib/types";
  import { t } from "$lib/i18n";
  export let audioId: string;
  export let audioTitle: string;
  export let audioPath: string;
  export let isFavorited: boolean | undefined = undefined;
  export let favoriteCount: number | undefined = 0;
  export let currentUser: ClientsideUser | null = null;
  // 'listen' renders server forms; 'quickfeed' uses callbacks
  export let context: 'listen' | 'quickfeed' = 'listen';
  // Quickfeed callback for toggling favorite
  export let onToggleFavorite: (() => void) | undefined = undefined;
  // Optional follow state (only used on listen page)
  export let showFollow: boolean = false;
  export let isFollowing: boolean = false;

  // Debug log
  $: if (context === 'quickfeed') {
    console.log('AudioActions - currentUser:', currentUser);
    console.log('AudioActions - isFavorited:', isFavorited);
    console.log('AudioActions - favoriteCount:', favoriteCount);
  }

  // Share handler (client only)
  function share() {
    if (typeof navigator !== 'undefined' && (navigator as any).share) {
      (navigator as any).share({ url: `/listen/${audioId}`, title: audioTitle })
        .catch((err: any) => {
          console.log('Share cancelled or failed:', err);
        });
    } else if (typeof navigator !== 'undefined' && navigator.clipboard) {
      const url = (window?.location?.origin ?? '') + `/listen/${audioId}`;
      navigator.clipboard.writeText(url)
        .then(() => {
          alert(t('audio_actions.share_copied'));
        })
        .catch((err) => {
          console.error('Failed to copy:', err);
          alert(t('upload.error.generic'));
        });
    }
  }
</script>

<div class="audio-actions" role="group" aria-label={t('audio_actions.aria')}>
  <a class="action-btn download" href={`/${audioPath}`} download
     aria-label={t('listen.download')}>
    ⬇ {t('listen.download')}
  </a>

  <button type="button" class="action-btn share" on:click={share} aria-label={t('audio_actions.share')}>
    🔗 {t('audio_actions.share')}
  </button>

  {#if currentUser}
    {#if context === 'quickfeed'}
      <button type="button"
              class="action-btn favorite"
              class:favorited={!!isFavorited}
              on:click={onToggleFavorite}
              aria-pressed={!!isFavorited}
              aria-label={isFavorited ? t('listen.unfavorite') : t('listen.favorite')}>
        ❤️ {isFavorited ? t('listen.unfavorite') : t('listen.favorite')} ({favoriteCount || 0})
      </button>
    {:else}
      {#if isFavorited}
        <form use:enhance action="?/unfavorite" method="POST" style="display:inline">
          <button type="submit" class="action-btn favorite favorited" aria-pressed="true">
            ❤️ {t('listen.unfavorite')} ({favoriteCount || 0})
          </button>
        </form>
      {:else}
        <form use:enhance action="?/favorite" method="POST" style="display:inline">
          <button type="submit" class="action-btn favorite" aria-pressed="false">
            🤍 {t('listen.favorite')} ({favoriteCount || 0})
          </button>
        </form>
      {/if}
    {/if}
  {/if}

  {#if context === 'listen' && showFollow && currentUser}
    {#if isFollowing}
      <form use:enhance action="?/unfollow" method="POST" style="display:inline">
        <button type="submit" class="action-btn follow">
          🔕 {t('listen.unfollow')}
        </button>
      </form>
    {:else}
      <form use:enhance action="?/follow" method="POST" style="display:inline">
        <button type="submit" class="action-btn follow">
          🔔 {t('listen.follow')}
        </button>
      </form>
    {/if}
  {/if}
</div>

<style>
  .audio-actions {
    display: flex;
    gap: .5rem;
    align-items: center;
    flex-wrap: wrap;
  }
  .action-btn {
    display: inline-flex;
    align-items: center;
    gap: .35rem;
    padding: .5rem .75rem;
    border-radius: .5rem;
    border: 1px solid rgba(255,255,255,.2);
    background: rgba(255,255,255,.06);
    color: inherit;
    text-decoration: none;
    cursor: pointer;
  }
  .action-btn:hover { background: rgba(255,255,255,.12); }
  .favorite.favorited { background: rgba(255, 107, 107, .25); border-color: rgba(255, 107, 107, .6); }
</style>
