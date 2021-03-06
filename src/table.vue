<template>
  <div class="v-parts-table-wrapper"  ref="wrapper">
    <div :style="{height,overflow:'auto'}">
      <table ref="table" class="v-table" :class="{bordered,compact,striped}">
        <thead>
        <tr>
          <th><input ref="allChecked" :checked="areAllItemsSelected" type="checkbox" @change="onChangeAllItems"></th>
          <th v-if="numberVisible">#</th>
          <th v-for="column in columns" :key="column.field">
            <div class="v-parts-table-column">{{ column.text }}
              <span v-if="column.field in orderBy" class="v-parts-table-sorter" @click="changeOrderBy(column.field)">
            <g-icon name="asc" :class="{active:orderBy[column.field] === 'asc'}"></g-icon><g-icon name="desc"
                                                                                                  :class="{active:orderBy[column.field] === 'desc'}"></g-icon></span>
            </div>
          </th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="(item,index) in dataSource" :key="item.id">
          <td><input type="checkbox" :checked="inSelectedItems(item)" @change="onChangeItem(item,index,$event)"></td>
          <td v-if="numberVisible">{{ index + 1 }}</td>
          <template v-for="column in columns">
            <td :key="column.field">{{ item[column.field] }}</td>
          </template>
        </tr>
        </tbody>
      </table>
    </div>

    <div v-if="loading" class="v-parts-table-loading">
      <g-icon name="loading"></g-icon>
    </div>
  </div>
</template>
<script>
import GIcon from './icon'

export default {
  name: "v-parts-table",
  components: {
    GIcon
  },
  props: {
    height:{
      type:[String,Number]
    },
    loading:{
      type:Boolean,
      default:false
    },
    orderBy: {
      type: Object,
      default: () => ({}),
    },
    striped: {
      type: Boolean,
      default: true
    },
    selectedItems: {
      type: Array,
      default: () => []
    },
    compact: {
      type: Boolean,
      default: false
    },
    columns: {
      type: Array,
      required: true
    },
    dataSource: {
      type: Array,
      required: true,
      validator(array) {
        if (array.filter(item => item.id === undefined).length > 0) {
          return false
        }
        return true
      }
    },
    numberVisible: {
      type: Boolean,
      default: false
    },
    bordered: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    areAllItemsSelected() {
      let a = this.dataSource.map(item => item.id).sort()
      let b = this.selectedItems.map(item => item.id).sort()
      if (a.length != b.length) {
        return false
      }
      let equal = true
      for (let i = 0; i < a.length; i++) {
        if (a[i] !== b[i]) {
          equal = false
          break
        }
      }
      return equal
    }
  },
  watch: {
    selectedItems() {
      if (this.selectedItems.length === this.dataSource.length) {
        this.$refs.allChecked.indeterminate = false
      } else if (this.selectedItems.length === 0) {
        this.$refs.allChecked.indeterminate = false
      } else {
        this.$refs.allChecked.indeterminate = true
      }
    }
  },
  mounted() {
    let table2 = this.$refs.table.cloneNode(true)
    this.table2 = table2
    table2.classList.add('v-parts-table-copy')
    this.$refs.wrapper.appendChild(table2)
    this.updateHeadersWidth()
    this.onWindowResize = ()=> this.updateHeadersWidth()
    window.addEventListener('resize',this.onWindowResize)
  },
  beforeDestroy() {
    window.removeEventListener('resize',this.onWindowResize)
    this.table2.remove()
  },
  methods: {
    updateHeadersWidth() {
      let table2 = this.table2
      const tableHeader = Array.from(this.$refs.table.children).filter(node=>node.tagName.toLowerCase()==='thead')[0]
      let tableHeader2
      Array.from(table2.children).map(node=>{
        if(node.tagName.toLowerCase()!=='thead') {
          node.remove()
        } else {
          tableHeader2 = node
        }
      })
      Array.from(tableHeader.children[0].children).map((th,i)=>{
        const {width} = th.getBoundingClientRect()
        tableHeader2.children[0].children[i].style.width = width + 'px'
      })
    },
    changeOrderBy(key) {
      const copy = JSON.parse(JSON.stringify(this.orderBy))
      let oldValue = copy[key]
      if (oldValue === 'asc') {
        copy[key] = 'desc'
      } else if (oldValue === 'desc') {
        copy[key] = true
      } else {
        copy[key] = 'asc'
      }
      this.$emit('update:orderBy', copy)
    },
    inSelectedItems(item) {
      return this.selectedItems.filter(i => i.id === item.id).length > 0
    },
    onChangeItem(item, index, e) {
      let selected = e.target.checked
      let copy = JSON.parse(JSON.stringify(this.selectedItems))
      if (selected) {
        copy.push(item)
      } else {
        copy = copy.filter(i => i.id !== item.id)
      }
      this.$emit('update:selectedItems', copy)
    },
    onChangeAllItems(e) {
      let selected = e.target.checked
      this.$emit('update:selectedItems', selected ? this.dataSource : [])
    }
  }


}
</script>
<style scoped lang="scss">
@import "../styles/_var";

.v-parts-table-wrapper {
  $grey: darken($grey, 10%);
  position: relative;
  .v-table {
    width: 100%;
    border-collapse: collapse;
    border-spacing: 0;
    border-bottom: 1px solid $grey;
    &.bordered {
      border: 1px solid $grey;
      td, th {
        border: 1px solid $grey;
      }
    }

    &.compact {
      td, th {
        padding: 4px;
      }
    }

    td, th {
      border-bottom: 1px solid $grey;
      text-align: left;
      padding: 8px;
    }

    &.striped {
      tbody {
        > tr {
          &:nth-child(odd) {
            background: white;
          }

          &:nth-child(even) {
            background: lighten($grey, 10%);
          }
        }
      }
    }

    .v-parts-table-column {
      display: flex;
      align-items: center;

      .v-parts-table-sorter {
        display: inline-flex;
        flex-direction: column;
        margin: 0 4px;
        cursor: pointer;

        svg {
          width: 10px;
          height: 10px;
          fill: $grey;

          &.active {
            fill: red;
          }

          &:first-child {
            position: relative;
            bottom: -1px;
          }

          &:nth-child(2) {
            position: relative;
            top: -1px;
          }
        }
      }
    }

  }

  .v-parts-table-loading {
    position: absolute;
    top:0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background: rgba(255,255,255,0.7);

    svg {
      @include spin;
      height: 48px;
      width: 48px;
    }
  }
  .v-parts-table-copy {
    position: absolute;
    top:0;
    left: 0;
    width: 100%;
    background: white;
  }
}
</style>