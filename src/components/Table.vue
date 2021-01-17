<template>
  <div>
    <div class="options mt-5">
      <h3>Options</h3>
      <form>
        <div class="row my-4">
          <div class="col-5">
            <label class="form-label"> Data to display </label>
            <input
              type="text"
              class="form-control"
              :value="getAvailableColumnList"
            />
            <div class="form-text fs-5 fw-light">
              Available: {{ getAvailableColumnList }}
            </div>
          </div>
          <div class="col-5">
            <label class="form-label"> Endpoint </label>
            <input type="text" class="form-control" :value="endpoint" />
          </div>
          <div class="col-2">
            <label class="form-label invisible"> Update </label>
            <a class="form-control btn btn-primary">
              Update
            </a>
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
            />
          </div>
          <div class="col-auto">
            <a
              class="btn btn-primary"
              :disabled="!getOptions.search"
              @click="search(this)"
            >
              Search
            </a>
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
              :class="[
                getCurrentPage == index
                  ? 'btn-secondary'
                  : 'btn-outline-secondary'
              ]"
              v-for="(item, index) in getPageMaxNumber"
              :key="index"
              @click="updatePagination(index)"
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
              scope="col"
              v-for="(item, index) in getColumnList"
              :key="index"
              @click="sortData(item)"
            >
              {{ item }}
              <span v-if="canSort && sort.lastChoice === item">
                {{ sort.lastChoice === item && sort.asc ? "↑" : "↓" }}
              </span>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(data, index) in tableData" :key="index">
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
  name: "Table",
  props: {
    endpoint: {
      type: String,
      default: "https://jsonplaceholder.typicode.com/users"
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
      default: "name,email,company,city,website"
    }
  },
  data() {
    return {
      options: {},
      tableData: [],
      sort: {
        asc: true,
        lastChoice: "name"
      },
      pagin: {
        sizeByPage: 3,
        leftItems: null,
        currentPage: 0,
        maxPages: null
      }
    };
  },
  mounted() {
    // set options from props to local storage
    this.options = {
      endpoint: this.endpoint,
      search: this.search,
      sorting: this.sorting,
      pagination: this.pagination,
      dataTypes: this.dataTypes
    };

    // get and save external data
    this.getExternalData(this.options.endpoint).then(data => {
      this.pagin.maxPages = Math.floor(data.length / 3) + 1;
      this.updateExternalData(data);
    });
  },
  computed: {
    getOptions() {
      return this.options;
    },
    getAvailableColumnList() {
      return this.options.dataTypes;
    },
    getColumnList() {
      return this.options.dataTypes
        ? this.options.dataTypes.split(",")
        : this.options.dataTypes;
    },
    getPageMaxNumber() {
      return Math.floor(this.tableData.length / 3) + 1;
    },
    getCurrentPage() {
      return this.pagin.currentPage;
    },
    canSort() {
      return this.options.sorting && !this.options.pagination;
    },
    getPagination() {
      return this.options.pagination;
    }
  },
  methods: {
    async getExternalData(endpoint) {
      // get and format data from endpoint
      try {
        const { data } = await this.axios.get(endpoint);
        let oldData = data;
        let newData = [];
        oldData.forEach(item => {
          newData.push({
            id: item.id,
            name: item.name,
            email: item.email,
            company: item.company.name,
            city: item.address.city,
            website: item.website
          });
        });
        return newData;
      } catch (error) {
        console.log(error);
      }
    },
    updateExternalData(data) {
      // save data to local storage
      this.tableData = data;
    },
    updatePagination(index) {
      this.setPage(index);
      let data = this.tableData;
      const pagin = this.pagin;
      let currentValue = pagin.currentPage * 3;
      if (this.getPagination) {
        data = data.slice(currentValue, currentValue + 3).map(i => {
          return i;
        });
      }
      return data;
    },
    setPage(index) {
      console.log(index);
      // set current page by index
      this.pagin.currentPage = index;
      // // update view
      // this.tableData;
    },

    sortData(columName) {
      if (!this.canSort) return;

      const newData = this.tableData.sort((a, b) =>
        this.sort.asc
          ? a[columName].localeCompare(b[columName])
          : b[columName].localeCompare(a[columName])
      );

      // toggle asc / desc
      this.sort.asc = !this.sort.asc;
      this.sort.lastChoice = columName;
      // update data
      this.updateExternalData(newData);
    },
    toggleOptions(optionName) {
      this.options[optionName] = !this.options[optionName];
    },
    disabledForm(e) {
      e.preventDefault();
    },
    beEmail(trItem, tdItem) {
      return tdItem == "email" ? `<a>${trItem[tdItem]}</a>` : trItem[tdItem];
    }
  }
};
</script>

<style lang="scss" scoped></style>
