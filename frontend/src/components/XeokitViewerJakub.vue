<script setup lang="ts">
import { XKTLoaderPlugin, WebIFCLoaderPlugin } from '@xeokit/xeokit-sdk';
import { onMounted } from 'vue';
import { useRoute } from 'vue-router';
import {buildLineGeometry, buildBoxGeometry, buildSphereGeometry, Viewer, Mesh, ReadableGeometry, PhongMaterial} from '@xeokit/xeokit-sdk';
import {ContextMenu,  buildCylinderGeometry, math, transformToNode} from '@xeokit/xeokit-sdk';
import { GridGen} from '../../../backend/jakub/gridGen';
import { aStarClass } from '../../../backend/jakub/aStar';
import * as data from '../../../backend/jakub/exampleForTomek.json';
//import * as obstacles from '../../../backend/dawid/bboxes-global-coordinates.json';
import * as obstacles from '../../../frontend/server/models/BUILDING_boxes_simple.json';
// import * as obstacles from '../../../frontend/server/models/Duplex_boxes.json';
import * as pathJSON from '../../../frontend/server/models/0.json';
import { checkLine } from '@/utils/check-line';
import { Result } from 'postcss';
import ColorPicker from 'primevue/colorpicker';

const route = useRoute()
const fileName = route.params['name']

