<template>
  <div class="wrap">
    <div class="left-bar">
      <ul>
        <li
          v-for="(item, index) in nav"
          :key="index"
          :class="isActive == index ? 'active' : ''"
          @click="pickNav(index)"
        >
          <el-tooltip
            popper-class="offset-left"
            effect="dark"
            :content="item.title"
            placement="right"
          >
            <i :class="'el-icon-' + item.icon"></i>
          </el-tooltip>
        </li>
      </ul>
    </div>
    <div class="place-info" draggable="true">
      <p class="title">商场{{Number(storeIndex)+1}}区</p>
      <div
        v-for="(parentItem, parentIndex) in detailList"
        :key="parentIndex"
      >
        <el-tabs
          v-if="storeIndex == parentIndex"
          v-model="activeName"
          type="card"
        >
          <el-tab-pane label="基本情况" name="first">
            <ul
            >
              <li
                v-for="(item, index) in parentItem.basic"
                :key="index"
                class="item"
              ><span>{{item.title}}</span><span>{{item.value}}</span></li>
            </ul>
          </el-tab-pane>
          <el-tab-pane label="管理人员" name="second">
            <ul>
              <li
                v-for="(item, index) in parentItem.manager"
                :key="index"
                class="item"
              ><span>{{item.title}}</span><span>{{item.value}}</span></li>
            </ul>
          </el-tab-pane>
          <el-tab-pane label="租期" name="third">
            <ul>
              <li
                v-for="(item, index) in parentItem.range"
                :key="index"
                class="item"
              ><span>{{item.title}}</span><span>{{item.value}}</span></li>
            </ul>
          </el-tab-pane>
        </el-tabs>
      </div>
    </div>
    <div class="heatmap" ref="heatmap"></div>
    <div class="container" ref="things"></div>
  </div>
</template>

<script>
import * as THREE from 'three'

import Heatmap from 'heatmap.js'

const OrbitControls = require('three-orbit-controls')(THREE)
const ThreeBSP = require('tthreebsp')(THREE)


