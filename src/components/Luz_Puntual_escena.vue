<script >
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { GUI } from 'dat.gui'
import { OBJLoader } from 'three/addons/loaders/OBJLoader.js';
import { MTLLoader } from 'three/addons/loaders/MTLLoader.js';

export default {
    name: 'Puntual',
    methods: {

    main: function (){
        class ColorGUIHelper {
            constructor(object, prop) {
                this.object = object;
                this.prop = prop;
            }
            get value() {
                return `#${this.object[this.prop].getHexString()}`;
            }
            set value(hexString) {
                this.object[this.prop].set(hexString);
            }
        }

    //========================= Escenas ============================
        const scene = new THREE.Scene();
        var canvas = document.querySelector('#mi_canvas');
    
    //========================== Ejes =============================
    
    //-------------------------- Eje X ----------------------------
        const ejeXvertices = [];
        ejeXvertices.push( new THREE.Vector3( 0, 7, 0 ) );
        ejeXvertices.push( new THREE.Vector3( 2, 7, 0 ) );
        const colorEjeX = new THREE.LineBasicMaterial( { color:  0x943126 } );
    
        const geometriaX = new THREE.BufferGeometry().setFromPoints( ejeXvertices );
        const ejeX = new THREE.Line( geometriaX, colorEjeX );
    
        scene.add(ejeX);
    
    //-------------------------- Eje Y ----------------------------
        const ejeYvertices = [];
        ejeYvertices.push( new THREE.Vector3( 0, 7, 0 ) );
        ejeYvertices.push( new THREE.Vector3(  0, 9, 0 ) );
        const colorEjeY = new THREE.LineBasicMaterial( { color:   0x2ecc71  } );
    
        const geometriaY = new THREE.BufferGeometry().setFromPoints( ejeYvertices );
        const ejeY = new THREE.Line( geometriaY, colorEjeY );
    
        scene.add(ejeY);
    
    //-------------------------- Eje Z ----------------------------
        const ejeZvertices = [];
        ejeZvertices.push( new THREE.Vector3( 0, 7, 0 ) );
        ejeZvertices.push( new THREE.Vector3(  0, 7, 2 ) );
        const colorEjeZ = new THREE.LineBasicMaterial( { color:   0x3498db  } );
        
        const geometriaZ = new THREE.BufferGeometry().setFromPoints( ejeZvertices );
        const ejeZ = new THREE.Line( geometriaZ, colorEjeZ );
    
        scene.add(ejeZ);
    
    //========================= Luces ============================= 
        const pivot = new THREE.Object3D();
        pivot.position.set(0, 7, 0);
        scene.add(pivot);

        const color = 0xFFFFFF;
        const intensity = 1;
        const light = new THREE.PointLight(color, intensity);
        light.position.set(0, 0, 3);
        pivot.add(light);

        light.castShadow = true;
        light.shadow.mapSize.width = 800,
        light.shadow.mapSize.height = 800; 
        light.shadow.camera.near = 0.5; 
        light.shadow.camera.far = 500;

    //-------------------------- Bombilla ----------------------------     
        const bombillaGeometria = new THREE.SphereGeometry(0.05, 8, 8);
		const bombillaMaterial = new THREE.MeshToonMaterial( { color: 0xFFFF00});
		var bombilla = new THREE.Mesh(bombillaGeometria, bombillaMaterial);
        light.add(bombilla);
    
    //========================= Camara =============================
        const camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 0.1, 50);
        camera.position.x = 0;
        camera.position.y = 7;
        camera.position.z = 4;
        camera.lookAt(0,7,0);
        scene.add(camera);         
    
    //========================== Modelos ==============================
    
        const planeGeometry = new THREE.PlaneGeometry( 15, 7 );
        const textureLoader = new THREE.TextureLoader();
        var planeMat = new THREE.MeshToonMaterial( { 
            map: textureLoader.load('./src/assets/img/eva01.png')
         } );
        const plano = new THREE.Mesh( planeGeometry, planeMat );
        plano.position.set(0,7,-3);
        plano.receiveShadow = true;
        scene.add( plano );

        //-------------------------- OBJ ----------------------------
        
        const mtlLoader = new MTLLoader();
        const objLoader = new OBJLoader();

        mtlLoader.load('./src/assets/Models/eva01.mtl', (mtl) => {
            mtl.preload();
            objLoader.setMaterials(mtl);
            
            objLoader.load('./src/assets/Models/eva01.obj', (root) => {
                scene.add(root);

                root.traverse(function(node){
                        if(node.isMesh){
                            node.castShadow = true;
                            node.receiveShadow = true;
                        }
                    });
              });      
        });
        
    
    //========================== Render =============================
        
        const renderer = new THREE.WebGLRenderer({canvas: canvas});
        renderer.shadowMap.enabled = true;
        renderer.shadowMapSoft = true;

        renderer.render(scene, camera);
    
        function resizeRendererToDisplaySize(renderer) {
            const canvas = renderer.domElement;
            const width = canvas.clientWidth;
            const height = canvas.clientHeight;
            const needResize = canvas.width !== width || canvas.height !== height;
            if (needResize) {
                renderer.setSize(width, height, false);
            }
            return needResize;
        }

    //========================= Controls =============================
        const controls = new OrbitControls( camera, renderer.domElement );
        controls.target.set(0, 7, 0);
        controls.update();
    
    //============================ GUI ============================
    
        const gui1 = new GUI( { autoPlace: false } );
        var customContainer = document.querySelector('#gui').append(gui1.domElement);
        
        const puntualGUI = gui1.addFolder('Puntal');
        puntualGUI.addColor(new ColorGUIHelper(light, 'color'), 'value').name('Color');
        puntualGUI.add(light, 'intensity', 0, 2, 0.01).name("Intensiadad");
        puntualGUI.add(light, 'distance', 1, 10, 1).setValue(5).name("Atenuación"); //onChange( helper.update()).    
    
    //========================= Visualiza =========================
    
        function animate() {
            requestAnimationFrame( animate );

            pivot.rotation.y += 0.01;
            pivot.rotation.x += 0.01;
    
            if (resizeRendererToDisplaySize(renderer)) {
                const canvas = renderer.domElement;
                camera.aspect = canvas.clientWidth / canvas.clientHeight;
                camera.updateProjectionMatrix();
            }
    
            renderer.render( scene, camera );
        };  
    
        animate()
    }
  },
  mounted() {
      this.main();
  }
}
</script>


<template>
    <h3>Luz puntual</h3>
    <p>Como su nombre indica, simula una fuente luminosa situada en una posición concreta y hemite luz 
        en todas direcciones. Parecido a una bombilla.</p>
    <p>
        Parámetros:
        <ul>
            <li><b>Intensidad</b>.</li>
            <li><b>Posición</b> dadas por coordenadas (x, y, z).</li>
            <li><b>Atenuación</b> que delimita el alcance de los rayos que emite.</li>
        </ul>
    </p>
</template>
