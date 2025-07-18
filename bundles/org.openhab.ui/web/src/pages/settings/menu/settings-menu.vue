<template>
  <f7-page @page:init="onPageInit" @page:afterin="onPageAfterIn" class="page-settings">
    <f7-navbar large :large-transparent="false" title-large="Settings" title="Settings" back-link="Back" back-link-url="/" back-link-force>
      <f7-nav-right>
        <developer-dock-icon />
        <f7-link
          class="searchbar-enable"
          data-searchbar=".searchbar-demo"
          icon-ios="f7:search_strong"
          icon-aurora="f7:search_strong"
          icon-md="material:search" />
      </f7-nav-right>
      <f7-searchbar
        class="searchbar-demo"
        expandable
        search-container=".search-list"
        search-in=".item-title"
        @searchbar:search="expandAll"
        :disable-button="!$theme.aurora" />
    </f7-navbar>
    <f7-block class="block-narrow after-big-title settings-menu">
      <f7-row>
        <f7-col :class="!addonsLoaded || (addonsLoaded && addonsInstalled.length > 0) ? 'settings-col' : ''" width="100" medium="50">
          <f7-block-title>Configuration</f7-block-title>
          <f7-list media-list class="search-list">
            <f7-list-item
              v-if="$store.getters.apiEndpoint('things')"
              media-item
              link="things/"
              title="Things"
              :badge="(inboxCount > 0) ? inboxCount : undefined"
              :after="(inboxCount > 0) ? (thingsCount + '+') : thingsCount"
              :badge-color="inboxCount ? 'red' : 'blue'"
              :footer="objectsSubtitles.things">
              <f7-icon slot="media" f7="lightbulb" color="gray" />
            </f7-list-item>
            <f7-list-item
              v-if="$store.getters.apiEndpoint('items')"
              media-item
              link="model/"
              title="Model"
              badge-color="blue"
              :footer="objectsSubtitles.model">
              <f7-icon slot="media" f7="list_bullet_indent" color="gray" />
            </f7-list-item>
            <f7-list-item
              v-if="$store.getters.apiEndpoint('items')"
              media-item
              link="items/"
              title="Items"
              :after="itemsCount"
              badge-color="blue"
              :footer="objectsSubtitles.items">
              <f7-icon slot="media" f7="square_on_circle" color="gray" />
            </f7-list-item>
            <f7-list-item
              v-if="$store.getters.apiEndpoint('ui')"
              link="pages/"
              title="Pages"
              :after="$store.getters.pages.length + sitemapsCount"
              badge-color="blue"
              :footer="objectsSubtitles.pages">
              <f7-icon slot="media" f7="tv" color="gray" />
            </f7-list-item>
          </f7-list>
          <f7-list media-list class="search-list">
            <f7-list-item
              v-if="$store.getters.apiEndpoint('transformations')"
              media-item
              link="transformations/"
              title="Transformations"
              :after="transformationsCount"
              badge-color="blue"
              :footer="objectsSubtitles.transform">
              <f7-icon slot="media" f7="function" color="gray" />
            </f7-list-item>
            <f7-list-item
              media-item
              link="persistence/"
              title="Persistence"
              badge-color="blue"
              :footer="objectsSubtitles.persistence">
              <f7-icon slot="media" f7="download_circle" color="gray" />
            </f7-list-item>
          </f7-list>
          <f7-block-title v-if="$store.getters.apiEndpoint('rules')">
            Automation
          </f7-block-title>
          <f7-list media-list class="search-list">
            <f7-list-item
              media-item
              link="rules/"
              title="Rules"
              :after="rulesCount"
              badge-color="blue"
              :footer="objectsSubtitles.rules">
              <f7-icon slot="media" f7="wand_stars" color="gray" />
            </f7-list-item>
            <f7-list-item
              media-item
              link="scenes/"
              title="Scenes"
              :after="scenesCount"
              badge-color="blue"
              :footer="objectsSubtitles.scenes">
              <f7-icon slot="media" f7="film" color="gray" />
            </f7-list-item>
            <f7-list-item
              media-item
              link="scripts/"
              title="Scripts"
              :after="scriptsCount"
              badge-color="blue"
              :footer="objectsSubtitles.scripts">
              <f7-icon slot="media" f7="doc_plaintext" color="gray" />
            </f7-list-item>
            <f7-list-item
              media-item
              link="schedule/"
              title="Schedule"
              badge-color="blue"
              :footer="objectsSubtitles.schedule">
              <f7-icon slot="media" f7="calendar" color="gray" />
            </f7-list-item>
          </f7-list>
        </f7-col>
        <f7-col :class="!addonsLoaded || (addonsLoaded && addonsInstalled.length > 0) ? 'settings-col' : ''" width="100" medium="50">
          <div v-show="servicesLoaded">
            <f7-block-title>System Settings</f7-block-title>
            <f7-list class="search-list">
              <f7-list-item
                v-for="service in systemSettings"
                :key="service.id"
                :link="'services/' + service.id"
                :title="service.label"
                v-show="!service.hidden" />
              <f7-list-button v-if="!expanded('systemSettings')" color="blue" @click="expand('systemSettings')">
                {{ $t('dialogs.showAll') }}
              </f7-list-button>
            </f7-list>
          </div>
          <!-- skeleton for not servicesLoaded -->
          <div v-if="!servicesLoaded">
            <f7-block-title>System Settings</f7-block-title>
            <f7-list>
              <f7-list-item
                v-for="n in 9"
                :key="n"
                :class="`skeleton-text skeleton-effect-blink`"
                title="Service Label" />
            </f7-list>
          </div>
          <div v-show="$f7.width < 1450">
            <div v-show="addonsLoaded && addonsInstalled.length > 0">
              <addon-section class="add-on-section" :addonsInstalled="addonsInstalled" :addonsServices="addonsServices" :expanded="expanded('addonsSettings')" @expand="expand('addonsSettings')" />
            </div>
            <!-- skeleton for not addonsLoaded -->
            <div v-if="!addonsLoaded">
              <f7-block-title>Add-on Settings</f7-block-title>
              <f7-list>
                <f7-list-item
                  v-for="n in 4"
                  :key="n"
                  :class="`skeleton-text skeleton-effect-blink`"
                  title="Service Label" />
              </f7-list>
            </div>
          </div>
        </f7-col>
        <f7-col width="33" class="add-on-col" v-show="$f7.width >= 1450">
          <div v-show="addonsLoaded && addonsInstalled.length > 0">
            <addon-section :addonsInstalled="addonsInstalled" :addonsServices="addonsServices" :expanded="expanded('addonsSettings')" @expand="expand('addonsSettings')" />
          </div>
          <!-- skeleton for not addonsLoaded -->
          <div v-if="!addonsLoaded">
            <f7-block-title>Add-on Settings</f7-block-title>
            <f7-list>
              <f7-list-item
                v-for="n in 9"
                :key="n"
                :class="`skeleton-text skeleton-effect-blink`"
                title="Service Label" />
            </f7-list>
          </div>
        </f7-col>
      </f7-row>
      <f7-block-footer v-if="$t('home.overview.title') !== 'Overview'" class="margin text-align-center">
        <small v-t="'admin.notTranslatedYet'" />
      </f7-block-footer>
    </f7-block>

    <f7-fab v-if="healthCount > 0" position="center-bottom" :text="`Health Issues (${healthCount})`" slot="fixed" color="red" href="health/">
      <f7-icon f7="heart" />
    </f7-fab>
  </f7-page>
