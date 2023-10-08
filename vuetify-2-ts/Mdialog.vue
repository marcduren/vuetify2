<template>
  <div>
    <div
      v-if="modelValue"
      disabled
      style="position: fixed; left: 0; right: 0; top: 0; bottom: 0; background-color: #000; opacity: 0.2; z-index: 100"
      @click="onClickOut"
    ></div>
    <div
      v-if="modelValue"
      class="elevation-6"
      style="position: fixed; z-index: 101"
      :style="{ width: width + 'px', height: pageHeight, left: left + 'px', top: top + 'px' }"
      :id="unique_id"
    >
      <div style="display: flex; flex-direction: column; height: 100%">
        <v-toolbar
          class="shrink deplacable"
          :color="couleurTitre"
          :light="themeTitre == 'light'"
          :dark="themeTitre == 'dark'"
          dense
          flat
          @mousedown="handleMouseDown"
        >
          <v-toolbar-title class="text-center" style="width: 100%">{{ titre }}</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon @click="onFermer"><v-icon>mdi-close</v-icon></v-btn>
        </v-toolbar>
        <div class="modal-body grow" :class="[couleur]" style="overflow: auto">
          <div class="fill-height" :class="defaultpadding"><slot></slot></div>
        </div>
        <template v-if="boutons.length > 0">
          <div class="shrink pa-2 d-flex dlg-actions" :class="couleur">
            <v-btn class="mx-1" rounded text color="error" v-if="boutons.includes('supprimer')" @click="onSupprimer">Supprimer</v-btn>
            <v-spacer></v-spacer>
            <v-btn class="mx-1" rounded depressed color="grey lighten-2" v-if="boutons.includes('fermer')" @click="onFermer">Fermer</v-btn>
            <v-btn class="mx-1" rounded depressed color="grey lighten-2" v-if="boutons.includes('annuler')" @click="onFermer">Annuler</v-btn>
            <v-btn class="mx-1" rounded depressed color="primary" v-if="boutons.includes('valider')" @click="onValider" :disabled="!validerActif"
              >Valider</v-btn
            >
          </div>
        </template>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  name: 'm-dialog',
  props: {
    value: Boolean,
    titre: {
      type: String,
      required: true
    },
    boutons: {
      type: Array,
      required: false,
      default: function () {
        return ['valider', 'annuler'] /** valider,annuler,fermer,supprimer */
      }
    },
    width: {
      type: Number,
      default: 900
    },
    couleur: {
      type: String,
      default: 'grey lighten-3'
    },
    couleurTitre: {
      type: String,
      default: 'primary'
    },
    themeTitre: {
      type: String,
      default: 'dark'
    },
    validerActif: {
      type: Boolean,
      default: true
    },
    defaultpadding: { type: String, default: 'pa-3' }
  },
  data() {
    return {
      left: 100,
      top: 20,
      xdown: 0,
      ydown: 0,
      modelValue: this.value,
      maxHeight: 1000,
      pageHeight: 'inherit',
      unique_id: '1'
    }
  },
  watch: {
    value() {
      this.modelValue = this.value
    }
  },
  methods: {
    onFermer() {
      this.$emit('fermer')
    },
    onValider() {
      this.$emit('valider')
    },
    onSupprimer() {
      this.$emit('supprimer')
    },
    onClickOut() {
      if ('persistent' in this.$attrs) return
      this.onFermer()
    },
    handleMouseDown(e: MouseEvent) {
      // To prevent selection while moving cursor
      e.preventDefault()

      this.xdown = e.screenX - this.left
      this.ydown = e.screenY - this.top
      window.addEventListener('mousemove', this.handleMouseMove, false)
      window.addEventListener('mouseup', this.handleMouseUp, false)
    },
    handleMouseMove(e: MouseEvent) {
      this.left = e.screenX - this.xdown
      this.top = e.screenY - this.ydown
      if (this.top < 0) this.top = 0
      if (this.top + 100 > window.innerHeight) this.top = window.innerHeight - 100
      if (this.left < 0) this.left = 0
      if (this.left + this.width > window.innerWidth) this.left = window.innerWidth - this.width
    },
    handleMouseUp() {
      window.removeEventListener('mousemove', this.handleMouseMove, false)
      window.removeEventListener('mouseup', this.handleMouseUp, false)
    },
    centrer() {
      const vnode = document.body
      const rect = vnode.getBoundingClientRect()
      this.left = (rect.width - Number(this.width)) / 2
      const fen = document.getElementById(this.unique_id)
      if (fen) {
        const rectDlg = fen?.getBoundingClientRect()
        this.top = (rect.height - Number(rectDlg?.height)) / 2
      } else {
        this.top = 10
      }
    },
    changeHauteur() {
      if (this.pageHeight == 'inherit') {
        this.top = 0
        const vnode = document.body
        const rect = vnode.getBoundingClientRect()
        this.pageHeight = rect.height - 20 + 'px'
      } else {
        this.pageHeight = 'inherit'
      }
    }
  },
  beforeMount() {
    this.unique_id = `${new Date().getTime()}-${Math.random().toString(36).substring(7)}`
  },
  mounted() {
    if (this.$attrs['height']) {
      this.pageHeight = this.$attrs['height'] + 'px'
    }
    const vnode = document.body
    const rect = vnode.getBoundingClientRect()
    this.maxHeight = rect.height - 100
    this.centrer()
  }
})
</script>
<style scoped>
.deplacable {
  cursor: grab;
}
.v-application .dlg-actions {
  border-top: 1px solid #d0d0d0 !important;
}
</style>