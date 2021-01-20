<template>
  <div>
    <div class="options mt-5">
      <h3>Options</h3>
      <form @change="changeForm()">
        <div class="row my-4">
          <div class="col">
            <label class="form-label"> Data to display </label>
            <input id="elmTableCol" type="text" class="form-control" :value="$TableListAvailable" />
            <div class="form-text fs-5 fw-light">Available: {{ $TableListAvailable }}</div>
          </div>
          <div class="col-auto">
            <label class="form-label invisible"> Reset form </label>
            <button type="button" class="form-control btn btn-danger" @click="resetForm()">
              Reset
            </button>
          </div>
        </div>
        <!-- start - search -->
        <div class="row my-4 d-flex align-items-center">
          <div class="col-auto ml-4">
            <input
              id="elmSearch"
              class="form-check-input"
              type="checkbox"
              :checked="$Options.search"
              @click="toggleOption('search')"
            />
            <label class="form-check-label" for="elmSearch">
              Search
            </label>
          </div>
          <div class="col">
            <input
              id="elmSearchText"
              class="form-control btn-sm"
              type="text"
              :disabled="!$Options.search"
              placeholder="some name, email, company..."
            />
          </div>
        </div>
        <!-- end - search -->
        <div class="row my-4 d-flex align-items-center">
          <div class="col-auto mx-4">
            <input
              id="elmSorting"
              class="form-check-input"
              type="checkbox"
              :checked="$Options.sorting"
              @click="toggleOption('sorting')"
            />
            <label class="form-check-label" for="elmSorting">
              Sorting
            </label>
          </div>
          <div class="col-auto">
            <input
              id="elmPagination"
              class="form-check-input"
              type="checkbox"
              :checked="$Options.pagination"
              @click="toggleOption('pagination')"
            />
            <label class="form-check-label" for="elmPagination">
              Pagination
            </label>
          </div>
          <div class="col">
            Page:
            <a
              class="btn mx-1"
              :class="[$PaginationPageCurrent == index ? 'btn-secondary' : 'btn-outline-secondary']"
              v-for="(item, index) in $PaginationPageMax"
              :key="index"
              @click="setPaginationPage(index)"
              :disabled="!$Options.pagination"
            >
              {{ index }}
            </a>
          </div>
        </div>
      </form>
    </div>

    <div class="mt-5">
      <h3>Table</h3>
      <table class="table table-hover">
        <thead>
          <tr>
            <th
              class="table-head"
              :class="{ 'table-head-active': $Options.sorting }"
              scope="col"
              v-for="(item, index) in $TableList"
              :key="index"
              @click="doSort(item)"
            >
              <span>{{ item }}</span>
              <span v-if="$CanSort && dynamic.sort.lastChoice === item">
                {{ dynamic.sort.lastChoice === item && dynamic.sort.asc ? '↑' : '↓' }}
              </span>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(data, index) in $DataLocal" :key="index">
            <td v-for="(type, index) in $TableList" :key="index">
              <a v-if="type == 'email'" :href="`mailto:${data[type]}`">
                {{ data[type] }}
              </a>
              <span v-if="type != 'email'"> {{ data[type] }} </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
//
function assign(obj, keyPath, value) {
  let lastKeyIndex = keyPath.length - 1
  for (var i = 0; i < lastKeyIndex; ++i) {
    let key = keyPath[i]
    if (!(key in obj)) {
      obj[key] = {}
    }
    obj = obj[key]
  }
  obj[keyPath[lastKeyIndex]] = value
}
//

