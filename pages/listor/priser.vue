<template>
  <section class="">
    <h2>Priser för översättare</h2>
    <ul class="resultlist">
        <li v-for="item in items">
            <nuxt-link :to="'/artiklar/' + item.URLName">{{item.ArticleName}}</nuxt-link>
        </li>
    </ul>
  </section>
</template>

<script>

import backend from "assets/backend"


export default {
    name : "Prizes",
    head : {
      title : "Priser för översättare – Svenskt översättarlexikon"
    },
    data() {
      return {
        items: null
      }
    },
    async asyncData ({error, env}) {
      try {
        return {items : await backend.listPrizeArticles()} 
      } catch(err) {
        console.log("err", err)
        error("Ett fel uppstod, vänligen försök igen senare.")
        return {items : null}
      }
    },
}

    
</script>

<style>
</style>
