<script >
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { GUI } from 'dat.gui'
import { FBXLoader } from 'three/addons/loaders/FBXLoader.js';

export default {
  name: 'Ambiental',
  //data() {},
methods: {

    main:function main(){

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

    //========================= Escena ============================
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x5499c7);
        var canvas = document.querySelector('#mi_canvas');
  
    //========================== Ejes =============================
  
    //-------------------------- Eje X ----------------------------
        const ejeXvertices = [];
        ejeXvertices.push( new THREE.Vector3( 0, 0, 0 ) );
        ejeXvertices.push( new THREE.Vector3( 2, 0, 0 ) );
        const colorEjeX = new THREE.LineBasicMaterial( { color:  0x943126 } );
    
        const geometriaX = new THREE.BufferGeometry().setFromPoints( ejeXvertices );
        const ejeX = new THREE.Line( geometriaX, colorEjeX );
    
        scene.add(ejeX);
  
    //-------------------------- Eje Y ----------------------------
        const ejeYvertices = [];
        ejeYvertices.push( new THREE.Vector3( 0, 0, 0 ) );
        ejeYvertices.push( new THREE.Vector3(  0, 2, 0 ) );
        const colorEjeY = new THREE.LineBasicMaterial( { color:   0x2ecc71  } );
    
        const geometriaY = new THREE.BufferGeometry().setFromPoints( ejeYvertices );
        const ejeY = new THREE.Line( geometriaY, colorEjeY );
    
        scene.add(ejeY);
  
    //-------------------------- Eje Z ----------------------------
        const ejeZvertices = [];
        ejeZvertices.push( new THREE.Vector3( 0, 0, 0 ) );
        ejeZvertices.push( new THREE.Vector3(  0, 0, 2 ) );
        const colorEjeZ = new THREE.LineBasicMaterial( { color:   0x3498db  } );
        
        const geometriaZ = new THREE.BufferGeometry().setFromPoints( ejeZvertices );
        const ejeZ = new THREE.Line( geometriaZ, colorEjeZ );

        scene.add(ejeZ);
  
    //========================= Luces =============================
        const color = 0xFFFFFF;
        const intensity = 1;
        var ambientLight = new THREE.AmbientLight(color, intensity);        
        scene.add(ambientLight);

        const skyColor = 0xB1E1FF;  // light blue
        const groundColor = 0xB97A20;  // brownish orange
        const  hemisphereLight = new THREE.HemisphereLight(skyColor, groundColor, intensity);
        scene.add(hemisphereLight);
  
  
    //========================= Camara =============================
        const camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 0.1, 20);
        camera.position.x = 4;
        camera.position.y = 3;
        camera.position.z = 4;
        camera.lookAt(0,0,0);
        scene.add(camera);
  
    //========================== Modelos ==============================
    
        let mixer;
        const loader = new FBXLoader();
        loader.load( './src/assets/Models/Whale/Whale.fbx', function ( object ) { //Whale by Quaternius via Poly Pizza

            mixer = new THREE.AnimationMixer( object );

            const action = mixer.clipAction( object.animations[ 0 ] );
            action.play();

            scene.add( object );
            object.scale.x = 0.005;
            object.scale.y = 0.005;
            object.scale.z = 0.005;
        } );
      
    //========================== Render =============================
  
        const renderer = new THREE.WebGLRenderer({canvas: canvas});    
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
        controls.target.set( 0, 0, 0 );
        controls.update();
  
    //============================ GUI ============================
  
        const gui1 = new GUI( { autoPlace: false } );
        var customContainer = document.querySelector('#gui').append(gui1.domElement);
        
        const ambientalGUI = gui1.addFolder('Ambiental');
        ambientalGUI.addColor(new ColorGUIHelper(ambientLight, 'color'), 'value').name('Color');
        ambientalGUI.add(ambientLight, 'intensity', 0, 2, 0.01).name("Intensidad");

        const hemisphereGUI = gui1.addFolder('Hemisferio');
        hemisphereGUI.addColor(new ColorGUIHelper(hemisphereLight, 'color'), 'value').name('Superior');
        hemisphereGUI.addColor(new ColorGUIHelper(hemisphereLight, 'groundColor'), 'value').name('Inferior');
        hemisphereGUI.add(hemisphereLight, 'intensity', 0, 2, 0.01).name("Intensidad");
  
  //========================= Visualiza =========================
        const clock = new THREE.Clock();

        function animate() {
            requestAnimationFrame( animate );

            const delta = clock.getDelta();
            if ( mixer ) mixer.update( delta );            
    
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
    <h3>Luz ambiente</h3>
    <p>Fuente luminosa difusa no direccional, resultado de la reflexión de la 
        luz en el entorno de la escena.</p>
    <p>Viene dado por dos parámetros:
        <ul>
            <li><b>Intensidad</b> de la luz.</li>
            <li><b>Coeficiente</b> (K) de reflexión.</li>
        </ul>
    </p>
    <p>En este caso se permite modificar tanto la intensidad como el color de la luz.</p>

    <h3>Luz de hemisferio</h3>
    <p>Es una variante de la luz ambiental y se puede combinar con esta. Representa la luz reflejada por el cielo, 
        que incide desde arriba, y la luz reflejada por el suelo, que indice desde abajo.
    </p>
</template>

