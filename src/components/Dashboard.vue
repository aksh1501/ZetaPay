<template>
  <div class='dashboard'>
    <div v-if='!showProdForm'>
      <ul class='heading'>
        <li>
          <img class='logo' :src='logoImg' />
        </li>
        <li class='page'>{{ pageInfo }}</li>
      </ul>
      <ul class='product'>
        <li class='product-text'><strong>Products</strong></li>
        <li>
          <button @click='toggleShowForm' class='create-product-button'>
            +Create Product
          </button>
        </li>
        <li>
          An overview of all your products available in Authentication Centre
        </li>
      </ul>
      <ul class='all-product-container'>
        <li
          v-for='product in productList'
          :key='product.name'
          :id='product.name' :data-product-id="product.id"
          class='product-container'
        ></li>
      </ul>
    </div>

    <div v-if='showProdForm'>
      <ul class='heading'>
        <li>
          <img class='logo' :src='logoImg' />
        </li>
        <li class='page'>{{ pageInfo }}</li>
      </ul>
      <ul class='product'>
        <li class='product-text'><strong>Create Product</strong></li>
        <li>Create a new Product:</li>
      </ul>

      <div>
        <h2>Basic Information</h2>
        <h3>Enter the Basic Details about your product</h3>
      </div>

      <form class='new-product-form' @submit.prevent='onSubmit'>
        <div class='error' v-if='errors.length'>
          <b>Please correct the following error(s):</b>
          <ul>
            <li v-for='(error, idx) in errors' :key='idx'>{{ error }}</li>
          </ul>
        </div>

        <p>
          <label for='name'>Product Name:</label><br />
          <input
            id='name'
            placeholder='Enter Product Name'
            v-model='newProduct.name'
          />
        </p>

        <p>
          <label for='description'>Description:</label><br />
          <textarea
            id='description'
            placeholder='Enter Product Description'
            v-model='newProduct.description'
          ></textarea>
        </p>

        <p>
          <label for='keyBundleId'>Key Bundle ID</label><br />
          <select id='keyBundleId'>
            <option value='Select Key Bundle ID'>Select Key Bundle ID</option>
          </select>
        </p>

        <p>
          <label for='cardNetwork'>Card Network</label><br />
          <select id='cardNetwork' v-model='newProduct.cardNetwork'>
            <option v-for='(card, idx) in cards' :key='idx' :value='card'>
              {{ card }}
            </option>
          </select>
        </p>

        <p>
          <label for='protocol'>Protocol Version</label><br />
          <select id='protocol' v-model='newProduct.version'>
            <option value="threeDSecure_1_0">3D Secure 1.0</option>
            <option value="threeDSecure_2_0">3D Secure 2.0</option>
            <option value="threeDSecure_3_0">2D Secure 3.0</option>
          </select>
        </p>

        <p>
          <label for='avvAlgorithm'>AVV Algorithm</label><br />
          <select id='avvAlgorithm' placeholder='Select AVV algorithm'>
            <option value='AVV Algorithm 1'>AVV Algorithm 1</option>
            <option value='AVV Algorithm 2'>AVV Algorithm 2</option>
            <option value='AVV Algorithm 3'>AVV Algorithm 3</option>
          </select>
        </p>

        <p>
          <label for='binNo'>Bin No.</label> <br />
          <input id='binNo' v-model='newProduct.bin' />
        </p>

        <p>
          <label for='prodId'>Product ID.</label><br />
          <input
            id='prodId'
            placeholder='Enter Product ID'
            v-model='newProduct.id'
          />
        </p>

        <p>
          <input class='form-buttons' type='submit' value='Submit' />
          <button class='form-buttons' @click='onCancel'>Cancel</button>
        </p>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Dashboard',
  data () {
    return {
      zeta: 'zeta ',
      pageInfo: 'Cipher - Authentication Centre ',
      logoImg: require('../assets/zeta.jpeg'),
      showProdForm: false,
      errors: [],
      prodCount: 0,
      productFromJson: require('@/assets/product.json'),
      newProduct: {
        id: null,
        name: null,
        bin: null,
        cardNetwork: null,
        config: {
          connectorURL:
            'https://ciphertest.amex-cipher.gw.zetapay.in//amexcipher/vereq'
        },
        description: null,
        version: null,
        authPlans: ['swipe_to_pay', 'super_pin']
      }
    }
  },
  computed: {
    productInfo () {
      if (localStorage.getItem('storedProducts')) {
        return JSON.parse(localStorage.getItem('storedProducts'))
      } else {
        localStorage.setItem(
          'storedProducts',
          JSON.stringify(this.productFromJson)
        )
        return this.productFromJson
      }
    },
    productList () {
      return this.productInfo.products
    },
    imageMap () {
      return this.productInfo.cardNetworkLogos
    },
    cards () {
      return Object.keys(this.imageMap)
    }
  },
  methods: {
    toggleShowForm () {
      this.showProdForm = !this.showProdForm
    },
    createImage (product) {
      const card = product.cardNetwork
      const url = this.imageMap[card].logoURL

      fetch(url).then(response => {
        response.blob().then(data => {
          const li = document.getElementById(product.name)
          const div = document.createElement('div')
          const enclosingDiv = document.createElement('div')
          const h1 = document.createElement('h1')
          const h2 = document.createElement('h3')
          const h3 = document.createElement('h3')
          h2.setAttribute('style', 'background-color:lightgrey; width:120px;padding:5px;')
          h1.setAttribute('style', 'padding:5px;')
          h3.setAttribute('style', 'padding:5px;')
          h1.innerText = 'Name: ' + product.name
          h2.innerText = 'ID:' + product.id
          h3.innerText = 'Bin-' + product.bin
          div.setAttribute('class', 'product-container-div')
          enclosingDiv.setAttribute(
            'style',
            'background-color:' +
              this.imageMap[product.cardNetwork].logoBgColor +
              '; height : 150px;'
          )
          li.append(div)
          div.append(enclosingDiv)
          const img = document.createElement('img')
          img.style = 'position: relative; width:150px; float : left;'
          img.src = URL.createObjectURL(data)
          enclosingDiv.append(img)
          div.append(h1)
          div.append(h2)
          div.append(h3)
        })
      })
    },
    onSubmit () {
      if (
        this.newProduct.name &&
        this.newProduct.bin &&
        this.newProduct.id &&
        this.newProduct.cardNetwork &&
        this.newProduct.description &&
        this.newProduct.version
      ) {
        console.log('hi if')
        this.prodCount++
        this.updateProductList(this.newProduct)
        this.newProduct = {
          id: null,
          name: null,
          bin: null,
          cardNetwork: null,
          config: {
            connectorURL:
              'https://ciphertest.amex-cipher.gw.zetapay.in//amexcipher/vereq'
          },
          description: null,
          version: null,
          authPlans: ['swipe_to_pay', 'super_pin']
        }

        this.toggleShowForm()
      } else {
        if (!this.newProduct.name) {
          this.errors.push('Product Name required.')
        }
        if (!this.newProduct.bin) {
          this.errors.push('Product Bin required.')
        }
        if (!this.newProduct.id) {
          this.errors.push('Product Id required.')
        }
        if (!this.newProduct.cardNetwork) {
          this.errors.push('Product CardNetwork required.')
        }
        if (!this.newProduct.description) {
          this.errors.push('Product Description required.')
        }
        if (!this.newProduct.version) {
          this.errors.push('Product version required.')
        }
      }
    },
    updateProductList (product) {
      if (localStorage.getItem('storedProducts')) {
        const productData = JSON.parse(localStorage.getItem('storedProducts'))
        console.log(productData.products)
        productData.products.push(product)
        localStorage.setItem('storedProducts', JSON.stringify(productData))
      } else {
        console.log('no object in local storage')
      }
      location.reload()
    },
    onCancel () {
      this.toggleShowForm()
      location.reload()
    },
    handleClick (e) {
      const li = e.target.closest('.product-container')
      const prodID = li.getAttribute('data-product-id')
      this.$router.push({ name: 'details', params: { Pid: prodID } })
    }
  },
  mounted: function () {
    this.prodCount = this.productList.length
    for (let iter = 0; iter < this.productList.length; iter++) {
      this.createImage(this.productList[iter])
    }
    const allProductContainer = document.getElementsByClassName('all-product-container')[0]
    allProductContainer.addEventListener('click', this.handleClick)
  }
}
</script>

