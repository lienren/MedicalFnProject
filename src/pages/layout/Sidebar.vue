<template>
  <a-layout-sider :trigger="null" collapsible v-model="collapsed" :style="{ position: 'relative' }">
    <div class="logo">{{collapsed?shortLogoName:logoName}}
      <span>{{version}}</span>
    </div>
    <a-menu theme="dark" mode="inline" :defaultOpenKeys="defaultOpenKeys" :defaultSelectedKeys="defaultSelectedKeys" v-model="selectMenuKeys" @select="select">
      <template v-for="menu in menus">
        <a-sub-menu v-if="menu.children&&menu.children.length>0" :key="menu.id">
          <span slot="title">
            <a-icon :type="menu.icon" />
            <span>{{menu.name}}</span>
          </span>
          <a-menu-item v-for="child in menu.children" :key="child.id">{{child.name}}</a-menu-item>
        </a-sub-menu>
        <a-menu-item v-else :key="menu.id">
          <a-icon :type="menu.icon" />
          <span>{{menu.name}}</span>
        </a-menu-item>
      </template>
    </a-menu>
    <div class="trigger-list">
      <a-icon class="trigger menu-collapsed-switch" :title="menuSwitchText" :type="menuSwitchClassName" @click="collapse" />
      <!-- <a-icon class="trigger logout-button" title="安全退出" type="logout" @click="logout" /> -->
    </div>
  </a-layout-sider>
</template>

<script>

export default {
  props: {
    menus: {
      type: Array
    },
    logoName: {
      type: String,
      default: 'Demo'
    },
    shortLogoName: {
      type: String,
      default: 'Demo'
    },
    version: {
      type: String,
      default: 'Beta1.0.0'
    },
    collapsed: {
      type: Boolean,
      default: false
    },
    selectMenuKey: {
      type: String
    }
  },
  components: {},
  data () {
    return {
      selectMenuKeys: []
    }
  },
  watch: {
    collapsed (val) {
      this.$emit('update:collapsed', val)
    },
    selectMenuKey (val) {
      this.selectMenuKeys = [val]
    },
    selectMenuKeys (val) {
      this.$emit('update:selectMenuKey', val[0])
    }
  },
  computed: {
    rountKey () {
      let path = window.location.href.split('/')
      return path && path.length > 0 ? `/${path[path.length - 1]}` : ''
    },
    defaultOpenKeys () {
      let rountKey = this.rountKey
      let openKeys = ''
      this.menus.forEach((menu) => {
        if (menu.children && menu.children.length > 0) {
          openKeys = menu.children.some((child) => {
            return child.id === rountKey
          }) ? menu.id : ''
        }
      })
      return openKeys === '' ? [] : [openKeys]
    },
    defaultSelectedKeys () {
      return this.rountKey === '' ? [] : [this.rountKey]
    },
    menuSwitchText () {
      return this.collapsed ? '打开菜单' : '收起菜单'
    },
    menuSwitchClassName () {
      return this.collapsed ? 'menu-unfold' : 'menu-fold'
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
    init () {
      this.$emit('on-init', {
        defaultOpenKeys: this.defaultOpenKeys,
        defaultSelectedKeys: this.defaultSelectedKeys
      })
    },
    logout () {
      this.$emit('on-logout')
    },
    collapse () {
      this.$emit('update:collapsed', !this.collapsed)
      this.$emit('on-collapse', this.collapsed)
    },
    select (selected) {
      this.$emit('on-select', selected)
    }
  }
}
</script>

<style lang="less" rel="stylesheet/less">
  .trigger-list {
    position: absolute;
    width: 100%;
    height: 38px;
    bottom: 0;
    background-color: #001529;
    .trigger {
      color: rgba(255, 255, 255, 0.65);
      font-size: 18px;
      cursor: pointer;
      transition: color 0.3s;
      &:hover {
        color: #1890ff;
      }
      &.menu-collapsed-switch {
        position: absolute;
        left: 10px;
        top: 10px;
      }
      &.logout-button {
        position: absolute;
        right: 10px;
        top: 10px;
      }
    }
  }

  .logo {
    position: relative;
    height: 48px;
    font-size: 18px;
    text-align: center;
    margin: 8px 16px 0;
    color: #fff;
    overflow: hidden;
    span {
      position: absolute;
      left: 50%;
      top: 50%;
      display: inline-block;
      font-size: 9px;
      padding: 1px 5px;
      border-radius: 20px;
      background: #f56c6c;
      color: #fff;
      margin: 0px 0 0 -30px;
      transform: scale(0.7);
    }
  }
</style>
