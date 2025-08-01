<template>
  <f7-popup ref="modelPicker" class="modelpicker-popup" close-on-escape
            @popup:open="onOpen" @popup:close="onClose">
    <f7-page>
      <f7-navbar>
        <f7-nav-left>
          <f7-link icon-ios="f7:arrow_left" icon-md="material:arrow_back" icon-aurora="f7:arrow_left" popup-close />
        </f7-nav-left>
        <f7-nav-title>{{ popupTitle || 'Pick from Model' }}</f7-nav-title>
        <f7-nav-right>
          <f7-link v-if="ready && ((multiple && checkedItems.length > 0) || selectedItem || allowEmpty)" @click="pickItems">
            {{ actionLabel || 'Pick' }}<span v-if="multiple && checkedItems.length > 0">&nbsp;{{ checkedItems.length }}</span>
          </f7-link>
        </f7-nav-right>
      </f7-navbar>
      <f7-subnavbar :inner="false" v-show="initSearchbar">
        <f7-searchbar
          v-if="initSearchbar"
          :init="initSearchbar"
          search-container=".model-treeview"
          search-item=".treeview-item"
          search-in=".treeview-item-label"
          :disable-button="!$theme.aurora" />
        <div class="expand-button">
          <f7-button v-if="!expanded" icon-size="24" tooltip="Expand" icon-f7="rectangle_expand_vertical" @click="toggleExpanded()" />
          <f7-button v-else color="gray" icon-size="24" tooltip="Collapse" icon-f7="rectangle_compress_vertical" @click="toggleExpanded()" />
        </div>
      </f7-subnavbar>

      <!-- Toolbar -->
      <f7-toolbar v-if="$f7.width >= 500" bottom class="toolbar-details">
        <f7-link v-if="!multiple" :disabled="selectedItem != null" class="left" @click="selectedItem = null">
          Clear
        </f7-link>
        <span v-else />
        <div class="padding-right text-align-right">
          <f7-checkbox :checked="includeNonSemantic" @change="toggleNonSemantic" />
          <label @click="toggleNonSemantic" class="advanced-label">Show non-semantic</label>
          <f7-checkbox style="margin-left: 5px" :checked="includeItemName" @change="toggleItemName" />
          <label @click="toggleItemName" class="advanced-label">Show name</label>
          <f7-checkbox style="margin-left: 5px" :checked="includeItemTags" @change="toggleItemTags" />
          <label @click="toggleItemTags" class="advanced-label">Show tags</label>
        </div>
        <span />
      </f7-toolbar>
      <f7-toolbar v-else bottom class="toolbar-details" style="height: calc(50px + var(--f7-safe-area-bottom))">
        <f7-link v-if="!multiple" :disabled="selectedItem != null" class="left" @click="selectedItem = null">
          Clear
        </f7-link>
        <span v-else />
        <div class="padding-left padding-right text-align-center" style="font-size: 12px">
          <div>
            <f7-checkbox :checked="includeNonSemantic" @change="toggleNonSemantic" />
            <label @click="toggleNonSemantic" class="advanced-label">Show non-semantic</label>
          </div>
          <div>
            <f7-checkbox :checked="includeItemName" @change="toggleItemName" />
            <label @click="toggleItemName" class="advanced-label">Show name</label>
            <f7-checkbox style="margin-left: 5px" :checked="includeItemTags" @change="toggleItemTags" />
            <label @click="toggleItemTags" class="advanced-label">Show tags</label>
          </div>
        </div>
        <span />
      </f7-toolbar>

      <f7-block strong class="no-padding" v-if="ready">
        <model-treeview class="model-picker-treeview" :root-nodes="rootNodes"
                        :includeItemName="includeItemName" :includeItemTags="includeItemTags"
                        :selected="selectedItem" @selected="selectItem" @checked="checkItem" />
      </f7-block>
      <f7-block v-else-if="!ready" class="text-align-center">
        <f7-preloader />
        <div>Loading...</div>
      </f7-block>
    </f7-page>
  </f7-popup>
</template>

<style lang="stylus">
.expand-button
  margin-right 8px
  text-overflow: unset
  align-self: center
  .icon
    margin-bottom: 2.75px !important
</style>

<script>
import ModelTreeview from '@/components/model/model-treeview.vue'

import ModelMixin from '@/pages/settings/model/model-mixin'

