<template>
  <div class="app">
    <Header class="page-header"/>
    <div class="app-container" :class="showHeader ? 'show-header' : ''">
      <router-view class="wrapper"/>
      <notifications/>
      <b-modal
        v-model="statisticVisible"
        size="xxl"
        cancel-title="Close"
        footer-class="statistic-footer"
      >
        <template v-slot:modal-title>{{ modalShow === 'statistic' ? 'Statistic' : 'Confusion Matrix' }}</template>
        <statistic-popup v-if="modalShow === 'statistic'" ref="statistic"/>
        <confusion-matrix v-else ref="matrix"></confusion-matrix>
      </b-modal>
    </div>
    <page-footer/>
  </div>
</template>

<script>
import {
  mapGetters,
} from 'vuex'
import Header from '../components/Header.vue'
import EventBus from '../utils/eventbus'
import PageFooter from '../components/PageFooter.vue'
import StatisticPopup from '../components/StatisticPopup.vue'
import ConfusionMatrix from '../components/ConfusionMatrix.vue'

const ERROR_DISPLAY_DURATION = 10000

export default {
  name: 'LoggedInLayout',
  components: {
    ConfusionMatrix,
    Header,
    PageFooter,
    StatisticPopup,
  },
  data() {
    return {
      statisticVisible: false,
      modalShow: 'statistic',
    }
  },
  computed: {
    ...mapGetters(['showHeader']),
  },
  mounted() {
    EventBus.$on('auth_api_error', this.handleApiError)
    EventBus.$on('notify-error', this.handleErrorMessage)
    EventBus.$on('login', this.handleLogin)
    EventBus.$on('show-statistic', this.showStatistic)
    EventBus.$on('show-confusion-matrix', this.showConfusionMatrix)
    EventBus.$on('statistic-folder-selected', this.selectFolder)
  },
  destroyed() {
    EventBus.$off('auth_api_error')
    EventBus.$off('notify-error')
    EventBus.$off('login')
    EventBus.$off('show-statistic')
    EventBus.$off('show-confusion-matrix')
    EventBus.$off('statistic-folder-selected')
  },
  methods: {
    showConfusionMatrix(dir) {
      this.statisticVisible = true
      this.modalShow = 'matrix'
      this.$nextTick(() => {
        if (this.$refs.matrix) {
          this.$refs.matrix.load(dir)
        }
      })
    },
    showStatistic(dir) {
      this.statisticVisible = true
      this.modalShow = 'statistic'
      this.$nextTick(() => {
        if (this.$refs.statistic) {
          this.$refs.statistic.open(dir)
        }
      })
    },
    handleApiError(err) {
      let message = 'Error'
      if (err && err.response && err.response.data && err.response.data.error) {
        message = err.response.data.error.message
      } else if (err) {
        message = err.toString()
      }
      this.$notify({
        type: 'error',
        text: message,
        duration: ERROR_DISPLAY_DURATION,
      })
    },
    handleErrorMessage(message) {
      this.$notify({
        type: 'error',
        text: message,
        duration: ERROR_DISPLAY_DURATION,
      })
    },
    handleLogin() {
      this.$notify({
        type: 'error',
        text: 'You need to login first',
        duration: ERROR_DISPLAY_DURATION,
      })
      this.$router.push({
        name: 'LoginPage',
      })
    },
    selectFolder(item) {
      this.statisticVisible = false
      const gotoDir = item.folder
      this.$router.push({
        name: 'explorer',
        query: {
          dir: gotoDir,
        },
      })
    },
  },
  async created() {
    const wsExpandedStr = localStorage.getItem('ws_expanded')
    if (wsExpandedStr) {
      await this.$store.commit('app/SET_EXPANDED', wsExpandedStr.split(','))
    }
  },
}
</script>
<style lang="scss">
.app {
  background: #fff;
  height: 100vh;
  display: flex;
  flex-direction: column;

  .app-container {
    flex: 1;
    /*background: #fff;*/
    margin: 0 30px;
    /*margin-bottom: 50px;*/
    position: relative;

    .wrapper {
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: -.5rem;
      overflow: auto;
      padding-right: .5rem;
    }

    &.show-header {
    }
  }

  .title-container {
    display: flex;
    justify-content: space-between;
    padding-bottom: 20px;
  }

  .cmd-main-menu {
    .cmd {
      padding-top: 10px;
    }

    .btn-stop-command {
      background: #D9D4CF !important;
      border: 1px solid #D9D4CF !important;
      color: #000 !important;
    }

    .btn-command {
      background: #0060FF !important;
      border: 1px solid #0060FF !important;
      color: #fff;
    }

    .svtrain-cmd-btn {
      width: 200px;
      text-align: left;
    }
  }

}

.statistic-footer {
  .btn-secondary {
    color: #000;
    background: #D9D4CF;
    border: 1px solid #D9D4CF;

    &:hover, &:focus, &:active {
      background: #D9D4CF !important;
      color: #000 !important;
    }
  }
}

@media (min-width: 576px) {
  .modal-dialog.modal-xxl {
    max-width: calc(100vw - 3.5rem);
    margin: 1.75rem auto;
  }
}

@media (min-width: 1200px) {
  .modal-dialog.modal-xxl {
    .modal-content {
      margin: 0 auto;
      min-width: 1100px;
      width: fit-content;
    }
  }
}
.opacity-0 {
  opacity: 0;
}
</style>
