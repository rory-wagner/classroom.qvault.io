<template>
  <div id="container">
    <LoadingOverlay
      :is-loading="isLoading" 
    />

    <div id="title-container">
      <FontAwesomeIcon
        icon="store"
        class="icon"
      />
      <span @click="$router.push({name: 'CouponCode'})">
        Coupon Code?
      </span>
    </div>

    <div id="sub-title">
      <span>
        Gems unlock courses and certifications
      </span>
    </div>

    <div id="cards">
      <ImageCard
        v-for="(product, i) of products"
        :key="i"
        class="card"
        :img-src="product.ImageURL"
        :click="() => { checkout(product.ID) }"
      >
        <div class="title">
          <span> Get {{ product.GemAmount }} Gems</span>
        </div>
        <div class="price">
          <span>${{ (product.Price.UnitAmount / 100) }}</span>
        </div>
      </ImageCard>
    </div>
  </div>
</template>

<script>
import { loadStripe } from '@stripe/stripe-js';
import 'vue-loading-overlay/dist/vue-loading.css';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';

import { publicKey } from '@/lib/stripeConsts';
import LoadingOverlay from '@/components/LoadingOverlay';
import ImageCard from '@/components/ImageCard';
import {
  startProductCheckout,
  completePayments,
  getLastGemTransaction
} from '@/lib/cloudClient.js';

export default {
  components: {
    ImageCard,
    LoadingOverlay,
    FontAwesomeIcon
  },
  data() {
    return {
      stripeCustomerID: null,
      stripeCustomerEmail: null,
      error: null,
      isLoading: false
    };
  },
  computed: {
    products(){
      let products = this.$store.getters.getProducts;
      return products;
    }
  },
  async mounted(){
    (async () => {
      try {
        await completePayments();
        const lastGemTransaction = await getLastGemTransaction();
        this.$store.commit('setBalance', lastGemTransaction.Balance);
      } catch (err) {
        this.$notify({
          type: 'error',
          text: err
        });
      } 
    })();
  },
  methods: {
    async checkout(productID){
      this.isLoading = true;
      const checkoutSession = await startProductCheckout(productID);
      const stripe = await loadStripe(publicKey);
      const {error} = await stripe.redirectToCheckout({
        sessionId: checkoutSession.id
      });
      this.isLoading = false;
      this.error = error.message;
    }
  }
};
</script>

<style scoped lang="scss">
@import '@/styles/colors.scss';

#container {
  margin: 20px;
  display: block;
  text-align: center;
}

#title-container{
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  margin: 20px;

  .icon {
    font-size: 4em;
    color: $purple-mid;
  }

  span {
    color: $gold-mid;
    cursor: pointer;

    &:hover{
      color: $purple-mid;
      text-decoration: underline;
    }
  }
}

#sub-title {
  text-align: center;
  color: $gray-mid;
  font-size: 1.5em;
  margin-bottom: 20px;
}

#cards {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  width: 100%;

  .card{
    flex: 1 1 calc(22% - 1em);
    margin: 20px;
    max-width: 400px;
    min-width: 200px;

    .title {
      color: $purple-lighter;
      font-size: 2em;
      margin: 1em;

      span {
        margin-left: 10px;
      }
    }

    .price {
      font-size: 24px;
      color: $green-dark;
      margin: 1em;
    }
  }
}

</style>
