<template>
	<div id="app">
		<h1>Hello Kendo UI for Vue!</h1>
		<p>
			<dropdownlist :data-items="categories" :data-item-key="'CategoryID'" :text-field="'CategoryName'"
				:default-item="defaultItems" @change="handleDropDownChange">
			</dropdownlist>
			&nbsp; Selected category ID: <strong>{{dropdownlistCategory}}</strong>
		</p>

		<grid :data-items="dataResult" :pageable="pageable" :sortable="sortable" :sort="sort" :page="page"
			:columns="columns" @datastatechange="dataStateChange" :style="{ height: '400px' }">

			<template v-slot:discontinuedTemplate="{ props }">
				<td colspan="1">
					<input type="checkbox" :checked = props.dataItem.Discontinued disabled="disabled" />
              </td>
			</template>
		</grid>

	</div>
</template>

<script>
	import categories from './appdata/categories.json';
import products from './appdata/products.json';
import { process } from '@progress/kendo-data-query';
import { Grid } from '@progress/kendo-vue-grid';
import { DropDownList } from '@progress/kendo-vue-dropdowns';
import '@progress/kendo-theme-default/dist/all.css';


export default {
  name: 'App',
  components: {
    'dropdownlist': DropDownList,
    'grid': Grid
  },
  data: function() {
    return {
      categories: categories,
      products: products,
      defaultItems: {CategoryID: null, CategoryName: "Product categories"},
      dropdownlistCategory: null,
      pageable: true,
      sortable: true,
      page: {
          skip: 0,
          take: 10
      },
      sort: [
          { field: "ProductName", dir: "asc" }
      ],
      filter: {},
      columns: [
          { field: 'ProductName', title: 'Product Name'},
          { field: 'UnitPrice', title: 'Price' },
          { field: 'UnitsInStock', title: 'Units in Stock' },
          { field: 'Discontinued', cell: 'discontinuedTemplate' }
      ],
      dataResult:[]
      }
  },
  created() {
      const dataState = {
          skip: this.skip,
          take: this.take,
          sort: this.sort,
      };

      this.dataResult = process(products, dataState);
  },
  methods: {
      handleDropDownChange (e) {
          this.dropdownlistCategory = e.target.value.CategoryID;

          if (e.target.value.CategoryID !== null) {
            this.filter = {
              logic: 'and',
              filters: [{ field: 'CategoryID', operator: 'eq', value: e.target.value.CategoryID }]
            }
            this.skip = 0
          } else {
            this.filter = []
            this.skip = 0
          }
          let event = {data:{
              skip: this.page.skip,
              take: this.page.take,
              sort: this.sort,
              filter: this.filter
          }};
          this.dataStateChange(event);
      },
      createAppState: function(dataState) {
          this.page.take = dataState.take;
          this.page.skip = dataState.skip;
          this.sort = dataState.sort;
          this.filter = dataState.filter;
      },
      dataStateChange (event) {
          this.createAppState(event.data);
          this.dataResult = process(products, {
              skip: this.page.skip,
              take: this.page.take,
              sort: this.sort,
              filter: this.filter
          });
      }
  }
}
</script>

<style>
	#app {
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: left;
		color: #2c3e50;
		margin-top: 60px;
	}
</style>