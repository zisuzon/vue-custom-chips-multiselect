<template>
  <div v-on-click-outside="deactivate">
    <div class="selecteditems" @click.prevent="activate()">
      <ul class="chips">
        <template>
          <li v-for="item in selectedItems" :key="item.id" class="chips__item">
            {{ item.name }}
            <span class="chips--remove" @click.stop="removeItem(item)"><b>x</b></span>
          </li>
        </template>
        <li class="chips__itemInput">
          <input 
            v-model="searchedText"
            @focus.prevent="activate()"
            @keyup.esc="deactivate()"
            class="chips__input--fake"
            type="text"
            placeholder="Type to search"
            ref="search"
          >
        </li>
      </ul>
    </div>

    <div v-if="showList" class="allitems">
      <ul>
        <li class="allitems__list" v-for="(item) in filteredAllItems" :key="item.id" @click="atSelectItem(item)">
          <span>{{ item.name }}</span>
        </li>
      </ul>
    </div>

  </div>
</template>

<script>
import { mixin as onClickOutside } from 'vue-on-click-outside'

import { EVENT_CATEGORIES_SELECTION_CUSTOM_CHIPS_MULTISELECT } from '~/store/constants.yaml'

import filter from 'lodash/filter'
import cloneDeep from 'lodash/cloneDeep'
import sortBy from 'lodash/sortBy'

export default {
  mounted () {
    this.selectedItems = []
  },
  props: ['items', 'selectedPrevItems'],
  mixins: [onClickOutside],
  data () {
    return {
      searchedText: null,
      selectedItems: null,
      showList: false
    }
  },
  watch: {
    'selectedPrevItems.length': {
      handler: function (newVal) {
        this.selectedItems = cloneDeep(this.selectedPrevItems)
        setTimeout(() => {
          this.$root.$emit(EVENT_CATEGORIES_SELECTION_CUSTOM_CHIPS_MULTISELECT, this.selectedPrevItems)
        }, 0)
      },
      deep: true,
      immediate: true
    }
  },
  computed: {
    filteredAllItems () {
      let clonedItems = cloneDeep(this.items)
      if (this.searchedText) {
        let filteredItems = filter(clonedItems, (i) => {
          return i.name.toLowerCase().includes(this.searchedText.toLowerCase())
        })
        return sortBy(filteredItems, ['id'])
      } else {
        return sortBy(clonedItems, ['id'])
      }
    }
  },
  methods: {
    atSelectItem (item) {
      this.selectedItems.push(item)
      let indexOfItem = this.filteredAllItems.findIndex((i) => {
        return i.id === item.id
      })
      this.filteredAllItems.splice(indexOfItem, 1)
      this.searchedText = null
      this.$root.$emit(EVENT_CATEGORIES_SELECTION_CUSTOM_CHIPS_MULTISELECT, this.selectedItems)
    },
    removeItem (item) {
      this.filteredAllItems.push(item)
      let indexOfItem = this.selectedItems.findIndex((i) => {
        return i.id === item.id
      })
      this.selectedItems.splice(indexOfItem, 1)
      this.$root.$emit(EVENT_CATEGORIES_SELECTION_CUSTOM_CHIPS_MULTISELECT, this.selectedItems)
    },
    activate () {
      // this.$refs.search.focus()
      this.showList = true
    },
    deactivate () {
      this.showList = false
    }
  }
}
</script>

<style lang="sass" scoped>
  .chips
    background-color: #ffffff
    border: 1px solid #dbdbdb
    border-radius: 4px
    color: #363636
    padding-bottom: calc(0.375em - 1px)
    padding-left: calc(0.625em - 1px)
    padding-right: calc(0.625em - 1px)
    padding-top: calc(0.375em - 1px)
    cursor: pointer 
    &--remove
      margin-right: 1px
      margin-left: 3px
      color: #cc0029
    &__item
      display: inline-block
      background-color: rgba(50, 115, 220, .2)
      margin: 4px
      padding: 6px
      border-radius: 6px
      transition: 0.3s
      &:hover
        background-color: rgba(50, 115, 220, .3)
    &__itemInput
      display: inline-block
    &__input--fake
      border: none
      width: 100%
      line-height: 1.42857143
      font-size: 1rem
      height: 36px
      &:focus, 
      &:active 
        outline: none
  .allitems
    cursor: pointer
    border: 1px solid #dbdbdb
    max-height: 150px
    height: calc(100vh - 240px)
    overflow-y: scroll
    &__list
      padding: 6px
      transition: 0.3s
      &:hover
        background: rgba(50, 115, 220, 1)
        color: #ffffff
</style>
