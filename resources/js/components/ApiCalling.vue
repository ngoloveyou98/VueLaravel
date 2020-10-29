<template>
  <div class="api-calling">
    <div class="error" v-if="errors.length">
      <span v-for="(err, index) in errors" :key="index">{{ err }}</span>
      <hr />
    </div>
    <div class="create-form">
      <div class="product-name-input">
        <input type="text" v-model="product.name" />
      </div>
      <div class="product-name-input">
        <input type="text" v-model.number="product.price" />
      </div>
      <div class="button-create">
        <button @click="createProduct">Create</button>
      </div>
    </div>

    <div class="list-products">
      <h2>LIST PRODUCT</h2>
      <div class="product-table">
        <table class="table table-bordered">
          <thead>
            <tr>
              <th>ID</th>
              <th>Name</th>
              <th>Price</th>
              <th>Date created</th>
              <th>Edit</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(prod, index) in list_products" :key="index">
              <td>{{ prod.id }}</td>
              <td v-if="!prod.isEdit">{{ prod.name }}</td>
              <td v-else>
                <input type="text" v-model="selectedProduct.name" />
              </td>
              <td v-if="!prod.isEdit">{{ prod.price }}</td>
              <td v-else>
                <input type="text"  v-model.number="selectedProduct.price" />
              </td>

              <td>{{ prod.created_at }}</td>
              <td v-if="!prod.isEdit">
                  <button @click="selectProduct(prod)">
                      Edit
                  </button>
                  <button @click="deleteProduct(prod, index)">
                      Delete
                  </button>
                </td>
              <td v-else>
                <button @click="UpdateProduct(index)">Update</button>
                <button @click="prod.isEdit = false">Cancel</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      product: {
        name: "",
        price: 0
      },
      errors: [],
      list_products: [],
      selectedProduct: {}
    };
  },
  created() {
    this.getListProducts();
  },

  methods: {
    createProduct() {
      axios
        .post("/products", {
          name: this.product.name,
          price: this.product.price
        })
        .then(response => {
          console.log(response.data.product);
          this.list_products.push(...response.data.product, (isEdit = false));
        })
        .catch(error => {
          this.errors = error.response.data.errors.name;
        });
    },

    getListProducts() {
      axios
        .get("/products")
        .then(response => {
          //    console.log(response.data.product)
          this.list_products = response.data.product;
          this.list_products.forEach(item => {
            Vue.set(item, "isEdit", false);
          });
        })
        .catch(error => {
          this.errors = error.response.data.errors.name;
        });
    },
    selectProduct(product) {
      this.selectedProduct = {...product};
      product.isEdit = true;
    },
    UpdateProduct(index) {
      axios
        .put("/products/" + this.selectedProduct.id, {
          name: this.selectedProduct.name,
          price: this.selectedProduct.price
        })
        .then(response => {
          this.list_products[index].name = this.selectedProduct.name;
          this.list_products[index].price = this.selectedProduct.price;
          this.list_products[index].isEdit = false;
        })
        .catch(errors => {
          this.errors = error.response.data.errors.name;
        });
    },
    deleteProduct(prod,index){
        axios.delete('/products/' + prod.id)
        .then(response =>{
            console.log(response.data.result)
            this.list_products.splice(index,1)
        })
        .catch(errors=>{
            this.errors = error.response.data.errors.name
        })
    }
  }
};
</script>

<style lang="scss" scoped>
.error {
  span {
    color: red;
  }
}
</style>