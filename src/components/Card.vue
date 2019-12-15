<template>
  <li class="cards__card">
    <div class="cards__wrapper">
      <span class="cards__descr">Туда <b>{{ date(0) }}</b></span>
      <span class="cards__descr">Назад <b>{{ date(1) }}</b></span>
    </div>
    <div class="cards__wrapper">
      <button class="button button--buy"><span class="button__small">Купить за</span> <br/><b class="button__price">{{ price(item.price) }} ₽</b></button>
      <img :src="`//pics.avs.io/99/36/${item.carrier}.png`" alt />
    </div>
  </li>
</template>

<script>
export default {
  name: "Card",
  props: {
    item: Object
  },
  methods: {
    price(x) {
      return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ");
    },
    date(route) {
			let date = new Date(this.item.segments[route].date);		
      let pDay = date.getDate();
      let day = pDay.toString().length > 1 ? pDay : "0" + pDay;
      let pMonth = date.getMonth() + 1;
      let month = pMonth.toString().length > 1 ? pMonth : "0" + pMonth;
      let year = date.getFullYear();

      return `${day}.${month}.${year}`;
    }
  }
};
</script>