</template>

<script>
import AddonSection from './addon-section.vue'

export default {
  components: {
    AddonSection
  },
  data () {
    return {
      addonsLoaded: false,
      servicesLoaded: false,
      addonsInstalled: [],
      persistenceAddonsInstalled: [],
      addonsServices: [],
      systemServices: [],
      objectsSubtitles: {
        health: 'Manage detected system health issues',
        things: 'Manage the physical layer',
        model: 'The semantic model of your home',
        items: 'Manage the functional layer',
        pages: 'Design displays for user interaction',
        transform: 'Make raw data human-readable',
        persistence: 'How to keep data for future use',
        rules: 'Automate with triggers and actions',
        scenes: 'Store a set of desired states to recall',
        scripts: 'Rules dedicated to running code',
        schedule: 'View upcoming time-based rules'
      },
      inboxCount: '',
      thingsCount: '',
      itemsCount: '',
      transformationsCount: '',
      sitemapsCount: 0,
      rulesCount: '',
      scenesCount: '',
      scriptsCount: '',

      orphanLinkCount: 0,
      semanticsProblemCount: 0,

      advancedSystemServices: [
        'org.openhab.storage.json',
        'org.openhab.restauth',
        'org.openhab.addons',
        'org.openhab.marketplace',
        'org.openhab.jsonaddonservice',
        'org.openhab.inbox',
        'org.openhab.sitemap',
        'org.openhab.lsp'
      ],

      expandedTypes: {
        systemSettings: this.$f7.width >= 1450,
        addonsSettings: false
      }
    }
  },
  computed: {
    apiEndpoints () {
      return this.$store.state.apiEndpoints
    },
    systemSettings () {
      if (this.expandedTypes.systemSettings) return this.systemServices
      return this.systemServices.map((service) => {
        const hide = this.advancedSystemServices.includes(service.id)
        return Object.assign({ hidden: hide }, service)
      })
    },
    healthCount () {
      const problemCount = this.orphanLinkCount + this.semanticsProblemCount
      return problemCount.toString()
    }
  },
  watch: {
    apiEndpoints () {
      this.loadMenu()
      this.loadCounters()
    }
  },
  methods: {
    loadMenu () {
      if (!this.apiEndpoints) return

      // can be done in parallel!
      if (this.$store.getters.apiEndpoint('services')) {
        this.$oh.api.get('/rest/services').then((data) => {
          this.systemServices = data
            .filter(s => (s.category === 'system') && (s.id !== 'org.openhab.persistence'))
            .sort((s1, s2) => this.sortByLabel(s1, s2))
          this.addonsServices = data.filter(s => s.category !== 'system').sort((s1, s2) => this.sortByLabel(s1, s2))
          this.servicesLoaded = true
        })
      }
      if (this.$store.getters.apiEndpoint('addons')) {
        this.$oh.api.get('/rest/addons?serviceId=all').then((data) => {
          this.addonsInstalled = data
            .filter(a => a.installed && !['application/vnd.openhab.ruletemplate', 'application/vnd.openhab.uicomponent;type=widget', 'application/vnd.openhab.uicomponent;type=blocks'].includes(a.contentType))
            .sort((s1, s2) => this.sortByLabel(s1, s2))
          this.persistenceAddonsInstalled = this.addonsInstalled.filter(a => a.installed && a.type === 'persistence')
          this.addonsLoaded = true
        })
      }
    },
    sortByLabel (s1, s2) {
      return s1.label.toLowerCase() > s2.label.toLowerCase() ? 1 : -1
    },
    loadCounters () {
      if (!this.apiEndpoints) return
      if (this.$store.getters.apiEndpoint('links')) this.$oh.api.get('/rest/links/orphans').then((data) => { this.orphanLinkCount = data.length })
      if (this.$store.getters.apiEndpoint('items')) this.$oh.api.get('/rest/items/semantics/health').then((data) => { this.semanticsProblemCount = data.length })
      if (this.$store.getters.apiEndpoint('inbox')) this.$oh.api.get('/rest/inbox?includeIgnored=false').then((data) => { this.inboxCount = data.filter((e) => e.flag === 'NEW').length.toString() })
      if (this.$store.getters.apiEndpoint('things')) this.$oh.api.get('/rest/things?staticDataOnly=true').then((data) => { this.thingsCount = data.length.toString() })
      if (this.$store.getters.apiEndpoint('items')) this.$oh.api.get('/rest/items?staticDataOnly=true').then((data) => { this.itemsCount = data.length.toString() })
      if (this.$store.getters.apiEndpoint('ui')) this.$oh.api.get('/rest/ui/components/system:sitemap').then((data) => { this.sitemapsCount = data.length })
      if (this.$store.getters.apiEndpoint('transformations')) this.$oh.api.get('/rest/transformations').then((data) => { this.transformationsCount = data.length.toString() })
      if (this.$store.getters.apiEndpoint('rules')) {
        this.$oh.api.get('/rest/rules?staticDataOnly=true').then((data) => {
          this.rulesCount = data.filter((r) => r.tags.indexOf('Scene') < 0 && r.tags.indexOf('Script') < 0).length.toString()
          this.scenesCount = data.filter((r) => r.tags.indexOf('Scene') >= 0).length.toString()
          this.scriptsCount = data.filter((r) => r.tags.indexOf('Script') >= 0).length.toString()
        })
      }
    },
    expand (type) {
      this.$set(this.expandedTypes, type, true)
    },
    expanded (type) {
      return this.expandedTypes[type] || (this[type] && this[type].length <= 5)
    },
    expandAll () {
      Object.keys(this.expandedTypes).forEach((type) => this.expand(type))
    },
    onPageInit () {
      this.loadMenu()
    },
    onPageAfterIn () {
      // this.loadMenu()
      this.loadCounters()
    }
  }
}
</script>

<style lang="stylus">
.device-desktop .settings-menu
  --f7-list-item-footer-line-height 1.3
  @media (min-width 1450px)
    .row
      width 1065px
      max-width 100%
    .settings-col
      width 33%
.settings-menu .icon
  color var(--f7-color-blue)
.theme-filled .settings-menu .icon
  color var(--f7-color-gray) !important
.aurora .settings-menu .icon
  font-size 24px
</style>
