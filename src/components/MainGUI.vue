<template>
  <div class="text-body-2 font-weight-light mb-n1">Count {{ store.count }}</div>
  <div class="connections" @contextmenu="onContextMenu($event)">
    <canvas id="pixi"></canvas>
  </div>
  <div class="pixiBody">
    <div class="buttons">
      <button v-on:click="stop()">Stop</button>
      <button v-on:click="start()">Start</button>
      <button v-on:click="reset()">Reset</button>
    </div>
    <br/>
  </div>

  <dialogForm
    :visible="showDialogForm"
    v-model:first_name="dialog_data.first_name"
    v-model:age="dialog_data.age"
    @closeVMOFnc="showDialogForm=false;" @saveDialogFormFnc="showDialogForm=false; testDialogFormFnc(dialog_data);"></dialogForm> <!-- can access var here -->

</template>

<script lang="ts">
import * as PIXI from "pixi.js";

var app;
var stars = [];
var t = 0;

const starTexture = PIXI.Texture.from('assets/star.png');

const starAmount = 1000;
let cameraZ = 0;
const fov = 20;
const baseSpeed = 0.025;
let speed = 0;
let warpSpeed = 0;
const starStretch = 5;
const starBaseSize = 0.05;

export default {
  name: "pixi",
  ready: function() {},
  methods: {
    testDialogFormFnc: function(test_var) {
      console.log(test_var.first_name);
      //we can't access vmo_dialog_data in this scope
    },
    stop() {
      app.stop();
    },
    start() {
      app.start();
    },
    reset() {
      console.log("reset");
      for (var i = 0; i < stars.length; i++) {
        var star = stars[i];
        star.x = 0;
        star.y = 0;
      }
    },
    bigBang() {},
    drawPixi(introAnim) {
      let graphics = new PIXI.Graphics() 
      graphics.lineStyle(8, 0xffff00)

      //start
      graphics.moveTo(300, 250)
      //end
      graphics.lineTo(500, 250)

      introAnim.stage.addChild(graphics)
    },
    drawAnim(introAnim) {

        // Create the stars
        const stars = [];
        for (let i = 0; i < starAmount; i++) {
            const star = {
                sprite: new PIXI.Sprite(starTexture),
                z: 0,
                x: 0,
                y: 0,
            };
            star.sprite.anchor.x = 0.5;
            star.sprite.anchor.y = 0.7;
            randomizeStar(star, true);
            app.stage.addChild(star.sprite);
            stars.push(star);
        }

        function randomizeStar(star, initial) {
            star.z = initial ? Math.random() * 2000 : cameraZ + Math.random() * 1000 + 2000;

            // Calculate star positions with radial random coordinate so no star hits the camera.
            const deg = Math.random() * Math.PI * 2;
            const distance = Math.random() * 50 + 1;
            star.x = Math.cos(deg) * distance;
            star.y = Math.sin(deg) * distance;
        }


        // Change flight speed every 5 seconds
        setInterval(() => {
            warpSpeed = warpSpeed > 0 ? 0 : 1;
        }, 5000);

        // Listen for animate update
        app.ticker.add((delta) => {
            // Simple easing. This should be changed to proper easing function when used for real.
            speed += (warpSpeed - speed) / 20;
            cameraZ += delta * 10 * (speed + baseSpeed);
            for (let i = 0; i < starAmount; i++) {
                const star = stars[i];
                if (star.z < cameraZ) randomizeStar(star);

                // Map star 3d position to 2d with really simple projection
                const z = star.z - cameraZ;
                star.sprite.x = star.x * (fov / z) * app.renderer.screen.width + app.renderer.screen.width / 2;
                star.sprite.y = star.y * (fov / z) * app.renderer.screen.width + app.renderer.screen.height / 2;

                // Calculate star scale & rotation.
                const dxCenter = star.sprite.x - app.renderer.screen.width / 2;
                const dyCenter = star.sprite.y - app.renderer.screen.height / 2;
                const distanceCenter = Math.sqrt(dxCenter * dxCenter + dyCenter * dyCenter);
                const distanceScale = Math.max(0, (2000 - z) / 2000);
                star.sprite.scale.x = distanceScale * starBaseSize;
                // Star is looking towards center so that y axis is towards center.
                // Scale the star depending on how fast we are moving, what the stretchfactor is and depending on how far away it is from the center.
                star.sprite.scale.y = distanceScale * starBaseSize + distanceScale * speed * starStretch * distanceCenter / app.renderer.screen.width;
                star.sprite.rotation = Math.atan2(dyCenter, dxCenter) + Math.PI / 2;
            }
        });


    },
  },
  mounted() {
    var canvas = document.getElementById('pixi');

    // 1. Create a Pixi renderer and define size and a background color
    app = new PIXI.Application({
      width: window.innerWidth /2,
      height: window.innerHeight / 2,
      // change background color to blue
      backgroundColor: "0x000000",
      //resolution: window.devicePixelRatio || 1, //will zoom to max screen
      view: canvas,
    });

    this.drawAnim(app);
    this.drawPixi(app);
  },

};

