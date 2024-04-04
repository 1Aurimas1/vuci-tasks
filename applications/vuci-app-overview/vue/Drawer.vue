<template>
  <a-space direction="vertical">
    <a-drawer
      title="Card config"
      placement="right"
      width="300"
      :closable="false"
      :visible="visible"
      @close="onDrawerClose"
    >
      <vuci-form :uci-config="config">
        <vuci-typed-section
          :type="type"
          :columns="columns"
          :collapsible="false"
        >
          <template #name="{ s }">
            <vuci-form-item-dummy :uci-section="s" name="name" />
          </template>
          <template #visible="{ s }">
            <a-switch
              :default-checked="isSectionVisible(s)"
              @change="(e) => onCardVisibilityChange(e, s)"
            />
          </template>
        </vuci-typed-section>
        <template #footer><div /></template>
      </vuci-form>
    </a-drawer>
  </a-space>
</template>

<script>
const columns = [
  { name: 'name', label: 'Name' },
  { name: 'visible', label: 'Visible' }
]

export default {
  components: {},
  props: {
    config: String,
    type: String
  },
  data () {
    return {
      sections: [],
      columns,
      visible: false
    }
  },
  methods: {
    onDrawerClose () {
      this.visible = false
    },
    isSectionVisible (s) {
      return s.visible === '1'
    },
    async onCardVisibilityChange (newValue, s) {
      const uciValue = newValue ? '1' : '0'
      this.$uci.set(this.config, s['.name'], 'visible', uciValue)
      await this.$uci.save()
      await this.$uci.apply()
      await this.$uci.load(this.config)
      this.$emit('onCardChange', s.name, uciValue)
    }
  }
}
</script>
