<template>
  <section>
    <breadcrumbs-component :items="path" />
    <CategoriesNavigation class="class='column is-2 is-pulled-left is-desktop is-narrow'" :id='category.id'></CategoriesNavigation>
    <div class="section">
      <div class="card is-clearfix columns">
        <div class="columns column is-one-thirds is-multiline">
          <figure class="card-image is-480x480 column is-full">
            <img src="https://bulma.io/images/placeholders/480x480.png">
          </figure>
            <div class="card-image is-480x480 column is-full">
              <product-detail-images-component v-bind:product-id="product.id"></product-detail-images-component>
           </div>
          </div>
          <div class="card-content column is-two-thirds">
            <div class="card-content__title">
              <h2 class="title is-4">{{ product.title }}
                <button class="button is-small" :title="removeFromFavouriteLabel" v-show="product.isFavourite" @click="removeFromFavourite(product.id)">
                  <span class="icon is-small">
                    <i class="fas fa-heart"></i>
                  </span>
                </button>
                <button class="button is-small" :title="addToFavouriteLabel" v-show="!product.isFavourite" @click="saveToFavorite(product.id)">
                  <span class="icon is-small">
                    <i class="far fa-heart"></i>
                  </span>
                </button>
              </h2>
            </div>
            <div class="card-content__text">
              <p>
              Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
              Ut enim ad minim veniam, quis nostrud
              </p>
            </div>
            <div class="card-content__ratings">
              <b-rate v-model="productRating"
                      :icon-pack="packs"
                      :disabled="isDisabled"
                      size="is-small">
              </b-rate>
            </div>
            <div class="card-content__reviews">
              <div class="is-pulled-left">
                <p>
                  <strong>
                    <router-link :to="{
                      name: 'product-reviews-component',
                      params: {
                        id: product.id
                      }
                    }">
                      {{ textCountReviews }}
                    </router-link>
                  </strong>
                </p>
              </div>
              <div class="select is-rounded is-small is-pulled-right">
                <select @change="onSelectQuantity(product.id)" v-model="selected">
                  <option v-for="quantity in quantityArray" :value="quantity">{{ quantity }}</option>
                </select>
              </div>
            </div>
            <div class="card-content__price is-pulled-left">
              <span class="title is-3"><strong>{{ product.price }}&euro;</strong></span>
            </div>
            <div class="card-content__btn is-pulled-right">
              <button class="button is-primary" v-if="!isAddedBtn" @click="addToCart(product.id)">{{ addToCartLabel }}</button>
              <button class="button is-text" v-if="isAddedBtn" @click="removeFromCart(product.id)">{{ removeFromCartLabel }}</button>
            </div>
        </div>
      </div> 
      <div >
        <p class="title is-4" style="padding-left: 120px" > Similar products:</p>
         <OtherProductComponent /> </div>
     </div>
   </section>
</template>

<script>
import BreadcrumbsComponent from '../Breadcrumbs'
import ProductDetailImagesComponent from './ProductDetailImages'
import OtherProductComponent from '../other_product/OtherProduct'
import CategoriesNavigation from '../categories_nav/CategoriesNavigation';

export default {
  name: 'product-detail-component',
  components: {
    BreadcrumbsComponent,
    ProductDetailImagesComponent,
    OtherProductComponent,
    CategoriesNavigation,
  },
  data () {
    return {
      addToCartLabel: 'Add to cart',
      removeFromCartLabel: 'Remove from cart',
      addToFavouriteLabel: 'Add to favourite',
      removeFromFavouriteLabel: 'Remove from favourite',
      product: {},
      selected: 1,
      quantityArray: [],
      packs: 'fas',
      icons: 'star',
      isDisabled: true
    };
  },

  created () {
    this.product = this.$store.getters.getProductById(this.$route.params.id);
    this.selected = this.product.quantity;

    for (let i = 1; i <= 20; i++) {
      this.quantityArray.push(i);
    }
  },

  computed: {
    isAddedBtn () {
      return this.product.isAddedBtn;
    },
    category () {
      return this.$store.getters.getCategoryById(this.product.category);
    },
    path () {
      this.product = this.$store.getters.getProductById(this.$route.params.id);
      return [
        {
          text: 'Home',
          to: { path: '/' }
        },
        {
          text: this.category ? this.category.title : '',
          to: { 
            name: 'category-products-component', 
            params: {
              id: this.category ? this.category.id : 0
            } 
          }
        },
        {
          text: this.product.title,
        }
      ]
    },
    productRating () {
      return this.$store.getters.getOverallRatingProductById(this.$route.params.id);
    },
    textCountReviews () {
      const count = this.$store.getters.getCountReviewsById(this.product.id);
      return count > 0 ? `${count} Reviews` : 'No product reviews';
    },
  },

  methods: {
    addToCart (id) {
      let data = {
        id: id,
        status: true
      }
      this.$store.commit('addToCart', id);
      this.$store.commit('setAddedBtn', data);
    },
    removeFromCart (id) {
      let data = {
        id: id,
        status: false
      }
      this.$store.commit('removeFromCart', id);
      this.$store.commit('setAddedBtn', data);
    },
    onSelectQuantity (id) {
      let data = {
        id: id,
        quantity: this.selected
      }
      this.$store.commit('quantity', data);
    },
    saveToFavorite (id) {
      let isUserLogged = this.$store.state.userInfo.isLoggedIn;

      if (isUserLogged) {
        this.$store.commit('addToFavourite', id);
      } else {
        this.$store.commit('showLoginModal', true);
      }
    },
    removeFromFavourite (id) {
      this.$store.commit('removeFromFavourite', id);
    }
  }
};
</script>

<style lang="scss" scoped>
  .card-content {
    padding: 15px 10px 15px 0;

    &__text {
      margin: 15px 0;
    }
    &__reviews {
      display: inline-block;
      width: 100%;
      margin-bottom: 10px;
    }
  }
</style>

