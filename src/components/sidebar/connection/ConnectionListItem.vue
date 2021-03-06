<template>
  <div class="list-item" :title="title">
    <a
      href=""
      class="list-item-btn"
      :class="classList"
      @click.prevent="click(config)"
      @dblclick.prevent="doubleClick(config)" 
    >
      <span :class="`connection-label connection-label-color-${labelColor}`"></span>
      <div class="connection-title flex-col expand">
        <span class="title">{{label}}</span>
        <span class="subtitle">{{subtitle}}</span>
      </div>
      <span class="badge"><span>{{config.connectionType}}</span></span>
      <x-button class="btn-fab" skin="iconic">
        <i class="material-icons">more_horiz</i>
        <x-menu style="--target-align: right; --v-target-align: top;">
          <x-menuitem @click.prevent.stop="remove(config)">
            <x-label class="text-danger">Remove</x-label>
          </x-menuitem>
        </x-menu>
      </x-button>
    </a>
  </div>
</template>
<script>
import path from 'path'
import TimeAgo from 'javascript-time-ago'
export default {
  // recent list is 'recent connections'
  // if that is true, we need to find the companion saved connection
  props: ['config', 'isRecentList', 'selectedConfig'],
  data: () => ({
    timeAgo: new TimeAgo('en-US'),
    split: null
  }),
  computed: {
    classList() {
      return {
        'active': this.savedConnection && this.selectedConfig ? this.savedConnection.id === this.selectedConfig.id : false
      }
    },
    labelColor() {
      return this.savedConnection ? this.savedConnection.labelColor : 'default'
    },
    label() {
      if (this.savedConnection) {
        return this.savedConnection.name
      }
      if (this.config.connectionType === 'sqlite') {
        return path.basename(this.config.defaultDatabase)
      } else {
        return this.config.simpleConnectionString
      }
      
    },
    subtitle() {
      if (this.isRecentList) {
        return this.timeAgo.format(this.config.updatedAt)
      } else {
        return this.config.simpleConnectionString
      }
    },
    title() {
      return this.config.fullConnectionString
    },
    savedConnection() {

      if (this.isRecentList) {
        if (!this.config.savedConnectionId) return null
        return this.$store.state.connectionConfigs.find(c => c.id === this.config.savedConnectionId)
      } else {
        return this.config
      }
    },
  },
  mounted() {

  },
  methods: {
    click() {
      if (this.savedConnection) {
        this.$emit('edit', this.savedConnection)
      } else {
        this.$emit('edit', this.config.toNewConnection())
      }
    },
    doubleClick() {
      if (this.savedConnection) {
        this.$emit('doubleClick', this.savedConnection)
      } else {
        this.$emit('doubleClick', this.config.toNewConnection())
      }
    },
    remove() {
      this.$emit('remove', this.config)
    }
  }

}
</script>