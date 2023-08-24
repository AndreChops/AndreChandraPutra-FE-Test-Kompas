<script>
//Import Axios
import axios from "axios";

// Import the Moment Indonesian locale
import moment from "moment";
import "moment/dist/locale/id";

export default {
  // Decraling Initival Data Value
  data() {
    return {
      itemName: "",
      itemHarga: "",
      detail: [],
      isModalVisible: false,
    };
  },

  // Calling fetchItem Method
  created() {
    this.fetchItem();
  },

  // Methods Use in this Project
  methods: {
    // Show and Close Modal Method
    showModal() {
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
      this.itemName = "";
      this.itemHarga = "";
    },

    // Fetch Data Method
    async fetchItem() {
      try {
        const res = await axios.get(`http://localhost:3000/detail`);

        // Sort the data by date in descending and hour by ascending order
        res.data.sort((a, b) => {
          const dateA = moment(a.tanggal, "DD/MM/YYYY");
          const dateB = moment(b.tanggal, "DD/MM/YYYY");

          if (dateA.isSame(dateB, "day")) {
            const timeA = moment(a.jam, "HH:mm");
            const timeB = moment(b.jam, "HH:mm");
            return timeA.diff(timeB);
          }

          return dateB.diff(dateA);
        });

        const groupedItems = {};

        res.data.forEach((item) => {
          const tanggal = item.tanggal;
          if (!groupedItems[tanggal]) {
            groupedItems[tanggal] = [];
          }
          groupedItems[tanggal].push(item);
        });

        this.detail = groupedItems;
      } catch (error) {
        console.error("Error fetching items:", error);
      }
    },

    // Formatting Date
    formatDate(tanggal) {
      moment.locale("id");
      return moment(tanggal, "DD/MM/YYYY").format("DD MMMM");
    },

    // Format Currency
    formatCurrency(harga) {
      return harga.toLocaleString("id-ID");
    },

    // Calculating Total Expense per Date
    calculatedTotal(detail) {
      let total = 0;
      detail.forEach((item) => {
        total += item.pengeluaraan;
      });

      return total;
    },

    // Adding Data Method
    async addItem() {
      if (!this.itemHarga || !this.itemName) {
        alert("Input field is empty!");
        return;
      }

      try {
        const response = await axios.post(`http://localhost:3000/detail`, {
          nama: this.itemName,
          pengeluaraan: this.itemHarga,
          tanggal: moment().format("DD/MM/YYYY"),
          jam: moment().format("HH:mm"),
        });

        console.log("Server response:", response.data);

        const tanggal = response.data.tanggal;
        if (!this.detail[tanggal]) {
          this.detail[tanggal] = [];
        }
        this.detail[tanggal].push(response.data);

        console.log("Updated detail array:", this.detail);
      } catch (error) {
        console.error("Error adding item:", error);
      }
      this.closeModal();
    },
  },

  // Define Computed Properti
  computed: {
    // Calculating Total Expenses
    calculateTotalExpenses() {
      let total = 0;
      Object.values(this.detail).forEach((itemsByDate) => {
        itemsByDate.forEach((item) => {
          total += item.pengeluaraan;
        });
      });
      return total;
    },
  },
};
</script>

<template>
  <!-- Header Container-->
  <header class="leading-6 text-center mb-4">
    <div class="headers">
      <h2 class="text-2xl font-bold text-gray-900">Diari Jajan Agustus 2023</h2>
      <p class="text-lg py-1">
        Pengeluaran Bulan ini Rp {{ formatCurrency(calculateTotalExpenses) }}
      </p>
      <button
        class="bg-violet-900 hover:bg-violet-700 text-white py-2 px-4"
        @click="showModal"
      >
        TAMBAH ITEM
      </button>
    </div>
  </header>

  <!-- Main Container -->
  <main class="mt-5">
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
      <div
        id="table-cards"
        class="shadow-xl border border-gray-400 px-4 py-2 m-2"
        v-for="(itemsByDate, tanggal) in detail"
        :key="tanggal"
      >
        <div>
          <h1 class="font-bold py-2 text-xl">{{ formatDate(tanggal) }}</h1>
          <div
            class="flex flex-row justify-between border-t border-gray-200 py-2"
            v-for="item in itemsByDate"
            :key="item.tanggal"
          >
            <div class="flex flex-row gap-4">
              <h2 class="w-10">{{ item.jam }}</h2>
              <h2>{{ item.nama }}</h2>
            </div>
            <div>
              <h2>Rp {{ formatCurrency(item.pengeluaraan) }}</h2>
            </div>
          </div>
          <div
            class="flex flex-row justify-end gap-5 border-t border-gray-400 py-2"
          >
            <h2 class="font-bold text-l">Total</h2>
            <h2 class="font-bold text-l">
              Rp {{ formatCurrency(calculatedTotal(itemsByDate)) }}
            </h2>
          </div>
        </div>
      </div>
    </div>
  </main>

  <!-- Modal Container -->
  <div
    v-if="isModalVisible"
    class="fixed inset-0 flex items-center justify-center bg-gray-700 bg-opacity-50"
  >
    <div class="max-w-2xl w-[70%] p-6 bg-white rounded-md shadow-xl">
      <div class="flex justify-start">
        <h3 class="text-2xl">Tambah Entri</h3>
      </div>
      <div class="mt-4">
        <form @submit.prevent="addItem">
          <p>Nama</p>
          <input
            class="w-[100%] border border-gray-400 px-4 py-2 rounded mb-4"
            type="text"
            required
            placeholder="Nama"
            v-model="itemName"
          />
          <p>Harga</p>
          <input
            class="w-[100%] border border-gray-400 px-4 py-2 rounded mb-4"
            type="number"
            placeholder="Harga"
            v-model="itemHarga"
          />

          <div class="flex justify-end">
            <button
              @click="closeModal"
              class="bg-gray-500 hover:bg-gray-700 text-white py-2 px-6 rounded-sm"
            >
              BATAL
            </button>
            <button
              type="submit"
              class="bg-violet-900 hover:bg-violet-700 text-white ml-2 py-2 px-6 rounded-sm"
            >
              KIRIM
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
