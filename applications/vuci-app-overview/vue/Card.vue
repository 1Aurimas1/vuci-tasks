<template>
  <a-card
    :title="titleUppercase"
    size="small"
    :style="{ width: '270px', height: '300px' }"
  >
    <div slot="extra" class="header">
      <div v-for="hS in headerSections" :key="hS.title">
        <div v-if="hS.type === 'status'">
          <StatusBar :label="hS.title" :value="hS.value" />
        </div>
        <div v-else>
          {{ s.value }}
        </div>
      </div>
    </div>
    <div v-for="(bS, idx1) in bodySections" :key="idx1">
      <Section :section="bS" />
      <a-divider v-if="idx1 < bodySections.length - 1" />
    </div>
  </a-card>
</template>

<script>
import StatusBar from "./StatusBar.vue";
import Section from "./Section.vue";

export default {
  components: { StatusBar, Section },
  props: {
    title: String,
    sections: Array,
  },
  computed: {
    titleUppercase() {
      return this.title.toUpperCase() || "Unnamed";
    },
    headerSections() {
      if (this.sections) {
        return this.sections.filter((s) => s.position === "header");
      }
      return [];
    },
    bodySections() {
      if (this.sections) {
        return this.sections.filter((s) => s.position === "body");
      }
      return [];
    },
  },
};
</script>

<style scoped>
.ant-divider-horizontal {
  margin-top: 8px;
  margin-bottom: 8px;
}
</style>
