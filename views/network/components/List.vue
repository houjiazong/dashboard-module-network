<template>
  <page-list
    :list="list"
    :columns="columns"
    :group-actions="groupActions"
    :singleActions="singleActions"
    :export-data-options="exportDataOptions" />
</template>

<script>
import * as R from 'ramda'
import { mapGetters } from 'vuex'
import {
  getProjectTableColumn,
  getRegionTableColumn,
  getBrandTableColumn,
  getStatusTableColumn,
  getCopyWithContentTableColumn,
  getNameDescriptionTableColumn,
  isPublicTableColumn,
} from '@/utils/common/tableColumn'
import windows from '@/mixins/windows.js'
import i18n from '@/locales'
import { findPlatform } from '@/utils/common/hypervisor'
import { getBrandFilter, getAccountFilter, getTenantFilter } from '@/utils/common/tableFilter'

const PROVIDER_FILTER_CN = i18n.t('env')

const disableAdjustConfig = ['private', 'public']
const canAdjustConfig = (obj) => {
  const config = {
    hasbrand: true,
    canUpdate: true,
    brand: '',
  }
  let brand = obj.brand || obj.provider
  config.brand = brand
  if (!brand) config.hasbrand = false
  brand = brand.toLowerCase()
  const platform = findPlatform(brand)
  if (disableAdjustConfig.includes(platform)) config.canUpdate = false
  return config
}

