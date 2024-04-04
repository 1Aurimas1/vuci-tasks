<template>
  <a-space direction="vertical">
    <a-button type="primary" @click="showDrawer"> Open card config </a-button>

    <draggable
      v-if="!isDataLoading"
      :list="cards"
      v-bind="dragOptions"
      class="card-group"
      @start="drag = true"
      @end="drag = false"
      @change="onDragChange"
    >
      <transition-group
        type="transition"
        :name="!drag ? 'flip-card' : null"
        class="flex"
      >
        <Card
          v-for="card in visibleCards"
          :key="card.name"
          :title="card.name"
          :sections="card.sections"
          class="card-group-item"
        >
        </Card>
      </transition-group>
    </draggable>

    <Drawer
      ref="drawer"
      :config="config"
      :type="type"
      @onCardChange="onCardChange"
    />
  </a-space>
</template>

<script>
import Drawer from "./Drawer.vue";
import draggable from "vuedraggable";
import Card from "./Card.vue";

export default {
  components: {
    Drawer,
    draggable,
    Card,
  },
  data() {
    return {
      config: "vuci_overview_task",
      type: "card",
      sections: [],
      drag: false,
      cards: [],

      lastCPUTime: null,
      isDataLoading: true,
    };
  },
  timers: {
    loadSystemInfo: {
      time: 5000,
      autostart: true,
      immediate: true,
      repeat: true,
    },
  },
  computed: {
    visibleCards() {
      return this.cards.filter((c) => this.isCardVisible(c));
    },
    dragOptions() {
      return {
        animation: 200,
        group: "description",
        disabled: false,
        ghostClass: "ghost",
      };
    },
  },
  methods: {
    showDrawer() {
      this.$refs.drawer.visible = true;
    },
    isCardVisible(c) {
      return c.visible === "1";
    },
    onCardChange(cardName, visibleValue) {
      const card = this.cards.find((c) => c.name === cardName);
      if (card) {
        this.$set(card, "visible", visibleValue);
      }
    },
    onDragChange({ moved }) {
      if (moved) {
        this.cards.forEach((c, idx) => {
          this.$uci.set(this.config, c[".name"], "index", idx);
        });
        this.$uci.save().then((_) => {
          this.$uci.apply();
        });
      }
    },
    addCardData(name, data) {
      const card = this.cards.find((c) => c.name === name);
      if (card) {
        this.$set(card, "sections", data);
      }
    },
    async loadUci() {
      await this.$uci.load(this.config);
      this.cards = this.$uci
        .sections(this.config, this.type)
        .sort((a, b) => a.index - b.index);
    },
    async loadInterfaces() {
      await this.$network.load();
      const ifaces = this.$network.getInterfaces();

      for (const iface of ifaces) {
        const ifInfo = [];
        ifInfo.push({
          title: "Type",
          position: "body",
          type: "text",
          value: iface.status.device,
        });
        ifInfo.push({
          title: "IP address",
          position: "body",
          type: "text",
          value: iface.getIPv4Addrs().join(", "),
        });
        this.addCardData(iface.name, ifInfo);
      }
    },
    async getLogsInfo(table) {
      return this.$log.get({ table: table, limit: 4 }).then((log) => {
        const logInfo = [];
        for (const l of log.log) {
          logInfo.push({
            title: new Date(l.TIME * 1000).toLocaleString("lt-LT"),
            position: "body",
            type: "text",
            value: l.TEXT,
          });
        }
        return logInfo;
      });
    },
    async load() {
      await this.loadUci();
      await this.loadInterfaces();

      const networksLogs = await this.getLogsInfo("NETWORK");
      this.addCardData("recent network events", networksLogs);

      const systemLogs = await this.getLogsInfo("SYSTEM");
      this.addCardData("recent system events", systemLogs);

      this.isDataLoading = false;
    },
    updateCpuUsage() {
      return this.$rpc.call("system", "cpu_time").then((times) => {
        if (!this.lastCPUTime) {
          this.lastCPUTime = times;
          return 0;
        }

        const idle1 = this.lastCPUTime[3];
        const idle2 = times[3];

        let total1 = 0;
        let total2 = 0;

        this.lastCPUTime.forEach((t) => {
          total1 += t;
        });

        times.forEach((t) => {
          total2 += t;
        });

        this.lastCPUTime = times;
        return Math.floor(
          ((total2 - total1 - (idle2 - idle1)) / (total2 - total1)) * 100
        );
      });
    },
    async loadSystemInfo() {
      const { release, localtime, uptime, memory } =
        await this.$system.getInfo();
      const routerUptime = "%t".format(uptime);
      const localTime = new Date(localtime * 1000).toLocaleString("lt-LT");
      const memPercentage = Math.floor(
        ((memory.total - memory.free) / memory.total) * 100
      );
      const firmwareVersion = release.revision;

      const { root } = await this.$system.getDiskInfo();
      const storagePercentage = Math.floor((root.used / root.total) * 100);

      const cpuPercentage = await this.updateCpuUsage();

      const sysInfo = [
        {
          title: "CPU load",
          position: "header",
          type: "status",
          value: cpuPercentage,
        },
        {
          title: "Router uptime",
          position: "body",
          type: "text",
          value: routerUptime,
        },
        {
          title: "Local device time",
          position: "body",
          type: "text",
          value: localTime,
        },
        {
          title: "Memory usage",
          position: "body",
          type: "status",
          value: [
            {
              title: "Memory",
              value: memPercentage,
            },
            {
              title: "Flash",
              value: storagePercentage,
            },
          ],
        },
        {
          title: "Firmware version",
          position: "body",
          type: "text",
          value: firmwareVersion,
        },
      ];

      this.addCardData("system info", sysInfo);
    },
  },
  created() {
    this.load();
  },
};
</script>

<style scoped>
.flip-card-move {
  transition: transform 0.5s;
}

.no-move {
  transition: transform 0s;
}

.card-group {
  min-height: 20px;
}

.card-group-item {
  cursor: move;
}

.card-group-item i {
  cursor: pointer;
}

.flex {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}
</style>
