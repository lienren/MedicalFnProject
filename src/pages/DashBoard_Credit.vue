<template>
  <div style="padding:20px;">
    <div style="margin-bottom: 20px;">
      <h2>您好，{{managerName}}</h2>
      <div>{{todayCalendar}} 农历{{lunarCalendar}}</div>
      <div v-if="isShowQuickButton">
        <br/> 快捷功能：
        <a-button icon="warning" size="small" type="primary" @click="goToOrderList(1)">今日到期</a-button>
        <a-button icon="warning" size="small" type="primary" @click="goToOrderList(2)">明日到期</a-button>
        <a-button icon="warning" size="small" type="primary" @click="goToOrderList(3)">即将到期</a-button>
        <a-button icon="warning" size="small" type="primary" @click="goToOrderList(4)">已经逾期</a-button>
      </div>
    </div>
    <div v-if="isShowDataBox" style="background-color: #ececec; padding: 20px; margin-bottom: 20px;">
      <a-row :gutter="16">
        <a-col :span="6">
          <a-card hoverable title="今日需还款总金额" :bordered="false" class="sts-card sts-card-1">
            <iNumberGrow :value="lastloanPrice" style="letter-spacing:2px;"></iNumberGrow>
          </a-card>
        </a-col>
        <a-col :span="6">
          <a-card hoverable title="今日已还款总金额" :bordered="false" class="sts-card sts-card-2">
            <iNumberGrow :value="realReturnPrice" style="letter-spacing:2px;"></iNumberGrow>
          </a-card>
        </a-col>
        <a-col :span="6">
          <a-card hoverable title="累计逾期金额" :bordered="false" class="sts-card sts-card-3">
            <iNumberGrow :value="lastloanPrice1" style="letter-spacing:2px;"></iNumberGrow>
          </a-card>
        </a-col>
        <a-col :span="6">
          <a-card hoverable title="累计利息收入" :bordered="false" class="sts-card sts-card-4">
            <iNumberGrow :value="lastloanInterest" style="letter-spacing:2px;"></iNumberGrow>
          </a-card>
        </a-col>
      </a-row>
    </div>
    <div style="margin-bottom: 20px;">
      <a-card style="height:300px;">
        <div slot="title">
          <span style="color:#f56c6c;font-weight:bold;">30天</span> 贷款总金额/还款总金额</div>
        <ve-line v-if="chartVisible" :data="chartData" height="252px"></ve-line>
      </a-card>
    </div>
    <div style="margin-bottom: 20px;">
      <a-card>
        <div slot="title">总业绩</div>
        <iTable :btns="buttons" :actionBtns="actionButtons" :columns="columnsall" :data="dataall"></iTable>
      </a-card>
    </div>
    <div style="margin-bottom: 20px;">
      <a-card>
        <div slot="title">排行榜</div>
        <iTable :btns="buttons" :actionBtns="actionButtonsSort" :columns="columns" :data="data"></iTable>
      </a-card>
    </div>
  </div>
</template>

<script>

import api from '../api/credit'
import { iNumberGrow, iTable } from '../components/'

