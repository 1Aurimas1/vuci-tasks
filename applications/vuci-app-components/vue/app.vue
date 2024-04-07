<template>
  <div class="components">
    <vuci-form :uci-config="config" :key="interfaceListKey">
      <vuci-typed-section :type="type" :columns="columns" :collapsible="false">
        <template #interfacename="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="interfacename" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="address" />
        </template>
        <template #actions="{ s }">
          <a-space>
            <a-button type="primary" @click="editInterface(s)"> Edit </a-button>
            <a-popconfirm
              title="Are you sure you want delete this interface?"
              ok-text="Yes"
              cancel-text="No"
              @confirm="deleteInterface(s)"
            >
              <a-button type="danger"> Delete </a-button>
            </a-popconfirm>
          </a-space>
        </template>
      </vuci-typed-section>

      <template #footer>
        <a-row :gutter="50">
          <a-col :span="12">
            <a-input
              v-model="form.interfaceName"
              addonBefore="Interface name"
            ></a-input>
          </a-col>
          <a-col :span="2">
            <a-button type="primary" @click="addInterface">Create</a-button>
          </a-col>
        </a-row>
      </template>
    </vuci-form>

    <Modal
      :config="config"
      :selectedSection="selectedSection"
      :modal="modal"
      @applied="onApplied"
    />
  </div>
</template>

<script>
import Modal from './Modal.vue'

const config = 'vuci_components_task'
const type = 'interface'

const columns = [
  { name: 'interfacename', label: 'Interface name' },
  { name: 'address', label: 'Address' },
  { name: 'actions', label: '' }
]

export default {
  components: {
    Modal
  },
  data () {
    return {
      config,
      type,
      columns,
      form: {
        interfaceName: ''
      },
      modal: {
        isVisible: false,
        name: ''
      },
      selectedSection: '',
      interfaceListKey: 0
    }
  },
  methods: {
    addInterface () {
      this.modal = {
        isVisible: true,
        name: this.form.interfaceName
      }

      const sid = this.$uci.add(this.config, this.type)
      this.$uci.set(this.config, sid, 'interfacename', this.form.interfaceName)

      this.selectedSection = sid
    },
    async deleteInterface (section) {
      this.$uci.del(this.config, section['.name'])
      await this.$uci.save()
      await this.$uci.apply()

      this.interfaceListKey++
    },
    editInterface (section) {
      this.modal = {
        isVisible: true,
        name: section.interfaceName
      }

      this.selectedSection = section['.name']
    },
    onApplied () {
      this.interfaceListKey++
    }
  },
  watch: {
    'modal.isVisible': function (newVal) {
      if (!newVal) {
        this.modal.name = ''
        this.form.interfaceName = ''
      }
    }
  }
}
</script>
