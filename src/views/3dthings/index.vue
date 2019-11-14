<template>
  <div class="container" ref="things"></div>
</template>

<script>
import * as THREE from 'three'

const OrbitControls = require('three-orbit-controls')(THREE)

export default {
  mounted() {
    this.createScence()
    document.addEventListener('mousedown', this.onDocumentMouseMove, false)
  },
  data() {
    return {
      INTERSECTED: ''
    }
  },
  methods: {
    createScence() {
      this.createScene()
      this.createCamera()
      this.createRender()
      this.createControls()
      this.createAxesHelper()
      this.createGround()
      this.createLight()
      this.rendering()
    },
    onDocumentMouseMove(event) {
      event.preventDefault()
      // 新建一个Vector2对象保存鼠标位置信息,监听鼠标移动事件
      const mouse = new THREE.Vector2()
      mouse.x = (event.clientX / window.outerWidth) * 2 - 1
      mouse.y = -(event.clientY / window.outerHeight) * 2 + 1
      // 新建一个Raycasting对象
      const raycaster = new THREE.Raycaster()
      raycaster.setFromCamera(mouse, this.camera)
      const intersects = raycaster.intersectObjects(this.scene.children)
      // 拾取物体数大于0时
      if (intersects.length > 0) {
        // 获取第一个物体
        if (this.INTERSECTED !== intersects[0].object) {
          if (this.INTERSECTED) {
            this.INTERSECTED.material.color.setHex(this.INTERSECTED.currentHex)
          }
          this.INTERSECTED = intersects[0].object
          this.INTERSECTED.currentHex = this.INTERSECTED.material.color.getHex()
          // 改变物体的颜色(黑色)
          console.log('this.INTERSECTED:', this.INTERSECTED.name)
          this.INTERSECTED.material.color.set('black')
        }
      } else {
        if (this.INTERSECTED) {
          this.INTERSECTED.material.color.set(this.INTERSECTED.currentHex)
        }
        this.INTERSECTED = null
      }
    },
    createCamera() {
      // 相机.视场，长宽比，近面，远面
      const v = window.outerWidth / window.outerHeight
      this.camera = new THREE.PerspectiveCamera(45, v, 1, 50000)
      this.camera.position.set(100, 180, -180)
      this.camera.lookAt(this.scene.position)
    },
    createControls() {
      // 鼠标拖拽
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.autoRotate = true
      this.controls.minDistance = 45// 设置移动的最短距离（默认为零）
      this.controls.maxDistance = 500// 设置移动的最长距离（默认为无穷）
    },
    createAxesHelper() {
      // 坐标系
      this.axisHelper = new THREE.AxesHelper(600)

      this.scene.add(this.axisHelper)
    },
    createRender() {
      // 渲染器
      this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
      this.renderer.setClearAlpha(0.2)
      this.renderer.setSize(window.outerWidth, window.outerHeight)
      this.renderer.shadowMap.enabled = true

      this.$refs.things.append(this.renderer.domElement)
    },
    createScene() {
      // 创建场景
      this.scene = new THREE.Scene()
      this.scene.position.set(0, 40, 0)
    },
    createGround() {
      // 一个生成平面 长，宽，分段数
      const planeGeometry = new THREE.PlaneGeometry(200, 100)
      // 指定材质
      const planeMaterial = new THREE.MeshLambertMaterial({
        color: '#fff',
        side: THREE.DoubleSide // 双面（正面及背面）渲染
      })
      const plane = new THREE.Mesh(planeGeometry, planeMaterial)
      plane.rotation.x = -0.5 * Math.PI
      plane.position.set(0, 0, 0)
      plane.receiveShadow = true
      plane.name = 'ground'

      this.scene.add(plane)
    },
    createCube() {
      // 创建一个集合体，10，10，10为正方体
      const cubeGeometry = new THREE.CubeGeometry(10, 10, 10)
      // 指定材质
      const cubeMaterial = new THREE.MeshLambertMaterial({ color: '#ff0000' })
      const cube = new THREE.Mesh(cubeGeometry, cubeMaterial)
      cube.position.set(0, 5, 0)
      cube.castShadow = true
      cube.receiveShadow = true
      cube.name = 'thing01'

      this.scene.add(cube)

      return cube
    },
    createLight() {
      // 增加灯光
      const light = new THREE.DirectionalLight('#fff')
      light.position.set(0, 180, -360)
      light.castShadow = true
      light.shadow.mapSize.width = 2048
      light.shadow.mapSize.height = 2048
      light.target = this.createCube()

      // 再加环境光，让阴影透明
      const environmentLight = new THREE.AmbientLight('#ccc')
      this.scene.add(light)
      this.scene.add(environmentLight)
    },
    rendering() {
      this.renderer.render(this.scene, this.camera)
      requestAnimationFrame(this.rendering)
    }
  }
}
</script>

<style lang="scss">
.container {
  background-image: linear-gradient( 135deg, #ABDCFF 10%, #0396FF 100%)
}
</style>
