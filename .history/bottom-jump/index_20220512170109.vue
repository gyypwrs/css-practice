<template>
  <div
    id="dialogHook"
    class="app-content app-content--quote invoice-entry-page"
  >
    <quote v-if="routeParams.action === 'detail'" text="发票详情" />
    <div class="app-content-wrapper">
      <el-tabs
        v-model="componentName"
        v-tabs-permission="tabs"
        class="tabs-top"
      >
        <el-tab-pane
          v-for="(item, index) in tabsList"
          :key="index"
          :label="item.tabName"
          :name="item.compName"
          :lazy="true"
        />
      </el-tabs>
      <div class="component-container" @scroll="handleScroll">
        <keep-alive include="InvoiceEntryEntity">
          <component
            :is="componentName"
            :route-params="routeParams"
            :can-edit="false"
          />
        </keep-alive>
        <el-backtop target=".component-container" :bottom="100" />
        <div
          v-show="showAnimation"
          :class="{ 'bottom-jump': showAnimation }"
        />
      </div>
    </div>
    <!-- <div class="app-content-wrapper">
      <div class="component-container">
        <invoice-entry-entity :route-params="routeParams" :can-edit="false" />
        <el-backtop target=".component-container" :bottom="100" />
      </div>
    </div> -->
  </div>
</template>
<script>
import Quote from 'components/Quote/index'
import { judgePermission } from 'utils'
const TAB_LIST = [
  {
    tabName: '发票详情',
    compName: 'InvoiceEntryEntity',
    permissionId: 'a668a1fa43ba499b87814a046183ec7d'
  },
  {
    tabName: '审批详情',
    compName: 'ApprovalDetail',
    permissionId: '64db340e4118440c94fac7940a677d8a'
  }
]
export default {
  name: 'InvoiceDetail',
  components: {
    Quote,
    InvoiceEntryEntity: () => import('./InvoiceEntryEntity.vue'),
    ApprovalDetail: () => import('./ApprovalDetail.vue')
  },
  data() {
    return {
      tabs: Object.freeze(TAB_LIST),
      routeParams: this.$route.query,
      componentName: 'InvoiceEntryEntity',
      tabsList: [],
      showAnimation: false,
      key: 0
    }
  },
  provide() {
    return {
      saveId: () => {
        return this.saveId
      },
      handleSave: this.handleSave
    }
  },
  computed: {},
  watch: {
    $route: function(to, from) {
      if (to.name !== 'InvoiceDetail' || this.fullPath === to.fullPath) return
      this.$store.dispatch('tagsView/refreshCurrentTag', this)
    }
  },
  created() {
    this.initTabs()
  },
  methods: {
    initTabs() {
      this.tabsList = judgePermission(TAB_LIST)
    },
    saveId() {
      return ''
    },
    handleScroll() {
      this.showAnimation = false
      const scrollTop =
        document.documentElement.scrollTop || document.body.scrollTop
      // windowHeight是可视区的高度
      const windowHeight =
        document.documentElement.clientHeight || document.body.clientHeight
      // scrollHeight是滚动条的总高度
      const scrollHeight =
        document.documentElement.scrollHeight || document.body.scrollHeight
      // 滚动条到底部的条件
      if (scrollTop + windowHeight == scrollHeight) {
        // this.key++
        this.showAnimation = true
      }
    },
    handleSave() {}
  }
}
</script>

<style lang="scss">
.invoice-entry-page {
  overflow: hidden;
  .app-content-wrapper {
    display: flex;
    flex-direction: column;
    height: 0;
    padding: 0;
  }
  .tabs-top {
    height: 40px;
    padding: 0 10px;
    .el-tabs__header {
      margin-bottom: 0px;
    }
    .el-tabs__content {
      display: none;
    }
  }
  .component-container {
    overflow-y: auto;
    flex: 1;
    padding-top: 10px;
    .el-backtop {
      width: 40px;
      height: 40px;
      font-size: 25px;
      border-radius: 0px;
      &:hover {
        background-color: #e2e6ea;
        border-color: #dae0e5;
      }
    }
    .bottom-jump {
      // height: 200px;
      animation: jump 1.5s ease;
    }
    @keyframes jump {
      from {
        height: 200px;
      }
      to {
        height: 0px;
      }
    }
  }
}
</style>
