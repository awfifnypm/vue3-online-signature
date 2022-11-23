# vue3-online-signature

> Canvas 手写签字/签名板

> 本组件使用Vue3.x版本开发，Vue3.x必须使用vue3-online-signature，如果使用Vue2.x版本的项目，请使用Vue2.x版本组件

## 版本提示
1. Vue2.x npm链接  或 npm install vue-online-signature --save
2. Vue2.x gitee链接 https://github.com/awfifnypm/vue-online-signature

## 功能
1. 兼容 PC 和 Mobile；
2. 屏幕旋转自适应自定义画布屏幕大小（屏幕旋转后竖屏与横屏数据互通）；
3. 自定义画布尺寸（尺寸可通过获取id元素，幕屏尺寸，自定义宽高），画笔粗细、颜色，画布背景色；
4. 支持裁剪 （针对需求：有的签字需要裁剪掉四周空白）。
5. 自定义导出图旋转角度
6. 自定义画布背景（支持图片及图片背景重复）
7. 自定义导出图背景（支持图片及图片背景重复，可生成与画布背景不一样背景的图片）
8. 导出图片格式为 `base64`；
[示例demo](http://b.wujiang.pro/)

注： 本组件是基于vue-esign插件基础上进行二开和修改，使用方法与vue-esign插件有差异，建议查看本插件文档


## 安装

``` bash
npm install vue3-online-signature --save
```

## 使用
1. 引入 main.js 获vue页面引入
```js
import vueSignature from 'vue3-online-signature'
```
2. 页面中使用
    **必须设置 `ref` ，用来调用组件内暴露的内置方法 `reset()` 和 `confirm()`**<br />
   自定义的宽度超出屏幕宽度的话，组件样式宽度则是父元素的100%；<br />
   组件所有参数都非必填，组件emit出部份内置方法，具体查看下面说明文档

```html
<vueSignature ref="vueSignatureRef" v-bind="params"/>
<button @click="handleReset">清空画板</button> 
<button @click="handleGenerate">生成图片</button>
```
```js
const resultImg = ''
const params = reactive<object>({
    width: 0,
    height: 0,
    lineWidth: 5,
    lineColor: '',
    canvasBack: '',
    isCrop: false,
    edg: 0,
    fullScreen: true,
    domId: '',
    imgBack: '',
    isRepeat: '',
    noRotation: false,
    backIsCenter: false
  })
let vueSignatureRef = ref<any>(null)
const handleReset = () => {
    vueSignatureRef.value.reset()
  },
const = handleGenerate = () => {
    vueSignatureRef.value.confirm().then(res => {
      this.resultImg = res
    }).catch(err => {
      alert(err) // 画布没有签字时会执行这里 'Not Signned'
    })
  }
```
3. 说明

| 属性 | 类型 | 默认值 | 说明 |
| :-: | :-- | :-: | :-- |
| domId | String | 空 | 用于获取元素的宽高生成画布尺寸，优先级一级（建议使用canvas父级元素的ID, 父级元素的width值不可大于当前屏幕的宽度） |
| fullScreen | Boolean | false | 是否获取屏幕的宽高生成画布尺寸，优先级二级 |
| width | Number | 0 | 画布宽度，优先级三级 (此值不可大于当前屏幕的宽度) |
| height | Number | 0 | 画布高度，优先级三级 |
| lineWidth | Number | 4 | 画笔粗细 |
| lineColor | String | #000000 | 画笔颜色 |
| canvasBack | String | 空 | 画布背景，为空时画布背景透明，<br />支持多种格式 '#ccc'，'#E5A1A1'，'rgb(229, 161, 161)'，'rgba(0,0,0,.6)'，'red', 'http'、'https'、文件路径及'base64'类型图片链接 |
| imgBack | String | 空 | 画布最终导出图的图片背景，如果此参数不为空，生成图片时会覆盖canvasBack的背景图，<br />支持多种格式 '#ccc'，'#E5A1A1'，'rgb(229, 161, 161)'，'rgba(0,0,0,.6)'，'red', 'http'、'https'、文件路径及'base64'类型图片链接 |
| edg | Number | 0 | 画布导出图需要旋转的角度（如竖屏导出图会生成竖屏尺寸的图片，此参数值为270时,会生成一张横向的图片） |
| imgType | String | image/png | 画布导出图的图片类型（可以是其他'image/jpeg'等） |
| isRepeat | String | 'no-repeat' | 画布背景是否重复(参数:'repeat','repeat-x','repeat-y' ) |
| noRotation | Boolean | false | 横屏时导出图是否旋转角度 (当值为true时，横屏时导出图不会旋转角度)|
| isCrop | Boolean | false | 是否裁剪，在画布设定尺寸基础上裁掉四周空白部分 |
| backIsCenter | Boolean | false | 背景图片是否居中显示(使用domId或输入固定宽度时生效并且只有图片宽度大于canvas宽度才会生效) |
| verticalDeductWidth | Number | 0 | 获取屏幕的宽高生成画布尺时，竖屏时宽度需要减除的尺寸 |
| verticalDeductHeight | Number | 0 | 获取屏幕的宽高生成画布尺时，竖屏时高度需要减除的尺寸 |
| acrossDeductWidth | Number | 0 | 获取屏幕的宽高生成画布尺时，横屏时宽度需要减除的尺寸 |
| acrossDeductHeight | Number | 0 | 获取屏幕的宽高生成画布尺时，横屏时高度需要减除的尺寸 |

| 方法  | 参数 | 说明 |
| :-: | :-- | :-- |
| onDrawingStatus | status | 返参可监听画板是否已绘画，true或false |
| onMouseDown | event | PC 当鼠标指针移动到元素上方，并按下鼠标按键（左、右键均可）时，会发生mousedown事件。 |
| onMouseMove | event | PC 当鼠标指针移动时发生的事件。 |
| onMouseUp | event | PC 当在元素上松开鼠标按键（左、右键均可）时，会发生 mouseup 事件。 |
| onTouchStart | event | Mobile 当手指触摸屏幕时候触发。 |
| onTouchMove | event | 当手指在屏幕上滑动的时候连续地触发。 |
| onTouchEnd | event | 当手指从屏幕上离开的时候触发。 |

两个内置方法，通过给组件设置 `ref` 调用：

**清空画布**
```js
vueSignatureRef.value.reset()
```
**生成图片**
```js
vueSignatureRef.value.confirm.then(res => {
  console.log(res) // base64图片
}).catch(err => {
  alert(err) // 画布没有签字时会执行这里 'Not Signned'
})
```
