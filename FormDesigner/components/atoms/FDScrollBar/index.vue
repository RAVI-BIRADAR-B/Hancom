<template style="position:relative;z-index:6 ">
<div  class="" style="position:relative;z-index:7" >
  <div
  class=""
  ref="scrollbar"
  v-on="eventStoppers()"
  @click="scrollBarClick,changeStatus()"
  :style="outerScrollBarDivObj"
  @mouseover="updateMouseCursor"
  @keydown.enter="setContentEditable($event, true)"
  @keydown.esc="setContentEditable($event, false)"
  :tabindex="0"
  @mousedown="controlEditMode"
  >

    <div class="slidecontainer " :style="cssVars">

      <button :style="scrollBarButtonStyleObj"
      @mouseover="updateMouseCursor, text()"
      @mousedown="!getDisableValue?properties.Min > properties.Max ? increaseTheValue() : decreaseTheValue():''"
      @mouseup="setIsSpinButtonScrollBarMouseDown"
      @mouseout="setIsSpinButtonScrollBarMouseDown"
      id="abc"
      class="tooltip"
      style="position:relative;z-index:15"
      >
          <pre style="position:fixed;z-index:50;margin-top:-40px;overflow:visible;clear:both"><div id="divContainer"  class="tooltiptext" :input="properties.ControlTipText"  >{{this.properties.ControlTipText}}</div></pre>

        <FdSvgImage
          key="leftArrow"
          name="left-arrow.svg"
          @hook:mounted="changeForeColor"
          class="svgLeftRightStyle "
          :style="svgLeftRightStyleObj"
          @mouseover="text()"
          style="position:fixed;z-index:10"
        />

      </button>
      <div class="tooltip" style="position:relative;z-index:11">
         <pre style="position:fixed;z-index:50;" ><div id="divContainer"  class="tooltiptext" :input="properties.ControlTipText"  >{{this.properties.ControlTipText}}</div></pre>

      <input
        :disabled="getDisableValue"
        type="range"
        :min="properties.Min > properties.Max ? properties.Max : properties.Min"
        :max="properties.Min > properties.Max ? properties.Min : properties.Max"
        :value="properties.Value"
        class="slider "
        :style="inputStyleObj"
        @input="updateValueProperty"
        orient="vertical"
        @mouseover="updateMouseCursor, text"
        style="posittion:relative;z-index:9"
      />

      </div>
      <button :style="scrollBarButtonStyleObj"
      @mouseover="updateMouseCursor, text()"
      @mousedown="!getDisableValue?properties.Min > properties.Max ? decreaseTheValue() : increaseTheValue():''"
      @mouseup="setIsSpinButtonScrollBarMouseDown"
      @mouseout="setIsSpinButtonScrollBarMouseDown"
      id="abcd"
      class="tooltip"
      style="position:relative;z-index:8"
      >
         <pre  style="position:fixed;z-index:50;margin-top:-40px;clear:both" ><div id="divContainer"  class="tooltiptext" :input="properties.ControlTipText"  >{{this.properties.ControlTipText}}</div></pre>

        <FdSvgImage
          key="rightArrow"
          name="right-arrow.svg"
          @hook:mounted="changeForeColor"
          class="svgLeftRightStyle  "
          :style="svgLeftRightStyleObj"
          style="position:relative;z-index:14"
        />

      </button>

    </div>

</div>

  </div>
</template>

<script lang="ts">
import { Component, Emit, Mixins, Watch, Ref } from 'vue-property-decorator'
import FdControlVue from '@/api/abstract/FormDesigner/FdControlVue'
import FdSvgImage from '@/FormDesigner/components/atoms/FDSVGImage/index.vue'
import { controlProperties } from '@/FormDesigner/controls-properties'
import $ from 'jquery'
import { orientation } from '@/FormDesigner/controls-select-types'

@Component({
  name: 'FDScrollBar',
  components: {
    FdSvgImage
  }
})
export default class FDScrollBar extends Mixins(FdControlVue) {
  @Ref('scrollbar') scrollbar: HTMLDivElement;

