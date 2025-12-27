<template>
  <button :disabled="disabled" @click="toggle">
    <mdicon class="icon" :name="icon" :title="$t(`player.button.${icon}`)" />
  </button>
</template>

<script>
import player from '@/api/player'
import { usePlayerStore } from '@/stores/player'
import { useQueueStore } from '@/stores/queue'

export default {
  name: 'ControlPlayerPlay',
  setup() {
    return {
      playerStore: usePlayerStore(),
      queueStore: useQueueStore()
    }
  },
  computed: {
    disabled() {
      return this.queueStore.isEmpty
    },
    icon() {
      if (!this.playerStore.isPlaying) {
        return 'play'
      } else if (this.queueStore.isPauseAllowed) {
        return 'pause'
      }
      return 'stop'
    }
  },
  watch: {
    // Watch for changes in the playback state and sync with Media Session
    'playerStore.isPlaying': {
      handler() {
        this.syncPlaybackState()
      },
      immediate: true
    }
  },
  mounted() {
    navigator.mediaSession.setActionHandler("play", this.toggle)
    navigator.mediaSession.setActionHandler("pause", this.toggle)
  },

  methods: {
    toggle() {
      if (this.playerStore.isPlaying && this.queueStore.isPauseAllowed) {
        player.pause()
      } else if (
        this.playerStore.isPlaying &&
        !this.queueStore.isPauseAllowed
      ) {
        player.stop()
      } else {
        player.play()
      }
    },
    syncPlaybackState() {
      if ('mediaSession' in navigator) {
        if (this.playerStore.isPlaying) {
          navigator.mediaSession.playbackState = 'playing'
        } else {
          navigator.mediaSession.playbackState = 'paused'
        }
      }
    }
  }
}
</script>
