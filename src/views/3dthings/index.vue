<template>
  <div class="container" ref="things"></div>
</template>

<script>
import * as THREE from 'three'

const OrbitControls = require('three-orbit-controls')(THREE)

export default {
  data() {
    return {
      controls: {
        scene: null,
        camera: null,
        renderer: null,
        rotationSpeed: 0.02
      }
    }
  },
  mounted() {
    this.initMesh()
  },
  methods: {
    initMesh() {
      // 创建场景
      this.scene = new THREE.Scene()

      // 相机.视场，长宽比，近面，远面
      const v = window.innerWidth / window.innerHeight
      this.camera = new THREE.PerspectiveCamera(45, v, 1, 500)
      this.camera.position.set(-20, 40, 30)
      this.camera.lookAt(this.scene.position)

      // 渲染器
      this.renderer = new THREE.WebGLRenderer({ antialias: true })
      this.renderer.setSize(window.outerWidth, window.outerHeight)

      // 鼠标拖拽
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.autoRotate = true;
      this.controls.minDistance = 80// 设置移动的最短距离（默认为零）
      this.controls.maxDistance = 400// 设置移动的最长距离（默认为无穷）

      // 一个生成平面
      const planeGeometry = new THREE.PlaneGeometry(60, 60, 10, 10)
      // 指定材质
      const planeMaterial = new THREE.MeshLambertMaterial({ color: 0xffffff })
      const plane = new THREE.Mesh(planeGeometry, planeMaterial)
      plane.rotation.x = -0.5 * Math.PI
      plane.position.set(0, 0, 0)
      plane.receiveShadow = true

      // 创建一个集合体，10，10，10为正方体
      const cubeGeometry = new THREE.CubeGeometry(10, 10, 10)
      const cubeMaterial = new THREE.MeshLambertMaterial({ color: 0xff0000 })
      this.cube = new THREE.Mesh(cubeGeometry, cubeMaterial)
      this.cube.position.set(-4, 3, 0)

      // 增加灯光
      const spotLight = new THREE.SpotLight(0xffffff)
      spotLight.position.set(-60, 60, -10)

      // 把创建的东西都添加到场景中
      this.scene.add(plane)
      this.scene.add(this.cube)
      this.scene.add(spotLight)

      this.$refs.things.append(this.renderer.domElement)
      this.rendering()
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
  height: 400px;
}
</style>
