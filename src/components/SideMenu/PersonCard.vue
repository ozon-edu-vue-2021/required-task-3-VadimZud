<template>
  <div class="person">
    <div class="person__photo">
      <img :src="person.picture" alt="photo" />
      <div class="button home"><span class="material-icons"> home </span></div>
    </div>
    <div class="person__info">
      <div class="person__info-text">
        <div class="person__info-name">
          <b>{{ person.name }} ({{ person.age }})</b>
        </div>
        <div class="person__info-date">
          <span class="section-title">Дата регистрации:</span>
          {{ formatedDate }}
        </div>
        <div class="person__info-about">
          <span class="section-title">О себе:</span> {{ person.about }}
        </div>
      </div>
      <div class="person__info-buttons">
        <div class="button">
          <a :href="`mailto:${person.email}`">
            <span class="material-icons"> mail </span>
          </a>
        </div>
        <div class="button link">
          <span>КЗ</span>
          <span class="material-icons"> link </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { format } from "date-fns";

export default {
  props: {
    person: {
      type: Object,
      default: null,
    },
  },
  computed: {
    formatedDate() {
      return format(new Date(this.person.registered), "d MMMM Y");
    },
  },
};
</script>

<style scoped>
.person {
  display: grid;
  grid-template-columns: 150px 1fr;
  grid-gap: 8px;
}

.person__photo {
  position: relative;
  height: fit-content;
}

.button {
  border-radius: 8px;
  background-color: #e2ebfd;
  color: #255af6;
  width: fit-content;
  height: 31px;
  padding: 0 3px;
  cursor: pointer;
}

.button span {
  vertical-align: middle;
  line-height: 31px;
}

.home {
  position: absolute;
  bottom: 5px;
  left: 5px;
  background-color: rgba(40, 39, 41, 0.7);
  color: #fff;
}

.person__photo img {
  height: 150px;
  width: 150px;
  display: block;
  border-radius: 8px;
}

.person__info {
  height: 145px;
}

.person__info-date {
  margin-bottom: 5px;
  font-size: 0.8em;
}
.person__info-text {
  height: 114px;
}

.person__info-buttons {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  height: 31px;
}

.link {
  font-weight: bold;
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-gap: 10px;
}

.person__info-name {
  margin-bottom: 5px;
  color: #255af6;
}

.person__info-about {
  overflow: auto;
  height: 60px;
  font-size: 0.6em;
}

.section-title {
  font-weight: bold;
}
</style>
