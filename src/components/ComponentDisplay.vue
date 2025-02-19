<template>
  <!--the sandbox area where the component boxes are rendered-->
  <div class="component-display">
    <Vue3DraggableResizable
      class="component-box"
      v-for="componentData in activeRouteArray"
      :draggable="true"
      :resizable="true"
      :disabledX="false"
      :disabledY="false"
      :parent="true"
      :key="componentData.componentName"
      :x="componentData.x"
      :y="componentData.y"
      :w="componentData.w"
      :h="componentData.h"
      :handles="['tl', 'tm', 'tr', 'ml', 'mr', 'bl', 'bm', 'br']"
      @click="onClick(componentData)"
      @activated="onActivated(componentData)"
      @deactivated="onDeactivated"
      @drag-start="activeComponentData, onDrag"
      @drag-end="onDragEnd"
      @resize-start="activeComponentData, onResize"
      @resize-end="onResizeEnd"
      @dblclick.native="onDoubleClick(componentData)"
    >
      <h3>{{ componentData.componentName }}</h3>
    </Vue3DraggableResizable>

    <v-overlay v-model="modalOpen" class="overlay">
      <v-dialog v-model="modalOpen" class="modal" width="auto">
        <v-card>
          <Modal />
        </v-card>
      </v-dialog>
    </v-overlay>
  </div>
</template>
<script>
import { mapState, mapActions } from 'vuex';
import Vue3DraggableResizable from 'vue3-draggable-resizable';
import Modal from './Modal/Modal.vue';

export default {
  name: 'ComponentDisplay',
  components: {
    Vue3DraggableResizable,
    Modal
  },
  data() {
    return {
      // by default modal associated with active component for further user custimaztion should be hidden
      modalOpen: false
    };
  },
  mounted() {
    // allows active user created component to be deleted when backspace is pressed
    window.addEventListener('keyup', event => {
      if (event.key === 'Backspace') {
        if (this.activeComponent && this.activeComponentData.isActive) {
          this.$store.dispatch('deleteActiveComponent');
        }
      }
    });
  },
  computed: {
    ...mapState(['routes', 'activeRoute', 'activeComponent', 'componentMap']),
    activeRouteArray() {
      // returns components associated with current active route
      return this.routes[this.activeRoute];
    },
    activeComponentData() {
      // returns object containing data associated with current active component
      return this.activeRouteArray.filter(componentData => {
        return componentData.componentName === this.activeComponent;
      })[0];
    }
  },
  methods: {
    ...mapActions(['setActiveComponent', 'updateOpenModal']),
    onResize: function(x) {
      // updates state associated with active component to reflect start of resize user has made to the component
      this.activeComponentData.x = x.x;
      this.activeComponentData.y = x.y;
      this.activeComponentData.w = x.w;
      this.activeComponentData.h = x.h;
    },
    onResizeEnd: function(x) {
      // updates state associated with active component to reflect end of resize user has made to the component
      this.activeComponentData.isActive = true;
      this.activeComponentData.x = x.x;
      this.activeComponentData.y = x.y;
      this.activeComponentData.w = x.w;
      this.activeComponentData.h = x.h;
    },
    onDrag: function(x) {
      // updates state associated with active component to reflect start of drag user has made to the component
      this.activeComponentData.x = x.x;
      this.activeComponentData.y = x.y;
    },
    onDragEnd: function(x) {
      // updates state associated with active component to reflect end of drag user has made to the component
      this.activeComponentData.x = x.x;
      this.activeComponentData.y = x.y;
    },
    onActivated(componentData) {
      // updates state to reflect current selected componenet (i.e. active component)
      this.setActiveComponent(componentData.componentName);
      this.activeComponentData.isActive = true;
    },
    onDeactivated() {
      // updates state when current selected component is unselected
      this.activeComponentData.isActive = false;
    },
    onClick(compData) {
      // sets clicked component as active in state
      this.setActiveComponent(compData.componentName);
      this.activeComponentData.isActive = true;
    },
    onDoubleClick(compData) {
      // sets double clicked component as active and opens modal providing options to further manipulate the component
      this.setActiveComponent(compData.componentName);
      this.activeComponentData.isActive = true;
      this.modalOpen = true;
    }
  }
};
</script>

<style scoped>
.component-display {
  border: 1px solid plum;
  height: 84vh;
}
.component-display {
  color: #3ab982;
  border: 1px solid salmon;
  position: relative;
}
.component-box {
  color: #3ab982;
  border: 1px solid salmon;
}
</style>
