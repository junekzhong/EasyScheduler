<template>
  <div class="main-layout-box">
    <m-secondary-menu :type="'projects'"></m-secondary-menu>
    <m-list-construction :title="$t('工作流实例')">
      <template slot="conditions">
        <m-conditions @on-query="_onQuery"></m-conditions>
      </template>
      <template slot="content">
        <template v-if="processInstanceList.length">
          <m-list :process-instance-list="processInstanceList" @on-update="_onUpdate" :page-no="searchParams.pageNo" :page-size="searchParams.pageSize">
          </m-list>
          <div class="page-box">
            <x-page :current="parseInt(searchParams.pageNo)" :total="total" show-elevator @on-change="_page"></x-page>
          </div>
        </template>
        <template v-if="!processInstanceList.length">
          <m-no-data></m-no-data>
        </template>
        <m-spin :is-spin="isLoading"></m-spin>
      </template>
    </m-list-construction>
  </div>
</template>
<script>
  import _ from 'lodash'
  import { mapActions } from 'vuex'
  import mList from './_source/list'
  import mConditions from './_source/conditions'
  import mSpin from '@/module/components/spin/spin'
  import localStore from '@/module/util/localStorage'
  import { setUrlParams } from '@/module/util/routerUtil'
  import mNoData from '@/module/components/noData/noData'
  import mSecondaryMenu from '@/module/components/secondaryMenu/secondaryMenu'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'

  export default {
    name: 'instance-list-index',
    data () {
      return {
        // loading
        isLoading: true,
        // 总条数
        total: null,
        // 数据
        processInstanceList: [],
        // 参数
        searchParams: {
          // 搜索关键词
          searchVal: '',
          // 一页条数
          pageSize: 10,
          // 当前页
          pageNo: 1,
          // host
          host: '',
          // 状态
          stateType: '',
          // 开始时间
          startDate: '',
          // 结束时间
          endDate: ''
        }
      }
    },
    props: {},
    methods: {
      ...mapActions('dag', ['getProcessInstance']),
      /**
       * 查询
       */
      _onQuery (o) {
        this.searchParams = _.assign(this.searchParams, o)
      },
      /**
       * 分页事件
       */
      _page (val) {
        this.searchParams.pageNo = val
        setUrlParams({
          pageNo: this.searchParams.pageNo
        })
      },
      /**
       * 获取list数据
       */
      _getProcessInstanceListP (flag) {
        this.isLoading = !flag
        this.getProcessInstance(this.searchParams).then(res => {
          this.processInstanceList = []
          this.processInstanceList = res.totalList
          this.total = res.total
          this.isLoading = false
        }).catch(e => {
          this.isLoading = false
        })
      },
      /**
       * 更新
       */
      _onUpdate () {
        this._debounceGET('false')
      },
      /**
       * 路由变动
       */
      _routerView () {
        return this.$route.name === 'projects-instance-details'
      },
      /**
       * 防抖请求接口
       * @desc 防止函数多次被调用
       */
      _debounceGET: _.debounce(function (flag) {
        this._getProcessInstanceListP(flag)
      }, 100, {
        'leading': false,
        'trailing': true
      })
    },
    watch: {
      // 路由变动
      '$route' (a, b) {
        if (a.name === 'instance' && b.name === 'projects-instance-details') {
          this._debounceGET()
        } else {
          // url no params get instance list
          this.searchParams.pageNo = !_.isEmpty(a.query) && a.query.pageNo || 1
        }
      },
      'searchParams': {
        deep: true,
        handler () {
          this._debounceGET()
        }
      }
    },
    created () {
      // 删除流程定义id
      localStore.removeItem('subProcessId')

      // 路由参数合并
      if (!_.isEmpty(this.$route.query)) {
        this.searchParams = _.assign(this.searchParams, this.$route.query)
      }

      // 根据路由判断请求数据
      if (!this._routerView()) {
        this._debounceGET()
      }
    },
    mounted () {
      // 轮循获取状态
      this.setIntervalP = setInterval(() => {
        this._debounceGET('false')
      }, 90000)
    },
    beforeDestroy () {
      // 销毁轮循
      clearInterval(this.setIntervalP)
    },
    components: { mList, mConditions, mSpin, mListConstruction, mSecondaryMenu, mNoData }
  }
</script>

<style lang="scss" rel="stylesheet/scss">
</style>
