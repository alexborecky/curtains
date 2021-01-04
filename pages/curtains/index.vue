<template>
    <div class="hero">
        <div id="canvas"></div>
            <!-- div used to create our plane -->
            <div class="plane">
            <!-- image that will be used as texture by our plane -->
            <img src="https://images.unsplash.com/photo-1544198365-f5d60b6d8190?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1350&q=80" />
        </div>
    </div>
</template>

<script>
import {Curtains, Plane} from 'curtainsjs';

    export default {
        mounted () {
            window.addEventListener("load", () => {
            // set up our WebGL context and append the canvas to our wrapper
            const curtains = new Curtains({
            container: "canvas"
            });
            // get our plane element
            const planeElement = document.getElementsByClassName("plane")[0];
            // set our initial parameters (basic uniforms)
            const params = {
            vertexShaderID: "plane-vs", // our vertex shader ID
            fragmentShaderID: "plane-fs", // our fragment shader ID
            uniforms: {
            time: {
            name: "uTime", // uniform name that will be passed to our shaders
            type: "1f", // this means our uniform is a float
            value: 0,
            },
            },
            };
            // create our plane using our curtains object, the HTML element and the parameters
            const plane = new Plane(curtains, planeElement, params);
            plane.onRender(() => {
            // use the onRender method of our plane fired at each requestAnimationFrame call
            plane.uniforms.time.value++; // update our time uniform value
            });
            });
        },
        head () {
            return { 
                __dangerouslyDisableSanitizers: ['script'],
                script: [
                {
                    innerHTML: `
                    precision mediump float;
                    // those are the mandatory attributes that the lib sets
                    attribute vec3 aVertexPosition;
                    attribute vec2 aTextureCoord;
                    // those are mandatory uniforms that the lib sets and that contain our model view and projection matrix
                    uniform mat4 uMVMatrix;
                    uniform mat4 uPMatrix;
                    // our texture matrix that will handle image cover
                    uniform mat4 uTextureMatrix0;
                    // pass your vertex and texture coords to the fragment shader
                    varying vec3 vVertexPosition;
                    varying vec2 vTextureCoord;
                    void main() {
                    vec3 vertexPosition = aVertexPosition;
                    gl_Position = uPMatrix * uMVMatrix * vec4(vertexPosition, 1.0);
                    // set the varyings
                    // here we use our texture matrix to calculate the accurate texture coords
                    vTextureCoord = (uTextureMatrix0 * vec4(aTextureCoord, 0.0, 1.0)).xy;
                    vVertexPosition = vertexPosition;
                    }
                    `,
                    type: 'x-shader/x-vertex',
                    id: 'plane-vs',
                    body: true
                },
                {
                    type: 'x-shader/x-fragment',
                    id: 'plane-fs',
                    body: true,
                    innerHTML: `
                    precision mediump float;
                    // get our varyings
                    varying vec3 vVertexPosition;
                    varying vec2 vTextureCoord;
                    // the uniform we declared inside our javascript
                    uniform float uTime;
                    // our texture sampler (default name, to use a different name please refer to the documentation)
                    uniform sampler2D uSampler0;
                    void main() {
                    // get our texture coords from our varying
                    vec2 textureCoord = vTextureCoord;
                    // displace our pixels along the X axis based on our time uniform
                    // textures coords are ranging from 0.0 to 1.0 on both axis
                    textureCoord.x += sin(textureCoord.y * 25.0) * cos(textureCoord.x * 25.0) * (cos(uTime / 50.0)) / 25.0;
                    // map our texture with the texture matrix coords
                    gl_FragColor = texture2D(uSampler0, textureCoord);
                    }
                    `,
                }
                ]
            }
        }
    }
</script>

<style lang="scss" scoped>

.hero {
    width: 100%;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
}

body {
/* make the body fits our viewport */
position: relative;
width: 100%;
height: 100vh;
margin: 0;
overflow: hidden;
}
#canvas {
/* make the canvas wrapper fits the document */
position: absolute;
top: 0;
right: 0;
bottom: 0;
left: 0;
}
.plane {
/* define the size of your plane */
width: 80%;
height: 80vh;
margin: 10vh auto;
}
.plane img {
/* hide the img element */
display: none;
}

</style>