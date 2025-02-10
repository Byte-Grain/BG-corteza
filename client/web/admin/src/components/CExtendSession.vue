<template>
  <b-modal
    id="session-extension-modal"
    centered
    hide-header
    hide-footer
    no-close-on-backdrop
    no-close-on-esc
    body-class="d-flex flex-column justify-content-center align-items-center gap-2 p-4"
    @hide="stopCountdown"
  >
    <h5>{{ labels.warning(countdownTime) }}</h5>

    <b-button
      variant="primary"
      size="lg"
      @click="extendSession"
    >
      {{ labels.extend }}
    </b-button>
  </b-modal>
</template>

<script>
const countdownTime = 60

export default {
  name: 'CExtendSession',

  props: {
    labels: {
      type: Object,
      default: () => ({
        extend: 'Extend Session',
        warning: (countdownTime) => `You will be logged out in ${countdownTime} seconds`,
      }),
    },
  },

  data () {
    return {
      countdownTime: countdownTime,
      countdownTimer: null,
    }
  },

  created () {
    this.setupActivityMonitoring()
    this.$root.$on('auth-logout-warning', this.show)
  },

  beforeDestroy () {
    this.stopCountdown()

    this.$root.$off('auth-logout-warning', this.show)
  },

  methods: {
    setupActivityMonitoring () {
      this.$auth.setupActivityMonitoring(this.$Settings.get('auth.autoLogoutTimeout', 0))
    },

    extendSession () {
      this.$bvModal.hide('session-extension-modal')
      this.setupActivityMonitoring()
    },

    logout () {
      this.$auth.logout()
    },

    startCountdown () {
      this.countdownTime = countdownTime

      this.countdownTimer = setInterval(() => {
        this.countdownTime--
        if (this.countdownTime <= 0) {
          clearInterval(this.countdownTimer)
          // this.logout()
        }
      }, 1000)
    },

    stopCountdown () {
      if (!this.countdownTimer) {
        return
      }

      clearInterval(this.countdownTimer)
    },

    show () {
      this.$bvModal.show('session-extension-modal')
      this.startCountdown()
    },
  },
}
</script>
