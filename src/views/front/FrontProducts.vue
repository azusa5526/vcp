<template>
  <div>
    <loading :active.sync="isLoading"></loading>
    <FrontSlidesProducts></FrontSlidesProducts>

    <div class="row mt-4 justify-content-center">
      <div class="col-xl-2 col-lg-3 col-md-3 col-sm-12 col-12 mb-3 side-wrap">
        <FrontSidebar
          :cateFilter="categoryFilter"
          :prodFilter="productsFilter"
          @filterUpdate="updateProductsFilter"
        ></FrontSidebar>
      </div>

      <div class="col-xl-10 col-lg-9 col-md-9 col-sm-12 col-12 product-wrap">
        <div class="row">
          <div
            class="col-2xl-3 col-xl-4 col-lg-6 col-md-6 col-sm-6 mb-4 product-hover card-wrap"
            v-for="(item, index) in productsInWindowList"
            :key="index"
          >
            <a
              class="link-block"
              :href="'#/front_single_product/' + item.id"
              @click="getProduct(item.id)"
              @click.middle="getProduct(item.id)"
            >
              <div class="card border-0 shadow-sm">
                <div class="promote-wrap" v-if="item.category.indexOf('hero') !== -1">
                  <img src="@/assets/img/hero_promote_b.png" />
                  <h6 class="promote-content">HREO</h6>
                </div>

                <div
                  style="height: 300px; background-size: contain; background-position: center; background-repeat: no-repeat; background-color: #FFFFFF;"
                  :style="{backgroundImage: `url(${item.imageUrl})`}"
                ></div>
                <div class="card-body">
                  <h5 class="card-title">{{item.title}}</h5>
                  <p class="card-text">{{item.content}}</p>
                </div>

                <div class="card-footer d-flex justify-content-between align-items-baseline">
                  <div class="h5 mb-0" v-if="!item.price">{{item.origin_price | currency}}</div>
                  <del class="h6 mb-0" v-if="item.price">{{item.origin_price | currency}}</del>
                  <div class="h5 mb-0" v-if="item.price">{{item.price | currency}}</div>
                </div>
              </div>
            </a>
          </div>
        </div>

        <div class="row justify-content-center">
          <Pgnation
            v-if="filteredProducts.length !== 0"
            class="mt-4"
            :pagination="pgnation"
            @changePage="changeCurrentPage"
          ></Pgnation>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import FrontSidebar from '@/components/front/FrontSidebar';
import Pgnation from '@/components/front/Pgnation';
import FrontSlidesProducts from './FrontSlidesProducts';

export default {
  components: {
    FrontSidebar,
    Pgnation,
    FrontSlidesProducts
  },

  data () {
    return {
      products: [],
      isLoading: false,
      status: {
        noProductsInWindow: false
      },
      pgnation: {
        current_page: 1,
        has_next: false,
        has_pre: false,
        total_pages: 1,
        page_size: 12
      },

      categoryFilter: '',
      productsFilter: [],
      filteredProducts: [],
      productsInWindow: [],
      categoryFilteredList: [],
      tempRandomProducts: []
    };
  },

  methods: {
    getAllProducts () {
      const api = `${process.env.API_PATH}/api/${process.env.CUSTOM_PATH}/products/all`;
      const vm = this;
      vm.isLoading = true;

      vm.$http.get(api).then(response => {
        vm.isLoading = false;
        vm.products = response.data.products;
      });
    },

    getProduct (id) {
      const api = `${process.env.API_PATH}/api/${process.env.CUSTOM_PATH}/product/${id}`;
      const vm = this;
      vm.isLoading = true;

      localStorage.setItem(
        'cateFilteredList',
        JSON.stringify(vm.categoryFilteredList)
      );

      vm.$http.get(api).then(response => {
        if (response.data.success) {
          vm.isLoading = false;
          vm.$router.push(
            `../front_single_product/${response.data.product.id}`
          );
        }
      });
    },

    activatedProductFilterList () {
      const vm = this;
      return vm.products.filter(function (item) {
        return item.is_enabled;
      });
    },

    categoryFilterList () {
      const vm = this;
      let tempProducts = vm.activatedProductFilterList();
      tempProducts.reverse();

      vm.categoryFilter = vm.$route.params.categoryFilter;
      if (vm.categoryFilter === 'all') {
        return tempProducts;
      } else {
        return tempProducts.filter(function (item) {
          return item.category.indexOf(vm.categoryFilter) !== -1;
        });
      }
    },

    productsFilterList () {
      const vm = this;
      let tempProducts = vm.categoryFilterList();
      vm.categoryFilteredList = tempProducts;

      if (vm.productsFilter.length === 0) {
        return tempProducts;
      } else {
        for (let filter of vm.productsFilter) {
          tempProducts = tempProducts.filter(function (item) {
            return item.category.indexOf(filter) !== -1;
          });
        }
        return tempProducts;
      }
    },

    updateProductsFilter (prodsFilter) {
      const vm = this;
      vm.productsFilter = prodsFilter;
    },

    pgnationCounter () {
      const vm = this;
      let productsLength = vm.filteredProducts.length;

      vm.pgnation.total_pages = Number(
        Math.floor(productsLength / vm.pgnation.page_size) + 1
      );

      if (vm.pgnation.current_page < vm.pgnation.total_pages) {
        vm.pgnation.has_next = true;
      } else {
        vm.pgnation.has_next = false;
      }

      if (vm.pgnation.current_page > 1) {
        vm.pgnation.has_pre = true;
      } else {
        vm.pgnation.has_pre = false;
      }
    },

    pageSpliter () {
      const vm = this;
      let pageMinIndex =
        vm.pgnation.current_page * vm.pgnation.page_size -
        vm.pgnation.page_size +
        1;
      let pageMaxIndex = vm.pgnation.current_page * vm.pgnation.page_size;

      vm.productsInWindow = [];
      vm.filteredProducts.forEach(function (item, index) {
        const num = index + 1;
        if (num >= pageMinIndex && num <= pageMaxIndex) {
          vm.productsInWindow.push(item);
        }
      });
    },

    changeCurrentPage (targetPage) {
      const vm = this;
      vm.pgnation.current_page = Number(targetPage);
      vm.pgnationCounter();
      vm.pageSpliter();
    }
  },

  computed: {
    productsInWindowList () {
      const vm = this;
      let productsInWindow = [];

      vm.filteredProducts = vm.productsFilterList();

      let pageMinIndex =
        vm.pgnation.current_page * vm.pgnation.page_size -
        vm.pgnation.page_size +
        1;
      let pageMaxIndex = vm.pgnation.current_page * vm.pgnation.page_size;

      vm.filteredProducts.forEach(function (item, index) {
        const num = index + 1;
        if (num >= pageMinIndex && num <= pageMaxIndex) {
          productsInWindow.push(item);
        }
      });
      vm.pgnationCounter();
      vm.pageSpliter();
      return productsInWindow;
    }
  },

  watch: {
    categoryFilter: {
      handler () {
        this.pgnation.current_page = 1;
      }
    },

    productsInWindow: {
      handler () {
        if (this.productsInWindow.length === 0) {
          this.status.noProductsInWindow = true;
        } else {
          this.status.noProductsInWindow = false;
        }
      }
    }
  },

  created () {
    this.getAllProducts();
    this.pgnationCounter();
    this.pageSpliter();
  }
};
</script>
