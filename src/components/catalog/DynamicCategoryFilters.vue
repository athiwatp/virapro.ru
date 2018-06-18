<template>
  <div id="filters">
    <el-row id="filter_title">
      <el-col :span="19">Фильтр</el-col>
      <el-col v-if="Object.keys(dynamicFilters).length" :span="5">
        <div id="clear_filter" @click="clearCheckedFilters">
          <i class="el-icon-close"></i>
        </div>
      </el-col>
    </el-row>
    <div v-if="filtersTree.length">
      <el-tree
        ref="filtersTree"
        :data="filtersTree"
        :props="defaultProps"
        @check="getCheckedFilterNodes"
        node-key="value"
        show-checkbox>
      </el-tree>
    </div>
    <p v-else align="middle">Нет фильтров</p>
  </div>
</template>
<script>
  export default {
    name: 'DynamicCategoryFilters',
    props: {},
    data() {
      return {
        filtersTree: '',
        defaultProps: {children: 'children', value: 'value', label: 'label'},
      }
    },
    methods: {
      async clearCheckedFilters() {
        if (this.$refs.filtersTree) {
          this.$refs.filtersTree.setCheckedNodes([])
        }
        await this.$store.dispatch('setProductDynamicFilters', [])
      },


      getCheckedFilterNodes(data, tree) { // active dynamic filters
        let filterObj = {} // { color: [red, green], size: [big, small] }
        let prevNodeProp
        tree.checkedNodes.forEach(node => {
          if (!node.children) {
            if (!prevNodeProp) {
              filterObj[node.prop] = [node.value]
            } else if (prevNodeProp === node.prop) {
              filterObj[node.prop].push(node.value)
            } else {
              filterObj[node.prop] = [node.value]
            }
            prevNodeProp = node.prop
          }
        })
        this.$store.dispatch('setProductDynamicFilters', filterObj).then(() => {
          this.filterProducts()
        })
      },


      createFiltersTree() { // get unique values of fetched products by every dynamic props
        if (!this.selectedNode || !this.selectedNode.filters) {
          return []
        }
        let tree = []
        let node = {}
        this.selectedNode.filters.forEach(prop => {
          node.value = this.$store.getters.DYNAMIC_PROPS[prop].value
          node.label = this.$store.getters.DYNAMIC_PROPS[prop].label
          node.children = []
          let unique = []
          for (let p in this.products) {
            if (this.products[p][prop] && unique.indexOf(this.products[p][prop]) === -1) {
              unique.push(this.products[p][prop])
            }
          }
          unique.forEach(el => {
            node.children.push({value: el, label: el, prop: prop})
          })
          if (node.children.length) {
            tree.push(node)
          }
          node = {}
        })
        this.filtersTree = tree
      }
    },
    computed: {
      dynamicFilters() {
        return this.$store.getters.productDynamicFilters
      }
    }
  }
</script>
<style scoped lang="scss">

  #filters {
    margin-top: 10px;
    flex: 0 0 300px;
  }

  #filter_title {
    display: flex;
    justify-content: start;
    padding-left: 18px;
    align-items: center;
    height: 40px;
    color: #909399;
    background: #f5f7fa;
    border: 1px solid #dcdfe6;
    border-radius: 4px;
    margin-right: 10px;
    width: 290px;
  }

  #clear_filter {
    background: #1A7CDD;
    padding: 9px 9px 9px 20px;
    color: white;
    border-top-right-radius: 4px;
    border-bottom-right-radius: 4px;
  }

  #clear_filter:hover {
    cursor: pointer;
  }

  .el-icon-close {
    transition: all .3s;
  }

  .el-icon-close:hover {
    @include rotate(90deg, scale(1.4))
  }

  @media only screen and (max-width: $sm-screen) {
    #filters {
      display: none;
    }
  }
</style>