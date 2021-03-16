<template>
<div class="tooltip"
@mouseover="text()"

>
  <div
    class="picture tooltip div"
    :style="cssStyleProperty"
    @mouseover="updateMouseCursor"
    @keydown.delete="deleteItem"
    @click="imageClick, changeStatus()"
    ref="imageRef1"
    @mouseleave="changeStatusAgain()"
  >
   <pre v-if="this.status && this.properties.ControlTipText" style="position:fixed; top:clientX,left:clientY"><div id="divContainer"  class="tooltiptext" :input="properties.ControlTipText" style="position:sticky;width:auto;left:e.pageX ">{{properties.ControlTipText}}</div></pre>
  </div>
  </div>
</template>

<script lang="ts">
import {
  Component,
  Mixins,
  Watch,
  Ref
} from 'vue-property-decorator'
import FdControlVue from '@/api/abstract/FormDesigner/FdControlVue'
import $ from 'jquery'

@Component({
  name: 'FDImage'
})
export default class FDImage extends Mixins(FdControlVue) {
  @Ref('imageRef1') imageRef1: HTMLDivElement;
  $el!: HTMLDivElement;
  /**
   * @description updating width and height of image control
   * @function updateAutoSize
   * @param oldVal previous AutoSize value
   * @param newVal  new/changed AutoSize value
   */
  @Watch('properties.AutoSize')
  updateAutoSize (newVal: boolean, oldVal: boolean) {
    this.setHeightWidth(newVal)
  }

  @Watch('properties.Picture')
  pictureChanges (newVal: string, oldVal: string) {
    this.setHeightWidth(this.properties.AutoSize!)
  }
  /**
   * @description When AutoSize is true update width and height property
   * @function setHeightWidth
   * @param isAutoSize AutoSize property value
   *
   */
  setHeightWidth (isAutoSize: boolean) {
    const that = this
    if (isAutoSize && this.properties.Picture) {
      const img = new Image()
      img.onload = function () {
        that.updateDataModel({
          propertyName: 'Height',
          value: img.height
        })
        that.updateDataModel({
          propertyName: 'Width',
          value: img.width
        })
      }
      img.src = this.properties.Picture
    } else {
      return undefined
    }
  }
  /**
   * @description style object is passed to :style attribute in label tag
   * dynamically changing the styles of the component based on propControlData
   * @function cssStyleProperty
   *
   */
  protected get cssStyleProperty (): Partial<CSSStyleDeclaration> {
    const controlProp = this.properties
    let display = ''
    if (this.isRunMode) {
      display = controlProp.Visible ? controlProp.Width === 0 || controlProp.Height === 0 ? 'none' : 'block' : 'none'
    } else {
      display = controlProp.Width === 0 || controlProp.Height === 0 ? 'none' : 'block'
    }
    let borderStyles = {}
    if (controlProp.SpecialEffect !== 0) {
      borderStyles = {
        borderStyle: controlProp.SpecialEffect === 3 || controlProp.SpecialEffect === 4 ? 'solid' : controlProp.SpecialEffect === 1 ? 'outset' : controlProp.SpecialEffect === 2 ? 'inset' : '',
        borderLeftColor: controlProp.SpecialEffect === 1 ? 'white' : controlProp.SpecialEffect === 3 ? 'rgb(169,169,169)' : controlProp.SpecialEffect === 4 ? 'black' : '',
        borderTopColor: controlProp.SpecialEffect === 1 ? 'white' : controlProp.SpecialEffect === 3 ? 'rgb(169,169,169)' : controlProp.SpecialEffect === 4 ? 'black' : '',
        borderRightColor: controlProp.SpecialEffect === 2 ? 'white' : controlProp.SpecialEffect === 3 ? 'rgb(169,169,169)' : controlProp.SpecialEffect === 4 ? 'black' : '',
        borderBottomColor: controlProp.SpecialEffect === 2 ? 'white' : controlProp.SpecialEffect === 3 ? 'rgb(169,169,169)' : controlProp.SpecialEffect === 4 ? 'black' : '',
        borderLeftWidth: controlProp.SpecialEffect === 1 ? '2px' : controlProp.SpecialEffect === 2 ? '3px' : controlProp.SpecialEffect === 3 ? '1px' : controlProp.SpecialEffect === 4 ? '1px' : '',
        borderTopWidth: controlProp.SpecialEffect === 1 ? '2px' : controlProp.SpecialEffect === 2 ? '3px' : controlProp.SpecialEffect === 3 ? '1px' : controlProp.SpecialEffect === 4 ? '1px' : '',
        borderRightWidth: controlProp.SpecialEffect === 2 ? '2px' : controlProp.SpecialEffect === 1 ? '3px' : controlProp.SpecialEffect === 3 ? '1px' : controlProp.SpecialEffect === 4 ? '1px' : '',
        borderBottomWidth: controlProp.SpecialEffect === 2 ? '2px' : controlProp.SpecialEffect === 1 ? '3px' : controlProp.SpecialEffect === 3 ? '1px' : controlProp.SpecialEffect === 4 ? '1px' : ''
      }
    } else if (controlProp.BorderStyle !== 0) {
      borderStyles = {
        borderLeft: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : '',
        borderRight: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : '',
        borderTop: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : '',
        borderBottom: controlProp.BorderStyle === 1 ? '1px solid ' + controlProp.BorderColor : ''
      }
    }
    return {
      ...this.baseStyle,
      ...borderStyles,
      left: `${controlProp.Left}px`,
      width: `${controlProp.Width}px`,
      height: `${controlProp.Height}px`,
      top: `${controlProp.Top}px`,
      backgroundColor: controlProp.BackStyle
        ? controlProp.BackColor
        : 'transparent',
      cursor: this.controlCursor,
      backgroundImage: `url(${controlProp.Picture})`,
      backgroundSize: controlProp.Picture === '' ? '' : this.getSizeMode,
      backgroundRepeat: this.getRepeatData,
      backgroundPosition: controlProp.Picture === '' ? '' : this.getPosition,
      display: display
    }
  }
  imageClick (event: MouseEvent) {
    if (this.toolBoxSelectControl === 'Select') {
      event.stopPropagation()
    }
  }
  text (e:MouseEvent) {
    const myRef1 = this.imageRef1
    console.log(this.properties.ControlTipText)
    debugger
    $(myRef1).mouseover(function () {
      $(myRef1).mouseover(function (e) {
        debugger
        $('.tooltiptext')
          .css({
            position: 'absolute',
            left: e.offsetX,
            top: e.offsetY + 20
          })
        console.log(e.pageX)
        console.log(e.pageY)
      })
    })
  }
   status = true
   changeStatus () {
     this.status = false
   }
   changeStatusAgain () {
     this.status = true
   }
}
</script>

<style scoped>
.tooltip {
  display: inline;
  position: sticky;
  }
.tooltip .tooltiptext {
  visibility: hidden;
  position:absolute;
  color: #fff;
  text-align: center;
  border: rgb(17, 17, 17)  solid 1px;
  padding:  0;
  background-color: white;
  font-size: 10px;
  height: 15px;
  padding: 4px;
color: black;
width: auto;
  /* Position the tooltip */
  }
  .tooltip:hover .tooltiptext {
  visibility: visible;
  position: static;
  top:0px;
  left:0px;
}
.picture {
  /* position: relative; */
  overflow: hidden;
  box-sizing: border-box;
  width: 0px;
  height: 0px;
  left: 0px;
  top: 0px;
}
</style>