  $el: HTMLDivElement
  isInvert: boolean = false
  intervalVariable: number = 0
  thumbHeight: string = ''
  minHeight: string = ''
  updateValueProperty (e: Event) {
    if (e.target instanceof HTMLInputElement) {
      const targetValue = parseInt(e.target!.value)
      this.updateDataModel({ propertyName: 'Value', value: targetValue })
    }
  }

  get svgLeftRightStyleObj () {
    const controlProp = this.properties
    return {
      position: 'relative',
      top: this.svgWidthCalc() + 'px'
    }
  }

  svgWidthCalc () {
    if (this.properties.Width! < 20 && this.properties.Width! >= 10) {
      return ((this.properties.Width! - 20) / 2)
    } else if (this.properties.Width! < 10) {
      return -5
    } else if (this.properties.Height! < 20 && this.properties.Height! >= 10) {
      return ((this.properties.Height! - 20) / 2)
    } else if (this.properties.Width! < 10) {
      return -5
    }
  }

  get cssVars () {
    const controlProp = this.properties
    return {
      position: 'relative',
      left: '0px',
      top: (this.properties.Min! > this.properties.Max!) ? this.checkOtherOrientations() ? `${controlProp.Height!}px` : '0px' : '0px',
      gridTemplateColumns: this.gridTemplateColumnsCalculate(),
      '--bg-color': this.properties.BackColor,
      '--height': this.checkOtherOrientations() ? `${this.properties.Width!}px` : `${this.properties.Height!}px`,
      transform: (this.properties.Min! > this.properties.Max!) ? this.scrollReAlign() : this.checkOtherOrientations() ? 'rotate(90deg)' : '',
      transformOrigin: (this.properties.Min! > this.properties.Max!) ? this.checkOtherOrientations() ? '0% 0%' : '' : '',
      '--invertValue': this.isEditMode ? this.isInvert ? '1' : '0' : '0',
      '--thumbHeight': this.thumbHeight,
      '--minHeight': this.minHeight
    }
  }

  gridTemplateColumnsCalculate () {
    const controlProp = this.properties
    if (this.checkOtherOrientations()) {
      if (controlProp.Height! <= 40) {
        return `${controlProp.Height! / 2}px` + ' 0px ' + `${controlProp.Height! / 2}px`
      } else if (controlProp.Height! > 40 && controlProp.Width! < 20) {
        return '20px ' + `${controlProp.Height! - 40}px` + ' 20px'
      } else if (controlProp.Height! < 40 && controlProp.Width! < 20) {
        return `${controlProp.Width! / 2}px` + ' 0px ' + `${controlProp.Width! / 2}px`
      } else {
        return '20px ' + `${controlProp.Height! - 40}px` + ' 20px'
      }
    } else {
      if (controlProp.Width! <= 40) {
        return `${controlProp.Width! / 2}px` + ' 0px ' + `${controlProp.Width! / 2}px`
      } else if (controlProp.Width! > 40 && controlProp.Height! < 20) {
        return '20px ' + `${controlProp.Width! - 40}px` + ' 20px'
      } else if (controlProp.Width! < 40 && controlProp.Height! < 20) {
        return `${controlProp.Height! / 2}px` + ' 0px ' + `${controlProp.Height! / 2}px`
      } else {
        return '20px ' + `${controlProp.Width! - 40}px` + ' 20px'
      }
    }
  }

