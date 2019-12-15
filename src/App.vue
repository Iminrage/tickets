<template>
  <div class="app">
    <div class="content-wrapper">
      <header class="header">
        <div class="container">
          <div class="header__logo">
            <img
              class="header__img"
              src="@/assets/img/logo.png"
              alt="Logo"
              height="auto"
              width="80"
            />
            <a href="#" class="header__link">TicketSearch!</a>
          </div>
        </div>
      </header>
      <main class="main">
        <section class="search">
          <Search class="search__form" @sorted="getTickets"></Search>
        </section>
        <section class="cards container">
          <div class="cards__not-found" v-if="!found">
            <p>Таких билетов к сожалению нет, попробуй другой запрос</p>
            <img src="@/assets/img/sad.png" alt="Очень жаль" width="256" height="auto" />
          </div>
          <ul class="cards__list">
            <Card v-for="(item, index) in sortedTickets" :item="item" :key="index"></Card>
          </ul>
          <button
            class="button cards__button"
            :class="{'button--warn': outOfTickets}"
            v-if="sortedTickets.length"
            @click="getTickets(sorted, true, count+20)"
          >{{ buttonMsg }}</button>
        </section>
      </main>
      <footer class="footer">
        <div class="container">
          <div class="footer__wrapper">
            <span class="footer__item">© Шевчук Е.А. и Ананьев Д.С. ЗФО 09.03.03 2019</span>
            <span class="footer__item">МГУТУ им. К. Г. Разумовского</span>
          </div>
        </div>
      </footer>
    </div>
  </div>
</template>

<script>
import Card from "./components/Card.vue";
import Search from "./components/Search.vue";
export default {
  name: "app",
  components: {
    Card,
    Search
  },
  data() {
    return {
      sortedTickets: [],
      count: 0,
      sorted: [],
      buttonMsg: "Показать еще",
      outOfTickets: false,
      found: true
    };
  },
  methods: {
    getTickets(sorted, response, count = 20) {
      if (response) {
        if (count < sorted.length) {
          this.sortedTickets = sorted.slice(0, count);
          this.count = count;
          this.sorted = sorted;
        } else {
          this.buttonMsg = "Билетов больше нет";
          this.outOfTickets = true;
        }
        this.found = true;
      } else {
        this.sortedTickets = [];
        this.found = false;
      }
    }
  }
};
</script>

