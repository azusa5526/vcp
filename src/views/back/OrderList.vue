<template>
  <div>
    <loading :active.sync="isLoading"></loading>
    <div class="dashboard-table-wrap my-3">
      <table class="table">
        <thead>
          <tr>
            <th width="150px">ORDER TIME</th>
            <th>EMAIL</th>
            <th class="d-lg-table-cell d-none">ORDER ITEMS</th>
            <th>OEDER NUM</th>
            <th width="130px">AMOUNT PAYABLE</th>
            <th width="130px">PAYMENT STATUS</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="item in orders" :key="item.id">
            <td>{{item.create_at | timestampToDate}}</td>
            <td>{{item.user.email}}</td>
            <td class="d-lg-table-cell d-none">
              <ul class="list-unstyled">
                <li
                  v-for="(product, id) in item.products"
                  :key="id"
                >{{product.product.title}} / {{product.qty}} {{product.product.unit}}</li>
              </ul>
            </td>
            <td>{{item.id}}</td>
            <td class="text-right">{{item.total}}</td>
            <td class="text-center">
              <span class="text-success" v-if="item.is_paid">PAID</span>
              <span class="text-danger" v-else>OUTSTANDING PAYMENT</span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <Pagination :pagination="pagination" @changePage="getOrders"></Pagination>
  </div>
</template>

<script>
import Pagination from '@/components/back/Pagination';

export default {
  components: {
    Pagination
  },

  data () {
    return {
      orders: [],
      tempProduct: {},
      isNew: false,
      isLoading: false,
      status: {
        fileUploading: false
      },
      pagination: {}
    };
  },

  methods: {
    getOrders (page = 1) {
      const api = `${process.env.API_PATH}/api/${process.env.CUSTOM_PATH}/admin/orders?page=${page}`;
      const vm = this;
      vm.isLoading = true;

      vm.$http.get(api).then(response => {
        vm.isLoading = false;
        vm.orders = response.data.orders;
        vm.pagination = response.data.pagination;
      });
    }
  },

  created () {
    this.getOrders();
  }
};
</script>