  @Watch('properties.ProportionalThumb')
  thumbValidate () {
    if (this.properties.ProportionalThumb) {
      if (this.properties.Max! >= this.properties.LargeChange! && this.properties.LargeChange! > 0) {
        const z = this.checkOtherOrientations() ? this.properties.Height! - 40 : this.properties.Width! - 40
        this.thumbHeight = ((z / 2) - ((this.properties.Max! - this.properties.LargeChange!) / this.properties.Max!) * (z / 2)) + 'px'
        this.minHeight = '15px'
      } else if (this.properties.LargeChange! === 0) {
        this.thumbHeight = '25px'
      } else {
        this.thumbHeight = '0px'
        this.minHeight = '0px'
      }
    } else {
      this.thumbHeight = '15px'
    }
  }
  @Watch('isEditMode')
  editModeValidate () {
    if (this.isEditMode) {
      this.intervalVariable = setInterval(() => {
        this.isInvert = !this.isInvert
      }, 1000)
    } else {
      clearInterval(this.intervalVariable)
    }
  }
  scrollReAlign () {
    if (this.checkOtherOrientations()) {
      return 'rotate(-90deg)'
    }
    return 'ScaleX(-1)'
  }
  get getDisableValue () {
    if (this.isRunMode) {
      return this.properties.Enabled === false
    } else if (this.isActivated || this.isEditMode) {
      return false
    } else {
      return true
    }
  }
  /**
   * @description changes ForeColor property and then updates the getForeColor variable which is given to fill attribute of the svg element
   * @function getForeColorValue
   *
   */
  @Watch('properties.ForeColor', { deep: true })
  changeForeColor () {
    if (this.properties.Enabled) {
      this.$el.querySelectorAll('.foreColor').forEach((e) => {
        (e as SVGGElement).style.fill = this.getForeColorValue
      })
    } else {
      this.$el.querySelectorAll('.foreColor').forEach((e) => {
        (e as SVGGElement).style.fill = 'rgb(200,200,200)'
      })
    }
  }

  @Watch('properties.Enabled', { deep: true })
  enabledValidate () {
    this.changeForeColor()
  }
  get outerScrollBarDivObj () {
    const controlProp = this.properties
    let display = ''
    if (this.isRunMode) {
      display = controlProp.Visible ? controlProp.Width === 0 || controlProp.Height === 0 ? 'none' : 'inline-block' : 'none'
    } else {
      display = controlProp.Width === 0 || controlProp.Height === 0 ? 'none' : 'inline-block'
    }
    return {
      width: `${controlProp.Width}px`,
      height: `${controlProp.Height}px`,
      left: `${controlProp.Left}px`,
      top: `${controlProp.Top}px`,
      display: display,
      cursor: this.controlCursor,
      backgroundColor: 'white',
      overflow: 'hidden'
    }
  }

  get inputStyleObj () {
    const controlProp = this.properties
    let a = null
    let temprgba
    if (controlProp.BackColor!!.startsWith('rgb')) {
      a = controlProp.BackColor!.split('rgba(')[1].split(',')
    } else {
      temprgba = this.hexToRgbA(controlProp.BackColor!)
    }
    return {
      '--display': this.inputDisplay(),
      width: this.checkOtherOrientations() ? `${controlProp.Height! - 40}px` : `${controlProp.Width! - 40}px`,
      height: this.checkOtherOrientations() ? `${controlProp.Width!}px` : `${controlProp.Height!}px`,
      cursor: this.controlCursor,
      backgroundColor: controlProp.BackColor!.startsWith('rgb') ? `rgba(${a![0]},${a![1]},${a![2]},0.5)` : temprgba,
      margin: '0px'
    }
  }

  inputDisplay () {
    if (this.checkOtherOrientations()) {
      if (this.properties.Height! < 40) {
        return 'none'
      }
    } else {
      if (this.properties.Width! < 40) {
        return 'none'
      }
    }
    if (this.properties.Min === this.properties.Max) {
      return 'none'
    } else {
      if (this.minHeight === '0px') {
        return 'none'
      } else {
        return 'block'
      }
    }
  }