export default {
  mixins: [ModelMixin],
  props: ['value', 'multiple', 'semanticOnly', 'groupsOnly', 'editableOnly', 'allowEmpty', 'popupTitle', 'actionLabel'],
  components: {
    ModelTreeview
  },
  data () {
    if (!this.$f7.data.modelPicker) this.$f7.data.modelPicker = {}
    return {
      initSearchbar: false,
      includeItemName: this.$f7.data.modelPicker.includeItemName || false,
      includeItemTags: this.$f7.data.modelPicker.includeItemTags || false,
      expanded: this.$f7.data.modelPicker.expanded || false,
      doubleClickStarted: null,
      doubleClickItem: null,
      checkedItems: []
    }
  },
  computed: {
    rootNodes () {
      if (this.semanticOnly) {
        return [this.rootLocations, this.rootEquipment, (!this.groupsOnly) ? this.rootPoints : []]
      } else {
        return [this.rootLocations, this.rootEquipment, (!this.groupsOnly) ? this.rootPoints : [], this.rootGroups, (!this.groupsOnly) ? this.rootItems : []].flat()
      }
    }
  },
  methods: {
    onOpen () {
      this.selectedItem = null
      this.initSearchbar = false
      this.$set(this, 'checkedItems', [])
      this.load()
    },
    onClose () {
      this.ready = false
      this.$emit('closed')
      this.$f7.emit('modelPickerClosed')
    },
    pickItems () {
      let pickedItems
      if (this.multiple) {
        pickedItems = this.checkedItems.map((i) => i.item)
      } else {
        pickedItems = (this.selectedItem) ? this.selectedItem.item : null
      }
      this.$emit('input', pickedItems)
      this.$f7.emit('itemsPicked', pickedItems)
      this.$refs.modelPicker.close()
    },
    modelItem (item) {
      const modelItem = {
        item,
        opened: false,
        checked: false,
        class: (item.metadata && item.metadata.semantics) ? item.metadata.semantics.value : '',
        children: {
          locations: [],
          equipment: [],
          points: [],
          groups: [],
          items: []
        }
      }
      // force the selection of the placeholder for a item being created
      if (item.created === false) {
        this.selectItem(modelItem)
      }
      if (this.previousSelection && item.name === this.previousSelection.item.name) {
        this.selectedItem = parent
        this.previousSelection = null
        this.selectItem(modelItem)
      }

      modelItem.checkable = this.multiple
      if (!this.multiple && this.value === item.name) {
        this.selectItem(modelItem)
      } else if (this.multiple && Array.isArray(this.value) && this.value.findIndex((i) => typeof i === 'string' ? i === item.name : i.name === item.name) >= 0) {
        modelItem.checked = true
        this.checkedItems.push(modelItem)
      }

      return modelItem
    },
    load () {
      this.loadModel().then(() => {
        this.$nextTick(() => {
          this.initSearchbar = true
          this.restoreExpanded()
          this.expandSelected()
        })
      })
    },
    selectItem (item) {
      if (!this.multiple) {
        this.selectedItem = item
        if (this.doubleClickStarted && this.doubleClickItem === item) {
          this.pickItems()
        } else {
          this.doubleClickStarted = setTimeout(() => { this.doubleClickStarted = null }, 500)
          this.doubleClickItem = item
        }
      } else if (item.children && item.opened !== undefined) {
        item.opened = !item.opened
      }
    },
    checkItem (item, check) {
      if (check) {
        this.checkedItems.push(item)
      } else {
        this.checkedItems.splice(this.checkedItems.indexOf(item), 1)
      }
    },
    toggleNonSemantic () {
      this.rootGroups = []
      this.rootItems = []
      this.includeNonSemantic = !this.includeNonSemantic
      this.load()
    },
    toggleItemName () {
      this.includeItemName = !this.includeItemName
      this.$f7.data.modelPicker.includeItemName = this.includeItemName
      this.load()
    },
    toggleItemTags () {
      this.includeItemTags = !this.includeItemTags
      this.$f7.data.modelPicker.includeItemTags = this.includeItemTags
      this.load()
    },
    toggleExpanded () {
      this.expanded = !this.expanded
      this.$f7.data.modelPicker.expanded = this.expanded
      this.applyExpandedOption()
    }
  }
}
</script>