export default {
  name: 'NetworkList',
  mixins: [windows],
  props: {
    id: String,
    getParams: {
      type: [Function, Object],
    },
    cloudEnv: String,
  },
  data () {
    return {
      list: this.$list.createList(this, {
        id: this.id,
        resource: 'networks',
        getParams: this.getParam,
        filterOptions: {
          name: {
            label: '实例名称',
            filter: true,
            formatter: val => {
              return `name.contains("${val}")`
            },
          },
          ip: {
            label: 'IP',
            distinctField: {
              type: 'extra_field',
              key: 'ip',
            },
          },
          status: {
            label: '状态',
            dropdown: true,
            items: [
              { label: '初始化', key: 'init' },
              { label: '正在创建', key: 'pending' },
              { label: '正常', key: 'available' },
              { label: '异常', key: 'failed' },
              { label: '开始删除', key: 'start_delete' },
              { label: '正在删除', key: 'deleting' },
              { label: '已删除', key: 'deleted' },
              { label: '删除失败', key: 'delete_failed' },
              { label: '未知', key: 'unknown' },
            ],
            filter: true,
            formatter: val => {
              return `status.in(${val})`
            },
          },
          server_type: {
            label: '类型',
            dropdown: true,
            items: [
              { label: '物理机', key: 'baremetal' },
              { label: '容器', key: 'container' },
              { label: '虚拟机', key: 'guest' },
              { label: 'PXE', key: 'pxe' },
              { label: 'IPMI', key: 'ipmi' },
            ],
          },
          brand: getBrandFilter(),
          account: getAccountFilter(),
          tenant: getTenantFilter(),
          region: {
            label: '区域',
          },
          vpc: {
            label: 'VPC',
          },
          wire: {
            label: '二层网络',
            filter: true,
            formatter: val => {
              return `wire.contains("${val}")`
            },
          },
        },
      }),
      exportDataOptions: {
        items: [
          { label: 'ID', key: 'id' },
          { label: '名称', key: 'name' },
          { label: '开始IP', key: 'guest_ip_start' },
          { label: '结束IP', key: 'guest_ip_end' },
          { label: '类型', key: 'server_type' },
          { label: '状态', key: 'status' },
          { label: '使用情况', key: 'ports' },
          { label: '平台', key: 'brand' },
          { label: '二层网络', key: 'wire' },
          { label: 'VLAN', key: 'vlan_id' },
          { label: 'VPC', key: 'vpc' },
          { label: '项目', key: 'tenant' },
          { label: '云账号', key: 'account' },
          { label: '区域', key: 'region' },
          { label: '可用区', key: 'zone' },
        ],
      },
      columns: [
        getNameDescriptionTableColumn({
          vm: this,
          hideField: true,
          slotCallback: row => {
            return (
              <side-page-trigger onTrigger={ () => this.sidePageTriggerHandle(row.id, 'NetworkSidePage') }>{ row.name }</side-page-trigger>
            )
          },
        }),
        {
          field: 'ip',
          title: 'IP地址',
          width: 140,
          slots: {
            default: ({ row }) => {
              return [
                <div>起：{ row.guest_ip_start }</div>,
                <div>止：{ row.guest_ip_end }</div>,
              ]
            },
          },
        },
        {
          field: 'server_type',
          title: '类型',
          width: 60,
          formatter: ({ cellValue }) => {
            if (cellValue === 'baremetal') {
              return '物理机'
            }
            if (cellValue === 'container') {
              return '容器'
            }
            if (cellValue === 'guest') {
              return '虚拟机'
            }
            if (cellValue === 'pxe') {
              return 'PXE'
            }
            if (cellValue === 'ipmi') {
              return 'IPMI'
            }
            return '未知'
          },
        },
        getStatusTableColumn({ statusModule: 'network' }),
        {
          field: 'ports',
          title: '使用情况',
          minWidth: 100,
          slots: {
            default: ({ row }) => {
              return [
                <div class='text-truncate'>总计:{ row.ports }</div>,
                <div class='text-truncate'>使用:{ row.ports_used }</div>,
              ]
            },
          },
        },
        isPublicTableColumn(),
        getBrandTableColumn(),
        getProjectTableColumn(),
        getRegionTableColumn(),
        getCopyWithContentTableColumn({ field: 'wire', title: '二层网络' }),
        {
          field: 'vlan_id',
          title: 'VLAN',
          width: 60,
        },
        getCopyWithContentTableColumn({ field: 'vpc', title: 'VPC' }),
        getCopyWithContentTableColumn({ field: 'account', title: '云账号' }),
      ],
      groupActions: [
        {
          label: '新建',
          permission: 'networks_create',
          action: () => {
            const creatPath = this.$router.resolve(this.$route.path)
            this.$router.push({ path: creatPath.resolved.path + '/create' })
          },
          meta: () => {
            return {
              buttonType: 'primary',
            }
          },
        },
        {
          label: '更多',
          actions: () => {
            return [
              {
                label: '更改项目',
                action: () => {
                  this.createDialog('ChangeOwenrDialog', {
                    data: this.list.selectedItems,
                    columns: this.columns,
                    list: this.list,
                  })
                },
                meta: () => {
                  const f = this.eachValidates((obj) => {
                    return this.isPower(obj)
                  })
                  return {
                    validate: f,
                  }
                },
              },
              {
                label: '设置共享',
                permission: 'networks_perform_public',
                action: () => {
                  this.createDialog('SetPublicDialog', {
                    data: this.list.selectedItems,
                    columns: this.columns,
                    list: this.list,
                  })
                },
                meta: () => {
                  const f = this.eachValidates((obj) => {
                    return this.isPower(obj)
                  })
                  return {
                    validate: f,
                  }
                },
              },
              // {
              //   label: '分割IP子网',
              //   permission: 'networks_perform_split',
              //   action: () => {
              //     this.createDialog('NetworkSplitDialog', {
              //       data: this.list.selectedItems,
              //       columns: this.columns,
              //       list: this.list,
              //     })
              //   },
              //   meta: () => {
              //     const f = this.eachValidates((obj) => {
              //       if (this.isPower(obj)) {
              //         if (obj.external_id && obj.external_id.length > 0) { // 是公网 IP
              //           return false
              //         } else {
              //           return true
              //         }
              //       }
              //     })
              //     return {
              //       validate: f,
              //       tooltip: '公网IP不支持该操作',
              //     }
              //   },
              // },
              {
                label: '合并IP子网',
                permission: 'networks_perform_merge',
                action: () => {
                  this.createDialog('NetworkConcatDialog', {
                    data: this.list.selectedItems,
                    columns: this.columns,
                    list: this.list,
                    itemData: {
                      IPfrom: this.IPfromObj.IPfrom,
                      nameFrom: this.IPfromObj.name,
                      IPto: this.IPtoObj.IPto,
                      nameTo: this.IPtoObj.name,
                    },
                  })
                },
                meta: () => {
                  let tooltip = ''
                  if (this.list.selectedItems.length === 2) {
                    if (this.list.selectedItems.every(v => v.external_id && v.external_id.length > 0)) { // 是公网 IP
                      tooltip = '公网 IP 不支持该操作'
                    } else {
                      if (!this.link(this.list.selectedItems)) {
                        tooltip = '请选择规范的两个IP子网'
                      }
                    }
                  } else {
                    tooltip = '请选择两个IP子网'
                  }
                  return {
                    validate: this.allowConcat,
                    tooltip,
                  }
                },
              },
              {
                label: '删除',
                permission: 'networks_delete',
                action: () => {
                  this.createDialog('DeleteResDialog', {
                    data: this.list.selectedItems,
                    columns: this.columns,
                    title: '删除',
                    list: this.list,
                  })
                },
                meta: () => {
                  return {
                    validate: this.list.allowDelete(),
                  }
                },
              },
            ]
          },
        },
      ],
      singleActions: [
        {
          label: '调整标签',
          action: (obj) => {
            this.createDialog('AdjustLabelDialog', {
              data: [obj],
              columns: this.columns,
              list: this.list,
            })
          },
          meta: (obj) => {
            return {
              validate: this.isPower(obj),
            }
          },
        },
        {
          label: '更多',
          actions: obj => {
            return [
              {
                label: '修改属性',
                permission: 'networks_update',
                action: () => {
                  const updatePath = this.$router.resolve(this.$route.path)
                  this.$router.push({ path: updatePath.resolved.path + '/edit', query: { network_id: obj.id } })
                },
                meta: () => {
                  let tooltip = ''
                  const { brand, canUpdate } = canAdjustConfig(obj)
                  const platform = findPlatform(brand)
                  if (!canUpdate) {
                    tooltip = `${PROVIDER_FILTER_CN[platform]}的IP子网不能修改属性`
                  }
                  if (this.isPower(obj)) {
                    const { hasbrand, canUpdate } = canAdjustConfig(obj)
                    if (!hasbrand) {
                      return {
                        validate: true,
                        tooltip,
                      }
                    }
                    return {
                      validate: canUpdate,
                      tooltip,
                    }
                  } else {
                    tooltip = '权限不足'
                    return {
                      validate: false,
                      tooltip,
                    }
                  }
                },
              },
              {
                label: '更改项目',
                permission: 'networks_perform_change_owner',
                action: () => {
                  this.createDialog('ChangeOwenrDialog', {
                    data: [obj],
                    columns: this.columns,
                    list: this.list,
                  })
                },
                meta: () => {
                  return {
                    validate: this.isPower(obj),
                  }
                },
              },
              {
                label: '设置为共享',
                permission: 'networks_perform_public',
                action: () => {
                  this.createDialog('SetPublicDialog', {
                    data: [obj],
                    title: '设置为共享',
                    columns: this.columns,
                    list: this.list,
                  })
                },
                meta: () => {
                  return {
                    validate: this.isPower(obj),
                  }
                },
              },
              {
                label: '分割IP子网',
                permission: 'networks_perform_split',
                action: () => {
                  this.createDialog('NetworkSplitDialog', {
                    data: [obj],
                    columns: this.columns,
                    list: this.list,
                  })
                },
                meta: () => {
                  if (this.isPower(obj)) {
                    if (obj.external_id && obj.external_id.length > 0) { // 是公网 IP
                      return {
                        validate: false,
                        tooltip: '公网 IP 不支持该操作',
                      }
                    } else {
                      return {
                        validate: true,
                      }
                    }
                  } else {
                    return {
                      validate: false,
                      tooltip: '权限不足',
                    }
                  }
                },
              },
              {
                label: '预留IP',
                permission: 'reservedips_create',
                action: (obj) => {
                  this.createDialog('NetworkReversedIPDialog', {
                    data: [obj],
                    columns: this.columns,
                    title: '预留IP',
                    list: this.list,
                  })
                },
                meta: () => {
                  if (this.isDomainMode) {
                    return {
                      validate: false,
                      tooltip: '权限不足',
                    }
                  }
                  return {
                    validate: true,
                  }
                },
              },
              {
                label: '删除',
                permission: 'networks_delete',
                action: () => {
                  this.createDialog('DeleteResDialog', {
                    data: [obj],
                    columns: this.columns,
                    title: '删除',
                    list: this.list,
                  })
                },
                meta: () => this.$getDeleteResult(obj),
              },
            ]
          },
        },
      ],
      IPfromObj: { IPfrom: '', name: '' },
      IPtoObj: { IPto: '', name: '' },
    }
  },
  computed: {
    ...mapGetters(['isAdminMode', 'isDomainMode', 'userInfo']),
    allowConcat () {
      if (this.list.selectedItems.length === 2) {
        if (this.list.selectedItems.every(v => v.external_id && v.external_id.length > 0)) { // 是公网 IP
          return false
        }
        return this.link(this.list.selectedItems)
      }
      return false
    },
  },
  watch: {
    cloudEnv (val) {
      this.$nextTick(() => {
        this.list.fetchData(0)
      })
    },
  },
  created () {
    this.initSidePageTab('network-detail')
    this.list.fetchData()
  },
  methods: {
    isPower (obj) {
      if (this.isAdminMode) return true
      if (this.isDomainMode) return obj.domain_id === this.userInfo.projectDomainId
      return obj.tenant_id === this.userInfo.projectId
    },
    eachValidates (callback) {
      let f = true
      if (this.list.selectedItems && this.list.selectedItems.length > 0) {
        for (let i = 0; i < this.list.selectedItems.length; i++) {
          const item = this.list.selectedItems[i]
          if (callback && !callback(item)) {
            f = false
            break
          }
        }
      } else {
        f = false
      }
      return f
    },
    _border (first, second) {
      let firstArr = first.split('.')
      let secondArr = second.split('.')
      let firstNum = Number(firstArr[firstArr.length - 1])
      let secondNum = Number(secondArr[secondArr.length - 1])
      let diff = firstNum - secondNum
      return Math.abs(diff) === 1
    },
    _comparePre (first, second) {
      let firstArr = first.split('.')
      let secondArr = second.split('.')
      firstArr.pop()
      secondArr.pop()
      return firstArr.join('') === secondArr.join('')
    },
    link ([first, second]) {
      const start = 'guest_ip_start'
      const end = 'guest_ip_end'
      if (this._border(first[end], second[start]) && this._comparePre(first[end], second[start])) {
        this.IPfromObj = {
          IPfrom: first[start],
          name: first.name,
        }
        this.IPtoObj = {
          IPto: second[end],
          name: second.name,
        }
        return true
      } else if (this._border(second[end], first[start]) && this._comparePre(second[end], first[start])) {
        this.IPfromObj = {
          IPfrom: second[start],
          name: second.name,
        }
        this.IPtoObj = {
          IPto: first[end],
          name: first.name,
        }
        return true
      } else {
        return false
      }
    },
    getParam () {
      const ret = {
        ...(R.is(Function, this.getParams) ? this.getParams() : this.getParams),
      }
      if (this.cloudEnv) ret.cloud_env = this.cloudEnv
      return ret
    },
  },
}
</script>
