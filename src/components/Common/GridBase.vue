<template>
  <div class="container">
    <div>现在使用json-server的模拟数据，模拟后端api.</div>
    {{queryUrl}}|page:{{page}}
    <filter-pane :filters-title="filters.title" :filters-fields.sync="filters.fields" :sort-fields.sync="gridColumns" :sort-key.sync="sortKey" :order.sync="order" :on-search="search"></filter-pane>
    <grid :grid-data="gridData" :columns="gridColumns" :is-show-action="isShowAction" :actions="actions" :sort-key.sync="sortKey" :order.sync="order" :on-save="onSave">
    </grid>
    <pagination :page.sync="page" :page-size.sync="pageSize" :page-size-list="pageSizeList" :records="records"></pagination>
  </div>
</template>

<script>
import grid from '../Common/Grid'
import filterPane from '../Common/filterPane'
import pagination from '../Common/Pagination'

export default {
  components: {
    filterPane,
    grid,
    pagination
  },
  props: {
    url: {
      type: String,
      default: function () {
        return '/api/users'
      }
    },
    filters: Object,
    gridColumns: Array,
    gridData: Array,
    isShowAction: Boolean,
    // actions 暂时无用！！！！！
    actions: Array,

    sortKey: String,
    order: String,
    page: {
      type: Number,
      default: function () {
        return 1
      }
    },
    pageSize: {
      type: Number,
      default: function () {
        return 10
      }
    },
    pageSizeList: Array,
    onSave: {
      type: Function,
      default: function (items) {
        console.log('gridBase onSave items:', items)
      }
    }
  },
  data () {
    return {
      queryUrl: '',
      records: 0
    }
  },
  methods: {
    getQueryUrl: function (isStart) {
      if (isStart) {
        this.page = 1
      }

      var arrFilters = []
      for (var it in this.filters.fields) {
        var field = this.filters.fields[it]
        if (field.value) {
          arrFilters.push(field.name + '_like=' + field.value)
        }
      }

      var start = (this.page - 1) * this.pageSize
      arrFilters.push('_start=' + start)
      arrFilters.push('_limit=' + this.pageSize)
      arrFilters.push('_sort=' + this.sortKey)
      arrFilters.push('_order=' + this.order)

      return this.url + '?' + arrFilters.join('&')
    },
    search: function (isStart) {
      console.log('parent search isStart:', isStart)
      this.queryUrl = this.getQueryUrl(isStart)

      console.log('search queryUrl:', this.queryUrl)

      this.$http({url: this.queryUrl, method: 'GET'})
      .then(function (response) {
        console.log('search response:', response)
        var count = response.headers('X-Total-Count')
        this.gridData = response.data
        this.records = parseInt(count)
      }, function (response) {
          // error callback
      })
    }
  },
  ready: function () {
    this.search(true)
  },
  watch: {
    page: function (oldValue, newValue) {
      console.log('arguments:', oldValue, newValue)
      this.search()
    },
    pageSize: function (oldValue, newValue) {
      console.log('arguments:', oldValue, newValue)
      this.search(true)
    },
    sortKey: function (oldValue, newValue) {
      console.log('arguments:', oldValue, newValue)
      this.search()
    },
    order: function (oldValue, newValue) {
      console.log('arguments:', oldValue, newValue)
      this.search()
    }
  }
}

</script>