export default {
  components: {
    iNumberGrow,
    iTable
  },
  data () {
    return {
      chartData: {},
      chartVisible: false,
      isShowDataBox: false,
      lastloanPrice: 0,
      lastloanPrice1: 0,
      realReturnPrice: 0,
      lastloanInterest: 0,
      isShowQuickButton: false,
      columns: [],
      columnsall: [],
      data: [],
      dataall: [],
      buttons: [],
      actionButtons: [],
      actionButtonsSort: [],
      pagination: {}
    }
  },
  computed: {
    managerName () {
      let userinfo = this.$utils.Store.get('userinfo')
      return userinfo ? userinfo.realName : ''
    },
    todayCalendar () {
      return this.$utils.Date.format(new Date(), 'yyyy年M月d日') + ' ' + this.$utils.Date.getWeek()
    },
    lunarCalendar () {
      return this.$utils.Date.getLunarCalendar()
    }
  },
  created () { },
  beforeDestroy () { },
  mounted () {
    this.$nextTick(() => {
      this.init()
    })
  },
  methods: {
    async init () {
      this.getStsOrders()
      this.getStsOrderSort()
    },
    async getStsOrders () {
      this.chartData = {
        columns: ['日期', '贷款总金额', '还款总金额'],
        rows: []
      }

      let userinfo = this.$utils.Store.get('userinfo')
      let superRole = userinfo.roles.filter(role => {
        return role.roleId === 1 || role.roleId === 2
      })
      let verfiyRole = userinfo.roles.filter(role => {
        return role.roleId === 3
      })
      if ((superRole && superRole.length > 0) || (verfiyRole && verfiyRole.length > 0)) {
        this.isShowQuickButton = true
      }

      let result
      if (superRole && superRole.length > 0) {
        result = await api.getStsOrdersAll({})
      } else {
        result = await api.getStsOrders({})
      }

      // let result = await api.getStsOrders({})

      if (result) {
        this.lastloanPrice = result.result.lastloanPrice ? parseInt(result.result.lastloanPrice) : 0
        this.realReturnPrice = result.result.realReturnPrice ? parseInt(result.result.realReturnPrice) : 0
        this.lastloanPrice1 = result.result.lastloanPrice1 ? parseInt(result.result.lastloanPrice1) : 0
        this.lastloanInterest = result.result.lastloanInterest ? parseInt(result.result.lastloanInterest) : 0
        this.chartData.rows = result.result.month
        this.isShowDataBox = true
        this.chartVisible = true
        this.mapVisible = true
      }
    },
    async getStsOrderSort () {
      // 列头
      this.columnsall = [{
        title: '贷款金额',
        dataIndex: 'lastloanPrice'
      }, {
        title: '贷款利息',
        dataIndex: 'lastloanInterest'
      }, {
        title: '贷款本金',
        dataIndex: 'lastloanServicePrice'
      }, {
        title: '逾期本金',
        dataIndex: 'yqbj'
      }, {
        title: '贷款数量',
        dataIndex: 'loanCount'
      }, {
        title: '逾期比例',
        dataIndex: 'yql'
      }]
      // 列头
      this.columns = [{
        title: '排名',
        dataIndex: 'sort'
      }, {
        title: '姓名',
        dataIndex: 'realName'
      }, {
        title: '性别',
        dataIndex: 'sexName'
      }, {
        title: '所属部门',
        dataIndex: 'depName'
      }, {
        title: '贷款金额',
        dataIndex: 'lastloanPrice'
      }, {
        title: '贷款利息',
        dataIndex: 'lastloanInterest'
      }, {
        title: '贷款本金',
        dataIndex: 'lastloanServicePrice'
      }, {
        title: '逾期本金',
        dataIndex: 'yqbj'
      }, {
        title: '贷款数量',
        dataIndex: 'loanCount'
      }, {
        title: '逾期比例',
        dataIndex: 'yql'
      }]
      let userinfo = this.$utils.Store.get('userinfo')
      let superRole = userinfo.roles.filter(role => {
        return role.roleId === 1 || role.roleId === 2
      })
      if (superRole && superRole.length > 0) {
        this.columns.push(
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            width: 150
          }
        )
      }
      this.actionButtonsSort = [{
        model: 'button',
        text: '订单详细',
        style: {},
        icon: '',
        click: async (e) => {
          this.$router.push({ path: '/allorderlist', query: {managerId: e.managerId} })
        }
      }]
      let result = await api.getStsOrderSort({})
      if (result && result.result) {
        this.data = []
        this.dataall = [{
          lastloanPrice: 0,
          lastloanInterest: 0,
          lastloanServicePrice: 0,
          yqbj: 0,
          loanCount: 0,
          yqlCount: 0,
          yql: '0%'
        }]
        result = result.result
        result.a.forEach((item, index) => {
          this.data.push({
            ...item,
            sort: (index + 1),
            lastloanPrice: item.lastloanPrice / 100,
            lastloanInterest: item.lastloanInterest / 100,
            lastloanServicePrice: item.lastloanServicePrice / 100,
            sexName: item.sex === 1 ? '男' : '女'
          })
          this.dataall[0].lastloanPrice += item.lastloanPrice / 100
          this.dataall[0].lastloanInterest += item.lastloanInterest / 100
          this.dataall[0].lastloanServicePrice += item.lastloanServicePrice / 100
          this.dataall[0].loanCount += item.loanCount
        })

        for (let i = 0, j = this.data.length; i < j; i++) {
          let findYQ = result.b.filter(yq => {
            return yq.managerId === this.data[i].managerId
          })
          if (findYQ && findYQ.length > 0) {
            this.data[i].yqbj = findYQ[0].lastloanServicePrice / 100
            this.data[i].yql = parseFloat((findYQ[0].loanCount / this.data[i].loanCount) * 100).toFixed(2) + '%'
          } else {
            this.data[i].yqbj = 0
            this.data[i].yql = '0%'
          }
        }

        result.b.forEach((item, index) => {
          this.dataall[0].yqbj += item.lastloanServicePrice / 100
          this.dataall[0].yqlCount += item.loanCount
          this.dataall[0].yql = parseFloat((this.dataall[0].yqlCount / this.dataall[0].loanCount) * 100).toFixed(2) + '%'
        })
      }
    },
    goToOrderList (type) {
      let userinfo = this.$utils.Store.get('userinfo')
      let superRole = userinfo.roles.filter(role => {
        return role.roleId === 1
      })

      let verfiyRole = userinfo.roles.filter(role => {
        return role.roleId === 3
      })

      let today = this.$utils.Date.getTodayTimeStamp()
      if (superRole && superRole.length > 0) {
        let query = {}
        switch (type) {
          case 1:
            query = {
              state: ['1', '2', '4', '5'],
              returnSTime: today.starttime,
              returnETime: today.endtime
            }
            break
          case 2:
            query = {
              state: ['1', '2', '4', '5'],
              returnSTime: today.starttime + 86400000,
              returnETime: today.endtime + 86400000
            }
            break
          case 3:
            query = {
              state: ['1', '2', '4', '5'],
              returnSTime: today.starttime + 172800000,
              returnETime: today.endtime + 172800000
            }
            break
          case 4:
            query = {
              state: ['4']
            }
            break
        }
        this.$router.push({ path: '/allorderlist', query: query })
      } else if (verfiyRole && verfiyRole.length > 0) {
        let query = {}
        switch (type) {
          case 1:
            query = {
              state: ['1', '2', '4', '5'],
              returnSTime: today.starttime,
              returnETime: today.endtime
            }
            break
          case 2:
            query = {
              state: ['1', '2', '4', '5'],
              returnSTime: today.starttime + 86400000,
              returnETime: today.endtime + 86400000
            }
            break
          case 3:
            query = {
              state: ['1', '2', '4', '5'],
              returnSTime: today.starttime + 172800000,
              returnETime: today.endtime + 172800000
            }
            break
          case 4:
            query = {
              state: ['4']
            }
            break
        }
        this.$router.push({ path: '/orderlist', query: query })
      }
    }
  }
}
</script>
<style lang="less" rel="stylesheet/less">
  .dashboard_body {
    width: 100%;
    display: flex;
    .left {
      flex: 1;
      padding: 0 10px 0 0;
    }
    .right {
      width: 200px;
      min-width: 200px;
      max-width: 200px;
    }
  }
  .sts-card {
    cursor: inherit;
    .ant-card-head {
      border: 0;
      padding: 0 0 0 12px;
    }
    .ant-card-head-title {
      color: #fff;
    }
  }
  .cs-items {
    cursor: inherit;
    .ant-card-body {
      padding: 12px;
    }
    .cs-item {
      border-bottom: 1px solid #eee;
      .ant-card-head {
        min-height: 32px;
        border: 0;
        padding: 12px 0 0 12px;
        .ant-card-head-title {
          padding: 0;
        }
      }
      &:last-child {
        border: 0;
      }
    }
  }
  .sts-card-1 {
    .ant-card-head {
      background-color: #409eff;
    }
  }
  .sts-card-2 {
    .ant-card-head {
      background-color: #67c23a;
    }
  }
  .sts-card-3 {
    .ant-card-head {
      background-color: #e6a23c;
    }
  }
  .sts-card-4 {
    .ant-card-head {
      background-color: #009688;
    }
  }
</style>