export default {
  name: 'Table',
  props: {
    endpoint: {
      type: String,
      default: 'https://jsonplaceholder.typicode.com/users'
    },
    search: {
      type: Boolean,
      default: false
    },
    sorting: {
      type: Boolean,
      default: false
    },
    pagination: {
      type: Boolean,
      default: false
    },
    dataTypes: {
      type: String,
      default: 'name,email,company,city,website'
    }
  },
  data() {
    return {
      options: {},
      dynamic: {
        sort: {
          asc: true,
          lastChoice: 'name'
        },
        pagination: {
          itemsByPage: 3,
          pageCurrent: 0,
          pageMax: null
        }
      },
      data: {
        external: [],
        local: []
      }
    }
  },
  mounted() {
    this.options = {
      endpoint: this.endpoint,
      search: this.search,
      sorting: this.sorting,
      pagination: this.pagination,
      dataTypes: this.dataTypes
    }

    this.getExternalData(this.$Options.endpoint).then(data => {
      this.dynamic.pagination.pageMax = Math.floor(data.length / 3) + 1
      this.setAllData(data)
    })
  },
  computed: {
    $Options() {
      return this.options
    },
    $DataExternal() {
      return this.data.external
    },
    $DataLocal() {
      return this.data.local
    },
    $TableList() {
      const dataTypes = this.$Options.dataTypes
      return dataTypes ? dataTypes.split(',') : dataTypes
    },
    $TableListAvailable() {
      return this.$Options.dataTypes
    },
    $PaginationPageMax() {
      return this.dynamic.pagination.pageMax
    },
    $PaginationPageCurrent() {
      return this.dynamic.pagination.pageCurrent
    },
    $CanSort() {
      return this.$Options.sorting && !this.$Options.pagination
    }
  },
  methods: {
    changeForm() {
      this.$Options.dataTypes = document.getElementById('elmTableCol').value
      this.doSearch()
    },
    resetForm() {
      this.resetTableColumns()
      this.clearSearch()
    },
    resetTableColumns() {
      this.$Options.dataTypes = this.dataTypes
    },
    // $data
    setAllData(data) {
      this.data.external = data
      this.data.local = data
    },
    setLocalData(data) {
      this.data.local = data
    },
    async getExternalData(endpoint) {
      try {
        const { data } = await this.axios.get(endpoint)
        let newData = []
        data.forEach(item => {
          newData.push({
            id: item.id,
            name: item.name,
            email: item.email,
            company: item.company.name,
            city: item.address.city,
            website: item.website
          })
        })

        return newData
      } catch (error) {
        console.log(error)
      }
    },
    updateTableData(indexes) {
      let searchedData = []
      this.$DataExternal.forEach((item, index) => {
        if (indexes.includes(index)) searchedData.push(item)
      })
      this.setLocalData(searchedData)
    },
    // $pagination
    updatePagination() {
      if (!this.$Options.pagination) return
      let currentValue = this.dynamic.pagination.pageCurrent * 3
      let oldData = this.$DataExternal
      let newData = []
      newData = oldData.slice(currentValue, currentValue + 3).map(i => {
        return i
      })
      this.setLocalData(newData)
    },
    setPaginationPage(index) {
      if (!this.$Options.pagination) return
      this.dynamic.pagination.pageCurrent = index
      this.updatePagination()
    },
    // $sort
    doSort(columnName) {
      if (!this.$CanSort) return
      const asc = this.dynamic.sort.asc
      const newData = this.$DataLocal.sort((a, b) => {
        a = a[columnName]
        b = b[columnName]
        if (typeof a == 'number') return asc ? a - b : b - a
        if (typeof a == 'string') return asc ? a.localeCompare(b) : b.localeCompare(a)
      })
      // toggle asc / desc
      this.dynamic.sort.asc = !this.dynamic.sort.asc
      this.dynamic.sort.lastChoice = columnName
      // update data
      this.setLocalData(newData)
    },
    // $search
    doSearch() {
      if (!this.$Options.search) return
      this.clearSearch()
      let phrase = document.getElementById('elmSearchText')
      let indexes = []
      this.$DataExternal.find((item, index) => {
        Object.keys(item).forEach(prop => {
          if (this.helpersNormString(item[prop]).search(this.helpersNormString(phrase.value)) != -1) {
            if (!indexes.includes(index)) indexes.push(index)
          }
        })
      })
      this.updateTableData(indexes)
    },
    clearSearch() {
      this.setLocalData(this.$DataExternal)
    },
    // $option
    toggleOption(name) {
      if (name == 'sorting') {
        this.options.pagination = false
        this.options.sorting = true
      } else if (name == 'pagination') {
        this.options.pagination = true
        this.options.sorting = false
      } else {
        this.options[name] = !this.options[name]
      }
    },
    // $helpers
    helpersNormString(string) {
      return string.toString().toLowerCase()
    },
    helpersDoEmail(trItem, tdItem) {
      return tdItem == 'email' ? `<a>${trItem[tdItem]}</a>` : trItem[tdItem]
    }
  }
}
</script>

<style lang="scss" scoped>
.table {
  &-head {
    text-transform: capitalize;

    &-active {
      & span {
        cursor: pointer;
      }
    }
  }
}
</style>
