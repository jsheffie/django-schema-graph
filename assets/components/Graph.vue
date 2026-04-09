<template>
  <network
    ref="visNetwork"
    :nodes=graphData.nodes
    :edges=graphData.edges
    :options=options
    @stabilization-progress="stabilizationProgress"
    @stabilization-iterations-done="stabilizationIterationsDone"
    @click="onNetworkClick"
    @drag-end="onDragEnd"
  />
</template>

<script>
import { Network } from "vue-vis-network";
import graphData from "../state/graphData.js";

export default {
  name: "Graph",
  components: { Network },
  props: ["completeLoad"],
  data() {
    const options = {
      edges: {
        smooth: {},
        arrows:{
          to: {scaleFactor: 0.8},
          from: {scaleFactor: 0.8},
        }
      }
    };
    return {
      options,
      graphData,
    };
  },
  methods: {
    stabilizationProgress: function (ev) {
      const progress = (ev.iterations / ev.total) * 100;
      console.log(`Stabilization progress ${progress}%`);
      this.$Progress.set(progress);
    },
    stabilizationIterationsDone: function () {
      console.log('Stabilization complete');
      this.$Progress.finish();
      this.completeLoad();
    },
    onNetworkClick: function (event) {
      if (event.nodes && event.nodes.length === 1) {
        this.$emit('node-click', event.nodes[0]);
      }
    },
    onDragEnd: function (event) {
      if (event.nodes && event.nodes.length > 0) {
        const net = this.$refs.visNetwork && this.$refs.visNetwork.network;
        if (net) {
          const positions = net.getPositions(event.nodes);
          event.nodes.forEach(nodeId => {
            const pos = positions[nodeId];
            if (pos) this.$emit('node-dragged', { nodeId, x: pos.x, y: pos.y });
          });
        }
      }
    },
    restabilize: function () {
      const net = this.$refs.visNetwork && this.$refs.visNetwork.network;
      if (net) net.stabilize(150);
    },
    zoomIn: function () {
      const net = this.$refs.visNetwork && this.$refs.visNetwork.network;
      if (net) net.moveTo({ scale: net.getScale() * 1.15 });
    },
    zoomOut: function () {
      const net = this.$refs.visNetwork && this.$refs.visNetwork.network;
      if (net) net.moveTo({ scale: net.getScale() / 1.15 });
    },
    getViewState: function () {
      const net = this.$refs.visNetwork && this.$refs.visNetwork.network;
      if (!net) return null;
      return {
        scale: net.getScale(),
        position: net.getViewPosition(),
      };
    },
    setViewState: function (viewState) {
      const net = this.$refs.visNetwork && this.$refs.visNetwork.network;
      if (!net || !viewState) return;
      net.moveTo({
        scale: viewState.scale,
        position: viewState.position,
      });
    },
    resetViewport: function () {
      const net = this.$refs.visNetwork && this.$refs.visNetwork.network;
      if (net) net.fit();
    },
  },
};
</script>
