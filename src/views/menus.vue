<!-- 顶部菜单 -->
<style lang="less">
    @import './menus.less';
</style>

<template>
  <ul class="menus">
      <li v-for="item in filterMenus" :key="item.id" :class="{select:item.id===selected}" @click="switchMenus(item)">{{item.name}}</li>
  </ul>
</template>

<script>
import Cookies from 'js-cookie';
import util from '../libs/util.js';
import {kycRouter, otcRouter, exchangeRouter, financeRouter, riskRouter, operationRouter, adminRouter, systemConfigRouter, systemLogsRouter, monitoringRouter, fundRouter } from '../router';
import breadcrumbNavVue from './main_components/breadcrumbNav.vue';
export default {
  data () {
    return {
      selected: localStorage.getItem('currentMenu') || 'kyc',
      datas: [
        {id: 'kyc', path: 'kycauditing_index', name: 'KYC管理', menus: kycRouter},
        {id: 'otc', path: 'otc_data_statistics_index', name: 'OTC管理', menus: otcRouter},
        {id: 'exchange', path: 'exchange_data_statistics_index', name: '币币管理', menus: exchangeRouter},
        {id: 'finance', path: 'finance_finance_index', name: '财务管理', menus: financeRouter},
        {id: 'risk', path: 'risk_exchange_index', name: '风险控制管理', menus: riskRouter},
        {id: 'operation', path: 'operation_distribute_index', name: '运维推广管理', menus: operationRouter},
        {id: 'admin', path: 'admin_index', name: '管理员权限管理', menus: adminRouter},
        {id: 'systemconfig', path: 'systemconfig_index', name: '系统参数', menus: systemConfigRouter},
        {id: 'systemcogs', path: 'systemlogs_index', name: '系统日志', menus: systemLogsRouter},
        {id: 'monitoring', path: 'monitoring_index', name: '监控平台', menus: monitoringRouter},
        {id: 'fund', path: 'fund_index', name: '平台资金管理', menus: fundRouter}
      ]
    }
  },
  computed: {
    filterMenus () {
      let accessCode = Cookies.get('roles');
      let newMenus = []
      this.datas.forEach((menu) => {
      let menuList = [];
            menu.menus.forEach((item, index) => {
                let access = item.meta && item.meta.roles;
                if (access !== undefined) {
                    if (util.showThisRoute(access, accessCode)) {
                        if (item.children.length <= 1) {
                            menuList.push(item);
                        } else {
                            let i = menuList.push(item);
                            let childrenArr = [];
                            childrenArr = item.children.filter(child => {
                                let childAccess = child.meta && child.meta.roles;
                                if (childAccess !== undefined) {
                                    if (childAccess === accessCode) {
                                        return child;
                                    }
                                } else {
                                    return child;
                                }
                            });
                            menuList[i - 1].children = childrenArr;
                        }
                    }
                } else {
                    if (item.children.length <= 1) {
                        menuList.push(item);
                    } else {
                        let i = menuList.push(item);
                        let childrenArr = [];
                        childrenArr = item.children.filter(child => {
                            if (access !== undefined) {
                                if (util.showThisRoute(access, accessCode)) {
                                    return child;
                                }
                            } else {
                                return child;
                            }
                        });
                        menuList[i - 1].children = childrenArr;
                    }
                }
            });
            if (menuList.length) {
                  newMenus.push({
                      id: menu.id,
                      menus: menuList,
                      name: menu.name,
                      path: menu.path
                  })
                }
          });
            return newMenus
    },
    tagsList () {
            return this.$store.state.tagsList;
    },
    menuList () {
        return this.$store.state.menuList;
    }
  },
  watch: {
    selected (newVal) {
      localStorage.setItem('currentMenu', newVal);
    },
    '$route' (to) {
      if (to.name === 'home_index' && this.filterMenus.length) {
          this.changeMenu(this.filterMenus[0].menus[0].children[0].name)
      }
    }
  },
  created () {
    let isExist = false
    for (let i = 0; i< this.filterMenus.length; i++) {
      let data = this.filterMenus[i];
      if (data.id === this.selected) {
        isExist = true
        this.switchMenus(data, false);
        break;
      }
    }
    if (!isExist && this.filterMenus.length) {
      this.switchMenus(this.filterMenus[0]);
    }
    /*let apiToken = Cookies.get('Authorization')
    if (apiToken && this.$route.name === 'home_index' && this.filterMenus.length) {
      this.$router.push({name: this.filterMenus[0].menus[0].children[0].name})
    }*/
  },
  methods: {
    switchMenus (item, notClear) {
      this.selected = item.id
      this.$store.commit('updateMenulist', item.menus);
      if (notClear === false) {
          return
      }
      this.$store.commit('clearTag');
      this.changeMenu(item.menus[0].children[0].name);
    },
    changeMenu (active) {
            if (active !== 'accesstest_index') {
                let pageOpenedList = this.$store.state.pageOpenedList;
                let openedPageLen = pageOpenedList.length;
                let i = 0;
                let tagHasOpened = false;
                while (i < openedPageLen) {
                    if (active === pageOpenedList[i].name) {  // 页面已经打开
                        this.$store.commit('moveToSecond', i);
                        tagHasOpened = true;
                        break;
                    }
                    i++;
                }
                if (!tagHasOpened) {
                    let tag = this.tagsList.filter((item) => {
                        if (item.children) {
                            return active === item.children[0].name;
                        } else {
                            return active === item.name;
                        }
                    });
                    tag = tag[0];
                    tag = tag.children ? tag.children[0] : tag;
                    this.$store.commit('increateTag', tag);
                    localStorage.pageOpenedList = JSON.stringify(this.$store.state.pageOpenedList); // 本地存储已打开页面
                }
                this.$store.commit('setCurrentPageName', active);
                this.$router.push({
                  name: active
                });
            }
        }
  }
}
</script>
