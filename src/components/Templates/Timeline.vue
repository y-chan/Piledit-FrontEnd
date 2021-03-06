<template>
  <svg
    id="timeline"
    width="2200"
    height="500"
    @mousedown.left="mouseDown"
    @mousemove.left="mouseMove"
    @mouseup.left="mouseUp"
    @click.right.stop="popupContextMenu"
  >
    <g :transform="`translate(${this.timelinePositionX}, 0)`">
      <rect
        x="1"
        y="1"
        :width="width"
        height="620"
        fill="none"
        stroke="black"
        stroke-width="1"
      />
      <SVGText
        v-for="i in 20"
        :key="`SVGText-${i}`"
        :x="`${10 + (i-1) * 200}`"
        y="16"
        color="black"
      >
        {{ getTimeDisplay(i) }}
      </SVGText>
      <g transform="translate(0, 20)">
        <ClipOnLayer
          v-for="(clip, uuid, index) in clips"
          :key="`Clip-${uuid}-${index}`"
          :clip="clip"
          @updatePosition="updatePosition"
          @updateWidth="updateWidth"
          @removeClip="removeClip"
        />
        <line
          v-for="i in 10"
          :key="i"
          x1="1"
          :y1="getLinePosY(i)"
          :x2="width + 1"
          :y2="getLinePosY(i)"
          stroke="black"
          stroke-width="1"
        />
      </g>
    </g>
  </svg>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import { clipsModule, projectsModule } from '@/store/store'
import { PClips, PPosition, PComponent } from '@/@types/piledit'
import { remote } from 'electron'
import SVGText from '@/components/Atoms/SVGText.vue'
import ClipOnLayer from '@/components/Molecules/ClipOnLayer.vue'
const Menu = remote.Menu
const MenuItem = remote.MenuItem

@Component({
  components: {
    SVGText,
    ClipOnLayer
  }
})
export default class Timeline extends Vue {
  @Prop({ required: true })
  public clips!: PClips

  @Prop({ required: true })
  public components!: { [key: string]: PComponent }

  public width = 4000
  public beforeMouseX = 0
  public beforeMouseY = 0
  public timelinePositionX = 0
  public isDragging = false

  public getTimeDisplay (seconds: number) {
    const minutes = Math.floor(seconds / 60)
    seconds -= minutes * 60
    const displayMinutes = minutes >= 10 ? minutes.toString() : '0' + minutes.toString()
    const displaySeconds = seconds >= 10 ? seconds.toString() : '0' + seconds.toString()
    return `${displayMinutes}:${displaySeconds}:00`
  }

  public getLinePosY (i: number) {
    return (i - 1) * 50 + 1
  }

  public popupContextMenu (event: any) {
    event.preventDefault()
    const menu = this.buildContextMenu()
    const currentWindow = remote.getCurrentWindow()
    menu.popup({ window: currentWindow })
  }

  public buildContextMenu () {
    const menu = new Menu()
    for (const [key, value] of Object.entries(this.components)) {
      menu.append(
        new MenuItem(({
          label: value.name || value.defaultName,
          click: () => this.setClip(key)
        }))
      )
    }
    return menu
  }

  public setClip (componentUuid: string) {
    const projectUuid = projectsModule.currentViewingProjectUuid
    clipsModule.add({ componentUuid, projectUuid })
  }

  public mouseDown (event: MouseEvent) {
    this.isDragging = true
    this.beforeMouseX = event.offsetX
    event.preventDefault()
  }

  public mouseMove (event: MouseEvent) {
    if (!this.isDragging) return
    const res = this.timelinePositionX + event.offsetX - this.beforeMouseX
    if (res <= 0 && res >= -this.width + 1500) {
      this.timelinePositionX = res
    }
    this.beforeMouseX = event.offsetX
    event.preventDefault()
  }

  public mouseUp (event: MouseEvent) {
    this.isDragging = false
    this.beforeMouseX = 0
    this.beforeMouseY = 0
    event.preventDefault()
  }

  public updatePosition (context: { position: PPosition; uuid: string }) {
    clipsModule.updatePosition(context)
  }

  public updateWidth (context: { width: number; uuid: string }) {
    clipsModule.updateWidth(context)
  }

  public removeClip (uuid: string) {
    clipsModule.removeClip(uuid)
  }
}
</script>

<style scoped>
</style>