<style scoped>
* {
  box-sizing: border-box; /*include paddding and border in height and width calculation to make sizing easier*/
  text-align: left;
}

.heading {
  color: white;
  background-color: red;
  opacity: 0.7;
  list-style-type: none;
  margin: 0px 0px 10px 0px;
}

.page {
  color: black;
  font-family: 'Nunito Sans', sans-serif;
  font-size: '4 rem';
}

.logo {
  margin-right: 10px;
  width: 80px;
  vertical-align: middle;
}
.product {
  padding-left: 20px;
  list-style-type: none;
  background-color: rgb(29, 118, 235);
  opacity: 0.7;
  color: white;
  height: 100px;
}

.product-text {
  color: red;
  font-family: 'Fira Sans', sans-serif;
  font-size: 30px;
  padding: 10px 20px 0px 0px;
  margin: 10px;
  /* padding-left: 20px; */
}

.create-product-button {
  float: right;
  color: white;
  height: 40px;
  background-color: rgb(55, 230, 128);
  margin-right: 10px;
}

.product-container {
  height: 400px;
  /* text-align: left; */
  margin-bottom: 20px;
  margin-top: 5px;
  cursor: pointer;
  width: 400px;
  /* border:1px solid lightgrey; */
  z-index: 4;
  box-shadow: 2px 2px 2px 2px grey;
  background-color: white;
}

.product-container:hover {
  background-color: yellow;
}

.all-product-container {
  display: flex;
  flex-flow: row wrap;
  justify-content: space-around;
  list-style: none;
  /* background-color: rgb(179, 37, 162); */
  box-shadow: 2px 2px 2px 2px grey;
  background-color: purple;
}

.product-container-div {
  position: relative;
  display: inline-block;
  height: 400px;
  margin-bottom: 20px;
}

.new-product-form {
  padding: 20px;
  margin: 40px;
  border: 2px solid #d8d8d8;
}

textarea {
  width: 100%;
  height: 200px;
}
input {
  width: 20%;
}

.form-buttons {
  width: 70px;
  background-color: lightblue;
  margin: 10px;
  padding: 10px;
}
</style>
