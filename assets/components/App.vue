<template>
  <v-app>
    <GraphEditor
      :loaded=loaded
    />
    <Graph
      ref="graph"
      class="graph"
      :completeLoad=completeLoad
      @node-click="onNodeClick"
      @node-dragged="onNodeDragged"
    />
    <vue-progress-bar></vue-progress-bar>

    <!-- Top-right controls: export / import -->
    <div class="top-right-controls">
      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn fab small v-on="on" @click="startExport()">
            <v-icon>mdi-download</v-icon>
          </v-btn>
        </template>
        <span>Export configuration</span>
      </v-tooltip>

      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn fab small v-on="on" @click="$refs.importFile.click()">
            <v-icon>mdi-upload</v-icon>
          </v-btn>
        </template>
        <span>Import configuration</span>
      </v-tooltip>

      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn fab small v-on="on" @click="clearDialog = true">
            <v-icon>mdi-broom</v-icon>
          </v-btn>
        </template>
        <span>Clear configuration</span>
      </v-tooltip>
    </div>

    <!-- Hidden file input for import -->
    <input
      ref="importFile"
      type="file"
      accept=".json"
      style="display:none"
      @change="onImportFile"
    />

    <!-- Clear config confirmation dialog -->
    <v-dialog v-model="clearDialog" max-width="360">
      <v-card>
        <v-card-title>Clear configuration?</v-card-title>
        <v-card-text>
          This will reset all visibility, field expansion, node positions, and zoom to their defaults. This cannot be undone.
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text @click="clearDialog = false">Cancel</v-btn>
          <v-btn color="error" text @click="confirmClear()">Clear</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Export name dialog -->
    <v-dialog v-model="exportDialog" max-width="360">
      <v-card>
        <v-card-title>Export configuration</v-card-title>
        <v-card-text>
          <v-text-field
            v-model="exportName"
            label="Configuration name"
            autofocus
            @keyup.enter="confirmExport()"
          />
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text @click="exportDialog = false">Cancel</v-btn>
          <v-btn color="primary" text @click="confirmExport()">Download</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Bottom-right controls -->
    <div class="bottom-right-controls">
      <v-tooltip left>
        <template v-slot:activator="{ on }">
          <v-btn fab small v-on="on" @click="$refs.graph.zoomIn()">
            <v-icon>mdi-magnify-plus-outline</v-icon>
          </v-btn>
        </template>
        <span>Zoom in</span>
      </v-tooltip>

      <v-tooltip left>
        <template v-slot:activator="{ on }">
          <v-btn fab small v-on="on" @click="$refs.graph.zoomOut()">
            <v-icon>mdi-magnify-minus-outline</v-icon>
          </v-btn>
        </template>
        <span>Zoom out</span>
      </v-tooltip>

      <v-tooltip left>
        <template v-slot:activator="{ on }">
          <v-btn fab small color="#64B5F6" v-on="on" @click="helpDialog = true">
            <v-icon>mdi-help-circle-outline</v-icon>
          </v-btn>
        </template>
        <span>Help</span>
      </v-tooltip>
    </div>

    <!-- Help dialog -->
    <v-dialog v-model="helpDialog" max-width="560" scrollable>
      <v-card>
        <v-card-title class="headline">How to use the Schema Graph</v-card-title>
        <v-divider />
        <v-card-text style="max-height: 70vh">

          <p class="subtitle-1 mt-3 mb-1"><strong>Graph canvas</strong></p>
          <v-simple-table dense>
            <tbody>
              <tr><td>Scroll wheel / trackpad pinch</td><td>Zoom in and out</td></tr>
              <tr><td>Click + drag canvas</td><td>Pan around the graph</td></tr>
            </tbody>
          </v-simple-table>

          <p class="subtitle-1 mt-4 mb-1"><strong>Export / Import (top-right)</strong></p>
          <v-simple-table dense>
            <tbody>
              <tr><td><v-icon small>mdi-download</v-icon> Export</td><td>Save the current graph layout (visibility, positions, zoom) as a named JSON file</td></tr>
              <tr><td><v-icon small>mdi-upload</v-icon> Import</td><td>Load a previously exported configuration file to restore that layout</td></tr>
            </tbody>
          </v-simple-table>

          <p class="subtitle-1 mt-4 mb-1"><strong>Nodes</strong></p>
          <v-simple-table dense>
            <tbody>
              <tr><td>Click a node</td><td>Expand / collapse its field list</td></tr>
              <tr><td>Drag a node</td><td>Pin it at a fixed position</td></tr>
            </tbody>
          </v-simple-table>

          <p class="subtitle-1 mt-4 mb-1"><strong>Zoom buttons (bottom-right)</strong></p>
          <v-simple-table dense>
            <tbody>
              <tr><td><v-icon small>mdi-magnify-plus-outline</v-icon> Zoom in</td><td>Zoom in by ~15%</td></tr>
              <tr><td><v-icon small>mdi-magnify-minus-outline</v-icon> Zoom out</td><td>Zoom out by ~15%</td></tr>
            </tbody>
          </v-simple-table>

          <p class="subtitle-1 mt-4 mb-1"><strong>Sidebar (☰ top-left)</strong></p>
          <v-simple-table dense>
            <tbody>
              <tr><td>Hide All / Show All</td><td>Toggle visibility of all models</td></tr>
              <tr><td>Collapse All / Expand All</td><td>Collapse all apps into a single box node, or expand them back</td></tr>
              <tr><td>Fold All / Unfold All</td><td>Collapse or expand the sidebar app list</td></tr>
              <tr><td>Expand All Fields / Collapse All Fields</td><td>Show or hide field details on all nodes at once</td></tr>
              <tr><td>Eye icon next to an app</td><td>Toggle visibility of all models in that app</td></tr>
              <tr><td>Arrow icon next to an app</td><td>Collapse all models in the app into a single group box</td></tr>
              <tr><td>Eye icon next to a model</td><td>Toggle visibility of that individual model</td></tr>
            </tbody>
          </v-simple-table>

          <p class="subtitle-1 mt-4 mb-1"><strong>Node styles</strong></p>
          <v-simple-table dense>
            <tbody>
              <tr><td>Dashed border</td><td>Abstract model</td></tr>
              <tr><td>White background</td><td>Proxy model</td></tr>
              <tr><td>Color by app</td><td>Each Django app gets a unique hue</td></tr>
            </tbody>
          </v-simple-table>

          <p class="subtitle-1 mt-4 mb-1"><strong>Edge styles</strong></p>
          <v-simple-table dense>
            <tbody>
              <tr><td>Solid, single arrow</td><td>ForeignKey</td></tr>
              <tr><td>Solid, middle arrow</td><td>OneToOneField</td></tr>
              <tr><td>Solid, bidirectional arrows</td><td>ManyToManyField</td></tr>
              <tr><td>Dashed line</td><td>Inheritance (subclass) or Proxy relationship</td></tr>
              <tr><td>Edge label</td><td>Field name; hover for type, field name, and reverse name</td></tr>
            </tbody>
          </v-simple-table>

        </v-card-text>
        <v-divider />
        <v-card-actions>
          <v-spacer />
          <v-btn text @click="helpDialog = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </v-app>
