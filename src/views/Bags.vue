<template>
  <div class="keranjang">
    <Navbar :updateBag="bags" />
    <div class="container">
      <!-- breadcrumb -->
      <div class="row mt-4">
        <div class="col">
          <nav aria-label="breadcrumb">
            <ol class="breadcrumb">
              <li class="breadcrumb-item">
                <router-link to="/" class="text-dark">Home</router-link>
              </li>
              <li class="breadcrumb-item">
                <router-link to="/foods" class="text-dark">Foods</router-link>
              </li>
              <li class="breadcrumb-item active" aria-current="page">Bags</li>
            </ol>
          </nav>
        </div>
      </div>

      <div class="row">
        <div class="col">
          <h2>My <strong>Bags</strong></h2>
          <div class="table-responsive mt-3">
            <table class="table">
              <thead>
                <tr>
                  <th scope="col">#</th>
                  <th scope="col">Foto</th>
                  <th scope="col">Foods</th>
                  <th scope="col">Info</th>
                  <th scope="col">Amount</th>
                  <th scope="col">Price</th>
                  <th scope="col">Total Price</th>
                  <th scope="col">Hapus</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(bag, index) in bags" :key="bag.id">
                  <th>{{ index + 1 }}</th>
                  <td>
                    <img
                      :src="'../assets/img/' + bag.products.gambar"
                      class="img-fluid shadow"
                      width="250"
                    />
                  </td>
                  <td>
                    <strong>{{ bag.products.nama }}</strong>
                  </td>
                  <td>{{ bag.keterangan ? bag.keterangan : "-" }}</td>
                  <td>{{ bag.jumlah_pesanan }}</td>
                  <td align="right">Rp. {{ bag.products.harga }}</td>
                  <td align="right">
                    <strong
                      >Rp. {{ bag.products.harga * bag.jumlah_pesanan }}</strong
                    >
                  </td>
                  <td align="center" class="text-danger">
                    <b-icon-trash @click="deleteBag(bag.id)"></b-icon-trash>
                  </td>
                </tr>

                <tr>
                  <td colspan="6" align="right">
                    <strong>Total Price : </strong>
                  </td>
                  <td align="right">
                    <strong>Rp. {{ totalPrice }}</strong>
                  </td>
                  <td></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- Form Checkout -->
      <div class="row justify-content-end">
        <div class="col-md-4">
          <form class="mt-4" v-on:submit.prevent>
            <div class="form-group">
              <label class="nama">Nama : </label>
              <input type="text" class="form-control" v-model="pesan.nama" />
            </div>
            <div class="form-group">
              <label class="notable">No Table : </label>
              <input type="text" class="form-control" v-model="pesan.notable" />
            </div>
            <button
              type="submit"
              class="btn btn-success float-right"
              @click="checkout"
            >
              <b-icon-cart></b-icon-cart>Pesan
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Navbar from "@/components/Navbar.vue";
import axios from "axios";

export default {
  name: "Bags",
  components: {
    Navbar,
  },
  data() {
    return {
      bags: [],
      pesan: {},
    };
  },
  methods: {
    setBags(data) {
      this.bags = data;
    },
    deleteBag(id) {
      axios
        .delete("http://localhost:3000/bags/" + id)
        .then(() => {
          this.$toast.success("Delete Success!.", {
            type: "error",
            position: "top-right",
            duration: 3000,
            dismissable: true,
          });
          //Update Keranjang
          axios
            .get("http://localhost:3000/bags")
            .then((response) => this.setBags(response.data))
            .catch((error) => console.log("Gagal", error));
        })
        .catch((error) => console.log("Gagal", error));
    },
    checkout() {
      if (this.pesan.nama && this.pesan.notable) {
        this.pesan.bags = this.bags;
        axios.post("http://localhost:3000/orders", this.pesan).then(() => {
          // Delete All Bags
          this.bags.map(function (item) {
            return axios
              .delete("http://localhost:3000/bags/" + item.id)
              .catch((error) => console.log("Gagal", error));
          });
          this.$router.push({ path: "/order-success" });
          this.$toast.success("Success Order.", {
            type: "success",
            position: "top-right",
            duration: 3000,
            dismissable: true,
          });
        });
      } else {
        this.$toast.success("Name and No Table must be filled.", {
          type: "error",
          position: "top-right",
          duration: 3000,
          dismissable: true,
        });
      }
    },
  },
  mounted() {
    axios
      .get("http://localhost:3000/bags")
      .then((response) => this.setBags(response.data))
      .catch((error) => console.log("Gagal", error));
  },
  computed: {
    totalPrice() {
      return this.bags.reduce(function (items, data) {
        return items + data.products.harga * data.jumlah_pesanan;
      }, 0);
    },
  },
};
</script>

<style>
</style>