export default {
  mounted() {
    this.createScence()
    document.addEventListener('mousedown', this.onDocumentMouseMove, false)
    document.addEventListener('mousewheel', this.onDocumentMousewheel, false)
    this.addHeatMap()
  },
  data() {
    return {
      INTERSECTED: '',
      textWz: {},
      nav: [{
        icon: 'house',
        title: '商场编号'
      }, {
        icon: 'user',
        title: '人流量'
      }, {
        icon: 'guide',
        title: '出口位置'
      }, {
        icon: 'camera',
        title: '监控位置'
      }],
      isActive: 0,
      detailList: [{
        basic: [{
          title: '店铺面积',
          value: '188'
        }, {
          title: '运营状况',
          value: '优'
        }],
        manager: [{
          title: '工商',
          value: '陈志'
        }, {
          title: '环保',
          value: '李青'
        }, {
          title: '安保',
          value: '王琪'
        }, {
          title: '店长',
          value: '马梅梅'
        }, {
          title: '副店长',
          value: '蒋一程'
        }],
        range: [{
          title: '当前租期',
          value: '1年',
        }, {
          title: '历史租期',
          value: '无',
        }, {
          title: '欠费情况',
          value: '无',
        }]
      }, {
        basic: [{
          title: '店铺面积',
          value: '69'
        }, {
          title: '运营状况',
          value: '良好'
        }],
        manager: [{
          title: '工商',
          value: '赵天骐'
        }, {
          title: '环保',
          value: '李志'
        }, {
          title: '安保',
          value: '王琪'
        }, {
          title: '店长',
          value: '马梅梅'
        }, {
          title: '副店长',
          value: '蒋一程'
        }],
        range: [{
          title: '当前租期',
          value: '5年',
        }, {
          title: '历史租期',
          value: '已租8年',
        }, {
          title: '欠费情况',
          value: '无'
        }]
      }, {
        basic: [{
          title: '店铺面积',
          value: '26'
        }, {
          title: '运营状况',
          value: '良好'
        }],
        manager: [{
          title: '工商',
          value: '赵天骐'
        }, {
          title: '环保',
          value: '李志'
        }, {
          title: '安保',
          value: '王琪'
        }, {
          title: '店长',
          value: '马梅梅'
        }, {
          title: '副店长',
          value: '蒋一程'
        }],
        range: [{
          title: '当前租期',
          value: '5年',
        }, {
          title: '历史租期',
          value: '已租8年',
        }, {
          title: '欠费情况',
          value: '无'
        }]
      }, {
        basic: [{
          title: '店铺面积',
          value: '77'
        }, {
          title: '运营状况',
          value: '良好'
        }],
        manager: [{
          title: '工商',
          value: '赵天骐'
        }, {
          title: '环保',
          value: '李志'
        }, {
          title: '安保',
          value: '王琪'
        }, {
          title: '店长',
          value: '马梅梅'
        }, {
          title: '副店长',
          value: '蒋一程'
        }],
        range: [{
          title: '当前租期',
          value: '5年',
        }, {
          title: '历史租期',
          value: '已租8年',
        }, {
          title: '欠费情况',
          value: '无'
        }]
      }],
      activeName: 'first',
      storeIndex: 0
    }
  },
  methods: {
    addHeatMap() {
      // 创建一个heatmap实例对象
      const heatmapInstance = Heatmap.create({
        container: this.$refs.heatmap
      })
      // 构建一些随机数据点,这里替换成你的业务数据
      const points = []
      let max = 0
      const width = 1000
      const height = 500
      let len = 200
      while (len > 0) {
        len -= 1
        const val = Math.floor(Math.random() * 100)
        max = Math.max(max, val)
        const point = {
          x: Math.floor(Math.random() * width),
          y: Math.floor(Math.random() * height),
          value: val
        }
        points.push(point)
      }
      const data = {
        max,
        data: points
      }
      // 因为data是一组数据,所以直接setData
      heatmapInstance.setData(data)
      this.heatMapTo3D()
    },
    heatMapTo3D() {
      const ctx = this.$refs.heatmap.getElementsByTagName('canvas')[0]
      const heatMapGeo = new THREE.PlaneGeometry(200, 100)
      const heatMapTexture = new THREE.Texture(ctx)
      const heatMapMaterial = new THREE.MeshBasicMaterial({
        map: heatMapTexture,
        transparent: true
      })
      heatMapMaterial.map.needsUpdate = true
      const heatMapPlane = new THREE.Mesh(heatMapGeo, heatMapMaterial)
      heatMapPlane.position.set(0, 0.1, 0)
      heatMapPlane.rotation.x = -Math.PI / 2
      this.scene.add(heatMapPlane)
    },
    pickNav(index) {
      this.isActive = index
      this.toggleCamera(index)
    },
    toggleCamera(index) {
      const children = this.scene.children

      children.map((value, childrenIndex, arr) => {
        const isCylinder = value.name.split('-')[0] === 'cylinder'
        const isIndex = index === 3
        if (isCylinder) {
          value.visible = isIndex
        }
        return arr
      })
    },
    createMesh(geom) {
      // 创建一个线框纹理
      const wireFrameMat = new THREE.MeshBasicMaterial({
        opacity: 0.5,
        wireframeLinewidth: 0.5
      });
      wireFrameMat.wireframe = true;

      // 创建模型
      const mesh = new THREE.Mesh(geom, wireFrameMat);

      return mesh;
    },
    createDoor() {
      // 创建球形几何体
      const sphereGeometry = new THREE.SphereGeometry(5, 2, 2)
      const sphere = this.createMesh(sphereGeometry)
      sphere.position.set(0, 50, 0)

      // 创建立方体几何体
      const cubeGeometry = new THREE.BoxGeometry(2, 20, 2)
      const cube = this.createMesh(cubeGeometry)
      cube.position.set(0, 50, 0)

      // 生成ThreeBSP对象
      const sphereBSP = new ThreeBSP(sphere)
      const cubeBSP = new ThreeBSP(cube)

      // 进行并集计算
      const resultBSP = sphereBSP.subtract(cubeBSP)

      // 从BSP对象内获取到处理完后的mesh模型数据
      const result = resultBSP.toMesh()
      // 更新模型的面和顶点的数据
      result.geometry.computeFaceNormals()
      result.geometry.computeVertexNormals()

      // 重新赋值一个纹理
      const material = new THREE.MeshPhongMaterial({ color: 0x00ffff })
      result.material = material

      // 将计算出来模型添加到场景当中
      this.scene.add(result)
    },
    createScence() {
      this.createScene()
      this.createCamera()
      this.createRender()
      this.createControls()
      this.createAxesHelper()
      this.createGround()
      this.createCube()
      this.createCircleCamera()
      this.createDoor()
      this.createLight()
      this.createGirdHelper()
      this.rendering()
      this.textPosition()
    },
    onDocumentMousewheel(e) {
      console.log(1)
      e.preventDefault();
      // e.stopPropagation();
      // 判断浏览器IE，谷歌滑轮事件
      if (e.wheelDelta) {
        // 当滑轮向上滚动时
        if (e.wheelDelta > 0) {
          console.log('上')
        }
        // 当滑轮向下滚动时
        if (e.wheelDelta < 0) {
          console.log('下')
        }
        // Firefox滑轮事件
      } else if (e.detail) {
        // 当滑轮向上滚动时
        if (e.detail > 0) {
          console.log('上')
        }
        // 当滑轮向下滚动时
        if (e.detail < 0) {
          console.log('下')
        }
      }
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
          // 不同的物体执行不同的方法
          const thingName = this.INTERSECTED.name
          const whatsThing = thingName.split('-')[0] === 'cube'
          const thingNumber = thingName.split('-')[1]
          console.log('this.INTERSECTED:', this.INTERSECTED)
          if (whatsThing) {
            this.switchName(thingNumber)
          }
          // 改变物体的颜色(黑色)
          // this.INTERSECTED.material.color.set('black')
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
      this.camera.position.set(100, 300, 100)
      this.camera.lookAt(this.scene.position)
    },
    createControls() {
      // 鼠标拖拽
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.autoRotate = true
      this.controls.minDistance = 45// 设置移动的最短距离（默认为零）
      this.controls.maxDistance = 500// 设置移动的最长距离（默认为无穷）
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
      this.scene.position.set(0, 80, 0)
    },
    createGround() {
      // 一个生成平面 长，宽，分段数
      const planeGeometry = new THREE.PlaneGeometry(200, 100)
      // 指定材质
      const planeMaterial = new THREE.MeshLambertMaterial({
        color: '#fff',
        side: THREE.DoubleSide // 双面（正面及背面）渲染
      })
      this.plane = new THREE.Mesh(planeGeometry, planeMaterial)
      this.plane.rotation.x = -0.5 * Math.PI
      this.plane.position.set(0, 0, 0)
      this.plane.receiveShadow = true
      this.plane.name = 'ground'

      this.scene.add(this.plane)
    },
    createCube() {
      // 创建一个集合体，10，10，10为正方体
      const cubeGeometry = new THREE.CubeGeometry(10, 10, 10)
      for (let i = 0; i < 4; i += 1) {
        // 指定材质
        const cubeMaterial = new THREE.MeshLambertMaterial({ color: '#ff0000' })
        const cube = new THREE.Mesh(cubeGeometry, cubeMaterial)
        const cubeWidth = i > 0 ? i * 50 : i
        cube.position.set(cubeWidth - 75, 5, 0)
        cube.castShadow = true
        cube.receiveShadow = true
        cube.name = `cube-${i}`

        this.textWz[i] = cube.position

        this.scene.add(cube)
      }
    },
    createCircleCamera() {
      // 创建圆柱体，作为监控设备
      const geometry = new THREE.CylinderGeometry(1.5, 1.5, 3, 50)
      for (let i = 0; i < 10; i += 1) {
        const material = new THREE.MeshLambertMaterial({
          color: '#FF7C01',
          wireframe: true
        })
        const cylinder = new THREE.Mesh(geometry, material)
        const x = Math.floor(Math.random() * 181) - 90
        const y = Math.floor(Math.random() * 81) - 40

        cylinder.position.set(x, 30, y)
        cylinder.rotation.x = 1
        cylinder.name = `cylinder-${i}`
        cylinder.visible = false
        this.scene.add(cylinder)
      }
    },
    createLight() {
      // 增加灯光
      const light = new THREE.DirectionalLight('#fff')
      light.position.set(0, 250, 360)
      light.castShadow = true
      light.shadow.mapSize.width = 2048
      light.shadow.mapSize.height = 2048

      light.shadow.camera.left = -100
      light.shadow.camera.right = 100
      light.shadow.camera.top = 20
      light.shadow.camera.bottom = 0

      light.target = this.plane

      // 再加环境光，让阴影透明
      const environmentLight = new THREE.AmbientLight('#ccc')
      this.scene.add(light)
      this.scene.add(environmentLight)

      // 显示光照区域
      const cameraHelper = new THREE.CameraHelper(light.shadow.camera)
      this.scene.add(cameraHelper)
    },
    createAxesHelper() {
      // 坐标系
      this.axisHelper = new THREE.AxesHelper(600)

      this.scene.add(this.axisHelper)
    },
    // 地面网格
    createGirdHelper() {
      const gridHelper = new THREE.GridHelper(3000, 100, 'red', 'gray')
      gridHelper.position.y = -5
      gridHelper.position.x = 0
      this.scene.add(gridHelper)
    },
    rendering() {
      this.renderer.render(this.scene, this.camera)
      requestAnimationFrame(this.rendering)
    },
    makeTextSprite(message) {
      // 先用画布将文字画出
      const canvas = document.createElement('canvas')
      const ctx = canvas.getContext('2d')
      ctx.fillStyle = '#333'
      ctx.font = '30px Verdana'
      ctx.lineWidth = 4
      ctx.fillText(message, 10, 90)
      const texture = new THREE.Texture(canvas)
      texture.needsUpdate = true

      // 使用Sprite显示文字
      const material = new THREE.SpriteMaterial({ map: texture })
      const textObj = new THREE.Sprite(material)
      textObj.scale.set(25, 12.5, 37.5)
      textObj.position.set(0, 0, 98)

      return textObj
    },
    textPosition() {
      for (let i = 0; i < 4; i += 1) {
        const repostoryOne = this.makeTextSprite(`商场${i + 1}区`)
        const x = this.textWz[i].x - 5.5
        const y = this.textWz[i].y + 5
        const z = this.textWz[i].z + 6.5
        repostoryOne.center = new THREE.Vector2(0, 0)
        this.scene.add(repostoryOne)
        repostoryOne.position.set(x, y, z)
      }
    },
    switchName(value) {
      this.storeIndex = value
    }
  }
}
</script>

