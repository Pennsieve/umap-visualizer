// D3 WebGL Scatterplot with Zoom and Pan
// This implementation uses D3 for scales and interactions,
// and regl for WebGL rendering

// Load required libraries
import * as d3 from 'd3';

// Generate sample data
const generateData = (count) => {
const data = [];
for (let i = 0; i < count; i++) {
data.push({
x: Math.random() * 100 - 50,
y: Math.random() * 100 - 50,
color: [Math.random(), Math.random(), Math.random()]
});
}
return data;
};

// Initialize the scatterplot
function initScatterplot(container, options = {}) {
// Default options
const defaults = {
width: 800,
height: 600,
pointSize: 3,
backgroundColor: [0, 0, 0, 1],
data: generateData(10000)
};

const config = { ...defaults, ...options };
const { width, height, pointSize, backgroundColor, data } = config;

// Create SVG container
const svg = d3.select(container)
.append('svg')
.attr('width', width)
.attr('height', height)
.style('position', 'absolute')
.style('top', '0px')
.style('left', '0px')
.style('pointer-events', 'none');

// Create WebGL canvas
const canvas = d3.select(container)
.append('canvas')
.attr('width', width)
.attr('height', height)
.style('position', 'absolute')
.style('top', '0px')
.style('left', '0px');

const gl = canvas.node().getContext('webgl', { preserveDrawingBuffer: true });

if (!gl) {
throw new Error('WebGL not supported');
}

// Set up scales
const xExtent = d3.extent(data, d => d.x);
const yExtent = d3.extent(data, d => d.y);

const xScale = d3.scaleLinear()
.domain([xExtent[0], xExtent[1]])
.range([-1, 1]);

const yScale = d3.scaleLinear()
.domain([yExtent[0], yExtent[1]])
.range([-1, 1]);

// Initialize regl (a functional abstraction for WebGL)
const regl = createREGL(gl);

// Create WebGL program for rendering points
const drawPoints = regl({
frag: `
precision mediump float;
varying vec3 vColor;
void main() {
float dist = length(gl_PointCoord.xy - vec2(0.5, 0.5));
if (dist > 0.5) discard;
gl_FragColor = vec4(vColor, 1.0);
}
`,
vert: `
precision mediump float;
attribute vec2 position;
attribute vec3 color;
uniform mat4 viewMatrix;
uniform float pointSize;
varying vec3 vColor;

void main() {
vColor = color;
gl_PointSize = pointSize;
gl_Position = viewMatrix * vec4(position, 0, 1);
}
`,
attributes: {
position: () => {
return data.map(d => [xScale(d.x), yScale(d.y)]);
},
color: () => {
return data.map(d => d.color);
}
},
uniforms: {
viewMatrix: regl.prop('viewMatrix'),
pointSize: regl.prop('pointSize')
},
count: () => data.length,
primitive: 'points'
});

// Set up zoom behavior
let transform = d3.zoomIdentity;
const zoom = d3.zoom()
.scaleExtent([0.1, 100])
.on('zoom', zoomed);

canvas.call(zoom);

function zoomed(event) {
transform = event.transform;
render();
}

// Render function
function render() {
// Clear canvas
regl.clear({
color: backgroundColor,
depth: 1
});

// Create view matrix from d3 transform
const viewMatrix = [
transform.k, 0, 0, 0,
0, transform.k, 0, 0,
0, 0, 1, 0,
transform.x / width * 2, transform.y / height * 2, 0, 1
];

// Draw points
drawPoints({
viewMatrix: viewMatrix,
pointSize: pointSize * Math.sqrt(transform.k)
});
}

// Initial render
render();

// Create axes on SVG layer
const xAxis = d3.axisBottom(
d3.scaleLinear()
.domain(xExtent)
.range([0, width])
);

const yAxis = d3.axisLeft(
d3.scaleLinear()
.domain(yExtent)
.range([height, 0])
);

const gX = svg.append('g')
.attr('transform', `translate(0, ${height})`)
.call(xAxis);

const gY = svg.append('g')
.call(yAxis);

// Update axes on zoom
zoom.on('zoom', (event) => {
transform = event.transform;

gX.call(xAxis.scale(transform.rescaleX(
d3.scaleLinear().domain(xExtent).range([0, width])
)));

gY.call(yAxis.scale(transform.rescaleY(
d3.scaleLinear().domain(yExtent).range([height, 0])
)));

render();
});

// Helper function to simulate regl library
function createREGL(gl) {
// This is a simplified placeholder for the full regl library
// In a real implementation, you would use the actual regl library

return {
clear: ({ color, depth }) => {
gl.clearColor(color[0], color[1], color[2], color[3]);
gl.clearDepth(depth);
gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
},
prop: (name) => (props) => props[name],
createBuffer: (data) => {
const buffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(data.flat()), gl.STATIC_DRAW);
return buffer;
},
// Simplified version - in real implementation, this would be more complex
buffer: (data) => createREGL.createBuffer(data),
// Simplified implementation of the regl function
// In a real implementation, you would use the actual regl function
draw: (props) => {
// This would execute the WebGL drawing commands
}
};
}

// Return API for the scatterplot
return {
update: (newData) => {
data = newData;
render();
},
resize: (newWidth, newHeight) => {
width = newWidth;
height = newHeight;
canvas.attr('width', width).attr('height', height);
svg.attr('width', width).attr('height', height);
render();
},
destroy: () => {
canvas.remove();
svg.remove();
}
};
}

// Usage example:
// const scatterplot = initScatterplot('#container');