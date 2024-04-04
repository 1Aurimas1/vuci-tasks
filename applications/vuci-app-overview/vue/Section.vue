<template>
  <div>
    <strong>{{ section.title }}</strong>
    <div v-if="section.type == 'status'">
      <a-row type="flex" :gutter="16" v-if="Array.isArray(section.value)">
        <a-col v-for="(subSection, idx) in section.value" :key="idx">
          <StatusBar :label="subSection.title" :value="subSection.value" />
        </a-col>
      </a-row>
      <div v-else>
        <StatusBar :label="section.title" :value="section.value" />
      </div>
    </div>
    <div v-else>
      {{ truncateMessage(section.value) }}
    </div>
  </div>
</template>

<script>
import StatusBar from "./StatusBar.vue";

export default {
  components: {
    StatusBar,
  },
  props: {
    section: Object,
  },
  methods: {
    truncateMessage(message) {
      if (!message) return "Not found";

      const limit = 35;
      if (message.length > limit) {
        return message.slice(0, limit) + "...";
      }
      return message;
    },
  },
};
</script>
