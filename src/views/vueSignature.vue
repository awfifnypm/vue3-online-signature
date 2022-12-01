<template>
    <canvas v-if="esignReset" ref="canvasRef" @mousedown="onMouseDown" @mousemove="onMouseMove" @mouseup="onMouseUp" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd"></canvas>
</template>

<script setup lang="ts">
    import { ref, reactive, watch, computed, onMounted, nextTick, onBeforeMount, onUnmounted, toRaw } from 'vue'
    const emits = defineEmits(['onDrawingStatus', 'onMouseDown', 'onMouseMove', 'onMouseUp', 'onTouchStart', 'onTouchMove', 'onTouchEnd'])
    interface pointsType {
      x: number,
      y: number,
      direction: string
    }
    interface Props {
        width?: number,
        height?: number,
        lineWidth?: number,
        lineColor?: string,
        canvasBack?: string,
        isCrop?: boolean,
        edg?: number,
        fullScreen?: boolean,
        domId?: string,
        imgBack?: string,
        isRepeat?: string,
        noRotation?: boolean,
        imgType?: string,
        backIsCenter?: boolean,
        acrossDeductWidth?: number,
        acrossDeductHeight?: number
        verticalDeductWidth?: number,
        verticalDeductHeight?: number,
        recoverPoints?: pointsType[]
    }
    const props = withDefaults(defineProps<Props>(), {
        width: 0,
        height: 0,
        lineWidth: 4,
        lineColor: '#000000',
        canvasBack: '',
        isCrop: false,
        edg: 0,
        fullScreen: false,
        domId: '',
        imgBack: '',
        isRepeat: '',
        noRotation: false,
        imgType: 'image/png',
        backIsCenter: false,
        verticalDeductWidth: 0,
        verticalDeductHeight: 0,
        acrossDeductWidth: 0,
        acrossDeductHeight: 0,
        recoverPoints: () => []
    })

    const hasDrew = ref<boolean>(false)
    const isOrientationchange = ref<boolean>(false)
    const esignReset = ref<boolean>(true)
    const resultImg = ref<string>('')
    const points = ref<any[]>([])
    let canvasTxt = ref<CanvasRenderingContext2D | null>(null)
    let canvasRef = ref<HTMLCanvasElement | null>(null)
    let cropCanvas = ref<HTMLCanvasElement | null>(null)
    let cropCanvasTxt = ref<CanvasRenderingContext2D | null>(null)
    const startX = ref<number>(0)
    const startY = ref<number>(0)
    const sratio = ref<number>(1)
    const isDrawing = ref<boolean>(false)
    const isLoad = ref<boolean>(false)
    let imgBackDom = ref<any>(null)
    let canvasBackDom = ref<any>(null)
    let screenPatams = reactive<{ width:number, height: number }>({
        width: 0,
        height: 0
    })
    let domPatams = reactive<{ width:number, height: number }>({
        width: 0,
        height: 0
    })

    const ratio = computed(() => (domPatams.height ? domPatams.height : props.fullScreen ? screenPatams.height : props.height) / (domPatams.width ? domPatams.width : props.fullScreen ? screenPatams.width : props.width) )
    const canvasBackground = computed(() => props.canvasBack ? props.canvasBack : 'rgba(255, 255, 255, 0)' )
    watch(canvasBackground, async (newVal: string) => {
        await nextTick()
        canvasRef.value && (canvasRef.value.style.background = newVal)
    })
    watch(hasDrew, (newVal: boolean) => {
        emits('onDrawingStatus', newVal)
    })
    const getSizeRatio = () => {
      return !props.fullScreen && props.backIsCenter
    }
    const setCanvasImageBack = (status: any) => {
      const canvas = canvasRef.value as HTMLCanvasElement
      let pat = canvasTxt.value?.createPattern(canvasBackDom.value, (props.isRepeat || "no-repeat"));
      canvasTxt.value?.rect(0,0,canvas.width ,canvas.height)
      canvasTxt!.value!.fillStyle = pat; 
      canvasTxt.value?.fill();
      if (status) {
          autoDraw(null, null)
      }
    }
    const setCanvasBack = (status: any) => {
       const canvas = canvasRef.value as HTMLCanvasElement
       if (props.canvasBack && canvasBackDom.value && isImgaes(props.canvasBack)) {
        setCanvasImageBack(status)
        } else {
          canvas.style.background = canvasBackground.value
        }
    }
    const getDomSize = () => {
        const canvas = canvasRef.value as HTMLCanvasElement
        if (props.domId) {
            let dom = document.getElementById(props.domId)
            let domWidth = dom ? dom.clientWidth || dom.offsetWidth : props.fullScreen ? screenPatams.width : props.width
            let domHeight = dom ? dom.clientHeight || dom.offsetHeight : props.fullScreen ? screenPatams.height : props.height
            canvas.height = domHeight
            canvas.width = domWidth
            domPatams.width = domWidth
            domPatams.height = domHeight
        } else {
            canvas.height = props.fullScreen ? screenPatams.height : props.height
            canvas.width = props.fullScreen ? screenPatams.width : props.width
        }
    }
    const resizeHandler = (status: any) => {
        if (isOrientationchange.value) return false
        const canvas = canvasRef.value as HTMLCanvasElement
        canvas.style.width = (domPatams.width ? domPatams.width : props.fullScreen ? screenPatams.width : props.width) + "px"
        const realw = parseFloat(window.getComputedStyle(canvas).width)
        canvas.style.height = ratio.value * realw + "px";
        canvasTxt.value = canvas.getContext('2d')
        canvasTxt.value?.scale(1 * sratio.value, 1 * sratio.value)
        sratio.value = realw / (domPatams.width ? domPatams.width : props.fullScreen ? screenPatams.width : props.width)
        canvasTxt.value?.scale(1 / sratio.value, 1 / sratio.value)
        if (props.canvasBack) {
          let IntervaId = setInterval(() => {
            if ((canvasBackDom.value && isLoad.value) || !isImgaes(props.canvasBack)) {
              setCanvasBack(status)
              clearInterval(IntervaId)
            }
          }, 100)
        } else {
          if (status) {
              autoDraw(null, null)
          }
        }
    }
    const orientationchangeEvent = () => {
        let directionWidth = window.orientation == 0 || window.orientation == 180 ? props.verticalDeductWidth : props.acrossDeductWidth
        let directionHeight = window.orientation == 0 || window.orientation == 180 ? props.verticalDeductHeight : props.acrossDeductHeight
        screenPatams.width = window.navigator.platform.indexOf('Win') || window.navigator.platform.indexOf('Mac') ? (document.body.clientHeight || document.body.offsetHeight) - directionWidth : (document.body.clientWidth || document.body.offsetWidth) - directionWidth
        screenPatams.height = window.navigator.platform.indexOf('Win') || window.navigator.platform.indexOf('Mac') ? (document.body.clientWidth || document.body.offsetWidth) - directionHeight : (document.body.clientHeight || document.body.offsetHeight) - directionHeight
        getImages()
        isOrientationchange.value = true
        esignReset.value = false
        let setIntervalId = setInterval(async() => {
          if (isLoad.value) {
            clearInterval(setIntervalId)
            esignReset.value = true
            isOrientationchange.value = false
            await nextTick()
            getDomSize()
            resizeHandler(true)
          }
        }, 100)
    }
    const isImgaes = (params: string) => {
      let imgType = ['.jpeg', '.bmp', '.jpg', '.gif', '.webp', '.pcx', '.tif', '.tga', '.exif', '.fpx', '.svg', '.cdr', '.pcd', '.dxf', '.ufo', '.eps', '.ai', '.png', '.hdri', '.raw', '.wmf', '.flic', '.emf', '.ico', '.avif', '.apng']
      let regex = /^\s*data:([a-z]+\/[a-z0-9-+.]+(;[a-z-]+=[a-z0-9-]+)?)?(;base64)?,([a-z0-9!$&',()*+;=\-._~:@\/?%\s]*?)\s*$/i;
      let status = params.includes('http://') || params.includes('https://') || regex.test(params) || imgType.some(item => params.includes(item))
      return status
    }
    const rotateBase64Img = (src: string, edg: number, type: string = 'not') => {
      return new Promise((resolve, reject) => {
        let canvas: HTMLCanvasElement = document.createElement("canvas");
        let ctx = canvas.getContext("2d") as CanvasRenderingContext2D
        let imgW;//图片宽度
        let imgH;//图片高度
        let size;//canvas初始大小
        if (edg % 90 != 0) {
            reject("旋转角度必须是90的倍数!");
            throw '旋转角度必须是90的倍数!';
        }
        (edg < 0) && (edg = (edg % 360) + 360)
        const quadrant = (edg / 90) % 4; //旋转象限
        const cutCoor = {sx: 0, sy: 0, ex: 0, ey: 0}; //裁剪坐标
        let image = new Image();
        image.crossOrigin = "anonymous"
        image.src = src;
        image.onload = function () {
            imgW = image.width;
            imgH = image.height;
            console.log(imgH, 'imgH')
            size = imgW > imgH ? imgW : imgH;
            canvas.width = size * 2;
            canvas.height = size * 2;
            let Cwidth = domPatams.width ? domPatams.width : props.fullScreen ? screenPatams.width : props.width
            let ratio = getSizeRatio() && type == 'init'
            switch (quadrant) {
                case 0:
                    cutCoor.sx = getSizeRatio() && type == 'init' && imgW > screenPatams.width ? size - ((imgW - (getDirection() == 'across' ? screenPatams.width : screenPatams.height)) / 2) : size
                    cutCoor.sy = size;
                    cutCoor.ex = size + imgW;
                    cutCoor.ey = size + imgH;
                    break;
                case 1:
                    cutCoor.sx = ratio ? size - props.height : window.orientation == 0 || window.orientation == 180 ? size - Cwidth : size - imgH
                    cutCoor.sy = size
                    cutCoor.ex = size;
                    cutCoor.ey = size + imgW;
                    break;
                case 2:
                    cutCoor.sx = size - imgW;
                    cutCoor.sy = size - imgH;
                    cutCoor.ex = size;
                    cutCoor.ey = size;
                    break;
                case 3:
                    cutCoor.sx = size;
                    cutCoor.sy = size - imgW;
                    cutCoor.ex = size + imgH;
                    cutCoor.ey = size + imgW;
                    break;
            }
            ctx.translate(size, size);
            ctx.rotate(edg * Math.PI / 180);
            ctx.drawImage(image, 0, 0);
            var imgData = ctx.getImageData(cutCoor.sx, cutCoor.sy, cutCoor.ex, cutCoor.ey);
            if (quadrant % 2 == 0) {
                canvas.width = imgW;
                canvas.height = imgH;
            } else {
                canvas.width = imgH;
                canvas.height = imgW;
            }
            //putImageData() 将图像数据放回画布
            ctx.putImageData(imgData, 0, 0);
            resolve(canvas.toDataURL(props.imgType))
        }
      })
    }
    const getImages = async () => {
        isLoad.value = false
        let edg = window.orientation == 0 || window.orientation == 180 ? 90 : 0
        let ratio = props.fullScreen ? edg : 0
        if (isImgaes(props.imgBack)) {
              let res = await rotateBase64Img(props.imgBack, ratio, 'init')
              if (res) {
                imgBackDom.value = new Image();
                imgBackDom.value.crossOrigin = "anonymous"
                imgBackDom!.value!.src = res;
              }
          }
          if (isImgaes(props.canvasBack)){
              let res = await rotateBase64Img(props.canvasBack, ratio, 'init')
              if (res) {
                canvasBackDom.value = new Image();
                canvasBackDom.value.crossOrigin = "anonymous"
                canvasBackDom!.value!.src = res;
                canvasBackDom!.value.onload = () => {
                  isLoad.value = true
                }
              }
          }
    }
    onMounted(() => {
        let directionWidth = window.orientation == 0 || window.orientation == 180 ? props.verticalDeductWidth : props.acrossDeductWidth
        let directionHeight = window.orientation == 0 || window.orientation == 180 ? props.verticalDeductHeight : props.acrossDeductHeight
        screenPatams.width = (document.body.clientWidth || document.body.offsetWidth) - directionWidth
        screenPatams.height = (document.body.clientHeight || document.body.offsetHeight) - directionHeight
        getImages()
        getDomSize()
        window.addEventListener("orientationchange", orientationchangeEvent)
        resizeHandler(props.recoverPoints && props.recoverPoints.length ? true : false)
        // 在画板以外松开鼠标后冻结画笔
        document.onmouseup= () => {
           isDrawing.value = false
        }
    })
    const getDirection = () => {
      return window.orientation == 90 || window.orientation == -90 ? 'across' : 'vertical'
    }
    // pc
    const onMouseDown = (e: any) => {
      e = e || event
      e.preventDefault()
      isDrawing.value = true
      hasDrew.value = true
      let params = {
        x: e.offsetX,
        y: e.offsetY,
        direction: getDirection()
      }
      drawStart(params)
      emits('onMouseDown', e)
    }
    const onMouseMove = (e: any) => {
      e = e || event
      e.preventDefault()
      if (isDrawing.value) {
        let obj = {
          x: e.offsetX,
          y: e.offsetY,
          direction: getDirection()
        }
        drawMove(obj)
      }
      emits('onMouseMove', e)
    }
    const onMouseUp = (e: any) => {
      e = e || event
      e.preventDefault()
      let obj = {
        x: e.offsetX,
        y: e.offsetY,
        direction: getDirection()
      }
      drawEnd(obj)
      isDrawing.value = false
      emits('onMouseUp', e)
    }
    // mobile
    const onTouchStart = (e: any) => {
      e = e || event
      e.preventDefault()
      hasDrew.value = true
      if (e.touches.length === 1) {
        let canvas = canvasRef.value as HTMLCanvasElement
        let obj = {
          x: e.targetTouches[0].clientX - canvas.getBoundingClientRect().left,
          y: e.targetTouches[0].clientY - canvas.getBoundingClientRect().top,
          direction: getDirection()
        }
        drawStart(obj)
      }
      emits('onTouchStart', e)
    }
    const onTouchMove = (e: any) => {
      e = e || event
      e.preventDefault()
      if (e.touches.length === 1) {
        let canvas = canvasRef.value as HTMLCanvasElement
        let obj = {
          x: e.targetTouches[0].clientX - canvas.getBoundingClientRect().left,
          y: e.targetTouches[0].clientY - canvas.getBoundingClientRect().top,
          direction: getDirection()
        }
        drawMove(obj)
      }
      emits('onTouchMove', e)
    }
    const onTouchEnd = (e: any) => {
      e = e || event
      e.preventDefault()
      if (e.touches.length === 1) {
        let canvas = canvasRef.value as HTMLCanvasElement
        let obj = {
          x: e.targetTouches[0].clientX - canvas.getBoundingClientRect().left,
          y: e.targetTouches[0].clientY - canvas.getBoundingClientRect().top,
          direction: getDirection()
        }
        drawEnd(obj)
      } else {
        points.value.push({x: -1, y: -1, direction: getDirection()})
      }
      emits('onTouchEnd', e)
    }
    // 绘制
    const drawStart = (params: { x: number, y: number}) => {
      startX.value = params.x
      startY.value = params.y
      canvasTxt.value?.beginPath()
      canvasTxt.value?.moveTo(startX.value, startY.value)
      canvasTxt.value?.lineTo(params.x, params.y)
      canvasTxt!.value!.lineCap = 'round'
      canvasTxt!.value!.lineJoin = 'round'
      canvasTxt!.value!.lineWidth = props.lineWidth * sratio.value
      canvasTxt.value?.stroke()
      canvasTxt.value?.closePath()
      points.value.push(params)
    }
    const drawMove = (params: { x: number, y: number}) => {
      canvasTxt.value?.beginPath()
      canvasTxt.value?.moveTo(startX.value, startY.value)
      canvasTxt.value?.lineTo(params.x, params.y)
      canvasTxt!.value!.strokeStyle = props.lineColor
      canvasTxt!.value!.lineWidth = props.lineWidth * sratio.value
      canvasTxt!.value!.lineCap = 'round'
      canvasTxt!.value!.lineJoin = 'round'
      canvasTxt.value?.stroke()
      canvasTxt.value?.closePath()
      startY.value = params.y
      startX.value = params.x
      points.value.push(params)
    }
    const drawEnd = (params: { x: number, y: number}) => {
      canvasTxt.value?.beginPath()
      canvasTxt.value?.moveTo(startX.value, startY.value)
      canvasTxt.value?.lineTo(params.x, params.y)
      canvasTxt!.value!.lineCap = 'round'
      canvasTxt!.value!.lineJoin = 'round'
      canvasTxt.value?.stroke()
      canvasTxt.value?.closePath()
      points.value.push(params)
      points.value.push({x: -1, y: -1})
    }
    const autoDraw = (canvasRefs: HTMLCanvasElement | null, canvas2d: CanvasRenderingContext2D | null) => {
        if (points.value && points.value.length || props.recoverPoints && props.recoverPoints.length) {
          let canvas = canvasRefs || canvasRef.value as HTMLCanvasElement
          let canvasText = canvas2d || canvasTxt.value
          let pointsList = props.recoverPoints && props.recoverPoints.length ? props.recoverPoints : points.value
          if (pointsList && pointsList.length) {
            hasDrew.value = true
          }
          pointsList.reduce((acc, cur) => {
              if (cur.x != -1 && cur.y != -1 && acc.x != -1 && acc.y != -1) {
                  canvasText?.beginPath()
                  let position = { accX: acc.x, accY: acc.y, curX: cur.x, curY: cur.y }
                  if (props.fullScreen) {
                    if ((window.orientation == 0 || window.orientation == 180) && acc.direction == 'across' && cur.direction == 'across') { // 横屏笔画横屏转竖屏
                      position = { accX: canvas.width - acc.y, accY: acc.x, curX: canvas.width - cur.y, curY: cur.x }
                    }  else if ((window.orientation == 90 || window.orientation == -90) && acc.direction == 'vertical' && cur.direction == 'vertical') { // 竖屏笔画竖屏转横屏
                      position = { accX: acc.y, accY: canvas.height - acc.x, curX: cur.y, curY: canvas.height - cur.x }
                    }
                  }
                  canvasText!.moveTo(position.accX, position.accY)
                  canvasText!.lineTo(position.curX, position.curY)
                  canvasText!.strokeStyle = props.lineColor
                  canvasText!.lineWidth = props.lineWidth * sratio.value
                  canvasText!.lineCap = 'round'
                  canvasText!.lineJoin = 'round'
                  canvasText!.stroke()
                  canvasText!.closePath()
                }
              return cur
            })
        }
    }
    // 操作
    const confirm = () => {
      return new Promise((resolve, reject) => {
        if (!hasDrew.value) {
          reject(`Warning: Not Signned!`)
          return
        }
        let canvas = canvasRef.value as HTMLCanvasElement
        let resImgData = (canvasTxt.value as CanvasRenderingContext2D).getImageData(0, 0, canvas.width, canvas.height)
        canvasTxt!.value!.globalCompositeOperation = "destination-over"
        if (props.canvasBack && props.imgBack) {
          canvasTxt.value?.clearRect(0,0,canvas.width ,canvas.height);
          autoDraw(null, null)
        }
        // canvas背景图没有的情况下才生成图片背景图
        if (props.imgBack && isImgaes(props.imgBack)){
          let pat = canvasTxt.value?.createPattern(imgBackDom.value, (props.isRepeat || "no-repeat"));
          canvasTxt.value?.rect(0,0,canvas.width ,canvas.height);
          canvasTxt!.value!.fillStyle = pat;
          canvasTxt.value?.fill();
        } else if(props.imgBack && !isImgaes(props.imgBack)) {
            canvasTxt!.value!.fillStyle = props.imgBack;
            canvasTxt.value?.fillRect(0,0, canvas.width, canvas.height);
        }
        resultImg.value = canvas.toDataURL()
        let resultImgs:any = resultImg.value
        canvasTxt.value?.clearRect(0, 0, canvas.width ,canvas.height)
        canvasTxt.value?.putImageData(resImgData, 0, 0)
        canvasTxt!.value!.globalCompositeOperation = "source-over"
        if (props.isCrop) {
          const crop_area = getCropArea(resImgData.data)
          let crop_canvas: HTMLCanvasElement | null = document.createElement('canvas')
          const crop_ctx = crop_canvas.getContext('2d')
          crop_canvas.width = crop_area[2] - crop_area[0]
          crop_canvas.height = crop_area[3] - crop_area[1]
          const crop_imgData = (cropCanvasTxt.value || canvasTxt.value)?.getImageData(...crop_area)
          crop_ctx!.globalCompositeOperation = "destination-over"
          crop_ctx?.putImageData(crop_imgData, 0, 0)
          resultImgs = crop_canvas.toDataURL()
          crop_canvas = null
        }
        let edg = (props.fullScreen && ((window.orientation == 0 || window.orientation == 180) || ((window.orientation == 90 || window.orientation == -90) && !props.noRotation))) || (window.orientation === undefined) && props.noRotation ? props.edg : 0
        rotateBase64Img(resultImgs, edg)
        .then(base64 => {
            resolve({
              base64, 
              points: points.value
            })
        })
      })
    }
    const reset = () => {
      let canvas = canvasRef.value as HTMLCanvasElement
      canvasTxt.value?.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
      )
      points.value = []
      hasDrew.value = false
      resultImg.value = ''
      setCanvasBack(false)
    }
    const getCropArea = (imgData: any) => {
        if (props.imgBack) {
          cropCanvas.value = document.createElement('canvas')
          if (props.domId) {
              let dom = document.getElementById(props.domId)
              let domWidth = dom ? dom.clientWidth || dom.offsetWidth : props.fullScreen ? screenPatams.width : props.width
              let domHeight = dom ? dom.clientHeight || dom.offsetHeight : props.fullScreen ? screenPatams.height : props.height
              cropCanvas.value.height = domHeight
              cropCanvas.value.width = domWidth
              domPatams.width = domWidth
              domPatams.height = domHeight
          } else {
              cropCanvas.value.height = props.fullScreen ? screenPatams.height : props.height
              cropCanvas.value.width = props.fullScreen ? screenPatams.width : props.width
          }
          cropCanvasTxt.value = cropCanvas.value.getContext('2d')
          if (isImgaes(props.imgBack)){
            let pat = cropCanvasTxt.value?.createPattern(imgBackDom.value, (props.isRepeat || "no-repeat"));
            cropCanvasTxt.value?.rect(0,0,cropCanvas.value.width ,cropCanvas.value.height);
            cropCanvasTxt!.value!.fillStyle = pat;
            cropCanvasTxt.value?.fill();
          } else if(!isImgaes(props.imgBack)) {
              cropCanvasTxt!.value!.fillStyle = props.imgBack;
              cropCanvasTxt.value?.fillRect(0,0, cropCanvas.value.width, cropCanvas.value.height);
          }
          autoDraw(cropCanvas.value, cropCanvasTxt.value)
        }
        let canvas = cropCanvas.value as HTMLCanvasElement || canvasRef.value as HTMLCanvasElement
        let topX = canvas.width;
        var btmX = 0;
        var topY = canvas.height;
        var btnY = 0
        for (var i = 0; i < canvas.width; i++) {
            for (var j = 0; j < canvas.height; j++) {
            var pos = (i + canvas.width * j) * 4
            if (imgData[pos] > 0 || imgData[pos + 1] > 0 || imgData[pos + 2] || imgData[pos + 3] > 0) {
                btnY = Math.max(j, btnY)
                btmX = Math.max(i, btmX)
                topY = Math.min(j, topY)
                topX = Math.min(i, topX)
            }
            }
        }
        topX++
        btmX++
        topY++
        btnY++
        const data = [topX, topY, btmX, btnY]
        return data
    }
    const recoverDraw = (pointsList: pointsType[]) => {
      let canvas = canvasRef.value as HTMLCanvasElement
      canvasTxt.value?.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
      )
      points.value = pointsList
      hasDrew.value = true
      resultImg.value = ''
      setCanvasBack(true)
    }
    onBeforeMount(() =>  {
      if (props.fullScreen) {
        let bodyDom = document.getElementsByTagName('body')
        bodyDom[0].style = 'height: 100vh';
      }
      window.addEventListener('resize', resizeHandler)
    })
    onUnmounted(() => {
      if (props.fullScreen) {
        let bodyDom = document.getElementsByTagName('body')
        bodyDom[0].style = '';
      }
      window.removeEventListener('resize', resizeHandler)
      window.removeEventListener("orientationchange", orientationchangeEvent)
    })
    defineExpose({
        confirm: toRaw(confirm),
        reset,
        recoverDraw
    })

</script>

<style scoped>
canvas {
  max-width: 100%;
  display: block;
}
</style>