onMounted(() => {
    const viewer = new Viewer({
        canvasId: "xeokit_canvas",
        transparent: true,
        dtxEnabled: true
    });




    viewer.camera.eye = [0, 0, 8];
    viewer.camera.look = [0, 0, 0];
    viewer.camera.up = [0, 1, 0];
    const scene = viewer.scene;
    const canvas = scene.canvas.canvas;

const buildPipe = function(segments) {
    // straight cylinders without elbows
    const elements = segments.map(
        function(segment) {
            const p0 = segment[0];
            const p1 = segment[1];
            const rad = 0.1;
            const pipe = new Mesh(
                scene,
                {
                    geometry: new ReadableGeometry(
                        scene,
                        buildCylinderGeometry({
                            center: [0,0,0],
                            radiusTop: rad,
                            radiusBottom: rad,
                            heightSegments: 60,
                            widthSegments: 60
                        })),
                    material: new PhongMaterial(scene, { diffuse: [0,0,1] }),
                    pickable: false
                });

            const mat = math.identityMat4();

            math.scaleMat4v([ 1, math.distVec3(p0, p1), 1], mat);

            const dir = math.vec3();
            math.subVec3(p0, p1, dir);
            math.normalizeVec3(dir);
            const r = math.vec4();
            math.vec3PairToQuaternion([ 0, 1, 0 ], dir, r);
            math.mulMat4(math.quaternionToRotationMat4(r, math.identityMat4()), mat, mat);

            const t = math.vec3();
            math.addVec3(p0, p1, t);
            math.mulVec3Scalar(t, 0.5, t);

            math.translateMat4v(t, mat);

            pipe.matrix = mat;

            return pipe;
        });
    return { destroy: () => elements.forEach(e => e.destroy()) };
};
//buildPipe([[[0,0,0], [10,10,10]]]);
for(let idx = 0; idx < pathJSON.points.length - 1; idx++){
    let startPoint = pathJSON.points[idx];
    let endPoint = pathJSON.points[idx + 1];
    startPoint = [startPoint[0], startPoint[2], -startPoint[1]];
    endPoint = [endPoint[0], endPoint[2], -endPoint[1]];
    //buildPipe([[startPoint, endPoint]]);
}

//lines = GridGen.test();
// lines = [];
// const linesJson = data;
// linesJson.grid.edges.forEach((edge) => {
//   const coords1 = linesJson.grid.nodes[edge.node1];
//   const coords2 = linesJson.grid.nodes[edge.node2];
//    lines.push({
//        startPoint: [coords1.x, coords1.y, coords1.z],
//        endPoint: [coords2.x, coords2.y, coords2.z]
//    });
//   });
//lines = GridGen.graphEdgesToLines(GridGen.exampleGraphWithHoles());

    const room = obstacles.roomBBox;
    const span = 30;

    let lines = [];
    let obstaclesBoxes = [];
     for(const obstacle of obstacles.obstacleBBoxes){
        const min = [obstacle.x, obstacle.y, obstacle.z];
        const max = [obstacle.x + obstacle.xDist, obstacle.y + obstacle.yDist, obstacle.z + obstacle.zDist];

        obstaclesBoxes.push({
            min: min,
            max: max
        });


        //startPoint and endPoints are diagonal of box. Create 6 lines to create box
        const bboxLines = GridGen.BBoxesToLines(min, max);
        for(const line of bboxLines){
            lines.push(line);
        }
     }

     const drawBox = (min_ : number[], max_: number[], transform = true) => {
      let min = min_;
      let max = max_;
      if(transform){
         min = [min_[0], min_[2], -min_[1]];
         max = [max_[0], max_[2], -max_[1]];
      }
        new Mesh(viewer.scene, {
            geometry: new ReadableGeometry(viewer.scene, buildBoxGeometry({
                center: [0.5*(min[0]+max[0]), 0.5*(min[1]+max[1]), 0.5*(min[2]+max[2])],
                  xSize: 0.5*(max[0] - min[0]),
                  ySize: 0.5*(max[1] - min[1]),
                  zSize: 0.5*(max[2] - min[2])

            })),
              material: new PhongMaterial(viewer.scene, {
                  diffuse: [0.5, 0.5, 0.5],
                  opacity: 0.5
              })
            });
     }
     const drawSmallBox= (center : [number,number, number], transform = true) => {
      const min = [center[0] - 0.5, center[1] - 0.5, center[2] - 0.5];
      const max = [center[0] + 0.5, center[1] + 0.5, center[2] + 0.5];
      drawBox(min, max, transform);
     }

     for(const obstacle of obstaclesBoxes){
        let min = obstacle.min;
        let max = obstacle.max;
        drawBox(min, max);
     }

     //drawSmallBox([0.5745122832679534, 6.390988588399216, 0.12019529864528122], false);
     //drawSmallBox([35.36324052388949, 7.771697169878537,-17.973580836785477], false);
     drawSmallBox([1.45, 1.25, 5.34]);
      drawSmallBox([37.16, 14.86, 8.74]);

 //START
//  {
//         "nodeNr": 817,
//         "x": 1.4596782297912037,
//         "y": 1.255766270208801,
//         "z": 5.34
//       },
//END
// {
//         "nodeNr": 1558,
//         "x": 37.1596782297912,
//         "y": 14.8557662702088,
//         "z": 8.739999999999998
//       },

    //  [0.5745122832679534, -0.12019529864528122, 6.390988588399216] // node
    //  [35.36324052388949, 17.973580836785477, 7.771697169878537]


     lines = lines.map((line) => {
        return {
            startPoint: [line.startPoint[0], line.startPoint[2], -line.startPoint[1]],
            endPoint: [line.endPoint[0], line.endPoint[2], -line.endPoint[1]]
        }
     });


     const graph = GridGen.createGrid(obstacles.roomBBox, obstacles.obstacleBBoxes, 1);

     lines = lines.filter((line) => {
        return checkLine(line.startPoint as [number, number, number], line.endPoint as [number, number, number] , 0.03);
     });


    // const path = aStarClass.test();
     let pathLines = [];
     //following coordinates span the building across its diagonal
     const startNode = GridGen.nodeClosestToCoords(graph, [1.45, 1.25, 5.34]);
     const endNode = GridGen.nodeClosestToCoords(graph, [37.16, 14.86, 8.74]);
     const path = aStarClass.main(graph, startNode.nr, endNode.nr);

    // //ITERATE over idx
    for (let i = 0; i < path.length - 1; i++) {
        const line = {
            startPoint: path[i],
            endPoint: path[i + 1]
        };
        pathLines.push(line);
    }
    let graphLines =  GridGen.graphEdgesToLines(graph);
     graphLines = graphLines.filter((line) => {
        return checkLine(line.startPoint as [number, number, number], line.endPoint as [number, number, number] , 0.03);
     });
    graphLines = graphLines.map((line) => {
        return {
            startPoint: [line.startPoint[0], line.startPoint[2], -line.startPoint[1]],
            endPoint: [line.endPoint[0], line.endPoint[2], -line.endPoint[1]]
        }
     });
    pathLines = pathLines.map((line) => {
        return {
            startPoint: [line.startPoint[0], line.startPoint[2], -line.startPoint[1]],
            endPoint: [line.endPoint[0], line.endPoint[2], -line.endPoint[1]]
        }
     });
    console.log(`pathLines ${pathLines}`);
    for (const line of [
       graphLines,
      lines
    ].flat()) {
    new Mesh(viewer.scene, {
        geometry: new ReadableGeometry(viewer.scene, buildLineGeometry({
            startPoint: line.startPoint,
            endPoint: line.endPoint,
        })),
        material: new PhongMaterial(viewer.scene, {
            emissive: [0, 1,],
            opacity: 1        })
    });}

    for(let idx = 0; idx < path.length - 1; idx++){
      let startPoint = path[idx];
      let endPoint = path[idx + 1];
      startPoint = [startPoint[0], startPoint[2], -startPoint[1]];
      endPoint = [endPoint[0], endPoint[2], -endPoint[1]];
      buildPipe([[startPoint, endPoint]]);
    }



    new Mesh(viewer.scene, {
        geometry: new ReadableGeometry(viewer.scene, buildSphereGeometry({
            radius: 0.5,
            heightSegments: 60,
            widthSegments: 60
        })),
        material: new PhongMaterial(viewer.scene, {
            ambient: [0.9, 0.3, 0.9],
            shininess: 30,

        })
    });



    const xktLoader = new XKTLoaderPlugin(viewer);

    const sceneModel = xktLoader.load({
        id: "myModel",
        src: `http://127.0.0.1:5200/BUILDING.xkt`,
        edges: true,
    });
    sceneModel.xrayed = true;

});


</script>

<template>
        <canvas id="xeokit_canvas"></canvas>
</template>

<style scoped>
#xeokit_canvas {
    height: 100%;
    width: 100%;
    background: lightblue;
    background-image: linear-gradient(lightblue, white);
}
</style>