</template>

<style>
  /* Override vuetify */
  .v-btn:not(.v-btn--text):not(.v-btn--outlined):focus::before {
    opacity: 0;
  }
  .v-application--is-ltr .v-list-item__icon:first-child {
    margin-right: 12px !important;
  }
  .v-application--is-ltr .v-list-item__icon:last-of-type:not(:only-child) {
    margin-left: 0 !important;
  }
  .v-list-group .v-list-group__header .v-list-item__icon.v-list-group__header__append-icon {
    min-width: 24px !important;
  }
</style>

<style scoped>
  /* Own styles */
  .graph {
    height: 100vh;
  }
  .top-right-controls {
    position: fixed;
    top: 16px;
    right: 16px;
    display: flex;
    flex-direction: row;
    gap: 8px;
    z-index: 10;
  }
  .bottom-right-controls {
    position: fixed;
    bottom: 16px;
    right: 16px;
    display: flex;
    flex-direction: column;
    gap: 8px;
    z-index: 10;
  }
</style>

<script>
import Graph from "./Graph.vue";
import GraphEditor from "./GraphEditor.vue";
import graphData from "../state/graphData.js";

export default {
  name: 'App',
  components: {Graph, GraphEditor},
  props: [],
  methods: {
    completeLoad: function() {
      this.loaded = true;
    },
    onNodeClick: function(nodeID) {
      graphData.toggleNodeFields(nodeID);
      this.$nextTick(() => this.$refs.graph.restabilize());
    },
    onNodeDragged: function({ nodeId, x, y }) {
      graphData.pinNode(nodeId, x, y);
    },
    startExport: function() {
      this.exportName = 'schema-config';
      this.exportDialog = true;
    },
    confirmExport: function() {
      const config = {
        version: 1,
        name: this.exportName,
        exportedAt: new Date().toISOString(),
        viewport: this.$refs.graph.getViewState(),
        ...graphData.getConfig(),
      };
      const blob = new Blob([JSON.stringify(config, null, 2)], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `${this.exportName.replace(/\s+/g, '-').toLowerCase()}.json`;
      a.click();
      URL.revokeObjectURL(url);
      this.exportDialog = false;
    },
    confirmClear: function() {
      graphData.resetConfig();
      this.$nextTick(() => this.$refs.graph.resetViewport());
      this.clearDialog = false;
    },
    onImportFile: function(event) {
      const file = event.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = (e) => {
        try {
          const config = JSON.parse(e.target.result);
          if (config.version !== 1) {
            alert('Unsupported configuration version.');
            return;
          }
          graphData.applyConfig(config);
          if (config.viewport) {
            this.$nextTick(() => this.$refs.graph.setViewState(config.viewport));
          }
        } catch {
          alert('Could not read configuration file. Make sure it is a valid JSON export.');
        }
        event.target.value = '';
      };
      reader.readAsText(file);
    },
  },
  data() {
    let loaded = false;
    graphData.setup('rgba(0, 0, 0, 0.54)');

    return {
      loaded,
      helpDialog: false,
      exportDialog: false,
      exportName: 'schema-config',
      clearDialog: false,
    }
  },
};
</script>