</script>

<style lang="scss">
body {
  height: 100%;
}
.buttons {
  position: fixed;
}
</style>



<script lang="ts" setup>

import { onMounted, reactive, ref } from 'vue'

import { MenuOptions } from '@imengyu/vue3-context-menu//src/ContextMenuDefine';
import ContextMenu from '@imengyu/vue3-context-menu'
import { useCounterStore } from "../store/counter";

import dialogForm from "@/components/DialogFormTest.vue"

const store = useCounterStore();

const dialog_data = ref({
  first_name: "sample name",
  age: ""
});
const showDialogForm = ref(false);

onMounted(() => {
  //highlight demo code
  (window as unknown as {
      hljs: {
        highlightAll: () => void
      }
    }).hljs?.highlightAll?.();
});
const menuData = reactive<MenuOptions>({
  items: [
    { 
      label: 'Increase counter value',
        onClick: () => {
           console.log("You click increase counter");
           store.increment(); 
         },
    },
    {
      label: "Show Dialog Form",
      onClick: () => {
         if (window.event) {
            var isShift = !!window.event.shiftKey;
            console.log("is shift: ", isShift);
         }
         showDialogForm.value = true;
      }
    },
    {
      label: "Sub menu Example",
      children: [
        {
          label: "Back",
          onClick: () => {
            if (window.event) {
              var isShift = !!window.event.shiftKey;
              console.log(isShift);
            }
             console.log("You click Back");
          }
        },
        { label: "Forward", disabled: true },
        { 
          label: "Reload", 
          divided: true, 
          icon: "icon-reload-1",
          onClick: () => {
            alert("You click Reload");
          }
        },
        { 
          label: "Save as...",
          icon: "icon-save",
          onClick: () => {
            alert("You click Save as");
          }
        },
        { 
          label: "Print...", 
          icon: "icon-print",
          onClick: () => {
            alert("You click Print");
          } 
        },
        { label: "View source", icon: "icon-terminal" },
        { label: "Inspect" }
      ],
    },
    {
      label: "Submenu with Submenu",
      children: [
        {
          label: "Very long submenu",
          divided: true, 
          children: [
            { label: "Test1" },
            { label: "Test2" },
            { label: "Test3" },
            { label: "Test4" },
            { label: "Test5" },
            { label: "Test6" },
            { label: "Test7" },
            { label: "Test8" },
            { label: "Test9" },
            { label: "Test10" },
          ]
        },
        { 
          label: "A submenu", 
          children: [
            { label: "日本語の基礎" },
            { label: "Item2" },
            { label: "Item3" },
          ]
        },
        { 
          label: "A submenu2", 
          children: [
            { label: "Item1" },
            { label: "Item2" },
            { label: "Item3" },
            { 
              label: "A submenu", 
              children: [
                { label: "Item1" },
                { label: "Item2" },
                { label: "Item3" },
              ]
            },
          ]
        },
      ]
    },
    { 
      label: 'Test item dynamic show and hide',
      clickClose: false,
      onClick: () => {
        menuData.items[4].hidden = !menuData.items[4].hidden;
      },
    },
    { 
      label: 'Click the item above to show/hide me',
    },
    { 
      label: 'Test item dynamic change the label',
      clickClose: false,
      onClick: () => {
        if (menuData.items[5].label === 'Test item dynamic change the label')
          menuData.items[5].label = 'My label CHANGED!';
        else
          menuData.items[5].label = 'Test item dynamic change the label';
      },
    },
    { 
      label: 'Item with icon',
      icon: "icon-reload-1",
    },
    { 
      label: "Item with svg icon",
      svgIcon: "#icon-clock",
    },
    { 
      label: "Item with svg icon",
      svgIcon: "#icon-multiply",
      svgProps: {
        fill: '#f60',
      },
    },
  ],
  iconFontClass: 'iconfont',
  customClass: "class-a",
  zIndex: 3,
  minWidth: 230,
  x: 0,
  y: 0,
});
function onContextMenu(e : MouseEvent) {
  //prevent the browser's default menu
  e.preventDefault();

  if (store.count % 2 == 0) {
    menuData.x = e.x;
    menuData.y = e.y;
    //show our menu
    ContextMenu.showContextMenu(menuData);
  }
  else {
    //menuData2.x = e.x;
    //menuData2.y = e.y;
    const menuData2 = {
        items: [
          { 
            label: 'Click to increase',
            onClick: () => {
              store.increment();
              console.log("You click increase counter");
            }
          },
          { 
            label: 'Simple item',
          },
        ],
        iconFontClass: 'iconfont',
        customClass: "class-a",
        zIndex: 3,
        minWidth: 230,
        x: e.x,
        y: e.y
      } as MenuOptions;
    ContextMenu.showContextMenu(menuData2);

  }


}

</script>