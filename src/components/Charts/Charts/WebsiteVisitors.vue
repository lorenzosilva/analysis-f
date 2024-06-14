<template>
  <div class="card mb-25 border-0 rounded-0 bg-white">
    <div class="card-body p-15 p-sm-20 p-sm-25 p-lg-30 letter-spacing">
      <div
        class="mb-lg-15 d-sm-flex align-items-center justify-content-between"
      >
        <h6 class="card-title fw-bold mb-0"> Categoría de producto más popular</h6>
      </div>
      <div id="websiteVisitorsChart" class="chart">
        <apexchart
          type="bar"
          :options="websiteVisitorsChartOptions"
          :series="[
            {
              name: 'Cantidad',
              data: topProducts.map((product) => product.quantity),
            },
          ]"
        ></apexchart>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import VueApexCharts from "vue3-apexcharts";

export default defineComponent({
  name: "WebsiteVisitors",
  props: {
    arrayData: {
      type: Object,
      required: true,
      default: () => ({}),
    },
  },
  components: {
    apexchart: VueApexCharts,
  },
  setup: (props) => {
    interface ProductRating {
      category: string;
      quantity: number;
    }

    const topProducts = ref<ProductRating[]>([]);

    const websiteVisitorsChartOptions = {
      chart: {
        type: "bar",
        height: 385,
        toolbar: {
          show: false,
        },
      },
      plotOptions: {
        bar: {
          borderRadius: 5,
          horizontal: false,
          columnWidth: "28%",
          endingShape: "rounded",
        },
      },
      dataLabels: {
        enabled: false,
      },
      colors: ["#6560F0"],
      stroke: {
        show: false,
      },
      xaxis: {
        categories: [] as string[],
        labels: {
          show: true,
          style: {
            fontFamily: "Red Hat Display, sans-serif",
            colors: "#9C9AB6",
            fontSize: "14px",
            fontWeight: 500,
          },
        },
      },
      yaxis: {
        show: true,
        tickAmount: 8,
        labels: {
          show: true,
          style: {
            fontFamily: "Red Hat Display, sans-serif",
            colors: ["#9C9AB6"],
            fontSize: "14px",
            fontWeight: 500,
          },
        },
      },
      fill: {
        opacity: 1,
      },
      tooltip: {
        style: {
          fontSize: "14px",
          fontFamily: "Red Hat Display, sans-serif",
        },
      },
      grid: {
        show: true,
        strokeDashArray: 5,
        borderColor: "#d9e9ef",
      },
    };

    const getTopCategory = async () => {
      try {
        const orders = Object.values(props.arrayData);
        const allOrders = orders[0];
        const allProducts = orders[1];

        const productMap = {};
        allProducts.forEach((product) => {
          productMap[product.objectID] = product.product_name;
        });

        const productQuantityMap = {};
        const productCategoryMap = {};

        allOrders.forEach((order) => {
          const productId = order.product_id;
          const quantity = parseInt(order.quantity);
          const category = order.product_category;

          if (productId.trim() !== "") {
            if (
              Object.prototype.hasOwnProperty.call(
                productQuantityMap,
                productId
              )
            ) {
              productQuantityMap[productId] += quantity;
            } else {
              productQuantityMap[productId] = quantity;
              productCategoryMap[productId] = category;
            }
          }
        });

        const products = Object.keys(productQuantityMap).map((productId) => ({
          category: productCategoryMap[productId],
          quantity: productQuantityMap[productId],
        }));

        products.sort((a, b) => b.quantity - a.quantity);

        const top10Products = products.slice(0, 10);

        topProducts.value = top10Products;
        websiteVisitorsChartOptions.xaxis.categories = top10Products.map(
          (product) => product.category
        );
      } catch (error) {
        console.error("Error al obtener los productos:", error);
      }
    };

    onMounted(async () => {
      await getTopCategory();
    });

    return {
      topProducts,
      websiteVisitorsChartOptions,
    };
  },
});
</script>
