<template>
   <div >
      <ul class="input__wrap">
         <li>
            <span>width: </span>
            <input type="text" v-model="params.width"/>
         </li>
         <p>画布宽度(此值不可大于当前屏幕的宽度)</p>
         <li>
            <span>height: </span>
            <input type="text" v-model="params.height"/>
         </li>
         <p>画布高度</p>
         <li>
            <span>lineWidth: </span>
            <input type="text" v-model="params.lineWidth"/>
         </li>
         <p>画笔粗细</p>
         <li>
            <span>lineColor: </span>
            <input type="text" v-model="params.lineColor"/>
         </li>
         <p>画笔颜色</p>
         <li>
            <span>canvasBack: </span>
            <input type="text" v-model="params.canvasBack"/>
         </li>
         <p>画布背景，为空时画布背景透明</p>
         <li>
            <span>imgBack: </span>
            <input type="text" v-model="params.imgBack"/>
         </li>
         <p>画布最终导出图的图片背景，如果此参数不为空，生成图片时会覆盖canvasBack的背景图</p>
         <li>
            <span>isCrop: </span>
            <input type="checkbox" v-model="params.isCrop"/>
         </li>
         <p>是否裁剪，在画布设定尺寸基础上裁掉四周空白部分</p>
         <li>
            <span>edg: </span>
            <input type="text" v-model="params.edg"/>
         </li>
         <p>画布导出图需要旋转的角度</p>
         <li>
            <span>fullScreen: </span>
            <input type="checkbox" v-model="params.fullScreen"/>
         </li>
         <p style="color: red">获取屏幕的宽高生成画布尺寸(注意：由于demo页面不是用全屏弹窗显示画布，当前demo移动端页面获取屏幕尺寸作为画布尺寸，在旋转后画笔会有偏差，正式开发下用全屏弹窗显示无此问题)</p>
         <li>
            <span>domId: </span>
            <input type="text" disabled v-model="params.domId"/>
         </li>
         <p>demo无法使用</p>
         <li>
            <span>isRepeat: </span>
            <input type="text" v-model="params.isRepeat"/>
         </li>
         <p>画布背景是否重复(参数:'repeat','repeat-x','repeat-y' )</p>
         <li>
            <span>noRotation: </span>
            <input type="checkbox" v-model="params.noRotation"/>
         </li>
         <p>横屏时导出图是否旋转角度 (当值为true时，横屏时导出图不会旋转角度)</p>
         <li>
            <span>ackIsCenter: </span>
            <input type="checkbox" v-model="params.backIsCenter"/>
         </li>
         <p>背景图片是否居中显示(使用domId或输入固定宽度时生效并且只有图片宽度大于canvas宽度才会生效)</p>
         <li>
            <span>imgType: </span>
            <input type="text" v-model="params.imgType"/>
         </li>
         <p>画布导出图的图片类型（可以是其他'image/jpeg'等）</p>
         <li>
            <span>vDeductWidth: </span>
            <input type="text" v-model="params.verticalDeductWidth"/>
         </li>
         <p>verticalDeductWidth 获取屏幕的宽高生成画布尺时，竖屏时宽度需要减除的尺寸</p>
         <li>
            <span>vDeductHeight: </span>
            <input type="text" v-model="params.verticalDeductHeight"/>
         </li>
         <p>verticalDeductHeight 获取屏幕的宽高生成画布尺时，竖屏时高度需要减除的尺寸</p>
         <li>
            <span>aDeductWidth: </span>
            <input type="text" v-model="params.acrossDeductWidth"/>
         </li>
         <p>acrossDeductWidth 获取屏幕的宽高生成画布尺时，横屏时宽度需要减除的尺寸</p>
         <li>
            <span>aDeductHeight: </span>
            <input type="text" v-model="params.acrossDeductHeight"/>
         </li>
         <p>acrossDeductHeight 获取屏幕的宽高生成画布尺时，横屏时高度需要减除的尺寸</p>
      </ul>
      <div class="button" @click="generate">创建画布</div>
      <div id="canvasWrap" style="width:100%;">
         <vueOnlineSignature v-if="isShow" ref="vueSignatureRef" v-bind="params"/>
      </div>
      <img v-if="imagesSRC" :src="imagesSRC" alt="" style="max-width: 100%">
     <div class="buttonList" v-if="isShow">
         <div  class="button"  @click="confirm">生成</div>
         <div  class="button"  @click="reset">重置</div>
     </div>
   </div>
</template>

<script setup lang="ts">
import vueOnlineSignature from '@/views/vueSignature.vue';
// import vueOnlineSignature from 'vue3-online-signature'
import { ref, toRaw, reactive } from 'vue'
const isShow = ref<boolean>(false)
const params = reactive<any>({
      width: 0,
      height: 0,
      lineWidth: 5,
      lineColor: '#000',
      canvasBack: new URL('@/assets/ceshi.png', import.meta.url).href,
      isCrop: true,
      edg: 0,
      fullScreen: false,
      domId: '',
      imgType: 'image/png',
      imgBack: new URL('@/assets/22.png', import.meta.url).href,
      isRepeat: '',
      noRotation: false,
      backIsCenter: false,
      verticalDeductWidth: 10,
      verticalDeductHeight: 24,
      acrossDeductWidth: 30,
      acrossDeductHeight: 20
    })
let vueSignatureRef = ref<any>(null)
const imagesSRC = ref<string>('')
const confirm = () => {
   vueSignatureRef.value.confirm()
   .then((res:string) => {
      imagesSRC.value = res
   })
   .catch(err => {
      alert('请先绘画')
   })
}
const reset = () => {
   vueSignatureRef.value.reset()
}
const generate = () => {
   imagesSRC.value = ''
   isShow.value = false
   setTimeout(() => {isShow.value = true}, 100)
}
</script>

<style lang="less" scoped>
#canvasWrap{
   margin: 15px 0;
}
canvas{
   border: 1px dashed #ccc
}
.input__wrap{
   list-style-type: none;
   width: 100%;
   padding: 0;
   li{
      line-height: 30px;
      display: flex;
      justify-content: flex-start;
      align-items: center;
      flex-wrap: wrap;
   }
   span{
      width: 130px;
      display: inline-block;
      text-align: right;
      margin-right: 10px
   }
   input[type="checkbox" i]{
      margin-left: 0
   }
   p{
      margin: 0 0 10px 140px;
      color:#aaa
   }
}
.button{
   width: 290px;
   height: 35px;
   display: flex;
   justify-content: center;
   align-items: center;
   font-size: 15px;
   border: 1px solid #ccc;
   border-radius: 5px;
   cursor: pointer;
}
.buttonList{
   display: flex;
   .button ~ .button{
      margin-left: 10px;
   }
}
</style>