<style lang="scss">
.offset-left{
  left: 55px !important;
}
.flex-style{
  display: flex;
  justify-content: space-between;
  flex-wrap:nowrap;
}
.wrap{
  .left-bar{
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    width: 60px;
    text-align: center;
    background: #28292d;
    color:#fff;
    li{
      padding:16px 0;
      cursor: pointer;
      border-left: 2px solid transparent;
      transition: 0.15s;
      &.active{
        border-left: 2px solid #ff762a;
        background: #060606;
      }
      &:hover{
        background: #060606;
      }
    }
    i{
      font-size:28px;
    }
  }
  .place-info{
    background: #252935;
    position: absolute;
    top: 100px;
    left: 200px;
    border-radius: 4px;
    padding:15px;
    font-size: 14px;
    color: #fff;
    .title{
      text-align: left;
      margin-bottom: 15px;
    }
    .item{
      font-size: 12px;
      @extend .flex-style;
      &:first-child{
        border:0;
        margin:0;
        padding:0;
      }
      border-top:1px solid #484747;
      margin-top:10px;
      padding-top: 10px;
    }
    span{
      display: inline-block;
    }
    .el-tabs__item{
      height: 25px;
      line-height: 25px;
      border:0;
      font-size: 12px;
      padding:0 10px;
      background: #3d414c;
      color: #86888f;
      transition: 0.15s;
    }
    .el-tabs--card>.el-tabs__header .el-tabs__item.is-active{
      color: #fff;
      background: #1b1e25;
    }
    .el-tabs--card>.el-tabs__header,.el-tabs--card>.el-tabs__header .el-tabs__nav{
      border:0;
    }
  }
  .container {
    background-image: linear-gradient( 135deg, #ABDCFF 10%, #0396FF 100%)
  }
  .heatmap{
    display: none;
    width:1000px;
    height: 500px;
  }
}
</style>
