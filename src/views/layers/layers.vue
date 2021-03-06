<template>
  <Container @drop="onDrop" :get-ghost-parent="getGhostParent">
    <Draggable class="fc-layer" v-for="(layer, index) in layers" :key="index">
      <div
        class="__item"
        :class="{ '__item--active': index === currentLayerIndex, 'has-syntax-error-tags': layer.hasSyntaxErrorTags }"
        tabindex="0"
        @keydown.exact.enter.prevent="focusEditor"
        @keydown.exact.shift.up="updateCheckedState(currentLayerIndex - 1, $event)"
        @keydown.exact.shift.down="updateCheckedState(currentLayerIndex + 1, $event)"
        @keydown.exact.shift.alt.up="onUpBlock"
        @keydown.exact.shift.alt.down="onDownBlock"
        @keydown.exact.up.prevent="onSelect(currentLayerIndex - 1)"
        @keydown.exact.down.prevent="onSelect(currentLayerIndex + 1)"
        @keydown.exact.page-up.prevent="onSelect(currentLayerIndex - 5)"
        @keydown.exact.page-down.prevent="onSelect(currentLayerIndex + 5)"
        @keydown.exact.home.prevent="onSelect(0)"
        @keydown.exact.end.prevent="onSelect(layers.length - 1)"
      >
        <div class="__item__group" v-if="layer.layout" @click="onSelect(index, $event, layer)">
          <label :for="`layer-${index}`" >
            <input :id="`layer-${index}`" type="checkbox" v-model="layer.isChecked"/>
          </label>
          <img :src="layer.layout.icon" alt="" />
          <span class="__item__group__info">
            <strong class="__item__group__name">{{ layer.layout.id }}</strong>
              {{ layer.layout.description }}
            </span>
        </div>
        <div class="__utils">
          <button class="__utils__btn" @click="onAddFavoriteLayout(layer.layout.id)">
            <i class="material-icons">{{ getFavoriteLayoutIconStyle(layer) }}</i>
          </button>
          <button class="__utils__btn" @click="onToggle(index)">
            <i class="material-icons">
              {{ getLayoutStateIconStyle(layer) }}
            </i>
          </button>
          <button class="__utils__btn" @click="onCloneLayer(index)">
            <i class="material-icons">file_copy</i>
          </button>
          <button class="__utils__btn" @click="onRemoveLayer(index)">
            <i class="material-icons">delete</i>
          </button>
        </div>
      </div>
    </Draggable>
  </Container>
</template>
<script>
  import { Container, Draggable } from "vue-smooth-dnd";
  import EventBus from '../../event-bus/event-bus';

  export default {
    components: {
      Container,
      Draggable,
    },
    props: {
      layers: {
        type: Array,
        default() {
          return []
        }
      },
      currentLayerIndex: {
        type: Number,
        default() {
          return -1
        }
      },
      favoriteLayoutIds: {
        type: Array,
        default() {
          return [];
        }
      }
    },
    data() {
      return {
        dropResult: null,
        checkedHistory: null,
      }
    },
    computed: {
      applyDrag() {
        const { removedIndex, addedIndex, payload } = this.dropResult;
        if (removedIndex === null && addedIndex === null) return this.layers;

        const result = this.layers;
        let itemToAdd = payload;

        if (removedIndex !== null) {
          itemToAdd = result.splice(removedIndex, 1)[0];
        }

        if (addedIndex !== null) {
          result.splice(addedIndex, 0, itemToAdd);
        }

        return result;
      }
    },
    methods: {
      onUpBlock() {
        this.$emit('up');
      },
      onDownBlock() {
        this.$emit('down');
      },
      updateCheckedState(index, event) {
        let newIndex = Math.min(Math.max(index, 0), this.layers.length - 1);

        if (this.checkedHistory !== event.key) {
          if (event.key === 'ArrowUp') {
            newIndex += 1;
          } else {
            newIndex -= 1;
          }
        }

        this.layers[newIndex].isChecked = !this.layers[newIndex].isChecked;

        this.$emit('selected-layer', newIndex);
        this.$emit('move-selected-layer');

        this.checkedHistory = event.key;
      },
      resetCheckedHistory() {
        this.checkedHistory = null;
      },
      isFavoriteLayout({ id } ) {
        return this.favoriteLayoutIds.includes(id);
      },
      getFavoriteLayoutIconStyle({ layout }) {
        return this.isFavoriteLayout(layout) ? 'favorite' : 'favorite_border';
      },
      getLayoutStateIconStyle({ hidden }) {
        return hidden ? 'visibility_off' : 'visibility';
      },
      onSelect(index) {
        const newIndex = Math.min(Math.max(index, 0), this.layers.length - 1);

        this.$emit('update:currentLayerIndex', newIndex);
        this.resetCheckedHistory();
      },
      onDrop(dropResult) {
        this.dropResult = dropResult;
        this.layers = this.applyDrag;
        this.$emit('update:currentLayerIndex', dropResult.addedIndex);
      },
      getGhostParent(){
        return document.body;
      },
      onAddFavoriteLayout(layoutId) {
        this.$emit('add-favorite-layout', layoutId);
      },
      onCloneLayer(index) {
        this.$emit('clone-layer', index);
      },
      onRemoveLayer(index) {
        this.$emit('remove-layer', index);
      },
      focus() {
        // XXX: focus the selected layer
        this.$el.focus();
        const el = this.$el.querySelector('.__item--active');
        if (el) {
          el.focus();
        }
      },
      focusEditor() {
        // XXX: focus to editor
        EventBus.$emit('focus-editor');
      },
      onToggle(index) {
        this.$emit('hidden', index, !this.layers[index].hidden);
      }
    },
  }
</script>
<style lang="scss" scoped>
  @import '../../assets/scss/utils/utilities';
  .__item {
    position: relative;
    border: 3px solid #ffffff;
    cursor: pointer;
    &:focus {
      outline-style: none;
    }
    &.has-syntax-error-tags {
      background-color: #ffba00;
      .__item__group {
        background-color: #ffba00;
      }
    }
    &--active {
      border: 3px solid #f74982;
    }
    &__group {
      display: flex;
      flex-direction: row;
      width: percentage(1);
      background: #0fd961;
      color: $white;
      &__info {
        flex: 1;
        padding-top: 0.5rem;
        padding-left: 1rem;
        padding-bottom: 0.5rem;
        text-align: left;
      }
      &__name {
        display: block;
        margin-bottom: 0.5rem;
      }
      label {
        display: inline-block;
        align-self: center;
      }
    }
  }
  .__utils {
    position: absolute;
    top: 0;
    right: 0;
  }
  .smooth-dnd-draggable-wrapper {
    & + .smooth-dnd-draggable-wrapper {
      margin-top: 0.5rem;
    }
  }
</style>
