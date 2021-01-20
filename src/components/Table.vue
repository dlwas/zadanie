<template>
  <div>
    <div class="options mt-5">
      <h3>Options</h3>
      <form>
        <div class="row my-4">
          <div class="col-5">
            <label class="form-label"> Data to display </label>
            <input type="text" class="form-control" :value="getAvailableColumnList" />
            <div class="form-text fs-5 fw-light">Available: {{ getAvailableColumnList }}</div>
          </div>
          <div class="col-5">
            <label class="form-label"> Endpoint </label>
            <input type="text" class="form-control" :value="endpoint" />
          </div>
          <div class="col-2">
            <label class="form-label invisible"> Update </label>
            <button
              type="button"
              class="form-control btn btn-primary"
              v-on:change="$emit('change', $event.target.checked)"
            >
              Update
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
              :checked="getOptions.search"
              @click="toggleOptions('search')"
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
              :disabled="!getOptions.search"
              placeholder="some name, email, company..."
              value="Sincere@april.biz"
            />
          </div>
          <div class="col-auto">
            <button type="button" class="btn btn-primary" :disabled="!getOptions.search" @click="doSearch()">
              Search
            </button>
            <button type="button" class="btn btn-warning ml-3" :disabled="!getOptions.search" @click="clearSearch()">
              Clear
            </button>
          </div>
        </div>
        <!-- end - search -->
        <div class="row my-4 d-flex align-items-center">
          <div class="col-auto mx-4">
            <input
              id="elmSorting"
              class="form-check-input"
              type="checkbox"
              :checked="getOptions.sorting"
              @click="toggleOptions('sorting')"
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
              :checked="getOptions.pagination"
              @click="toggleOptions('pagination')"
            />
            <label class="form-check-label" for="elmPagination">
              Pagination
            </label>
          </div>
          <div class="col">
            Page:
            <a
              class="btn"
              :class="[getCurrentPage == index ? 'btn-secondary' : 'btn-outline-secondary']"
              v-for="(item, index) in getPageMaxNumber"
              :key="index"
              @click="setPage(index)"
              :disabled="!getOptions.pagination"
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
              :class="{ 'table-head-active': getOptions.sorting }"
              scope="col"
              v-for="(item, index) in getColumnList"
              :key="index"
              @click="sortData(item)"
            >
              <span>{{ item }}</span>
              <span v-if="canSort && sort.lastChoice === item">
                {{ sort.lastChoice === item && sort.asc ? '↑' : '↓' }}
              </span>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(data, index) in newTableData" :key="index">
            <td v-for="(type, index) in getColumnList" :key="index">
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
      tableData: [],
      newTableData: [],
      sort: {
        asc: true,
        lastChoice: 'name'
      },
      pagin: {
        sizeByPage: 3,
        leftItems: null,
        currentPage: 0,
        maxPages: null
      }
    }
  },
  mounted() {
    // set options from props to local storage
    this.options = {
      endpoint: this.endpoint,
      search: this.search,
      sorting: this.sorting,
      pagination: this.pagination,
      dataTypes: this.dataTypes
    }

    // get and save external data
    this.getExternalData(this.options.endpoint).then(data => {
      this.pagin.maxPages = Math.floor(data.length / 3) + 1
      this.updateExternalData(data)
    })
  },
  computed: {
    getOptions() {
      return this.options
    },
    getAvailableColumnList() {
      return this.options.dataTypes
    },
    getColumnList() {
      return this.options.dataTypes ? this.options.dataTypes.split(',') : this.options.dataTypes
    },
    getPageMaxNumber() {
      return Math.floor(this.tableData.length / 3) + 1
    },
    getCurrentPage() {
      return this.pagin.currentPage
    },
    canSort() {
      return this.options.sorting || !this.options.pagination
    },
    getPagination() {
      return this.options.pagination
    }
  },
  methods: {
    async getExternalData(endpoint) {
      // get and format data from endpoint
      try {
        const { data } = await this.axios.get(endpoint)
        let oldData = data
        let newData = []
        oldData.forEach(item => {
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
    updateExternalData(data) {
      // save data to local storage
      this.tableData = data
      this.newTableData = data
    },
    updatePagination() {
      if (!this.getOptions.pagination) return
      const pagin = this.pagin
      let currentValue = pagin.currentPage * 3
      this.newTableData = this.tableData.slice(currentValue, currentValue + 3).map(i => {
        return i
      })
    },
    togglePagination() {
      if (!this.getOptions.pagination || !this.getOptions.sorting) this.newTableData = this.tableData
      this.setPage(0)
    },
    setPage(index) {
      if (!this.getOptions.pagination) return
      this.pagin.currentPage = index
      this.updatePagination(index)
    },
    sortData(columName) {
      if (!this.canSort) return

      const newData = this.newTableData.sort((a, b) => {
        a = a[columName]
        b = b[columName]

        if (typeof a == 'number') return this.sort.asc ? a - b : b - a
        if (typeof a == 'string') return this.sort.asc ? a.localeCompare(b) : b.localeCompare(a)
      })
      // toggle asc / desc
      this.sort.asc = !this.sort.asc
      this.sort.lastChoice = columName
      // update data
      this.newTableData = newData
    },
    toggleOptions(optionName) {
      // update option
      this.options[optionName] = !this.options[optionName]
      // do other stuff after changes
      this.togglePagination()
    },
    beEmail(trItem, tdItem) {
      return tdItem == 'email' ? `<a>${trItem[tdItem]}</a>` : trItem[tdItem]
    },
    doSearch() {
      if (!this.getOptions.search) return
      this.clearSearch()
      let phrase = document.getElementById('elmSearchText')
      let indexes = []
      this.newTableData.find((item, index) => {
        Object.keys(item).forEach(prop => {
          if (this.normString(item[prop]).search(this.normString(phrase.value)) != -1) {
            if (!indexes.includes(index)) indexes.push(index)
          }
        })
      })
      this.updateTableData(indexes)
    },
    clearSearch() {
      this.newTableData = this.tableData
    },
    normString(string) {
      return string.toString().toLowerCase()
    },
    updateTableData(indexes) {
      let searchedData = []
      console.log(indexes)
      this.tableData.forEach((item, index) => {
        if (indexes.includes(index)) searchedData.push(item)
      })
      this.newTableData = searchedData
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