  hexToRgbA (hex: string) {
    let c: any
    if (/^#([A-Fa-f0-9]{3}){1,2}$/.test(hex)) {
      c = hex.substring(1).split('')
      if (c.length === 3) {
        c = [c[0], c[0], c[1], c[1], c[2], c[2]]
      }
      c = '0x' + c.join('')
      return 'rgba(' + [(c >> 16) & 255, (c >> 8) & 255, c & 255].join(',') + ',0.5)'
    }
  }
  get scrollBarButtonStyleObj () {
    const controlProp = this.properties
    return {
      backgroundColor: this.cssVars['--bg-color'],
      overflow: 'hidden',
      outline: 'none',
      cursor: this.controlCursor,
      border: !controlProp.Enabled && this.isRunMode ? '1px solid gray' : '',
      padding: '0px',
      height: this.checkOtherOrientations() ? `${controlProp.Width!}px` : `${controlProp.Height!}px`
    }
  }
    @Watch('properties.Max')
  maxValidate () {
    this.thumbValidate()
  }
  @Watch('properties.Min')
    minValidate () {
      this.thumbValidate()
    }
  @Watch('properties.LargeChange')
  largeChangeValidate () {
    this.thumbValidate()
  }
  @Watch('properties.Width')
  widthValidate () {
    this.thumbValidate()
  }
  @Watch('properties.Height')
  heightValidate () {
    this.thumbValidate()
  }

  mounted () {
    this.$el.focus()
    this.thumbValidate()
  }

  /**
   * @description checkOtherOrientations returns string value from
   * controlProperties.controlsOrientationProp
   * @function checkOtherOrientations
   * @returns boolean value
   * @override
   */
  checkOtherOrientations (): boolean {
    return controlProperties.controlsOrientationProp(this.data)
  }
  eventStoppers () {
    const eventStop = (event: Event) => event.stopPropagation()
    return this.isEditMode === false ? null : {
      keydown: eventStop
    }
  }
  scrollBarClick (event: MouseEvent) {
    if (this.toolBoxSelectControl === 'Select') {
      event.stopPropagation()
    }
  }
  text (e:MouseEvent) {
    const myRef = this.scrollbar
    console.log(this.properties.ControlTipText)
    var flag = 0
    if (this.properties.Height >= this.properties.Width) {
      flag = -90
    } else {
      flag = 0
    }
    console.log('flag value', flag)
    $(myRef).hover(function () {
      $(myRef).mouseover(function (e) {
        $('.tooltiptext')
          .css({
            position: 'absolute',
            left: e.offsetX + 10,
            top: e.offsetY,
            transform: `rotate(${flag}deg)`
          })
        // console.log(e.offsetX)
        // console.log(e.offsetY)
      })
    })
  }
  // changeStatus () {
  //   const myRef = this.scrollbar
  //   console.log(this.properties.ControlTipText)
  //   $('.tooltiptext')
  //     .css({
  //       visibility: 'hidden'
  //     })
  //   // console.log(e.offsetX)
  //   // console.log(e.offsetY)
  // }
}
</script>

<style scoped>
.tooltip {
  display: inline;
  position: sticky;
  }
  .tooltip1 {
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
.tooltip1:hover .tooltiptext {
  visibility: visible;
  position: static;
  top:0px;
  left:0px;
}
.slidecontainer {
  display: grid;
  grid-template-columns: 20px auto 20px;
}

.slider {
  -webkit-appearance: none;
  content: '';
  --rgb:250,250,250;
  --alpha: 0.5;
  background-color: rgba(var(--rgb), var(--alpha));
  outline: none;
  overflow: hidden;
  pointer-events: none;
}

.slider:hover {
  opacity: 1;
}

.slider::-webkit-slider-thumb {
  display: var(--display);
  background-color: var(--bg-color);
  border: 2px solid gray;
  border-right: 2px solid black;
  height: var(--height);
  opacity: var(--opacityValue);
  -webkit-appearance: none;
  appearance: none;
  width: var(--thumbHeight);
  min-width: var(--minHeight);
  --alpha: 1;
  cursor: inherit;
  pointer-events:auto;
  filter: invert(var(--invertValue));
}

.slider::-moz-range-thumb {
  background: rgb(139, 138, 138);
  cursor: pointer;
}

.svgLeftRightStyle {
  width: 4px;
  height: 16px;
  margin: auto;

}
</style>
