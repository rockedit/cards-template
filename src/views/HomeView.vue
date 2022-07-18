<script lang="ts">
  import CardTemplate from '@/components/CardTemplate.vue'

  import {defineComponent, ref} from 'vue'

  export default defineComponent({
    components: {
      CardTemplate
    },
    setup() {
      let cardScheme: object = ref(null)
      let cardData: object = ref(null)

      return {cardScheme, cardData}
    },

    async mounted() {
      try {
        //todo: temp solution to get mock data
        const schemeResponse = await this.axios.get('/data/scheme.json');
        this.cardScheme = schemeResponse.data;

        let dataResponse = await this.axios.get('/data/example.json');
        this.cardData = dataResponse.data;
      } catch (e) {
        //todo: put data to sentry ?
      }
    }
  })
</script>

<template>
  <main>
    <div class="cards">
      <CardTemplate v-if="cardScheme && cardData" :scheme="cardScheme" :initial-data="cardData"/>
    </div>
  </main>
</template>

<style>
  .cards {
    display: flex;
  }

  .cards .card {
      margin: 20px;
  }
</style>