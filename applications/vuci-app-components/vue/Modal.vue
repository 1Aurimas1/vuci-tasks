<template>
  <a-modal
    v-model="modal.isVisible"
    :title="`Interface ${modal.name}`"
    :destroyOnClose="true"
  >
    <vuci-form ref="editForm" :uci-config="config" @applied="onApplied">
      <vuci-named-section
        :name="selectedSection"
        v-slot="{ s }"
        :card="false"
        @change-proto="onProtocolChange"
      >
        <vuci-form-item-select
          initial="static"
          label="Protocol"
          name="proto"
          :options="protocols"
          :uci-section="s"
        />
        <vuci-form-item-input
          :uci-section="s"
          label="Address"
          name="address"
          :depend="protocolDependency"
          rules="ipaddr"
        />
        <vuci-form-item-input
          :uci-section="s"
          label="Netmask"
          name="netmask"
          :depend="protocolDependency"
          rules="netmask4"
        />
        <vuci-form-item-input
          :uci-section="s"
          label="Gateway"
          name="gateway"
          :depend="protocolDependency"
          rules="ipaddr"
        />
        <vuci-form-item-list
          :uci-section="s"
          label="DNS"
          name="dns"
          :depend="protocolDependency"
        ></vuci-form-item-list>
      </vuci-named-section>
      <template #footer>
        <a-row type="flex" justify="end" :gutter="20">
          <a-col>
            <a-button type="primary" @click="save">Save</a-button>
          </a-col>
          <a-col>
            <a-button type="danger" @click="cancel">Cancel</a-button>
          </a-col>
        </a-row>
      </template>
    </vuci-form>
    <template #footer> </template>
  </a-modal>
</template>

<script>
const protocols = [
  ["static", "Static"],
  ["dhcp", "DHCP"],
];

export default {
  props: {
    modal: {
      type: Object,
      required: true,
    },
    config: {
      type: String,
      required: true,
    },
    selectedSection: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      protocols,
      protocolDependency: "proto == 'static'",
      selectedProtocol: "",
    };
  },
  methods: {
    getFormFieldKey(fieldName) {
      return `${this.selectedSection}_${fieldName}`;
    },
    save() {
      if (this.selectedProtocol === "dhcp") {
        const form = this.$refs.editForm.form;
        form[this.getFormFieldKey("address")] = "";
        form[this.getFormFieldKey("gateway")] = "";
        form[this.getFormFieldKey("netmask")] = "";
        form[this.getFormFieldKey("dns")] = [];
      }

      this.$refs.editForm.apply();
    },
    cancel() {
      this.modal.isVisible = false;
      this.$refs.editForm.reset();
    },
    onApplied() {
      this.modal.isVisible = false;
      this.$emit("applied");
    },
    onProtocolChange(self) {
      this.selectedProtocol = self.form[this.getFormFieldKey("proto")];
    },
  },
};
</script>
