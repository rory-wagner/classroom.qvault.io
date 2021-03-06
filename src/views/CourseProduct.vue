<template>
  <div id="container">
    <div id="title-container">
      <img
        :src="course.ImageURL"
      >

      <span>
        {{ course.Title }}
      </span>

      <GemDisplay :cost="course.GemCost" />
    </div>

    <div id="sub-title">
      <span>
        {{ course.Description }}
      </span>
    </div>

    <div id="body">
      <div id="modules">
        <span id="title">Included Modules</span>
        <div>
          <div
            v-for="(mod, i) of course.Modules"
            :key="i"
            class="module"
          >
            <span class="title">{{ mod.Title }}</span>
            <p class="description">
              {{ mod.Description }}
            </p>
          </div>
        </div>
      </div>
      <div id="store">
        <ImageCard
          v-if="product"
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
  </div>
</template>

<script>
import ImageCard from '@/components/ImageCard';
import GemDisplay from '@/components/GemDisplay';
import { publicKey } from '@/lib/stripeConsts';

import { loadStripe } from '@stripe/stripe-js';
import { 
  startProductCheckout
} from '@/lib/cloudClient.js';

export default {
  components: {
    ImageCard,
    GemDisplay
  },
  computed: {
    course(){
      const theCourse = this.$store.getters.getCourses.find(
        course => course.UUID === this.$route.params.courseUUID
      );
      if (!theCourse){
        return {};
      }
      return theCourse;
    },
    product(){
      let products =  this.$store.getters.getProducts;
      const balance = this.$store.getters.getBalance;

      products = products.filter(product => product.GemAmount >= this.course.GemCost - balance);
      if (products.length === 0){
        products = products.filter(product => product.GemAmount >= this.course.GemCost);
      }

      products.sort((p1, p2) => p1.GemAmount > p2.GemAmount ? 1 : -1);

      return products[0];
    }
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
  padding: 20px;
  display: block;
  text-align: center;
}

#title-container{
  width: 100%;
  margin: 20px;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  align-items: center;

  span {
    color: $gray-darker;
    font-size: 3em;
  }

  img {
    height: 75px;
  }
}

#sub-title {
  text-align: center;
  color: $gray-mid;
  font-size: 1.2em;
  margin-bottom: 20px;
}

#body {
  width: 100%;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  align-items: flex-start;
  color: $gray-darker;
  padding: 40px 0 40px 0;

  #modules {
    flex: 1;

    #title {
      font-size: 2.5em;
      color: $gold-mid;
    }

    .module {
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      margin: 15px;

      span {
        display: block;
      }

      .title {
        font-size: 1.5em;
        color: $gold-mid;
      }

      .description {
        font-size: 1em;
        text-align: left;
      }
    }
  }

  #store {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;

    .card{
      max-width: 400px;
      min-width: 200px;

      .title {
        color: $purple-lighter;
        font-size: 2em;
        margin: 1em;
      }

      .price {
        font-size: 24px;
        color: $green-dark;
        margin: 1em;
      }
    }
  }
}

</style>
