<template>
  <div class="nested__dropArea" :class="nestedDropAreaClasses">
    <!-- Accordion wrapper only if parentId > -1 and the row has children -->
    <a17-accordion v-if="parentId > -1 && rows.length > 0"
      :open="open"
      @toggleVisibility="notifyOpen"
      :useOverrideTransition="true"
      :beforeEnterOverride="beforeEnterOverride"
      :enterOverride="enterOverride"
      :beforeLeaveOverride="beforeLeaveOverride"
      :leaveOverride="leaveOverride"
    >
      <span slot="accordion__title"><slot>Expand to see more</slot></span>
      <!-- <div slot="accordion__value" v-html="currentLabel"></div> -->
      <div class="accordion__fields">
        <transition :css="false" :duration="275"
                    @before-enter="beforeEnterOverride"
                    @enter="enterOverride"
                    @before-leave="beforeLeaveOverride"
                    @leave="leaveOverride">
          <draggable v-model="rows"
                     :options="draggableOptions"
                     :tag="'ul'"
                     :component-data="draggableGetComponentData">
            <li class="nested-datatable__item"
                v-for="(row, index) in rows"
                :class="haveChildren(row.children)"
                :key="depth + '-' +  row.id">
              <a17-nested-item
                :index="index"
                :row="row"
                :columns="columns"
              />
              <a17-nested-list
                v-if="row.children && depth < maxDepth"
                :maxDepth="maxDepth"
                :depth="depth + 1"
                :parentId="row.id"
                :items="row.children"
                :nested="true"
                :draggable="true"
              />
            </li>
          </draggable>
        </transition>
      </div>
    </a17-accordion>

    <!-- Regular draggable list for parentId === -1 -->
    <draggable v-else
               v-model="rows"
               :options="draggableOptions"
               :tag="'ul'"
               :component-data="draggableGetComponentData">
      <li class="nested-datatable__item"
          v-for="(row, index) in rows"
          :class="haveChildren(row.children)"
          :key="depth + '-' +  row.id">
        <a17-nested-item
          :index="index"
          :row="row"
          :columns="columns"
        />
        <a17-nested-list
          v-if="row.children && depth < maxDepth"
          :maxDepth="maxDepth"
          :depth="depth + 1"
          :parentId="row.id"
          :items="row.children"
          :nested="true"
          :draggable="true"
        />
      </li>
    </draggable>
  </div>
</template>

<script>
  import { mapState } from 'vuex'
  import { DATATABLE } from '@/store/mutations'
  import draggable from 'vuedraggable'
  import { DatatableMixin, DraggableMixin, NestedDraggableMixin } from '@/mixins/index'
  import NestedItem from './NestedItem'

  import a17Accordion from '@/components/Accordion.vue'
  import VisibilityMixin from '@/mixins/toggleVisibility'

  export default {
    name: 'a17-nested-list',
    components: {
      'a17-nested-item': NestedItem,
      draggable,
      a17Accordion
    },
    mixins: [DatatableMixin, DraggableMixin, NestedDraggableMixin, VisibilityMixin],
    props: {
      index: {
        type: Number,
        default: 0
      },
      items: {
        type: Array,
        default: () => []
      },
      useOverrideTransition: {
        type: Boolean,
        default: false
      }
    },
    data: function () {
      return {
        handle: '.tablecell__handle'
      }
    },
    computed: {
      styleDepth: function () {
        return {
          marginLeft: this.depth === 0 ? '0px' : '60px'
        }
      },
      rows: {
        get () {
          // return this.items
          return this.parentId > -1 ? this.items : this.$store.state.datatable.data
        },
        set (value) {
          const data = {
            parentId: this.parentId,
            val: value
          }

          const isChangingParents = (this.rows.length !== data.val.length)

          if (this.parentId > -1) {
            this.$store.commit(DATATABLE.UPDATE_DATATABLE_NESTED, data)
          } else {
            this.$store.commit(DATATABLE.UPDATE_DATATABLE_DATA, value)
          }
          this.saveNewTree(isChangingParents)
        }
      },
      nestedDropAreaClasses: function () {
        return [
          this.rows.length === 0 ? 'nested__dropArea--empty' : '',
          this.depth ? `nested__dropArea--depth nested__dropArea--depth${Math.min(10, this.depth)}` : ''
        ]
      },
      draggableOptions: function () {
        return {
          ...this.dragOptions,
          fallbackTolerance: 5,
          group: {
            name: this.name
          }
        }
      },
      ...mapState({
        currentValue: state => state.parents.active
      })
    },
    methods: {
      haveChildren: function (children) {
        return {
          // children may be undefined and not an array
          // if it's undefined the component wouldn't load anything
          // without providing a fallback value
          'nested-datatable__item--empty': (children || []).length === 0 && this.depth < this.maxDepth
        }
      },
      notifyOpen: function (newValue) {
        this.$emit('open', newValue, this.$options.name)
      },
      beforeEnterOverride (el) {
        el.style.maxHeight = '0px'
      },
      enterOverride (el, done) {
        el.style.maxHeight = (this.$el.querySelector('.accordion__list').clientHeight + 1) + 'px'
        el.addEventListener('transitionend', done, { once: true })
      },
      beforeLeaveOverride (el, done) {
        el.style.maxHeight = (this.$el.querySelector('.accordion__list').clientHeight + 1) + 'px'
        el.addEventListener('transitionend', done, { once: true })
      },
      leaveOverride (el, done) {
        el.style.maxHeight = '0px'
        el.addEventListener('transitionend', done, { once: true })
      }
    }
  }
</script>

<style lang="scss" scoped>

  .accordion__trigger > span {
    font-style: italic;
    color: #d7d5d5;
    font-size: 13px;
  }

  .nested-datatable__item {
    border: 1px solid #F2F2F2;
    // padding:10px 0 0 10px;
    margin-top: -1px;

    .nested-datatable__item {
      border-right: 0 none;
    }

    &.sortable-ghost {
      opacity: 1 !important;
      background-color: $color__f--bg;
    }

    &.sortable-chosen {
      opacity: 0.5;
    }

    &.sortable-drag {
      display: block;
    }
  }

  .nested__dropArea {
    // border:1px solid grey;
    padding: 15px 0px 15px 0px;

    * {
      will-change: auto;
    }

    .nested__dropArea {
      padding-left: 15px;
    }

    &.nested__dropArea--empty {
      padding-top: 20px;
      min-height: 20px;
      margin-top: -20px;
    }
  }

  .nested-item:hover + .nested__dropArea {
    background: $color__f--bg;

    .nested-datatable__item {
      background: white;
    }
  }

  .nested__dropArea--depth > li > div {
    &::after {
      content: '';
      display: block;
      height: 6px;
      border-left: 1px solid #D9D9D9;
      border-bottom: 1px solid #D9D9D9;
      position: absolute;
      top: calc(50% - 3px);
      left: 20px;
      background-color: transparent;
      width: 0;
      pointer-events: none;
    }
  }

  .nested__dropArea--depth1 > li > div {
    padding-left: 50px;

    &::after {
      width: 30px;
    }
  }

  @for $i from 2 through 10 {
    .nested__dropArea--depth#{$i} > li > div {
      padding-left: #{$i * 35px};

      &::after {
        width: #{($i * 35px) - 20px};
      }
    }
  }

</style>

<style lang="scss">
  .nested__dropArea {
    &.nested__dropArea--empty {
      .nested-item {
        margin-bottom: 0;
      }
    }

    &.nested-datatable__item--empty {
      > .nested-item {
        margin-bottom: 0;
      }
    }
  }
</style>
