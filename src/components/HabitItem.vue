<template>
  <li class="habit-item" :key="componentKey">
    <div class="habit-item__data">
      <span class="habit-item__name">{{ habit.name }}</span>
      <span class="habit-item__description">{{ habit.description }}</span>
    </div>

    <div class="days-list">
      <span v-for="day in daysList"
            class="days-list__item"
            :key="day.id"
            @click="setData(day)"
            :style="{'background-color': day.color, 'border': day.color ? '1px solid transparent' :'1px solid #89ccc5'}">
      </span>
    </div>
    <div class="habit-item__button-group">
      <v-btn icon @click="changeHabit">
        <v-icon>mdi-pencil</v-icon>
      </v-btn>
      <v-btn icon @click="deleteHabit(habit.id)">
        <v-icon>mdi-delete</v-icon>
      </v-btn>
    </div>
  </li>
</template>

<script>
import lists from "/lists.json";

export default {
  name: "HabitItem",
  props: {
    habit: Object
  },
  data() {
    return {
      componentKey: 0,
      selected: [],
      colorsValue: []
    }
  },
  computed: {
    progressData() {
      return this.$store.getters.getProgressData;
    },
    habitsList() {
      return this.$store.getters.getHabitsList;
    },
    daysList() {
      const list = new Array(this.habit.period).fill({color: ""});
      const daysArray = list.map(item => Object.assign({}, item));
      daysArray.forEach(item => {
        item.id = Math.random() * list.length;
      });
      return daysArray;
    },
  },
  methods: {
    getMoodValue() {
      const colors = lists.moodList.map(item => Object.assign({}, item));
      this.daysList.forEach(item => {
        if (item.color !== "") {
          colors.find(el => el.color === item.color).value += 1;
        }
      })
      this.colorsValue = colors;
    },
    getColorValueArray(array) {
      const checkedColorAmount = array.reduce((acc, curr) => acc + curr.value, 0);
      return array.map(item => {
        Object.assign({}, item);
        if (item.value !== 0) {
          item.value = Math.round(item.value / checkedColorAmount * 100);
        }
        return item.value;
      });
    },
    getProgressValue() {
      const checkedDays = this.daysList.slice().filter(item => item.color !== "");
      if (checkedDays.length > 0) {
        const progressValue = Math.round(checkedDays.length / this.habit.period * 100);
        const progressItem = {
          name: this.habit.name,
          value: progressValue,
          key: this.habit.id,
          colorsValue: this.getColorValueArray(this.colorsValue),
          completedDays: checkedDays.length,
          period: this.habit.period
        };

        if (this.progressData.every(item => item.name !== progressItem.name)) {
          this.progressData.push(progressItem);
        } else {
          this.progressData.forEach(item => {
            // если поменяется имя
            if (item.name === progressItem.name) {
              //добавить цикл
              item.colorsValue = progressItem.colorsValue;
              item.value = progressItem.value;
              item.completedDays = progressItem.completedDays;
              item.period = progressItem.period;
            }
          });
        }
      }
    },
    setData(item) {
      const el = this.daysList.find(el => el.id === item.id);
      if (el.color === "") {
        el.color = this.$store.getters.getColorMood;
      } else {
        el.color = "";
      }
      this.componentKey += 1;
      this.getProgressValue();
      this.getMoodValue();
    },
    changeHabit() {
      this.$store.commit("setChangeableHabit", this.habit);
      this.$router.push({name: "createHabit"});
    },
    deleteHabit(habitId) {
      const filterList = this.habitsList.filter(item => item.id !== habitId);
      this.$store.commit("setHabitsList", filterList);
    }
  }
}
</script>

<style scoped lang="scss">
.habit-item {
  display: flex;
  flex-direction: row;
  align-items: center;
  margin-bottom: 35px;
  color: #000;
  align-self: flex-start;

  &__data {
    width: 200px;
    max-width: 200px;
    display: flex;
    flex-direction: column;
    font-family: "Montserrat-Regular", Arial, sans-serif;
    font-size: 18px;
    line-height: 18px;
  }

  &__name {
    margin-bottom: 10px;
  }

  &__description {
    font-size: 14px;
  }
}

.days-list {
  display: flex;
  flex-wrap: wrap;
  width: 320px;

  &__item {
    width: 35px;
    height: 35px;
    border: 1px solid #89ccc5;
    text-align: center;
    vertical-align: middle;
    margin-bottom: 20px;
    border-radius: 50%;
    cursor: pointer;
    display: inline-block;
    padding-top: 8px;
    margin-right: 10px;
  }
}
</style>