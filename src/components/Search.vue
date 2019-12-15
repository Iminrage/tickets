<template>
  <form class="form" @submit.prevent="submit">
    <div class="form__wrapper">
      <div class="form__input-wrapper">
        <span v-if="$v.dept.$error" class="form__error">Это поле обязательно!</span>
        <input class="form__input" placeholder="Откуда" type="text" v-model.trim="$v.dept.$model" />
      </div>
      <div class="form__input-wrapper">
        <span v-if="$v.dest.$error" class="form__error">Это поле обязательно!</span>
        <input class="form__input" placeholder="Куда" type="text" v-model.trim="$v.dest.$model" />
        <button class="form__swap" type="button" @click="swap"></button>
      </div>
      <div class="form__date-wrapper">
        <datepicker placeholder="Туда" v-model="date" :language="ru" :disabledDates="disabledDates"></datepicker>
        <datepicker
          placeholder="Назад"
          v-model="dateBack"
          :language="ru"
          :disabledDates="disabledDates"
        ></datepicker>
      </div>
      <button class="button form__button" type="submit">Найти билеты</button>
    </div>
  </form>
</template>

<script>
import Datepicker from "vuejs-datepicker";
import { ru } from "vuejs-datepicker/dist/locale";
import { required } from "vuelidate/lib/validators";
export default {
  name: "Search",
  components: {
    Datepicker
  },
  data() {
    return {
      dept: "",
      dest: "",
      date: null,
      dateBack: null,
      tickets: [],
      disabledDates: {
        customPredictor: function(date) {
          if (date < new Date(Date.now() - 86400000)) {
            return true;
          }
          if (date > new Date(Date.now() + 31536000000)) {
            return true;
          }
        }
      },
      ru
    };
  },
  validations: {
    dept: {
      required
    },
    dest: {
      required
    }
  },
  methods: {
    swap() {
      let tmp = this.dept;
      this.dept = this.dest;
      this.dest = tmp;
    },
    async submit() {
      this.$v.$touch();
      if (!this.$v.$invalid) {
        this.tickets = await [];
        let searchId = await this.getId();
        await this.getTickets(searchId);
        await this.sortEmAll();
      }
    },
    async getId() {
      let searchId;
      await (async () => {
        let promise = await fetch(
          "https://front-test.beta.aviasales.ru/search"
        );
        if (await promise.ok) {
          let json = await promise.json();
          searchId = await json.searchId;
        }
      })();
      return searchId;
    },
    async getTickets(searchId) {
      let newPromise = await fetch(
        `https://front-test.beta.aviasales.ru/tickets?searchId=${searchId}`
      );
      if (await newPromise.ok) {
        let newJson = await newPromise.json();
        this.tickets = this.tickets.concat(newJson.tickets);
        if (!newJson.stop) {
          this.getTickets(searchId);
        }
      } else if ((await newPromise.status) > 499) {
        this.getTickets(searchId);
      }
    },
    async getIata(search) {
      let promise = await fetch(
        `http://autocomplete.travelpayouts.com/places2?term=${search}&locale=ru&types[]=country&types[]=city`
      );
      if (await promise.ok) {
        let json = await promise.json();
        if (json.length === 0) {
          return "";
        } else {
          return json[0].code;
        }
      }
    },
    async sortEmAll() {
      let dept = await this.getIata(this.dept);
      let dest = await this.getIata(this.dest);
      let sorted;
      let response;
      if (dept.length && dest.length) {
        if (this.date || this.dateBack) {
          if (this.date && this.dateBack) {
            let toDate = `${this.date.getMonth()}-${this.date.getDate()}`;
            let bDate = `${this.dateBack.getMonth()}-${this.dateBack.getDate()}`;
            sorted = await this.tickets.filter(
              ticket =>
                ticket.segments[0].origin === dept &&
                ticket.segments[0].destination === dest &&
                `${new Date(ticket.segments[0].date).getMonth()}-${new Date(
                  ticket.segments[0].date
                ).getDate()}` === toDate &&
                `${new Date(ticket.segments[1].date).getMonth()}-${new Date(
                  ticket.segments[1].date
                ).getDate()}` === bDate
            );
          } else if (this.date) {
            let toDate = `${this.date.getMonth()}-${this.date.getDate()}`;
            sorted = await this.tickets.filter(
              ticket =>
                ticket.segments[0].origin === dept &&
                ticket.segments[0].destination === dest &&
                `${new Date(ticket.segments[0].date).getMonth()}-${new Date(
                  ticket.segments[0].date
                ).getDate()}` === toDate
            );
          } else if (this.dateBack) {
            let bDate = `${this.dateBack.getMonth()}-${this.dateBack.getDate()}`;
            sorted = await this.tickets.filter(
              ticket =>
                ticket.segments[0].origin === dept &&
                ticket.segments[0].destination === dest &&
                `${new Date(ticket.segments[1].date).getMonth()}-${new Date(
                  ticket.segments[1].date
                ).getDate()}` === bDate
            );
          }
        } else {
          sorted = await this.tickets.filter(
            ticket =>
              ticket.segments[0].origin === dept &&
              ticket.segments[0].destination === dest
          );
        }
        response = sorted.length > 0 ? true : false;
      } else {
        response = false;
      }
      this.$emit("sorted", sorted, response);
    }
  }
};
</script>