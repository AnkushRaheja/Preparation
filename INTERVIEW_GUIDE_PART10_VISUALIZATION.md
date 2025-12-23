# 📊 Part 10: Data Visualization Complete Guide

> **Master D3.js, Recharts, SVG, and visualization patterns for frontend interviews**

---

## 📋 Table of Contents

1. [SVG Fundamentals](#1-svg-fundamentals)
2. [D3.js Core Concepts](#2-d3js-core-concepts)
3. [D3 Scales](#3-d3-scales)
4. [D3 Axes](#4-d3-axes)
5. [D3 Shapes & Paths](#5-d3-shapes--paths)
6. [D3 Transitions & Animations](#6-d3-transitions--animations)
7. [D3 with React](#7-d3-with-react)
8. [Recharts Complete Guide](#8-recharts-complete-guide)
9. [Chart Types](#9-chart-types)
10. [Responsive Visualization](#10-responsive-visualization)
11. [Canvas vs SVG](#11-canvas-vs-svg)
12. [Performance Optimization](#12-performance-optimization)
13. [Interview Questions](#13-interview-questions)
14. [Coding Challenges](#14-coding-challenges)

---

# 1. SVG Fundamentals

## 1.1 What is SVG?

```typescript
// SVG = Scalable Vector Graphics
// XML-based format for 2D vector graphics
// Resolution-independent (scales perfectly)

const SVG_CHARACTERISTICS = {
  format: 'XML-based markup',
  rendering: 'Vector (not raster/bitmap)',
  scaling: 'Perfect at any size',
  accessibility: 'Text is searchable, elements accessible',
  animation: 'CSS and SMIL animations supported',
  interactivity: 'DOM events work on elements',
  fileSize: 'Usually smaller for simple graphics',
};

// SVG in React
function SimpleSVG() {
  return (
    <svg width="200" height="200" viewBox="0 0 200 200">
      <circle cx="100" cy="100" r="50" fill="blue" />
    </svg>
  );
}
```

## 1.2 SVG Coordinate System

```typescript
// SVG uses a coordinate system where:
// - Origin (0,0) is at TOP-LEFT
// - X increases to the right
// - Y increases DOWNWARD (opposite of math!)

function CoordinateDemo() {
  return (
    <svg width="400" height="300" style={{ border: '1px solid black' }}>
      {/* Origin point */}
      <circle cx="0" cy="0" r="5" fill="red" />
      <text x="10" y="20">Origin (0,0)</text>
      
      {/* X-axis demonstration */}
      <line x1="0" y1="50" x2="400" y2="50" stroke="gray" />
      <text x="380" y="45">X →</text>
      
      {/* Y-axis demonstration */}
      <line x1="50" y1="0" x2="50" y2="300" stroke="gray" />
      <text x="55" y="290">Y ↓</text>
      
      {/* Point example */}
      <circle cx="200" cy="150" r="5" fill="blue" />
      <text x="210" y="155">(200, 150)</text>
    </svg>
  );
}
```

## 1.3 ViewBox and Viewport

```typescript
// Viewport = actual size of SVG element (width/height)
// ViewBox = coordinate system used internally

// viewBox="minX minY width height"

function ViewBoxDemo() {
  return (
    <>
      {/* Viewport: 400x300, ViewBox: 0 0 100 100 */}
      {/* Content scales to fit! */}
      <svg 
        width="400" 
        height="300" 
        viewBox="0 0 100 100"
        preserveAspectRatio="xMidYMid meet"
      >
        <rect x="0" y="0" width="100" height="100" fill="lightgray" />
        <circle cx="50" cy="50" r="40" fill="blue" />
      </svg>
    </>
  );
}

// preserveAspectRatio options:
const ASPECT_RATIO_OPTIONS = {
  'none': 'Stretch to fill (distorts)',
  'xMinYMin meet': 'Align top-left, fit inside',
  'xMidYMid meet': 'Center, fit inside (default)',
  'xMaxYMax meet': 'Align bottom-right, fit inside',
  'xMidYMid slice': 'Center, fill and crop',
};
```

## 1.4 Basic SVG Shapes

```typescript
function AllShapes() {
  return (
    <svg width="600" height="400" viewBox="0 0 600 400">
      {/* Rectangle */}
      <rect 
        x="10" y="10" 
        width="80" height="60" 
        fill="red" 
        stroke="black" 
        strokeWidth="2"
        rx="10" ry="10"  // Rounded corners
      />
      
      {/* Circle */}
      <circle 
        cx="150" cy="40" 
        r="30" 
        fill="blue" 
        stroke="black" 
        strokeWidth="2"
      />
      
      {/* Ellipse */}
      <ellipse 
        cx="250" cy="40" 
        rx="40" ry="25"  // Different x and y radii
        fill="green"
      />
      
      {/* Line */}
      <line 
        x1="310" y1="10" 
        x2="380" y2="70" 
        stroke="purple" 
        strokeWidth="3"
        strokeLinecap="round"
      />
      
      {/* Polyline (connected lines, not closed) */}
      <polyline 
        points="400,70 430,20 460,50 490,10 520,60"
        fill="none" 
        stroke="orange" 
        strokeWidth="2"
      />
      
      {/* Polygon (closed shape) */}
      <polygon 
        points="50,150 100,100 150,150 130,200 70,200"
        fill="teal"
        stroke="black"
      />
      
      {/* Text */}
      <text 
        x="200" y="150" 
        fontSize="24" 
        fontFamily="Arial"
        fill="black"
        textAnchor="middle"  // Horizontal alignment
        dominantBaseline="middle"  // Vertical alignment
      >
        Hello SVG!
      </text>
      
      {/* Path (most powerful - can draw anything) */}
      <path
        d="M 300 100 L 350 150 Q 400 100 350 50 Z"
        fill="pink"
        stroke="black"
      />
    </svg>
  );
}
```

## 1.5 SVG Path Commands

```typescript
// The <path> element is the most powerful SVG shape
// d="..." attribute contains path commands

const PATH_COMMANDS = {
  // Move to (start point)
  'M x y': 'Move to absolute position',
  'm dx dy': 'Move relative to current',
  
  // Line commands
  'L x y': 'Line to absolute position',
  'l dx dy': 'Line relative',
  'H x': 'Horizontal line to x',
  'h dx': 'Horizontal line relative',
  'V y': 'Vertical line to y',
  'v dy': 'Vertical line relative',
  
  // Curve commands
  'C x1 y1 x2 y2 x y': 'Cubic Bezier curve',
  'S x2 y2 x y': 'Smooth cubic Bezier',
  'Q x1 y1 x y': 'Quadratic Bezier curve',
  'T x y': 'Smooth quadratic Bezier',
  
  // Arc
  'A rx ry rotation large-arc sweep x y': 'Elliptical arc',
  
  // Close
  'Z': 'Close path (line to start)',
};

// Examples
function PathExamples() {
  return (
    <svg width="400" height="300">
      {/* Triangle */}
      <path 
        d="M 50 100 L 100 20 L 150 100 Z" 
        fill="red" 
      />
      
      {/* Curved line (Quadratic Bezier) */}
      <path 
        d="M 200 100 Q 250 20 300 100" 
        fill="none" 
        stroke="blue" 
        strokeWidth="2"
      />
      
      {/* Heart shape */}
      <path
        d="M 100 200 
           C 100 160, 50 160, 50 200 
           C 50 240, 100 260, 100 300
           C 100 260, 150 240, 150 200
           C 150 160, 100 160, 100 200"
        fill="red"
      />
    </svg>
  );
}
```

## 1.6 SVG Transforms

```typescript
function TransformExamples() {
  return (
    <svg width="500" height="200">
      {/* Original */}
      <rect x="10" y="10" width="40" height="30" fill="gray" />
      
      {/* Translate (move) */}
      <rect 
        x="10" y="10" 
        width="40" height="30" 
        fill="red"
        transform="translate(60, 0)"
      />
      
      {/* Rotate */}
      <rect 
        x="10" y="10" 
        width="40" height="30" 
        fill="blue"
        transform="translate(140, 50) rotate(45)"
      />
      
      {/* Scale */}
      <rect 
        x="10" y="10" 
        width="40" height="30" 
        fill="green"
        transform="translate(230, 0) scale(1.5)"
      />
      
      {/* Skew */}
      <rect 
        x="10" y="10" 
        width="40" height="30" 
        fill="purple"
        transform="translate(320, 0) skewX(20)"
      />
      
      {/* Multiple transforms (applied right to left) */}
      <rect 
        x="10" y="10" 
        width="40" height="30" 
        fill="orange"
        transform="translate(400, 50) rotate(30) scale(1.2)"
      />
    </svg>
  );
}

// Transform origin in SVG (different from CSS!)
// SVG transforms around (0,0) by default
// To rotate around center, translate first:

function RotateAroundCenter() {
  const cx = 100, cy = 100;
  const width = 80, height = 60;
  
  return (
    <svg width="200" height="200">
      <rect
        x={cx - width / 2}
        y={cy - height / 2}
        width={width}
        height={height}
        fill="blue"
        transform={`rotate(45, ${cx}, ${cy})`}
      />
    </svg>
  );
}
```

## 1.7 SVG Groups and Defs

```typescript
function GroupsAndDefs() {
  return (
    <svg width="400" height="300">
      {/* Defs - reusable definitions (not rendered) */}
      <defs>
        {/* Gradient */}
        <linearGradient id="myGradient" x1="0%" y1="0%" x2="100%" y2="0%">
          <stop offset="0%" stopColor="red" />
          <stop offset="100%" stopColor="blue" />
        </linearGradient>
        
        {/* Pattern */}
        <pattern id="dots" width="10" height="10" patternUnits="userSpaceOnUse">
          <circle cx="5" cy="5" r="2" fill="gray" />
        </pattern>
        
        {/* Clip path */}
        <clipPath id="circleClip">
          <circle cx="100" cy="100" r="50" />
        </clipPath>
        
        {/* Symbol (reusable graphic) */}
        <symbol id="star" viewBox="0 0 24 24">
          <path d="M12 2 L15 9 L22 9 L17 14 L19 22 L12 17 L5 22 L7 14 L2 9 L9 9 Z" />
        </symbol>
      </defs>
      
      {/* Using gradient */}
      <rect x="10" y="10" width="100" height="50" fill="url(#myGradient)" />
      
      {/* Using pattern */}
      <rect x="120" y="10" width="100" height="50" fill="url(#dots)" />
      
      {/* Using clip path */}
      <image 
        href="/image.jpg" 
        width="200" height="200"
        clipPath="url(#circleClip)"
      />
      
      {/* Group - apply transforms/styles to multiple elements */}
      <g fill="green" transform="translate(250, 50)">
        <rect x="0" y="0" width="30" height="30" />
        <rect x="40" y="0" width="30" height="30" />
        <rect x="20" y="40" width="30" height="30" />
      </g>
      
      {/* Using symbol */}
      <use href="#star" x="10" y="150" width="50" height="50" fill="gold" />
      <use href="#star" x="70" y="150" width="30" height="30" fill="silver" />
    </svg>
  );
}
```

---

# 2. D3.js Core Concepts

## 2.1 What is D3?

```typescript
// D3 = Data-Driven Documents
// A library for binding data to DOM and applying data-driven transformations

// D3 is NOT a charting library - it's a data visualization toolkit
// You build visualizations from primitives

import * as d3 from 'd3';

const D3_PHILOSOPHY = {
  dataBinding: 'Bind data arrays to DOM elements',
  enterUpdateExit: 'Handle data changes elegantly',
  selections: 'jQuery-like element selection',
  scales: 'Map data values to visual values',
  generators: 'Create SVG paths from data',
  transitions: 'Animate between states',
};
```

## 2.2 Selections

```typescript
// D3 selections are like jQuery - select and manipulate elements

// Select single element
const svg = d3.select('#my-svg');

// Select all matching elements
const circles = d3.selectAll('circle');

// Chaining operations
d3.select('#chart')
  .append('svg')
  .attr('width', 400)
  .attr('height', 300)
  .style('background', '#f0f0f0');

// Selection methods
const SELECTION_METHODS = {
  // Selecting
  'd3.select(selector)': 'Select first matching',
  'd3.selectAll(selector)': 'Select all matching',
  'selection.select(selector)': 'Select descendant',
  'selection.selectAll(selector)': 'Select all descendants',
  
  // Modifying
  'selection.attr(name, value)': 'Set attribute',
  'selection.style(name, value)': 'Set CSS style',
  'selection.classed(names, bool)': 'Toggle CSS class',
  'selection.property(name, value)': 'Set DOM property',
  'selection.text(value)': 'Set text content',
  'selection.html(value)': 'Set inner HTML',
  
  // Creating/Removing
  'selection.append(type)': 'Append new element',
  'selection.insert(type, before)': 'Insert before element',
  'selection.remove()': 'Remove elements',
  
  // Traversing
  'selection.node()': 'Get first DOM node',
  'selection.nodes()': 'Get all DOM nodes',
  'selection.empty()': 'Check if empty',
  'selection.size()': 'Count elements',
};
```

## 2.3 Data Binding (The Enter-Update-Exit Pattern)

```typescript
// This is THE core concept of D3!

const data = [10, 20, 30, 40, 50];

// Step 1: Bind data to selection
const circles = d3.select('svg')
  .selectAll('circle')
  .data(data);

// Step 2: Handle the three cases

// ENTER: New data points (no existing elements)
circles.enter()
  .append('circle')
  .attr('cx', (d, i) => i * 50 + 25)
  .attr('cy', 50)
  .attr('r', d => d)
  .attr('fill', 'blue');

// UPDATE: Existing elements with new data
circles
  .attr('r', d => d)
  .attr('fill', 'green');

// EXIT: Elements without data (should be removed)
circles.exit()
  .remove();

// Modern D3 (v6+) with join() - simpler!
function modernDataBinding(data: number[]) {
  d3.select('svg')
    .selectAll('circle')
    .data(data)
    .join(
      enter => enter
        .append('circle')
        .attr('cx', (d, i) => i * 50 + 25)
        .attr('cy', 50)
        .attr('fill', 'blue'),
      update => update
        .attr('fill', 'green'),
      exit => exit
        .attr('fill', 'red')
        .call(exit => exit.transition().duration(500).attr('r', 0).remove())
    )
    .attr('r', d => d);
}
```

## 2.4 Data Key Function

```typescript
// By default, D3 binds data by index
// Use key function for object identity

interface DataPoint {
  id: string;
  value: number;
  name: string;
}

const data: DataPoint[] = [
  { id: '1', value: 10, name: 'A' },
  { id: '2', value: 20, name: 'B' },
  { id: '3', value: 30, name: 'C' },
];

// Without key: reordering data causes wrong animations
// With key: D3 tracks each element properly

d3.select('svg')
  .selectAll('rect')
  .data(data, d => d.id)  // Key function!
  .join('rect')
  .attr('x', (d, i) => i * 50)
  .attr('width', 40)
  .attr('height', d => d.value * 5)
  .attr('fill', 'steelblue');

// Now if data order changes, D3 will:
// - Move existing elements smoothly
// - Not recreate elements unnecessarily
// - Properly animate additions/removals
```

---

# 3. D3 Scales

## 3.1 What are Scales?

```typescript
// Scales map a domain (data values) to a range (visual values)
// domain → input data range
// range → output visual range

import { scaleLinear, scaleBand, scaleTime, scaleOrdinal } from 'd3-scale';

// Example: Map temperature (0-100°F) to pixels (0-500px)
const tempScale = scaleLinear()
  .domain([0, 100])    // Data range
  .range([0, 500]);    // Visual range

tempScale(0);    // 0
tempScale(50);   // 250
tempScale(100);  // 500
tempScale(75);   // 375

// Invert: get data value from pixel
tempScale.invert(250);  // 50
```

## 3.2 Continuous Scales

```typescript
// scaleLinear - linear mapping
const linearScale = scaleLinear()
  .domain([0, 100])
  .range([0, 500]);

// Methods
linearScale(50);           // 250
linearScale.invert(250);   // 50
linearScale.ticks(5);      // [0, 20, 40, 60, 80, 100]
linearScale.nice();        // Round domain to nice values

// Clamping - restrict output to range
const clampedScale = scaleLinear()
  .domain([0, 100])
  .range([0, 500])
  .clamp(true);

clampedScale(150);  // 500 (clamped, not 750)
clampedScale(-50);  // 0 (clamped, not -250)

// scaleLog - logarithmic (for exponential data)
const logScale = scaleLog()
  .domain([1, 1000])  // Cannot include 0!
  .range([0, 300]);

// scalePow - power/exponential
const sqrtScale = scaleSqrt()  // Same as scalePow().exponent(0.5)
  .domain([0, 100])
  .range([0, 50]);

// scaleTime - for dates
const timeScale = scaleTime()
  .domain([new Date('2024-01-01'), new Date('2024-12-31')])
  .range([0, 1000]);

timeScale(new Date('2024-07-01'));  // ~500
```

## 3.3 Ordinal and Band Scales

```typescript
// scaleOrdinal - discrete input, discrete output
const colorScale = scaleOrdinal<string>()
  .domain(['Apple', 'Banana', 'Cherry'])
  .range(['red', 'yellow', 'crimson']);

colorScale('Apple');   // 'red'
colorScale('Banana');  // 'yellow'
colorScale('Cherry');  // 'crimson'
colorScale('Date');    // 'red' (cycles back)

// D3 color schemes
import { schemeCategory10, schemeTableau10 } from 'd3-scale-chromatic';

const categoryColors = scaleOrdinal(schemeCategory10);

// scaleBand - for bar charts!
const categories = ['A', 'B', 'C', 'D', 'E'];

const bandScale = scaleBand<string>()
  .domain(categories)
  .range([0, 500])
  .padding(0.2);          // Gap between bars (0-1)

bandScale('A');            // 0
bandScale('B');            // 104
bandScale.bandwidth();     // 80 (width of each band)
bandScale.step();          // 100 (band + padding)

// For bar charts:
// x position = bandScale(category)
// width = bandScale.bandwidth()

// scalePoint - like band but for points (no width)
const pointScale = scalePoint<string>()
  .domain(categories)
  .range([0, 500])
  .padding(0.5);

pointScale('A');  // 50
pointScale('B');  // 162.5
```

## 3.4 Color Scales

```typescript
// Sequential scales - for continuous data
import { 
  scaleSequential, 
  interpolateViridis,
  interpolateBlues,
  interpolateRdYlGn 
} from 'd3';

const colorScale = scaleSequential(interpolateViridis)
  .domain([0, 100]);

colorScale(0);   // Dark purple
colorScale(50);  // Green
colorScale(100); // Yellow

// Diverging scales - for data with meaningful midpoint
import { scaleDiverging, interpolateRdBu } from 'd3';

const divergingScale = scaleDiverging(interpolateRdBu)
  .domain([-10, 0, 10]);  // [min, mid, max]

divergingScale(-10);  // Red
divergingScale(0);    // White
divergingScale(10);   // Blue

// Quantize - continuous to discrete buckets
const quantizeScale = scaleQuantize<string>()
  .domain([0, 100])
  .range(['low', 'medium', 'high']);

quantizeScale(25);   // 'low'
quantizeScale(50);   // 'medium'
quantizeScale(75);   // 'high'

// Threshold - explicit breakpoints
const thresholdScale = scaleThreshold<number, string>()
  .domain([60, 80])
  .range(['fail', 'pass', 'excellent']);

thresholdScale(55);   // 'fail'
thresholdScale(70);   // 'pass'
thresholdScale(90);   // 'excellent'
```

---

# 4. D3 Axes

## 4.1 Creating Axes

```typescript
import { axisBottom, axisLeft, axisRight, axisTop } from 'd3-axis';
import { scaleLinear, scaleBand, scaleTime } from 'd3-scale';

// Axes are created from scales
const xScale = scaleLinear()
  .domain([0, 100])
  .range([0, 500]);

const xAxis = axisBottom(xScale);

// Render axis to SVG group
d3.select('svg')
  .append('g')
  .attr('transform', 'translate(50, 260)')  // Position it
  .call(xAxis);  // .call() invokes the axis generator

// Different positions
const topAxis = axisTop(scale);     // Ticks above
const bottomAxis = axisBottom(scale); // Ticks below
const leftAxis = axisLeft(scale);   // Ticks left
const rightAxis = axisRight(scale); // Ticks right
```

## 4.2 Customizing Axes

```typescript
const yScale = scaleLinear()
  .domain([0, 1000])
  .range([300, 0]);  // Inverted for Y axis!

const yAxis = axisLeft(yScale)
  .ticks(5)                    // Approximate number of ticks
  .tickValues([0, 250, 500, 750, 1000])  // Exact tick values
  .tickSize(10)                // Tick line length
  .tickSizeInner(6)            // Inner tick length
  .tickSizeOuter(0)            // Outer tick length (at ends)
  .tickPadding(5)              // Space between tick and label
  .tickFormat(d => `$${d}`);   // Format labels

// Complete axis setup in React/D3
function ChartWithAxes({ data, width, height, margin }) {
  const svgRef = useRef<SVGSVGElement>(null);
  
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;
  
  const xScale = scaleBand()
    .domain(data.map(d => d.category))
    .range([0, innerWidth])
    .padding(0.2);
  
  const yScale = scaleLinear()
    .domain([0, d3.max(data, d => d.value)!])
    .range([innerHeight, 0])
    .nice();
  
  useEffect(() => {
    const svg = d3.select(svgRef.current);
    
    // Clear previous axes
    svg.selectAll('.axis').remove();
    
    // X axis
    svg.append('g')
      .attr('class', 'axis x-axis')
      .attr('transform', `translate(${margin.left}, ${height - margin.bottom})`)
      .call(axisBottom(xScale))
      .selectAll('text')
      .attr('transform', 'rotate(-45)')
      .style('text-anchor', 'end');
    
    // Y axis
    svg.append('g')
      .attr('class', 'axis y-axis')
      .attr('transform', `translate(${margin.left}, ${margin.top})`)
      .call(axisLeft(yScale).ticks(5).tickFormat(d => `${d}%`));
    
    // Y axis label
    svg.append('text')
      .attr('class', 'axis-label')
      .attr('transform', 'rotate(-90)')
      .attr('x', -height / 2)
      .attr('y', 15)
      .style('text-anchor', 'middle')
      .text('Value (%)');
    
  }, [data, xScale, yScale]);
  
  return <svg ref={svgRef} width={width} height={height} />;
}
```

## 4.3 Grid Lines

```typescript
function createGridLines(svg, xScale, yScale, width, height, margin) {
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;
  
  // Horizontal grid lines (from Y axis)
  svg.append('g')
    .attr('class', 'grid horizontal-grid')
    .attr('transform', `translate(${margin.left}, ${margin.top})`)
    .call(
      axisLeft(yScale)
        .tickSize(-innerWidth)  // Negative = goes right
        .tickFormat(() => '')    // No labels
    )
    .selectAll('line')
    .attr('stroke', '#e0e0e0')
    .attr('stroke-dasharray', '2,2');
  
  // Vertical grid lines (from X axis)
  svg.append('g')
    .attr('class', 'grid vertical-grid')
    .attr('transform', `translate(${margin.left}, ${margin.top})`)
    .call(
      axisBottom(xScale)
        .tickSize(innerHeight)  // Positive = goes down
        .tickFormat(() => '')
    )
    .selectAll('line')
    .attr('stroke', '#e0e0e0');
  
  // Remove domain lines
  svg.selectAll('.grid .domain').remove();
}
```

---

# 5. D3 Shapes & Paths

## 5.1 Line Generator

```typescript
import { line, curveLinear, curveCardinal, curveBasis } from 'd3-shape';

interface DataPoint {
  date: Date;
  value: number;
}

const data: DataPoint[] = [
  { date: new Date('2024-01'), value: 10 },
  { date: new Date('2024-02'), value: 25 },
  { date: new Date('2024-03'), value: 15 },
  { date: new Date('2024-04'), value: 40 },
];

// Create line generator
const lineGenerator = line<DataPoint>()
  .x(d => xScale(d.date))
  .y(d => yScale(d.value))
  .curve(curveCardinal);  // Smooth curve

// Generate path string
const pathD = lineGenerator(data);
// Returns: "M0,100L100,50L200,75L300,0"

// In JSX
function LineChart({ data }) {
  const lineGen = line<DataPoint>()
    .x(d => xScale(d.date))
    .y(d => yScale(d.value));
  
  return (
    <svg>
      <path
        d={lineGen(data) || ''}
        fill="none"
        stroke="steelblue"
        strokeWidth={2}
      />
    </svg>
  );
}

// Curve types
const CURVE_TYPES = {
  curveLinear: 'Straight lines between points',
  curveCardinal: 'Smooth curve through points',
  curveCatmullRom: 'Another smooth curve option',
  curveBasis: 'B-spline (may not pass through points)',
  curveStep: 'Step function',
  curveStepBefore: 'Step before each point',
  curveStepAfter: 'Step after each point',
  curveMonotoneX: 'Monotonic in X (good for time series)',
};
```

## 5.2 Area Generator

```typescript
import { area, curveMonotoneX } from 'd3-shape';

const areaGenerator = area<DataPoint>()
  .x(d => xScale(d.date))
  .y0(innerHeight)          // Bottom of area (baseline)
  .y1(d => yScale(d.value)) // Top of area
  .curve(curveMonotoneX);

function AreaChart({ data }) {
  return (
    <svg>
      {/* Area fill */}
      <path
        d={areaGenerator(data) || ''}
        fill="steelblue"
        fillOpacity={0.3}
      />
      {/* Line on top */}
      <path
        d={lineGenerator(data) || ''}
        fill="none"
        stroke="steelblue"
        strokeWidth={2}
      />
    </svg>
  );
}

// Stacked area chart
import { stack, stackOrderNone, stackOffsetNone } from 'd3-shape';

const stackedData = [
  { date: '2024-01', a: 10, b: 20, c: 15 },
  { date: '2024-02', a: 15, b: 25, c: 20 },
  // ...
];

const stackGenerator = stack<any>()
  .keys(['a', 'b', 'c'])
  .order(stackOrderNone)
  .offset(stackOffsetNone);

const series = stackGenerator(stackedData);
// Returns array of series, each with [y0, y1] for each point

function StackedAreaChart({ data }) {
  const series = stackGenerator(data);
  
  return (
    <svg>
      {series.map((s, i) => (
        <path
          key={s.key}
          d={area()
            .x((d, j) => xScale(data[j].date))
            .y0(d => yScale(d[0]))
            .y1(d => yScale(d[1]))
            (s) || ''}
          fill={colorScale(s.key)}
        />
      ))}
    </svg>
  );
}
```

## 5.3 Arc Generator (Pie/Donut Charts)

```typescript
import { arc, pie } from 'd3-shape';

interface PieData {
  label: string;
  value: number;
}

const data: PieData[] = [
  { label: 'A', value: 30 },
  { label: 'B', value: 50 },
  { label: 'C', value: 20 },
];

// Pie generator - converts data to angles
const pieGenerator = pie<PieData>()
  .value(d => d.value)
  .sort(null);  // Don't sort (keep original order)

const arcs = pieGenerator(data);
// Returns array with startAngle, endAngle, data, etc.

// Arc generator - converts angles to path
const arcGenerator = arc()
  .innerRadius(0)      // 0 = pie, >0 = donut
  .outerRadius(100)
  .padAngle(0.02)      // Gap between slices
  .cornerRadius(4);    // Rounded corners

function PieChart({ data }) {
  const arcs = pieGenerator(data);
  
  return (
    <svg width={300} height={300}>
      <g transform="translate(150, 150)">
        {arcs.map((arc, i) => (
          <path
            key={arc.data.label}
            d={arcGenerator(arc as any) || ''}
            fill={colorScale(arc.data.label)}
          />
        ))}
      </g>
    </svg>
  );
}

// Donut chart with labels
function DonutChart({ data }) {
  const arcs = pieGenerator(data);
  
  const arcGen = arc()
    .innerRadius(60)
    .outerRadius(100);
  
  // Separate arc for labels (positioned in middle)
  const labelArc = arc()
    .innerRadius(80)
    .outerRadius(80);
  
  return (
    <svg width={300} height={300}>
      <g transform="translate(150, 150)">
        {arcs.map(a => (
          <g key={a.data.label}>
            <path
              d={arcGen(a as any) || ''}
              fill={colorScale(a.data.label)}
            />
            <text
              transform={`translate(${labelArc.centroid(a as any)})`}
              textAnchor="middle"
              fontSize={12}
            >
              {a.data.label}
            </text>
          </g>
        ))}
        {/* Center label */}
        <text textAnchor="middle" fontSize={24} fontWeight="bold">
          {d3.sum(data, d => d.value)}
        </text>
      </g>
    </svg>
  );
}
```

---

# 6. D3 Transitions & Animations

## 6.1 Basic Transitions

```typescript
import { transition } from 'd3-transition';

// Create a transition
d3.select('circle')
  .transition()
  .duration(1000)           // 1 second
  .delay(200)               // Wait 200ms before starting
  .ease(d3.easeElasticOut)  // Easing function
  .attr('r', 50)            // Animate to this value
  .attr('fill', 'red');

// Chained transitions
d3.select('rect')
  .transition()
  .duration(500)
  .attr('x', 100)
  .transition()              // Chain another
  .duration(500)
  .attr('y', 100);

// Callback when transition ends
d3.select('circle')
  .transition()
  .duration(1000)
  .attr('r', 0)
  .on('end', function() {
    d3.select(this).remove();
  });
```

## 6.2 Easing Functions

```typescript
import { 
  easeLinear, 
  easeQuadIn, easeQuadOut, easeQuadInOut,
  easeCubicIn, easeCubicOut, easeCubicInOut,
  easeElasticIn, easeElasticOut, easeElasticInOut,
  easeBounceIn, easeBounceOut, easeBounceInOut,
  easeBackIn, easeBackOut, easeBackInOut
} from 'd3-ease';

const EASING_FUNCTIONS = {
  // Linear - constant speed
  easeLinear: 'No acceleration',
  
  // Polynomial
  easeQuadIn: 'Slow start',
  easeQuadOut: 'Slow end',
  easeQuadInOut: 'Slow start and end',
  
  // Elastic - spring-like
  easeElasticOut: 'Overshoot and spring back',
  
  // Bounce - ball bouncing
  easeBounceOut: 'Bounces at end',
  
  // Back - overshoots target
  easeBackOut: 'Slight overshoot',
};

// Custom easing
d3.select('circle')
  .transition()
  .ease(d3.easeElasticOut.amplitude(1.5).period(0.4))
  .attr('r', 50);
```

## 6.3 Transitioning Data

```typescript
// Key function is crucial for proper animations!

interface DataPoint {
  id: string;
  value: number;
  category: string;
}

function updateChart(data: DataPoint[]) {
  const bars = d3.select('svg')
    .selectAll('rect')
    .data(data, d => d.id);  // Key function
  
  // Exit - fade out and remove
  bars.exit()
    .transition()
    .duration(500)
    .attr('opacity', 0)
    .attr('width', 0)
    .remove();
  
  // Enter - start invisible, then fade in
  const enter = bars.enter()
    .append('rect')
    .attr('opacity', 0)
    .attr('x', d => xScale(d.category)!)
    .attr('y', innerHeight)
    .attr('width', xScale.bandwidth())
    .attr('height', 0)
    .attr('fill', 'steelblue');
  
  // Merge enter and update, then transition
  enter.merge(bars as any)
    .transition()
    .duration(750)
    .delay((d, i) => i * 50)  // Stagger animation
    .attr('opacity', 1)
    .attr('x', d => xScale(d.category)!)
    .attr('y', d => yScale(d.value))
    .attr('height', d => innerHeight - yScale(d.value));
}
```

## 6.4 Interpolators

```typescript
import { 
  interpolate, 
  interpolateNumber,
  interpolateString,
  interpolateRgb,
  interpolateArray,
  interpolateObject
} from 'd3-interpolate';

// Number interpolation
const numInterp = interpolateNumber(0, 100);
numInterp(0);    // 0
numInterp(0.5);  // 50
numInterp(1);    // 100

// Color interpolation
const colorInterp = interpolateRgb('red', 'blue');
colorInterp(0);    // rgb(255, 0, 0)
colorInterp(0.5);  // rgb(128, 0, 128)
colorInterp(1);    // rgb(0, 0, 255)

// String with embedded numbers
const strInterp = interpolateString('20px', '100px');
strInterp(0.5);  // '60px'

// Custom tweening
d3.select('text')
  .transition()
  .duration(2000)
  .tween('text', function() {
    const i = interpolateNumber(0, 1000);
    return function(t) {
      this.textContent = Math.round(i(t)).toLocaleString();
    };
  });

// Path morphing
const path1 = 'M0,0 L100,0 L100,100 L0,100 Z';
const path2 = 'M50,0 L100,50 L50,100 L0,50 Z';
const pathInterp = interpolate(path1, path2);

d3.select('path')
  .transition()
  .duration(1000)
  .attrTween('d', function() {
    return pathInterp;
  });
```

---

# 7. D3 with React

## 7.1 Integration Approaches

```typescript
// There are three main approaches to using D3 with React:

const APPROACHES = {
  // 1. D3 for math, React for DOM (Recommended!)
  reactDom: {
    description: 'Use D3 scales/generators, React renders SVG',
    pros: ['React manages DOM', 'Full React lifecycle', 'Better performance'],
    cons: ['Some D3 features harder to use'],
    bestFor: 'Most charts and visualizations',
  },
  
  // 2. D3 controls DOM inside ref
  d3Dom: {
    description: 'D3 directly manipulates DOM via useRef',
    pros: ['Full D3 power', 'Easy to port D3 examples'],
    cons: ['Two systems managing DOM', 'Harder to debug'],
    bestFor: 'Complex animations, existing D3 code',
  },
  
  // 3. Hybrid
  hybrid: {
    description: 'React for structure, D3 for specific parts',
    pros: ['Flexible', 'Use each where it excels'],
    cons: ['More complex'],
    bestFor: 'Large applications',
  },
};
```

## 7.2 React-First Approach (Recommended)

```typescript
// D3 for calculations, React for rendering

import { scaleLinear, scaleBand, max } from 'd3';

interface DataPoint {
  category: string;
  value: number;
}

interface BarChartProps {
  data: DataPoint[];
  width: number;
  height: number;
}

function BarChart({ data, width, height }: BarChartProps) {
  const margin = { top: 20, right: 20, bottom: 30, left: 40 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;
  
  // D3 for scales (pure functions)
  const xScale = scaleBand<string>()
    .domain(data.map(d => d.category))
    .range([0, innerWidth])
    .padding(0.2);
  
  const yScale = scaleLinear()
    .domain([0, max(data, d => d.value) || 0])
    .range([innerHeight, 0])
    .nice();
  
  // React renders SVG
  return (
    <svg width={width} height={height}>
      <g transform={`translate(${margin.left}, ${margin.top})`}>
        {/* Bars */}
        {data.map(d => (
          <rect
            key={d.category}
            x={xScale(d.category)}
            y={yScale(d.value)}
            width={xScale.bandwidth()}
            height={innerHeight - yScale(d.value)}
            fill="steelblue"
          />
        ))}
        
        {/* X Axis */}
        <g transform={`translate(0, ${innerHeight})`}>
          {data.map(d => (
            <g key={d.category} transform={`translate(${xScale(d.category)! + xScale.bandwidth() / 2}, 0)`}>
              <line y2={6} stroke="black" />
              <text y={9} dy="0.71em" textAnchor="middle" fontSize={12}>
                {d.category}
              </text>
            </g>
          ))}
        </g>
        
        {/* Y Axis */}
        <g>
          {yScale.ticks(5).map(tick => (
            <g key={tick} transform={`translate(0, ${yScale(tick)})`}>
              <line x2={-6} stroke="black" />
              <text x={-9} dy="0.32em" textAnchor="end" fontSize={12}>
                {tick}
              </text>
            </g>
          ))}
        </g>
      </g>
    </svg>
  );
}
```

## 7.3 D3-Controlled DOM Approach

```typescript
// D3 manipulates DOM via useRef

import { useRef, useEffect } from 'react';
import * as d3 from 'd3';

interface LineChartProps {
  data: { date: Date; value: number }[];
  width: number;
  height: number;
}

function LineChart({ data, width, height }: LineChartProps) {
  const svgRef = useRef<SVGSVGElement>(null);
  const margin = { top: 20, right: 20, bottom: 30, left: 40 };
  
  useEffect(() => {
    if (!svgRef.current || !data.length) return;
    
    const svg = d3.select(svgRef.current);
    const innerWidth = width - margin.left - margin.right;
    const innerHeight = height - margin.top - margin.bottom;
    
    // Scales
    const xScale = d3.scaleTime()
      .domain(d3.extent(data, d => d.date) as [Date, Date])
      .range([0, innerWidth]);
    
    const yScale = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.value)!])
      .range([innerHeight, 0])
      .nice();
    
    // Line generator
    const lineGenerator = d3.line<{ date: Date; value: number }>()
      .x(d => xScale(d.date))
      .y(d => yScale(d.value))
      .curve(d3.curveMonotoneX);
    
    // Clear previous content
    svg.selectAll('*').remove();
    
    // Create chart group
    const chart = svg.append('g')
      .attr('transform', `translate(${margin.left}, ${margin.top})`);
    
    // Add axes
    chart.append('g')
      .attr('transform', `translate(0, ${innerHeight})`)
      .call(d3.axisBottom(xScale));
    
    chart.append('g')
      .call(d3.axisLeft(yScale));
    
    // Add line with animation
    const path = chart.append('path')
      .datum(data)
      .attr('fill', 'none')
      .attr('stroke', 'steelblue')
      .attr('stroke-width', 2)
      .attr('d', lineGenerator);
    
    // Animate line drawing
    const pathLength = path.node()!.getTotalLength();
    
    path
      .attr('stroke-dasharray', `${pathLength} ${pathLength}`)
      .attr('stroke-dashoffset', pathLength)
      .transition()
      .duration(2000)
      .ease(d3.easeLinear)
      .attr('stroke-dashoffset', 0);
    
  }, [data, width, height, margin]);
  
  return <svg ref={svgRef} width={width} height={height} />;
}
```

## 7.4 Custom Hooks for D3

```typescript
// Reusable hooks for common D3 patterns

import { useMemo, useRef, useEffect, useState } from 'react';

// Hook for scales
function useScale<T>(
  type: 'linear' | 'band' | 'time',
  domain: any[],
  range: [number, number],
  options?: { padding?: number; nice?: boolean }
) {
  return useMemo(() => {
    let scale: any;
    
    switch (type) {
      case 'linear':
        scale = d3.scaleLinear().domain(domain).range(range);
        if (options?.nice) scale.nice();
        break;
      case 'band':
        scale = d3.scaleBand().domain(domain).range(range);
        if (options?.padding) scale.padding(options.padding);
        break;
      case 'time':
        scale = d3.scaleTime().domain(domain).range(range);
        break;
    }
    
    return scale;
  }, [type, JSON.stringify(domain), JSON.stringify(range), JSON.stringify(options)]);
}

// Hook for dimensions with resize
function useDimensions(ref: React.RefObject<HTMLElement>) {
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    const element = ref.current;
    if (!element) return;
    
    const observer = new ResizeObserver(entries => {
      const { width, height } = entries[0].contentRect;
      setDimensions({ width, height });
    });
    
    observer.observe(element);
    return () => observer.disconnect();
  }, [ref]);
  
  return dimensions;
}

// Hook for animated number
function useAnimatedNumber(value: number, duration = 500) {
  const [animatedValue, setAnimatedValue] = useState(value);
  const previousValue = useRef(value);
  
  useEffect(() => {
    const startValue = previousValue.current;
    const startTime = Date.now();
    
    const animate = () => {
      const elapsed = Date.now() - startTime;
      const progress = Math.min(elapsed / duration, 1);
      
      // Ease out quad
      const eased = 1 - (1 - progress) * (1 - progress);
      const current = startValue + (value - startValue) * eased;
      
      setAnimatedValue(current);
      
      if (progress < 1) {
        requestAnimationFrame(animate);
      }
    };
    
    animate();
    previousValue.current = value;
  }, [value, duration]);
  
  return animatedValue;
}

// Usage
function AnimatedStat({ value, label }: { value: number; label: string }) {
  const animatedValue = useAnimatedNumber(value);
  
  return (
    <div className="stat">
      <span className="value">{Math.round(animatedValue).toLocaleString()}</span>
      <span className="label">{label}</span>
    </div>
  );
}
```

---

# 8. Recharts Complete Guide

## 8.1 What is Recharts?

```typescript
// Recharts = React + D3 charting library
// Components-based API for building charts declaratively

// Installation
// npm install recharts

// Key features:
const RECHARTS_FEATURES = {
  declarative: 'Build charts with React components',
  responsive: 'Built-in responsive container',
  customizable: 'Style with props or custom components',
  animations: 'Built-in animations',
  tooltips: 'Easy-to-use tooltip system',
  legends: 'Built-in legend components',
};

import {
  LineChart,
  Line,
  BarChart,
  Bar,
  PieChart,
  Pie,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Legend,
  ResponsiveContainer,
} from 'recharts';
```

## 8.2 Line Chart

```tsx
import { 
  LineChart, Line, XAxis, YAxis, 
  CartesianGrid, Tooltip, Legend, ResponsiveContainer 
} from 'recharts';

const data = [
  { month: 'Jan', sales: 4000, profit: 2400 },
  { month: 'Feb', sales: 3000, profit: 1398 },
  { month: 'Mar', sales: 5000, profit: 3200 },
  { month: 'Apr', sales: 4780, profit: 2908 },
  { month: 'May', sales: 5890, profit: 3800 },
  { month: 'Jun', sales: 6390, profit: 4300 },
];

function SalesLineChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <LineChart data={data} margin={{ top: 20, right: 30, left: 20, bottom: 5 }}>
        <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
        
        <XAxis 
          dataKey="month" 
          tick={{ fill: '#666' }}
          axisLine={{ stroke: '#ccc' }}
        />
        
        <YAxis 
          tick={{ fill: '#666' }}
          axisLine={{ stroke: '#ccc' }}
          tickFormatter={(value) => `$${value}`}
        />
        
        <Tooltip 
          contentStyle={{ backgroundColor: '#fff', border: '1px solid #ccc' }}
          formatter={(value: number) => [`$${value.toLocaleString()}`, '']}
        />
        
        <Legend />
        
        <Line 
          type="monotone" 
          dataKey="sales" 
          stroke="#8884d8" 
          strokeWidth={2}
          dot={{ fill: '#8884d8', r: 4 }}
          activeDot={{ r: 6 }}
        />
        
        <Line 
          type="monotone" 
          dataKey="profit" 
          stroke="#82ca9d" 
          strokeWidth={2}
          dot={{ fill: '#82ca9d', r: 4 }}
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

## 8.3 Bar Chart

```tsx
import { 
  BarChart, Bar, XAxis, YAxis, 
  CartesianGrid, Tooltip, Legend, ResponsiveContainer,
  Cell
} from 'recharts';

const data = [
  { name: 'Product A', value: 4000, fill: '#8884d8' },
  { name: 'Product B', value: 3000, fill: '#83a6ed' },
  { name: 'Product C', value: 5000, fill: '#8dd1e1' },
  { name: 'Product D', value: 2780, fill: '#82ca9d' },
  { name: 'Product E', value: 1890, fill: '#a4de6c' },
];

// Basic bar chart
function BasicBarChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <BarChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="name" />
        <YAxis />
        <Tooltip />
        <Bar dataKey="value" fill="#8884d8" radius={[4, 4, 0, 0]} />
      </BarChart>
    </ResponsiveContainer>
  );
}

// With colored bars
function ColoredBarChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <BarChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="name" />
        <YAxis />
        <Tooltip />
        <Bar dataKey="value" radius={[4, 4, 0, 0]}>
          {data.map((entry, index) => (
            <Cell key={index} fill={entry.fill} />
          ))}
        </Bar>
      </BarChart>
    </ResponsiveContainer>
  );
}

// Stacked bar chart
const stackedData = [
  { month: 'Jan', desktop: 4000, mobile: 2400, tablet: 1000 },
  { month: 'Feb', desktop: 3000, mobile: 1398, tablet: 800 },
  { month: 'Mar', desktop: 5000, mobile: 3200, tablet: 1200 },
];

function StackedBarChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <BarChart data={stackedData}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="month" />
        <YAxis />
        <Tooltip />
        <Legend />
        <Bar dataKey="desktop" stackId="a" fill="#8884d8" />
        <Bar dataKey="mobile" stackId="a" fill="#82ca9d" />
        <Bar dataKey="tablet" stackId="a" fill="#ffc658" />
      </BarChart>
    </ResponsiveContainer>
  );
}

// Horizontal bar chart
function HorizontalBarChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <BarChart data={data} layout="vertical">
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis type="number" />
        <YAxis dataKey="name" type="category" width={80} />
        <Tooltip />
        <Bar dataKey="value" fill="#8884d8" />
      </BarChart>
    </ResponsiveContainer>
  );
}
```

## 8.4 Pie and Donut Charts

```tsx
import { PieChart, Pie, Cell, Tooltip, Legend, ResponsiveContainer } from 'recharts';

const data = [
  { name: 'Chrome', value: 400, color: '#0088FE' },
  { name: 'Firefox', value: 300, color: '#00C49F' },
  { name: 'Safari', value: 200, color: '#FFBB28' },
  { name: 'Edge', value: 100, color: '#FF8042' },
];

// Basic pie chart
function BasicPieChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <PieChart>
        <Pie
          data={data}
          dataKey="value"
          nameKey="name"
          cx="50%"
          cy="50%"
          outerRadius={150}
          label
        >
          {data.map((entry, index) => (
            <Cell key={index} fill={entry.color} />
          ))}
        </Pie>
        <Tooltip />
        <Legend />
      </PieChart>
    </ResponsiveContainer>
  );
}

// Donut chart
function DonutChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <PieChart>
        <Pie
          data={data}
          dataKey="value"
          nameKey="name"
          cx="50%"
          cy="50%"
          innerRadius={80}
          outerRadius={150}
          paddingAngle={2}
          label={({ name, percent }) => `${name}: ${(percent * 100).toFixed(0)}%`}
        >
          {data.map((entry, index) => (
            <Cell key={index} fill={entry.color} />
          ))}
        </Pie>
        <Tooltip />
      </PieChart>
    </ResponsiveContainer>
  );
}

// Donut with center label
function DonutWithLabel() {
  const total = data.reduce((sum, entry) => sum + entry.value, 0);
  
  return (
    <ResponsiveContainer width="100%" height={400}>
      <PieChart>
        <Pie
          data={data}
          dataKey="value"
          cx="50%"
          cy="50%"
          innerRadius={80}
          outerRadius={120}
        >
          {data.map((entry, index) => (
            <Cell key={index} fill={entry.color} />
          ))}
        </Pie>
        {/* Center text */}
        <text x="50%" y="50%" textAnchor="middle" dominantBaseline="middle">
          <tspan x="50%" dy="-0.5em" fontSize={24} fontWeight="bold">
            {total}
          </tspan>
          <tspan x="50%" dy="1.5em" fontSize={14} fill="#666">
            Total Users
          </tspan>
        </text>
        <Tooltip />
      </PieChart>
    </ResponsiveContainer>
  );
}
```

## 8.5 Area Charts

```tsx
import { 
  AreaChart, Area, XAxis, YAxis, 
  CartesianGrid, Tooltip, ResponsiveContainer 
} from 'recharts';

const data = [
  { date: '2024-01', value: 4000 },
  { date: '2024-02', value: 3000 },
  { date: '2024-03', value: 5000 },
  { date: '2024-04', value: 4780 },
  { date: '2024-05', value: 5890 },
  { date: '2024-06', value: 6390 },
];

function GradientAreaChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <AreaChart data={data}>
        <defs>
          <linearGradient id="colorValue" x1="0" y1="0" x2="0" y2="1">
            <stop offset="5%" stopColor="#8884d8" stopOpacity={0.8} />
            <stop offset="95%" stopColor="#8884d8" stopOpacity={0} />
          </linearGradient>
        </defs>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" />
        <YAxis />
        <Tooltip />
        <Area 
          type="monotone" 
          dataKey="value" 
          stroke="#8884d8" 
          fill="url(#colorValue)" 
        />
      </AreaChart>
    </ResponsiveContainer>
  );
}

// Stacked area
const stackedAreaData = [
  { date: '2024-01', organic: 4000, paid: 2400, direct: 1000 },
  { date: '2024-02', organic: 3000, paid: 1398, direct: 800 },
  { date: '2024-03', organic: 5000, paid: 3200, direct: 1200 },
];

function StackedAreaChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <AreaChart data={stackedAreaData}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" />
        <YAxis />
        <Tooltip />
        <Legend />
        <Area 
          type="monotone" 
          dataKey="organic" 
          stackId="1" 
          fill="#8884d8" 
          stroke="#8884d8" 
        />
        <Area 
          type="monotone" 
          dataKey="paid" 
          stackId="1" 
          fill="#82ca9d" 
          stroke="#82ca9d" 
        />
        <Area 
          type="monotone" 
          dataKey="direct" 
          stackId="1" 
          fill="#ffc658" 
          stroke="#ffc658" 
        />
      </AreaChart>
    </ResponsiveContainer>
  );
}
```

## 8.6 Custom Tooltips

```tsx
import { TooltipProps } from 'recharts';

// Custom tooltip component
function CustomTooltip({ 
  active, 
  payload, 
  label 
}: TooltipProps<number, string>) {
  if (!active || !payload || !payload.length) return null;
  
  return (
    <div className="custom-tooltip" style={{
      background: 'white',
      border: '1px solid #ccc',
      borderRadius: 8,
      padding: 12,
      boxShadow: '0 2px 8px rgba(0,0,0,0.15)',
    }}>
      <p style={{ fontWeight: 'bold', marginBottom: 8 }}>{label}</p>
      {payload.map((entry, index) => (
        <p key={index} style={{ color: entry.color, margin: '4px 0' }}>
          {entry.name}: ${entry.value?.toLocaleString()}
        </p>
      ))}
    </div>
  );
}

// Usage
function ChartWithCustomTooltip() {
  return (
    <LineChart data={data}>
      {/* ... other components */}
      <Tooltip content={<CustomTooltip />} />
    </LineChart>
  );
}

// Active shape for pie charts
function renderActiveShape(props: any) {
  const { 
    cx, cy, innerRadius, outerRadius, startAngle, endAngle,
    fill, payload, percent, value 
  } = props;
  
  return (
    <g>
      <text x={cx} y={cy} dy={-20} textAnchor="middle" fill={fill}>
        {payload.name}
      </text>
      <text x={cx} y={cy} dy={8} textAnchor="middle" fill="#333" fontSize={20}>
        {value.toLocaleString()}
      </text>
      <text x={cx} y={cy} dy={30} textAnchor="middle" fill="#999">
        {`${(percent * 100).toFixed(1)}%`}
      </text>
      <Sector
        cx={cx}
        cy={cy}
        innerRadius={innerRadius}
        outerRadius={outerRadius + 10}
        startAngle={startAngle}
        endAngle={endAngle}
        fill={fill}
      />
    </g>
  );
}

function InteractivePieChart() {
  const [activeIndex, setActiveIndex] = useState(0);
  
  return (
    <PieChart>
      <Pie
        data={data}
        activeIndex={activeIndex}
        activeShape={renderActiveShape}
        onMouseEnter={(_, index) => setActiveIndex(index)}
        dataKey="value"
        innerRadius={60}
        outerRadius={100}
      >
        {data.map((entry, index) => (
          <Cell key={index} fill={entry.color} />
        ))}
      </Pie>
    </PieChart>
  );
}
```

## 8.7 Responsive Charts

```tsx
import { ResponsiveContainer } from 'recharts';

// Always wrap charts in ResponsiveContainer
function ResponsiveChart() {
  return (
    <div style={{ width: '100%', height: 400 }}>
      <ResponsiveContainer width="100%" height="100%">
        <LineChart data={data}>
          {/* Chart components */}
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
}

// With aspect ratio
function AspectRatioChart() {
  return (
    <ResponsiveContainer width="100%" aspect={16 / 9}>
      <BarChart data={data}>
        {/* Chart components */}
      </BarChart>
    </ResponsiveContainer>
  );
}

// Responsive axis labels
function ResponsiveAxisChart() {
  const [width, setWidth] = useState(0);
  
  return (
    <ResponsiveContainer 
      width="100%" 
      height={400}
      onResize={(w) => setWidth(w)}
    >
      <BarChart data={data}>
        <XAxis 
          dataKey="name"
          interval={width < 500 ? 1 : 0}  // Skip labels on small screens
          tick={{ fontSize: width < 500 ? 10 : 12 }}
        />
        {/* ... */}
      </BarChart>
    </ResponsiveContainer>
  );
}
```

---

# 9. More Chart Types

## 9.1 Scatter Plot

```tsx
import { 
  ScatterChart, Scatter, XAxis, YAxis, 
  CartesianGrid, Tooltip, ResponsiveContainer,
  ZAxis
} from 'recharts';

const data = [
  { x: 100, y: 200, z: 200 },
  { x: 120, y: 100, z: 260 },
  { x: 170, y: 300, z: 400 },
  { x: 140, y: 250, z: 280 },
  { x: 150, y: 400, z: 500 },
  { x: 110, y: 280, z: 200 },
];

function BasicScatterChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <ScatterChart margin={{ top: 20, right: 20, bottom: 20, left: 20 }}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis type="number" dataKey="x" name="Price" unit="$" />
        <YAxis type="number" dataKey="y" name="Revenue" unit="k" />
        <ZAxis type="number" dataKey="z" range={[60, 400]} />
        <Tooltip cursor={{ strokeDasharray: '3 3' }} />
        <Scatter name="Products" data={data} fill="#8884d8" />
      </ScatterChart>
    </ResponsiveContainer>
  );
}

// Multiple data series
const series1 = [
  { x: 100, y: 200 },
  { x: 120, y: 100 },
  { x: 170, y: 300 },
];

const series2 = [
  { x: 200, y: 260 },
  { x: 240, y: 290 },
  { x: 190, y: 340 },
];

function MultiSeriesScatter() {
  return (
    <ScatterChart>
      <CartesianGrid />
      <XAxis type="number" dataKey="x" />
      <YAxis type="number" dataKey="y" />
      <Tooltip />
      <Scatter name="Series A" data={series1} fill="#8884d8" />
      <Scatter name="Series B" data={series2} fill="#82ca9d" />
    </ScatterChart>
  );
}
```

## 9.2 Radar Chart

```tsx
import { 
  Radar, RadarChart, PolarGrid, 
  PolarAngleAxis, PolarRadiusAxis, ResponsiveContainer 
} from 'recharts';

const data = [
  { subject: 'Math', A: 120, B: 110, fullMark: 150 },
  { subject: 'Chinese', A: 98, B: 130, fullMark: 150 },
  { subject: 'English', A: 86, B: 130, fullMark: 150 },
  { subject: 'Geography', A: 99, B: 100, fullMark: 150 },
  { subject: 'Physics', A: 85, B: 90, fullMark: 150 },
  { subject: 'History', A: 65, B: 85, fullMark: 150 },
];

function BasicRadarChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <RadarChart data={data}>
        <PolarGrid />
        <PolarAngleAxis dataKey="subject" />
        <PolarRadiusAxis angle={30} domain={[0, 150]} />
        <Radar 
          name="Student A" 
          dataKey="A" 
          stroke="#8884d8" 
          fill="#8884d8" 
          fillOpacity={0.6} 
        />
        <Radar 
          name="Student B" 
          dataKey="B" 
          stroke="#82ca9d" 
          fill="#82ca9d" 
          fillOpacity={0.6} 
        />
      </RadarChart>
    </ResponsiveContainer>
  );
}
```

## 9.3 Radial Bar Chart

```tsx
import { RadialBarChart, RadialBar, Legend, ResponsiveContainer } from 'recharts';

const data = [
  { name: '18-24', uv: 31.47, fill: '#8884d8' },
  { name: '25-29', uv: 26.69, fill: '#83a6ed' },
  { name: '30-34', uv: 15.69, fill: '#8dd1e1' },
  { name: '35-39', uv: 8.22, fill: '#82ca9d' },
  { name: '40-49', uv: 8.63, fill: '#a4de6c' },
  { name: '50+', uv: 2.63, fill: '#d0ed57' },
];

function RadialChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <RadialBarChart 
        innerRadius="10%" 
        outerRadius="80%" 
        data={data} 
        startAngle={180} 
        endAngle={0}
      >
        <RadialBar
          minAngle={15}
          label={{ position: 'insideStart', fill: '#fff' }}
          background
          clockWise
          dataKey="uv"
        />
        <Legend iconSize={10} layout="vertical" verticalAlign="middle" />
      </RadialBarChart>
    </ResponsiveContainer>
  );
}
```

## 9.4 Treemap

```tsx
import { Treemap, ResponsiveContainer, Tooltip } from 'recharts';

const data = [
  {
    name: 'Technology',
    children: [
      { name: 'Apple', size: 2800 },
      { name: 'Microsoft', size: 2500 },
      { name: 'Google', size: 2200 },
      { name: 'Amazon', size: 1800 },
    ],
  },
  {
    name: 'Healthcare',
    children: [
      { name: 'Johnson & Johnson', size: 1200 },
      { name: 'Pfizer', size: 1000 },
      { name: 'UnitedHealth', size: 900 },
    ],
  },
  {
    name: 'Finance',
    children: [
      { name: 'JPMorgan', size: 1500 },
      { name: 'Bank of America', size: 1100 },
      { name: 'Wells Fargo', size: 800 },
    ],
  },
];

const COLORS = ['#8889DD', '#9597E4', '#8DC77B', '#A5D297', '#E2CF45', '#F8C12D'];

function TreemapChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <Treemap
        data={data}
        dataKey="size"
        aspectRatio={4 / 3}
        stroke="#fff"
        fill="#8884d8"
        content={({ x, y, width, height, name, value }) => (
          <g>
            <rect
              x={x}
              y={y}
              width={width}
              height={height}
              fill={COLORS[Math.floor(Math.random() * COLORS.length)]}
              stroke="#fff"
            />
            {width > 50 && height > 30 && (
              <text
                x={x + width / 2}
                y={y + height / 2}
                textAnchor="middle"
                fill="#fff"
                fontSize={12}
              >
                {name}
              </text>
            )}
          </g>
        )}
      >
        <Tooltip />
      </Treemap>
    </ResponsiveContainer>
  );
}
```

## 9.5 Funnel Chart

```tsx
import { FunnelChart, Funnel, LabelList, Tooltip, ResponsiveContainer } from 'recharts';

const data = [
  { name: 'Visited', value: 5000, fill: '#8884d8' },
  { name: 'Cart', value: 2500, fill: '#83a6ed' },
  { name: 'Checkout', value: 1200, fill: '#8dd1e1' },
  { name: 'Paid', value: 800, fill: '#82ca9d' },
];

function FunnelChartExample() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <FunnelChart>
        <Tooltip />
        <Funnel
          dataKey="value"
          data={data}
          isAnimationActive
        >
          <LabelList position="right" fill="#000" stroke="none" dataKey="name" />
        </Funnel>
      </FunnelChart>
    </ResponsiveContainer>
  );
}
```

## 9.6 Composed Chart (Mixed Types)

```tsx
import {
  ComposedChart, Line, Bar, Area, XAxis, YAxis,
  CartesianGrid, Tooltip, Legend, ResponsiveContainer
} from 'recharts';

const data = [
  { name: 'Jan', revenue: 4000, profit: 2400, target: 2400 },
  { name: 'Feb', revenue: 3000, profit: 1398, target: 2210 },
  { name: 'Mar', revenue: 5000, profit: 3200, target: 2290 },
  { name: 'Apr', revenue: 4780, profit: 2908, target: 2000 },
  { name: 'May', revenue: 5890, profit: 3800, target: 2181 },
  { name: 'Jun', revenue: 6390, profit: 4300, target: 2500 },
];

function ComposedChartExample() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <ComposedChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="name" />
        <YAxis />
        <Tooltip />
        <Legend />
        <Area 
          type="monotone" 
          dataKey="target" 
          fill="#8884d8" 
          stroke="#8884d8" 
          fillOpacity={0.2}
        />
        <Bar dataKey="revenue" fill="#82ca9d" />
        <Line type="monotone" dataKey="profit" stroke="#ff7300" strokeWidth={2} />
      </ComposedChart>
    </ResponsiveContainer>
  );
}
```

---

# 10. Advanced Visualization Concepts

## 10.1 Brush for Zooming

```tsx
import {
  LineChart, Line, XAxis, YAxis, CartesianGrid,
  Tooltip, Brush, ResponsiveContainer
} from 'recharts';

// Large dataset
const data = Array.from({ length: 100 }, (_, i) => ({
  date: `2024-${String(i + 1).padStart(3, '0')}`,
  value: Math.floor(Math.random() * 1000) + 500,
}));

function ZoomableChart() {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <LineChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" />
        <YAxis />
        <Tooltip />
        <Line type="monotone" dataKey="value" stroke="#8884d8" />
        <Brush 
          dataKey="date" 
          height={30} 
          stroke="#8884d8"
          startIndex={0}
          endIndex={19}
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

## 10.2 Reference Lines and Areas

```tsx
import {
  LineChart, Line, XAxis, YAxis, CartesianGrid,
  Tooltip, ReferenceLine, ReferenceArea, ResponsiveContainer
} from 'recharts';

function ChartWithReferences() {
  return (
    <LineChart data={data}>
      <CartesianGrid strokeDasharray="3 3" />
      <XAxis dataKey="name" />
      <YAxis />
      <Tooltip />
      
      {/* Reference line */}
      <ReferenceLine 
        y={3000} 
        stroke="red" 
        strokeDasharray="3 3"
        label={{ value: 'Target', fill: 'red' }}
      />
      
      {/* Reference area */}
      <ReferenceArea 
        x1="Mar" 
        x2="May" 
        y1={0} 
        y2={5000}
        fill="green" 
        fillOpacity={0.1}
        label="Best Period"
      />
      
      <Line type="monotone" dataKey="value" stroke="#8884d8" />
    </LineChart>
  );
}
```

## 10.3 Real-Time Updates

```tsx
import { useState, useEffect, useCallback } from 'react';

function RealTimeChart() {
  const [data, setData] = useState<{ time: string; value: number }[]>([]);
  
  const addDataPoint = useCallback(() => {
    setData(prev => {
      const newData = [...prev, {
        time: new Date().toLocaleTimeString(),
        value: Math.floor(Math.random() * 100),
      }];
      
      // Keep only last 20 points
      return newData.slice(-20);
    });
  }, []);
  
  useEffect(() => {
    const interval = setInterval(addDataPoint, 1000);
    return () => clearInterval(interval);
  }, [addDataPoint]);
  
  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={data}>
        <XAxis dataKey="time" />
        <YAxis domain={[0, 100]} />
        <Line 
          type="monotone" 
          dataKey="value" 
          stroke="#8884d8"
          isAnimationActive={false}  // Disable animation for real-time
        />
      </LineChart>
    </ResponsiveContainer>
  );
}

// With D3 for smoother updates
function RealTimeD3Chart() {
  const svgRef = useRef<SVGSVGElement>(null);
  const dataRef = useRef<number[]>([]);
  
  useEffect(() => {
    const svg = d3.select(svgRef.current);
    const width = 600;
    const height = 300;
    
    const xScale = d3.scaleLinear().domain([0, 100]).range([0, width]);
    const yScale = d3.scaleLinear().domain([0, 100]).range([height, 0]);
    
    const line = d3.line<number>()
      .x((d, i) => xScale(i))
      .y(d => yScale(d))
      .curve(d3.curveMonotoneX);
    
    const path = svg.append('path')
      .attr('fill', 'none')
      .attr('stroke', 'steelblue')
      .attr('stroke-width', 2);
    
    const interval = setInterval(() => {
      dataRef.current.push(Math.random() * 100);
      if (dataRef.current.length > 100) {
        dataRef.current.shift();
      }
      
      path.attr('d', line(dataRef.current));
    }, 100);
    
    return () => clearInterval(interval);
  }, []);
  
  return <svg ref={svgRef} width={600} height={300} />;
}
```

---

# 11. Canvas vs SVG

## 11.1 Comparison

```typescript
const CANVAS_VS_SVG = {
  svg: {
    type: 'Vector graphics',
    rendering: 'Retained mode (DOM)',
    scaling: 'Perfect at any size',
    performance: 'Worse with many elements',
    interactivity: 'Built-in (DOM events)',
    accessibility: 'Text searchable, ARIA support',
    animation: 'CSS and SMIL',
    debugging: 'Easy (inspect DOM)',
    
    bestFor: [
      'Charts with < 1000 elements',
      'Interactive visualizations',
      'Accessible graphics',
      'Print/export quality',
    ],
  },
  
  canvas: {
    type: 'Raster graphics (pixels)',
    rendering: 'Immediate mode (draw then forget)',
    scaling: 'Must redraw at new size',
    performance: 'Better with many elements',
    interactivity: 'Manual hit detection',
    accessibility: 'Limited (fallback content only)',
    animation: 'requestAnimationFrame',
    debugging: 'Harder (just pixels)',
    
    bestFor: [
      'Large datasets (10k+ points)',
      'Gaming/simulations',
      'Pixel manipulation',
      'Real-time graphics',
    ],
  },
};

// When to use which:
const DECISION_GUIDE = {
  useSVG: [
    'Under 1000 data points',
    'Need tooltips on elements',
    'Need accessibility',
    'Need CSS styling',
    'Need to export/print',
  ],
  
  useCanvas: [
    'Over 10,000 data points',
    'Real-time updates (60fps)',
    'Particle systems',
    'Custom pixel effects',
    'Memory-sensitive environment',
  ],
  
  useWebGL: [
    'Over 100,000 data points',
    '3D visualizations',
    'GPU-accelerated effects',
    'Complex shaders',
  ],
};
```

## 11.2 Canvas Basics for Charts

```typescript
function CanvasBarChart({ data, width, height }: CanvasChartProps) {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  
  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    
    const ctx = canvas.getContext('2d')!;
    
    // Clear
    ctx.clearRect(0, 0, width, height);
    
    // Setup
    const padding = 40;
    const chartWidth = width - padding * 2;
    const chartHeight = height - padding * 2;
    const barWidth = chartWidth / data.length * 0.8;
    const maxValue = Math.max(...data.map(d => d.value));
    
    // Draw bars
    data.forEach((d, i) => {
      const x = padding + i * (chartWidth / data.length) + barWidth * 0.1;
      const barHeight = (d.value / maxValue) * chartHeight;
      const y = height - padding - barHeight;
      
      ctx.fillStyle = '#8884d8';
      ctx.fillRect(x, y, barWidth, barHeight);
      
      // Label
      ctx.fillStyle = '#333';
      ctx.textAlign = 'center';
      ctx.fillText(d.label, x + barWidth / 2, height - padding + 15);
    });
    
    // Draw axes
    ctx.strokeStyle = '#333';
    ctx.beginPath();
    ctx.moveTo(padding, padding);
    ctx.lineTo(padding, height - padding);
    ctx.lineTo(width - padding, height - padding);
    ctx.stroke();
    
  }, [data, width, height]);
  
  return <canvas ref={canvasRef} width={width} height={height} />;
}

// Canvas with interactivity (hit detection)
function InteractiveCanvasChart({ data, width, height }) {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [hoveredIndex, setHoveredIndex] = useState<number | null>(null);
  
  // Store bar positions for hit detection
  const barsRef = useRef<{ x: number; y: number; width: number; height: number; data: any }[]>([]);
  
  const handleMouseMove = (e: React.MouseEvent) => {
    const rect = canvasRef.current!.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    
    // Find hit bar
    const hitIndex = barsRef.current.findIndex(bar => 
      x >= bar.x && x <= bar.x + bar.width &&
      y >= bar.y && y <= bar.y + bar.height
    );
    
    setHoveredIndex(hitIndex >= 0 ? hitIndex : null);
  };
  
  useEffect(() => {
    // Draw and store bar positions
    const bars: typeof barsRef.current = [];
    
    // ... drawing code that also populates bars array
    
    barsRef.current = bars;
  }, [data, width, height, hoveredIndex]);
  
  return (
    <canvas 
      ref={canvasRef} 
      width={width} 
      height={height}
      onMouseMove={handleMouseMove}
      style={{ cursor: hoveredIndex !== null ? 'pointer' : 'default' }}
    />
  );
}
```

## 11.3 Hybrid Approach

```tsx
// Use SVG for axes and labels, Canvas for data points
function HybridChart({ data, width, height }) {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const margin = { top: 20, right: 20, bottom: 30, left: 40 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;
  
  const xScale = useMemo(() => 
    d3.scaleLinear()
      .domain([0, data.length - 1])
      .range([0, innerWidth]),
    [data, innerWidth]
  );
  
  const yScale = useMemo(() =>
    d3.scaleLinear()
      .domain([0, d3.max(data, d => d.value)!])
      .range([innerHeight, 0])
      .nice(),
    [data, innerHeight]
  );
  
  // Draw data on canvas (performant for many points)
  useEffect(() => {
    const ctx = canvasRef.current?.getContext('2d');
    if (!ctx) return;
    
    ctx.clearRect(0, 0, innerWidth, innerHeight);
    
    // Draw points
    ctx.fillStyle = '#8884d8';
    data.forEach((d, i) => {
      const x = xScale(i);
      const y = yScale(d.value);
      
      ctx.beginPath();
      ctx.arc(x, y, 3, 0, Math.PI * 2);
      ctx.fill();
    });
  }, [data, xScale, yScale, innerWidth, innerHeight]);
  
  return (
    <div style={{ position: 'relative', width, height }}>
      {/* SVG for axes (accessible, stylable) */}
      <svg width={width} height={height} style={{ position: 'absolute' }}>
        <g transform={`translate(${margin.left}, ${margin.top})`}>
          {/* Y Axis */}
          <g>
            {yScale.ticks(5).map(tick => (
              <g key={tick} transform={`translate(0, ${yScale(tick)})`}>
                <line x1={0} x2={-6} stroke="black" />
                <text x={-9} dy="0.32em" textAnchor="end" fontSize={12}>
                  {tick}
                </text>
              </g>
            ))}
          </g>
          {/* X Axis */}
          <g transform={`translate(0, ${innerHeight})`}>
            <line x1={0} x2={innerWidth} stroke="black" />
          </g>
        </g>
      </svg>
      
      {/* Canvas for data (performant) */}
      <canvas 
        ref={canvasRef}
        width={innerWidth}
        height={innerHeight}
        style={{
          position: 'absolute',
          left: margin.left,
          top: margin.top,
        }}
      />
    </div>
  );
}
```

---

# 12. Performance Optimization

## 12.1 Virtual Rendering

```typescript
// Only render visible elements
function VirtualizedChart({ 
  data, 
  width, 
  height,
  itemWidth = 50 
}: VirtualizedChartProps) {
  const [scrollLeft, setScrollLeft] = useState(0);
  const containerRef = useRef<HTMLDivElement>(null);
  
  // Calculate visible range
  const visibleStart = Math.floor(scrollLeft / itemWidth);
  const visibleEnd = Math.ceil((scrollLeft + width) / itemWidth);
  const visibleData = data.slice(
    Math.max(0, visibleStart - 1),
    Math.min(data.length, visibleEnd + 1)
  );
  
  const handleScroll = (e: React.UIEvent) => {
    setScrollLeft(e.currentTarget.scrollLeft);
  };
  
  const totalWidth = data.length * itemWidth;
  
  return (
    <div 
      ref={containerRef}
      style={{ width, height, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <svg width={totalWidth} height={height}>
        {visibleData.map((d, i) => {
          const actualIndex = visibleStart + i;
          return (
            <rect
              key={actualIndex}
              x={actualIndex * itemWidth}
              y={height - d.value}
              width={itemWidth - 2}
              height={d.value}
              fill="steelblue"
            />
          );
        })}
      </svg>
    </div>
  );
}
```

## 12.2 Data Aggregation

```typescript
// Aggregate data when zoomed out
function aggregateData(
  data: DataPoint[],
  targetPoints: number
): DataPoint[] {
  if (data.length <= targetPoints) return data;
  
  const bucketSize = Math.ceil(data.length / targetPoints);
  const aggregated: DataPoint[] = [];
  
  for (let i = 0; i < data.length; i += bucketSize) {
    const bucket = data.slice(i, i + bucketSize);
    
    aggregated.push({
      date: bucket[0].date,
      value: d3.mean(bucket, d => d.value)!,
      min: d3.min(bucket, d => d.value)!,
      max: d3.max(bucket, d => d.value)!,
    });
  }
  
  return aggregated;
}

// LTTB algorithm for downsampling (keeps visual shape)
function largestTriangleThreeBuckets(
  data: DataPoint[],
  threshold: number
): DataPoint[] {
  if (data.length <= threshold) return data;
  
  const sampled: DataPoint[] = [data[0]];
  const bucketSize = (data.length - 2) / (threshold - 2);
  
  let a = 0;
  
  for (let i = 0; i < threshold - 2; i++) {
    const bucketStart = Math.floor((i + 1) * bucketSize) + 1;
    const bucketEnd = Math.floor((i + 2) * bucketSize) + 1;
    
    // Average point in next bucket
    let avgX = 0;
    let avgY = 0;
    const avgCount = bucketEnd - bucketStart;
    
    for (let j = bucketStart; j < bucketEnd && j < data.length; j++) {
      avgX += j;
      avgY += data[j].value;
    }
    avgX /= avgCount;
    avgY /= avgCount;
    
    // Find point in current bucket with largest triangle area
    const bucketLow = Math.floor(i * bucketSize) + 1;
    const bucketHigh = Math.floor((i + 1) * bucketSize) + 1;
    
    let maxArea = -1;
    let maxAreaIndex = bucketLow;
    
    for (let j = bucketLow; j < bucketHigh && j < data.length; j++) {
      const area = Math.abs(
        (a - avgX) * (data[j].value - data[a].value) -
        (a - j) * (avgY - data[a].value)
      ) * 0.5;
      
      if (area > maxArea) {
        maxArea = area;
        maxAreaIndex = j;
      }
    }
    
    sampled.push(data[maxAreaIndex]);
    a = maxAreaIndex;
  }
  
  sampled.push(data[data.length - 1]);
  return sampled;
}
```

## 12.3 Memoization

```tsx
// Memoize expensive calculations
const MemoizedChart = memo(function Chart({ data, width, height }) {
  // Memoize scales
  const xScale = useMemo(() => 
    d3.scaleLinear()
      .domain([0, data.length - 1])
      .range([0, width]),
    [data.length, width]
  );
  
  // Memoize path generation
  const linePath = useMemo(() => {
    const lineGenerator = d3.line<DataPoint>()
      .x((d, i) => xScale(i))
      .y(d => yScale(d.value));
    
    return lineGenerator(data);
  }, [data, xScale, yScale]);
  
  // Memoize callback
  const handleClick = useCallback((d: DataPoint) => {
    console.log('Clicked:', d);
  }, []);
  
  return (
    <svg width={width} height={height}>
      <path d={linePath || ''} fill="none" stroke="steelblue" />
    </svg>
  );
});

// Split static and dynamic parts
function OptimizedChart({ data, selectedIndex }) {
  return (
    <svg>
      {/* Static background - won't rerender */}
      <StaticBackground />
      
      {/* Data layer - rerenders on data change */}
      <DataLayer data={data} />
      
      {/* Highlight layer - rerenders on selection */}
      <HighlightLayer selectedIndex={selectedIndex} />
    </svg>
  );
}

const StaticBackground = memo(() => (
  <g>
    <rect x={0} y={0} width={500} height={300} fill="#f5f5f5" />
    <text x={250} y={20} textAnchor="middle">Chart Title</text>
  </g>
));

const DataLayer = memo(({ data }) => (
  <g>
    {data.map((d, i) => (
      <circle key={i} cx={i * 10} cy={d.value} r={3} fill="steelblue" />
    ))}
  </g>
));
```

## 12.4 Debouncing Updates

```typescript
// Debounce window resize
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

function ResponsiveChart() {
  const containerRef = useRef<HTMLDivElement>(null);
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    const observer = new ResizeObserver(
      debounce((entries) => {
        const { width, height } = entries[0].contentRect;
        setDimensions({ width, height });
      }, 100)
    );
    
    if (containerRef.current) {
      observer.observe(containerRef.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  // Use debounced dimensions
  const debouncedDimensions = useDebounce(dimensions, 150);
  
  return (
    <div ref={containerRef} style={{ width: '100%', height: 400 }}>
      {debouncedDimensions.width > 0 && (
        <Chart 
          width={debouncedDimensions.width} 
          height={debouncedDimensions.height} 
        />
      )}
    </div>
  );
}
```

---

# 13. Interview Questions

## 13.1 Conceptual Questions

### Q1: What's the difference between SVG and Canvas?

```typescript
// SVG:
// - Vector-based (scales perfectly)
// - DOM-based (each element is a node)
// - Built-in interactivity (events on elements)
// - Better for < 1000 elements
// - Accessible text and ARIA support

// Canvas:
// - Pixel-based (must redraw to scale)
// - Immediate mode (draw and forget)
// - Manual hit detection for interactivity
// - Better for > 10000 elements
// - No built-in accessibility

// When to use which:
// SVG: Charts, icons, maps with interaction
// Canvas: Particle systems, games, large scatter plots
```

### Q2: Explain the D3 enter-update-exit pattern

```typescript
// The pattern handles data changes

const selection = svg.selectAll('circle').data(data, d => d.id);

// ENTER: Data without corresponding elements
// These are new items that need DOM elements
selection.enter()
  .append('circle')
  .attr('r', 0)
  .transition()
  .attr('r', d => d.radius);

// UPDATE: Data with corresponding elements
// These elements exist and need updates
selection
  .attr('cx', d => d.x);

// EXIT: Elements without corresponding data
// These should be removed
selection.exit()
  .transition()
  .attr('r', 0)
  .remove();

// Modern simplified with .join():
selection.join(
  enter => enter.append('circle'),
  update => update.attr('fill', 'green'),
  exit => exit.remove()
);
```

### Q3: How do D3 scales work?

```typescript
// Scales map input domain to output range

// scaleLinear: continuous → continuous
const linear = d3.scaleLinear()
  .domain([0, 100])  // Input: data values
  .range([0, 500]);  // Output: pixels

linear(50);  // 250 (maps data to pixels)

// scaleBand: discrete → continuous (for bar charts)
const band = d3.scaleBand()
  .domain(['A', 'B', 'C'])
  .range([0, 300])
  .padding(0.1);

band('A');           // 0 (x position)
band.bandwidth();    // 96 (bar width)

// scaleOrdinal: discrete → discrete
const color = d3.scaleOrdinal()
  .domain(['A', 'B', 'C'])
  .range(['red', 'green', 'blue']);

color('A');  // 'red'
```

### Q4: What are the key differences between D3 and Recharts?

```typescript
// D3:
// - Low-level toolkit (not a charting library)
// - Full control over DOM
// - Steeper learning curve
// - Can build any visualization
// - Direct DOM manipulation or with React refs
// - No built-in responsiveness

// Recharts:
// - High-level React component library
// - Declarative API
// - Built on D3
// - Easy to use
// - Built-in responsive container
// - Limited customization compared to D3

// Choose D3 when:
// - Need custom visualizations
// - Complex animations
// - Maximum control

// Choose Recharts when:
// - Standard chart types
// - Quick development
// - React-first approach
```

### Q5: How would you optimize a chart with 100,000 data points?

```typescript
// 1. Use Canvas instead of SVG
// Canvas performs better with many elements

// 2. Data aggregation/downsampling
const aggregated = largestTriangleThreeBuckets(data, 1000);

// 3. Virtual rendering
// Only render points in viewport

// 4. Web Workers for data processing
const worker = new Worker('dataProcessor.js');
worker.postMessage(data);
worker.onmessage = (e) => setProcessedData(e.data);

// 5. WebGL for GPU acceleration
// Libraries like deck.gl, regl

// 6. Level of detail
// Show different detail at different zoom levels
function getDetailLevel(zoom: number) {
  if (zoom < 0.5) return aggregateData(data, 100);
  if (zoom < 1) return aggregateData(data, 500);
  return data;
}

// 7. Debounce updates
const debouncedData = useDebounce(data, 100);
```

## 13.2 Coding Questions

### Q6: Implement a simple bar chart with D3 and React

```typescript
function BarChart({ data, width, height }: BarChartProps) {
  const margin = { top: 20, right: 20, bottom: 30, left: 40 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;
  
  const xScale = d3.scaleBand<string>()
    .domain(data.map(d => d.label))
    .range([0, innerWidth])
    .padding(0.2);
  
  const yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value)!])
    .range([innerHeight, 0])
    .nice();
  
  return (
    <svg width={width} height={height}>
      <g transform={`translate(${margin.left}, ${margin.top})`}>
        {/* Bars */}
        {data.map(d => (
          <rect
            key={d.label}
            x={xScale(d.label)}
            y={yScale(d.value)}
            width={xScale.bandwidth()}
            height={innerHeight - yScale(d.value)}
            fill="steelblue"
          />
        ))}
        
        {/* X Axis */}
        <g transform={`translate(0, ${innerHeight})`}>
          <line x1={0} x2={innerWidth} stroke="black" />
          {data.map(d => (
            <text
              key={d.label}
              x={xScale(d.label)! + xScale.bandwidth() / 2}
              y={20}
              textAnchor="middle"
            >
              {d.label}
            </text>
          ))}
        </g>
        
        {/* Y Axis */}
        <g>
          <line y1={0} y2={innerHeight} stroke="black" />
          {yScale.ticks(5).map(tick => (
            <text key={tick} x={-5} y={yScale(tick)} textAnchor="end">
              {tick}
            </text>
          ))}
        </g>
      </g>
    </svg>
  );
}
```

### Q7: Create an animated pie chart

```typescript
function AnimatedPieChart({ data }: PieChartProps) {
  const [animatedData, setAnimatedData] = useState(
    data.map(d => ({ ...d, value: 0 }))
  );
  
  useEffect(() => {
    // Animate to actual values
    const timer = setTimeout(() => {
      setAnimatedData(data);
    }, 100);
    
    return () => clearTimeout(timer);
  }, [data]);
  
  const pie = d3.pie<DataPoint>()
    .value(d => d.value)
    .sort(null);
  
  const arc = d3.arc()
    .innerRadius(0)
    .outerRadius(100);
  
  const arcs = pie(animatedData);
  
  return (
    <svg width={250} height={250}>
      <g transform="translate(125, 125)">
        {arcs.map((a, i) => (
          <path
            key={data[i].label}
            d={arc(a as any) || ''}
            fill={colorScale(data[i].label)}
            style={{
              transition: 'all 0.5s ease-out',
            }}
          />
        ))}
      </g>
    </svg>
  );
}
```

### Q8: Implement a tooltip that follows the mouse

```typescript
function ChartWithTooltip({ data }) {
  const [tooltip, setTooltip] = useState<{
    x: number;
    y: number;
    data: DataPoint | null;
  }>({ x: 0, y: 0, data: null });
  
  const handleMouseEnter = (e: React.MouseEvent, d: DataPoint) => {
    setTooltip({
      x: e.clientX,
      y: e.clientY,
      data: d,
    });
  };
  
  const handleMouseLeave = () => {
    setTooltip(prev => ({ ...prev, data: null }));
  };
  
  const handleMouseMove = (e: React.MouseEvent) => {
    if (tooltip.data) {
      setTooltip(prev => ({
        ...prev,
        x: e.clientX,
        y: e.clientY,
      }));
    }
  };
  
  return (
    <div style={{ position: 'relative' }}>
      <svg onMouseMove={handleMouseMove}>
        {data.map((d, i) => (
          <rect
            key={i}
            onMouseEnter={(e) => handleMouseEnter(e, d)}
            onMouseLeave={handleMouseLeave}
            // ... other props
          />
        ))}
      </svg>
      
      {tooltip.data && (
        <div
          style={{
            position: 'fixed',
            left: tooltip.x + 10,
            top: tooltip.y + 10,
            background: 'white',
            border: '1px solid #ccc',
            padding: '8px',
            pointerEvents: 'none',
          }}
        >
          <strong>{tooltip.data.label}</strong>
          <br />
          Value: {tooltip.data.value}
        </div>
      )}
    </div>
  );
}
```

---

# 14. More Coding Challenges

## 14.1 Animated Number Counter

```typescript
// Challenge: Create a smooth number counter animation

function AnimatedCounter({ 
  value, 
  duration = 1000,
  prefix = '',
  suffix = '',
}: AnimatedCounterProps) {
  const [displayValue, setDisplayValue] = useState(0);
  const previousValue = useRef(0);
  const frameRef = useRef<number>();
  
  useEffect(() => {
    const startValue = previousValue.current;
    const endValue = value;
    const startTime = performance.now();
    
    const animate = (currentTime: number) => {
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);
      
      // Easing: ease out cubic
      const eased = 1 - Math.pow(1 - progress, 3);
      const current = startValue + (endValue - startValue) * eased;
      
      setDisplayValue(current);
      
      if (progress < 1) {
        frameRef.current = requestAnimationFrame(animate);
      }
    };
    
    frameRef.current = requestAnimationFrame(animate);
    previousValue.current = value;
    
    return () => {
      if (frameRef.current) {
        cancelAnimationFrame(frameRef.current);
      }
    };
  }, [value, duration]);
  
  const formatNumber = (n: number) => {
    return new Intl.NumberFormat().format(Math.round(n));
  };
  
  return (
    <span className="animated-counter">
      {prefix}{formatNumber(displayValue)}{suffix}
    </span>
  );
}

// Usage
<AnimatedCounter value={1234567} prefix="$" duration={2000} />
```

## 14.2 Responsive Chart Container

```typescript
// Challenge: Create a reusable responsive chart wrapper

interface ResponsiveChartContainerProps {
  children: (dimensions: { width: number; height: number }) => React.ReactNode;
  aspectRatio?: number;
  minHeight?: number;
  maxHeight?: number;
  className?: string;
}

function ResponsiveChartContainer({
  children,
  aspectRatio,
  minHeight = 200,
  maxHeight = 600,
  className,
}: ResponsiveChartContainerProps) {
  const containerRef = useRef<HTMLDivElement>(null);
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    const container = containerRef.current;
    if (!container) return;
    
    const resizeObserver = new ResizeObserver(
      debounce((entries) => {
        const { width } = entries[0].contentRect;
        
        let height: number;
        if (aspectRatio) {
          height = width / aspectRatio;
        } else {
          height = entries[0].contentRect.height;
        }
        
        // Apply constraints
        height = Math.max(minHeight, Math.min(maxHeight, height));
        
        setDimensions({ width, height });
      }, 100)
    );
    
    resizeObserver.observe(container);
    return () => resizeObserver.disconnect();
  }, [aspectRatio, minHeight, maxHeight]);
  
  return (
    <div 
      ref={containerRef} 
      className={className}
      style={{ 
        width: '100%', 
        height: aspectRatio ? 'auto' : '100%',
        minHeight,
        maxHeight,
      }}
    >
      {dimensions.width > 0 && children(dimensions)}
    </div>
  );
}

// Usage
<ResponsiveChartContainer aspectRatio={16/9}>
  {({ width, height }) => (
    <LineChart width={width} height={height} data={data} />
  )}
</ResponsiveChartContainer>
```

## 14.3 Chart Legend Component

```typescript
// Challenge: Build a customizable legend component

interface LegendItem {
  id: string;
  label: string;
  color: string;
}

interface ChartLegendProps {
  items: LegendItem[];
  selected?: Set<string>;
  onToggle?: (id: string) => void;
  layout?: 'horizontal' | 'vertical';
}

function ChartLegend({ 
  items, 
  selected, 
  onToggle,
  layout = 'horizontal',
}: ChartLegendProps) {
  const handleClick = (id: string) => {
    if (onToggle) {
      onToggle(id);
    }
  };
  
  return (
    <div 
      className="chart-legend"
      style={{
        display: 'flex',
        flexDirection: layout === 'horizontal' ? 'row' : 'column',
        gap: layout === 'horizontal' ? 20 : 8,
        flexWrap: 'wrap',
      }}
    >
      {items.map(item => {
        const isActive = !selected || selected.has(item.id);
        
        return (
          <button
            key={item.id}
            className="legend-item"
            onClick={() => handleClick(item.id)}
            style={{
              display: 'flex',
              alignItems: 'center',
              gap: 8,
              padding: '4px 8px',
              border: 'none',
              background: 'none',
              cursor: onToggle ? 'pointer' : 'default',
              opacity: isActive ? 1 : 0.4,
            }}
          >
            <span 
              className="legend-color"
              style={{
                width: 12,
                height: 12,
                backgroundColor: item.color,
                borderRadius: 2,
              }}
            />
            <span className="legend-label">
              {item.label}
            </span>
          </button>
        );
      })}
    </div>
  );
}

// Usage with toggle functionality
function ChartWithLegend() {
  const [hiddenSeries, setHiddenSeries] = useState<Set<string>>(new Set());
  
  const legendItems = [
    { id: 'sales', label: 'Sales', color: '#8884d8' },
    { id: 'profit', label: 'Profit', color: '#82ca9d' },
    { id: 'cost', label: 'Cost', color: '#ffc658' },
  ];
  
  const handleToggle = (id: string) => {
    setHiddenSeries(prev => {
      const next = new Set(prev);
      if (next.has(id)) {
        next.delete(id);
      } else {
        next.add(id);
      }
      return next;
    });
  };
  
  const visibleSeries = legendItems
    .filter(item => !hiddenSeries.has(item.id))
    .map(item => item.id);
  
  return (
    <div>
      <ChartLegend 
        items={legendItems} 
        selected={new Set(visibleSeries)}
        onToggle={handleToggle}
      />
      <LineChart data={data}>
        {legendItems.map(item => 
          !hiddenSeries.has(item.id) && (
            <Line 
              key={item.id}
              dataKey={item.id} 
              stroke={item.color} 
            />
          )
        )}
      </LineChart>
    </div>
  );
}
```

## 14.4 Sparkline Component

```typescript
// Challenge: Create a minimal sparkline chart

interface SparklineProps {
  data: number[];
  width?: number;
  height?: number;
  color?: string;
  showPoint?: boolean;
  showArea?: boolean;
}

function Sparkline({
  data,
  width = 100,
  height = 30,
  color = '#3b82f6',
  showPoint = true,
  showArea = false,
}: SparklineProps) {
  if (data.length < 2) return null;
  
  const padding = 2;
  const innerWidth = width - padding * 2;
  const innerHeight = height - padding * 2;
  
  const min = Math.min(...data);
  const max = Math.max(...data);
  const range = max - min || 1;
  
  const points = data.map((value, i) => ({
    x: padding + (i / (data.length - 1)) * innerWidth,
    y: padding + innerHeight - ((value - min) / range) * innerHeight,
  }));
  
  const linePath = points
    .map((p, i) => `${i === 0 ? 'M' : 'L'} ${p.x},${p.y}`)
    .join(' ');
  
  const areaPath = showArea ? 
    `${linePath} L ${points[points.length - 1].x},${height - padding} L ${padding},${height - padding} Z` : 
    '';
  
  const lastPoint = points[points.length - 1];
  const isPositive = data[data.length - 1] >= data[0];
  
  return (
    <svg width={width} height={height} className="sparkline">
      {showArea && (
        <path
          d={areaPath}
          fill={color}
          fillOpacity={0.1}
        />
      )}
      <path
        d={linePath}
        fill="none"
        stroke={color}
        strokeWidth={1.5}
        strokeLinecap="round"
        strokeLinejoin="round"
      />
      {showPoint && (
        <circle
          cx={lastPoint.x}
          cy={lastPoint.y}
          r={2.5}
          fill={isPositive ? '#10b981' : '#ef4444'}
        />
      )}
    </svg>
  );
}

// Usage
<Sparkline 
  data={[10, 15, 8, 22, 18, 25, 20, 28]} 
  width={120} 
  height={40}
  showArea
/>
```

## 14.5 Heatmap Component

```typescript
// Challenge: Create a heatmap visualization

interface HeatmapData {
  row: string;
  col: string;
  value: number;
}

interface HeatmapProps {
  data: HeatmapData[];
  width?: number;
  height?: number;
  colorScale?: (value: number) => string;
}

function Heatmap({ 
  data, 
  width = 400, 
  height = 300,
  colorScale: customColorScale,
}: HeatmapProps) {
  const margin = { top: 50, right: 20, bottom: 50, left: 80 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;
  
  // Get unique rows and columns
  const rows = [...new Set(data.map(d => d.row))];
  const cols = [...new Set(data.map(d => d.col))];
  
  // Create scales
  const xScale = d3.scaleBand<string>()
    .domain(cols)
    .range([0, innerWidth])
    .padding(0.05);
  
  const yScale = d3.scaleBand<string>()
    .domain(rows)
    .range([0, innerHeight])
    .padding(0.05);
  
  const extent = d3.extent(data, d => d.value) as [number, number];
  
  const colorScale = customColorScale || d3.scaleSequential(d3.interpolateRdYlBu)
    .domain([extent[1], extent[0]]); // Reversed for heatmap
  
  // Create data map for quick lookup
  const dataMap = new Map(
    data.map(d => [`${d.row}-${d.col}`, d.value])
  );
  
  return (
    <svg width={width} height={height}>
      <g transform={`translate(${margin.left}, ${margin.top})`}>
        {/* Cells */}
        {rows.map(row =>
          cols.map(col => {
            const value = dataMap.get(`${row}-${col}`);
            if (value === undefined) return null;
            
            return (
              <g key={`${row}-${col}`}>
                <rect
                  x={xScale(col)}
                  y={yScale(row)}
                  width={xScale.bandwidth()}
                  height={yScale.bandwidth()}
                  fill={colorScale(value)}
                  rx={2}
                />
                <title>{`${row}, ${col}: ${value}`}</title>
              </g>
            );
          })
        )}
        
        {/* X Axis Labels */}
        <g transform={`translate(0, ${innerHeight})`}>
          {cols.map(col => (
            <text
              key={col}
              x={(xScale(col) || 0) + xScale.bandwidth() / 2}
              y={15}
              textAnchor="middle"
              fontSize={10}
            >
              {col}
            </text>
          ))}
        </g>
        
        {/* Y Axis Labels */}
        <g>
          {rows.map(row => (
            <text
              key={row}
              x={-5}
              y={(yScale(row) || 0) + yScale.bandwidth() / 2}
              textAnchor="end"
              dominantBaseline="middle"
              fontSize={10}
            >
              {row}
            </text>
          ))}
        </g>
      </g>
    </svg>
  );
}
```

---

# 15. Output-Based Questions

## 15.1 What Will Render?

```typescript
// Question 1: ViewBox scaling
<svg width="200" height="100" viewBox="0 0 100 50">
  <rect width="100" height="50" fill="blue" />
</svg>

// Answer: A blue rectangle that fills the entire 200x100 SVG
// The viewBox coordinates are scaled 2x to fit the viewport

// Question 2: Transform order
<rect 
  x="0" y="0" 
  width="50" height="30"
  transform="translate(100, 50) rotate(45)"
/>

// Answer: Rectangle is first translated to (100,50),
// then rotated 45 degrees around origin (0,0)
// The rotation happens AROUND THE SVG ORIGIN, not the rect center!

// Question 3: D3 data binding
const data = [1, 2, 3];
d3.select('svg').selectAll('circle').data(data);

// What's in enter(), update, exit()?
// Answer:
// - enter(): All 3 data points (assuming no existing circles)
// - update: Empty (no existing circles matched)
// - exit(): Empty (no existing circles)

// Question 4: Scale output
const scale = d3.scaleLinear().domain([0, 10]).range([0, 100]);
console.log(scale(5));   // ?
console.log(scale(15));  // ?
console.log(scale(-5));  // ?

// Answer:
// scale(5) = 50
// scale(15) = 150 (extrapolates beyond range)
// scale(-5) = -50 (extrapolates below domain)
```

## 15.2 Find the Bug

```typescript
// Bug 1: Memory leak in useEffect
function Chart({ data }) {
  const svgRef = useRef(null);
  
  useEffect(() => {
    const svg = d3.select(svgRef.current);
    
    svg.selectAll('circle')
      .data(data)
      .join('circle')
      .on('click', (e, d) => console.log(d));
    
    // BUG: No cleanup for event listeners!
  }, [data]);
  
  return <svg ref={svgRef} />;
}

// Fix: Return cleanup function
useEffect(() => {
  const svg = d3.select(svgRef.current);
  
  svg.selectAll('circle')
    .data(data)
    .join('circle')
    .on('click', (e, d) => console.log(d));
  
  return () => {
    svg.selectAll('circle').on('click', null);
  };
}, [data]);
```

```typescript
// Bug 2: Stale closure in animation
function AnimatedChart({ data }) {
  const [frame, setFrame] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      // BUG: data is stale - captured at mount time!
      console.log(data.length);
      setFrame(f => f + 1);
    }, 100);
    
    return () => clearInterval(interval);
  }, []); // Empty deps array!
  
  // ... render
}

// Fix: Include data in dependencies or use ref
const dataRef = useRef(data);
useEffect(() => {
  dataRef.current = data;
}, [data]);

useEffect(() => {
  const interval = setInterval(() => {
    console.log(dataRef.current.length); // Always current
    setFrame(f => f + 1);
  }, 100);
  
  return () => clearInterval(interval);
}, []);
```

```typescript
// Bug 3: Y-axis inverted
const yScale = d3.scaleLinear()
  .domain([0, max])
  .range([0, height]); // BUG: Not inverted!

// Bars will be upside down!

// Fix: Invert range for Y axis
const yScale = d3.scaleLinear()
  .domain([0, max])
  .range([height, 0]); // Now 0 is at bottom, max at top
```

---

# 16. Mock Interview Scenarios

## Scenario 1: Dashboard Design

**Interviewer**: "Design a real-time dashboard showing metrics for an e-commerce platform"

**Strong Answer:**

```typescript
// Dashboard architecture

interface DashboardProps {
  websocketUrl: string;
  initialData: DashboardData;
}

function EcommerceDashboard({ websocketUrl, initialData }: DashboardProps) {
  const { metrics, isConnected } = useRealtimeMetrics(websocketUrl, initialData);
  
  return (
    <div className="dashboard">
      <header>
        <h1>Sales Dashboard</h1>
        <ConnectionStatus isConnected={isConnected} />
      </header>
      
      <div className="metrics-grid">
        {/* Key metrics with trend indicators */}
        <MetricCard
          title="Revenue Today"
          value={metrics.revenue}
          trend={metrics.revenueTrend}
          format="currency"
        />
        
        <MetricCard
          title="Orders"
          value={metrics.orders}
          trend={metrics.ordersTrend}
          format="number"
        />
        
        <MetricCard
          title="Conversion Rate"
          value={metrics.conversionRate}
          trend={metrics.conversionTrend}
          format="percent"
        />
        
        <MetricCard
          title="Active Users"
          value={metrics.activeUsers}
          format="number"
          sparkline={metrics.activeUsersHistory}
        />
      </div>
      
      <div className="charts-section">
        {/* Revenue over time - area chart */}
        <ResponsiveChartContainer aspectRatio={2}>
          {({ width, height }) => (
            <RevenueChart
              data={metrics.revenueHistory}
              width={width}
              height={height}
            />
          )}
        </ResponsiveChartContainer>
        
        {/* Sales by category - pie chart */}
        <SalesByCategoryChart data={metrics.salesByCategory} />
        
        {/* Geographic distribution - choropleth map */}
        <SalesMap data={metrics.salesByRegion} />
      </div>
    </div>
  );
}

// Key considerations:
// 1. Real-time updates via WebSocket
// 2. Responsive design with ResponsiveContainer
// 3. Consistent color scheme across charts
// 4. Trend indicators for context
// 5. Sparklines for quick historical context
// 6. Performance optimization for high-frequency updates
```

## Scenario 2: Large Dataset Visualization

**Interviewer**: "You need to visualize 1 million data points in a scatter plot. How would you approach this?"

**Strong Answer:**

```typescript
// Multi-level approach for large data

interface LargeScatterPlotProps {
  data: DataPoint[];
  width: number;
  height: number;
}

function LargeScatterPlot({ data, width, height }: LargeScatterPlotProps) {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [zoom, setZoom] = useState(1);
  const [pan, setPan] = useState({ x: 0, y: 0 });
  
  // 1. Downsample data based on zoom level
  const displayData = useMemo(() => {
    const targetPoints = getTargetPointCount(data.length, zoom);
    return downsampleData(data, targetPoints);
  }, [data, zoom]);
  
  // 2. Use Canvas for rendering
  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    
    const ctx = canvas.getContext('2d')!;
    ctx.clearRect(0, 0, width, height);
    
    // Apply transformations
    ctx.save();
    ctx.translate(pan.x, pan.y);
    ctx.scale(zoom, zoom);
    
    // Draw points
    ctx.fillStyle = 'rgba(66, 133, 244, 0.5)';
    displayData.forEach(point => {
      ctx.beginPath();
      ctx.arc(xScale(point.x), yScale(point.y), 2, 0, Math.PI * 2);
      ctx.fill();
    });
    
    ctx.restore();
  }, [displayData, width, height, zoom, pan]);
  
  // 3. Quadtree for hit detection
  const quadtree = useMemo(() => {
    return d3.quadtree<DataPoint>()
      .x(d => xScale(d.x))
      .y(d => yScale(d.y))
      .addAll(displayData);
  }, [displayData]);
  
  const handleMouseMove = (e: React.MouseEvent) => {
    const [x, y] = getMousePosition(e);
    const nearest = quadtree.find(x, y, 20); // 20px radius
    if (nearest) {
      showTooltip(nearest);
    }
  };
  
  return (
    <div className="scatter-container">
      <canvas
        ref={canvasRef}
        width={width}
        height={height}
        onMouseMove={handleMouseMove}
        onWheel={handleZoom}
        onMouseDown={handlePanStart}
      />
      <Tooltip data={tooltipData} />
    </div>
  );
}

// Helper functions
function getTargetPointCount(total: number, zoom: number): number {
  // More points visible at higher zoom
  return Math.min(total, Math.round(5000 * zoom));
}

function downsampleData(data: DataPoint[], target: number): DataPoint[] {
  if (data.length <= target) return data;
  
  // Use spatial binning
  const gridSize = Math.ceil(Math.sqrt(target));
  const grid = new Map<string, DataPoint[]>();
  
  data.forEach(point => {
    const key = `${Math.floor(point.x / 10)}-${Math.floor(point.y / 10)}`;
    if (!grid.has(key)) grid.set(key, []);
    grid.get(key)!.push(point);
  });
  
  // Sample from each bin
  const sampled: DataPoint[] = [];
  grid.forEach(bin => {
    const sample = bin[Math.floor(Math.random() * bin.length)];
    sampled.push(sample);
  });
  
  return sampled;
}
```

---

# 17. Quick Reference

## 17.1 D3 Cheat Sheet

```typescript
// Selection
d3.select('#id')           // Select by ID
d3.selectAll('.class')     // Select all by class
selection.select('child')   // Select descendant
selection.filter(d => d.value > 0) // Filter

// Modification
.attr('name', value)       // Set attribute
.style('property', value)  // Set CSS style
.classed('name', bool)     // Toggle class
.text(value)               // Set text content
.html(value)               // Set innerHTML

// Data binding
.data(array)               // Bind data
.data(array, d => d.id)    // With key function
.datum(value)              // Bind single value
.enter()                   // New data selection
.exit()                    // Removed data selection
.join('element')           // Simplified enter/update/exit

// Creation
.append('element')         // Append new element
.insert('element', before) // Insert before
.remove()                  // Remove elements

// Scales
d3.scaleLinear()           // Continuous linear
d3.scaleBand()             // Discrete bands
d3.scaleTime()             // Time scale
d3.scaleOrdinal()          // Discrete to discrete
d3.scaleSequential()       // Continuous color

// Axes
d3.axisBottom(scale)       // Bottom axis
d3.axisLeft(scale)         // Left axis
axis.ticks(5)              // Number of ticks
axis.tickFormat(d => '$' + d)  // Format

// Shapes
d3.line()                  // Line generator
d3.area()                  // Area generator
d3.arc()                   // Arc generator
d3.pie()                   // Pie layout
```

## 17.2 SVG Cheat Sheet

```html
<!-- Basic shapes -->
<rect x="0" y="0" width="100" height="50" />
<circle cx="50" cy="50" r="25" />
<ellipse cx="50" cy="25" rx="50" ry="25" />
<line x1="0" y1="0" x2="100" y2="100" />
<polyline points="0,0 50,50 100,0" />
<polygon points="50,0 100,100 0,100" />
<path d="M 0,0 L 100,0 L 50,100 Z" />
<text x="50" y="50" text-anchor="middle">Hello</text>

<!-- Attributes -->
fill="color"              <!-- Fill color -->
stroke="color"            <!-- Stroke color -->
stroke-width="2"          <!-- Stroke thickness -->
opacity="0.5"             <!-- Transparency -->
transform="translate(x,y) rotate(angle) scale(x,y)"

<!-- Path commands -->
M x,y                     <!-- Move to -->
L x,y                     <!-- Line to -->
H x                       <!-- Horizontal line -->
V y                       <!-- Vertical line -->
C x1,y1 x2,y2 x,y        <!-- Cubic bezier -->
Q x1,y1 x,y              <!-- Quadratic bezier -->
A rx,ry rot large sweep x,y  <!-- Arc -->
Z                         <!-- Close path -->
```

## 17.3 Recharts Cheat Sheet

```tsx
// Container
<ResponsiveContainer width="100%" height={400}>

// Chart types
<LineChart data={data}>
<BarChart data={data}>
<AreaChart data={data}>
<PieChart>
<ScatterChart>
<ComposedChart data={data}>
<RadarChart data={data}>

// Chart elements
<Line type="monotone" dataKey="value" stroke="#8884d8" />
<Bar dataKey="value" fill="#82ca9d" />
<Area dataKey="value" fill="url(#gradient)" />
<Pie data={data} dataKey="value" />
<Scatter data={data} fill="#8884d8" />

// Axes
<XAxis dataKey="name" />
<YAxis tickFormatter={(v) => `$${v}`} />
<CartesianGrid strokeDasharray="3 3" />

// Interactive
<Tooltip content={<CustomTooltip />} />
<Legend />
<Brush dataKey="name" height={30} />
<ReferenceLine y={100} stroke="red" />
<ReferenceArea x1="A" x2="C" fill="green" fillOpacity={0.1} />
```

---

# 18. Final Summary

## What You Should Know

### Core Concepts
1. **SVG Fundamentals** - Shapes, viewBox, transforms, paths
2. **D3 Philosophy** - Data binding, enter-update-exit, scales
3. **React Integration** - D3 for math, React for DOM
4. **Recharts** - Declarative React charting

### Performance
1. SVG for < 1000 elements
2. Canvas for > 10000 elements
3. Data aggregation/downsampling
4. Virtual rendering
5. Memoization

### Common Interview Topics
1. SVG vs Canvas tradeoffs
2. D3 data binding pattern
3. Scales and axes
4. Responsive design
5. Animation techniques
6. Large dataset handling

### Key Libraries
- **D3.js** - Visualization toolkit
- **Recharts** - React charting
- **visx** - Low-level React + D3
- **Chart.js** - Simple charts
- **Highcharts** - Enterprise charts
- **deck.gl** - WebGL visualization

---

**End of Part 10: Data Visualization Complete Guide**

Total Sections: 18
Key Topics: SVG, D3.js, Recharts, Canvas, Performance
Interview Ready: ✅

---

# BONUS: Advanced Visualization Patterns

## Advanced Pattern 1: Force-Directed Graph

```typescript
// Network visualization using D3 force simulation

interface Node {
  id: string;
  name: string;
  group: number;
}

interface Link {
  source: string;
  target: string;
  value: number;
}

interface ForceGraphProps {
  nodes: Node[];
  links: Link[];
  width: number;
  height: number;
}

function ForceDirectedGraph({ nodes, links, width, height }: ForceGraphProps) {
  const svgRef = useRef<SVGSVGElement>(null);
  
  useEffect(() => {
    const svg = d3.select(svgRef.current);
    svg.selectAll('*').remove();
    
    // Create simulation
    const simulation = d3.forceSimulation(nodes as any)
      .force('link', d3.forceLink(links)
        .id((d: any) => d.id)
        .distance(50)
      )
      .force('charge', d3.forceManyBody().strength(-200))
      .force('center', d3.forceCenter(width / 2, height / 2))
      .force('collision', d3.forceCollide().radius(30));
    
    // Color scale
    const colorScale = d3.scaleOrdinal(d3.schemeCategory10);
    
    // Create links
    const link = svg.append('g')
      .attr('class', 'links')
      .selectAll('line')
      .data(links)
      .join('line')
      .attr('stroke', '#999')
      .attr('stroke-opacity', 0.6)
      .attr('stroke-width', d => Math.sqrt(d.value));
    
    // Create nodes
    const node = svg.append('g')
      .attr('class', 'nodes')
      .selectAll('g')
      .data(nodes)
      .join('g')
      .call(d3.drag()
        .on('start', dragStarted)
        .on('drag', dragged)
        .on('end', dragEnded) as any
      );
    
    node.append('circle')
      .attr('r', 10)
      .attr('fill', d => colorScale(String(d.group)));
    
    node.append('text')
      .attr('dx', 12)
      .attr('dy', 4)
      .text(d => d.name)
      .attr('font-size', 12);
    
    // Update positions on tick
    simulation.on('tick', () => {
      link
        .attr('x1', (d: any) => d.source.x)
        .attr('y1', (d: any) => d.source.y)
        .attr('x2', (d: any) => d.target.x)
        .attr('y2', (d: any) => d.target.y);
      
      node.attr('transform', (d: any) => `translate(${d.x}, ${d.y})`);
    });
    
    function dragStarted(event: any, d: any) {
      if (!event.active) simulation.alphaTarget(0.3).restart();
      d.fx = d.x;
      d.fy = d.y;
    }
    
    function dragged(event: any, d: any) {
      d.fx = event.x;
      d.fy = event.y;
    }
    
    function dragEnded(event: any, d: any) {
      if (!event.active) simulation.alphaTarget(0);
      d.fx = null;
      d.fy = null;
    }
    
    return () => {
      simulation.stop();
    };
  }, [nodes, links, width, height]);
  
  return <svg ref={svgRef} width={width} height={height} />;
}
```

## Advanced Pattern 2: Choropleth Map

```typescript
// Geographic visualization with color coding

interface GeoDataProps {
  geoJSON: any;
  data: Map<string, number>;
  width: number;
  height: number;
}

function ChoroplethMap({ geoJSON, data, width, height }: GeoDataProps) {
  const svgRef = useRef<SVGSVGElement>(null);
  
  useEffect(() => {
    const svg = d3.select(svgRef.current);
    svg.selectAll('*').remove();
    
    // Create projection
    const projection = d3.geoMercator()
      .fitSize([width, height], geoJSON);
    
    // Create path generator
    const pathGenerator = d3.geoPath().projection(projection);
    
    // Color scale
    const values = Array.from(data.values());
    const colorScale = d3.scaleSequential(d3.interpolateBlues)
      .domain([d3.min(values)!, d3.max(values)!]);
    
    // Draw regions
    svg.selectAll('path')
      .data(geoJSON.features)
      .join('path')
      .attr('d', pathGenerator as any)
      .attr('fill', (d: any) => {
        const value = data.get(d.properties.id);
        return value !== undefined ? colorScale(value) : '#ccc';
      })
      .attr('stroke', '#fff')
      .attr('stroke-width', 0.5)
      .on('mouseover', function(event, d: any) {
        d3.select(this).attr('stroke-width', 2);
        // Show tooltip
      })
      .on('mouseout', function() {
        d3.select(this).attr('stroke-width', 0.5);
      });
    
    // Add legend
    const legendWidth = 200;
    const legendHeight = 10;
    
    const legendScale = d3.scaleLinear()
      .domain(colorScale.domain())
      .range([0, legendWidth]);
    
    const legendAxis = d3.axisBottom(legendScale)
      .ticks(5)
      .tickSize(legendHeight + 3);
    
    const legend = svg.append('g')
      .attr('transform', `translate(${width - legendWidth - 20}, ${height - 40})`);
    
    // Gradient for legend
    const defs = svg.append('defs');
    const gradient = defs.append('linearGradient')
      .attr('id', 'legend-gradient');
    
    gradient.selectAll('stop')
      .data(d3.range(0, 1.01, 0.1))
      .join('stop')
      .attr('offset', d => `${d * 100}%`)
      .attr('stop-color', d => colorScale(d * d3.max(values)!));
    
    legend.append('rect')
      .attr('width', legendWidth)
      .attr('height', legendHeight)
      .attr('fill', 'url(#legend-gradient)');
    
    legend.append('g')
      .call(legendAxis)
      .select('.domain')
      .remove();
    
  }, [geoJSON, data, width, height]);
  
  return <svg ref={svgRef} width={width} height={height} />;
}
```

## Advanced Pattern 3: Sankey Diagram

```typescript
// Flow visualization showing relationships

import { sankey, sankeyLinkHorizontal } from 'd3-sankey';

interface SankeyNode {
  name: string;
}

interface SankeyLink {
  source: number;
  target: number;
  value: number;
}

interface SankeyDiagramProps {
  nodes: SankeyNode[];
  links: SankeyLink[];
  width: number;
  height: number;
}

function SankeyDiagram({ nodes, links, width, height }: SankeyDiagramProps) {
  const margin = { top: 10, right: 10, bottom: 10, left: 10 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;
  
  // Create sankey generator
  const sankeyGenerator = sankey<SankeyNode, SankeyLink>()
    .nodeWidth(20)
    .nodePadding(10)
    .extent([[0, 0], [innerWidth, innerHeight]]);
  
  // Generate layout
  const { nodes: layoutNodes, links: layoutLinks } = sankeyGenerator({
    nodes: nodes.map(d => ({ ...d })),
    links: links.map(d => ({ ...d })),
  });
  
  // Color scale
  const colorScale = d3.scaleOrdinal(d3.schemeCategory10);
  
  return (
    <svg width={width} height={height}>
      <g transform={`translate(${margin.left}, ${margin.top})`}>
        {/* Links */}
        <g fill="none" strokeOpacity={0.3}>
          {layoutLinks.map((link, i) => (
            <path
              key={i}
              d={sankeyLinkHorizontal()(link as any) || ''}
              stroke={colorScale(String((link.source as any).name))}
              strokeWidth={Math.max(1, (link as any).width)}
            />
          ))}
        </g>
        
        {/* Nodes */}
        {layoutNodes.map((node, i) => (
          <g key={i} transform={`translate(${(node as any).x0}, ${(node as any).y0})`}>
            <rect
              width={(node as any).x1 - (node as any).x0}
              height={(node as any).y1 - (node as any).y0}
              fill={colorScale(node.name)}
            />
            <text
              x={(node as any).x1 - (node as any).x0 < innerWidth / 2 ? 
                (node as any).x1 - (node as any).x0 + 5 : -5}
              y={((node as any).y1 - (node as any).y0) / 2}
              dy="0.35em"
              textAnchor={(node as any).x0 < innerWidth / 2 ? 'start' : 'end'}
              fontSize={12}
            >
              {node.name}
            </text>
          </g>
        ))}
      </g>
    </svg>
  );
}
```

## Advanced Pattern 4: Sunburst Chart

```typescript
// Hierarchical data as nested arcs

interface HierarchyNode {
  name: string;
  value?: number;
  children?: HierarchyNode[];
}

interface SunburstProps {
  data: HierarchyNode;
  width: number;
  height: number;
}

function SunburstChart({ data, width, height }: SunburstProps) {
  const radius = Math.min(width, height) / 2;
  
  // Create hierarchy
  const hierarchy = d3.hierarchy(data)
    .sum(d => d.value || 0)
    .sort((a, b) => (b.value || 0) - (a.value || 0));
  
  // Create partition layout
  const partition = d3.partition<HierarchyNode>()
    .size([2 * Math.PI, radius]);
  
  const root = partition(hierarchy);
  
  // Color scale
  const colorScale = d3.scaleOrdinal(d3.schemeSet3);
  
  // Arc generator
  const arc = d3.arc<any>()
    .startAngle(d => d.x0)
    .endAngle(d => d.x1)
    .padAngle(d => Math.min((d.x1 - d.x0) / 2, 0.005))
    .padRadius(radius / 2)
    .innerRadius(d => d.y0)
    .outerRadius(d => d.y1 - 1);
  
  return (
    <svg width={width} height={height}>
      <g transform={`translate(${width / 2}, ${height / 2})`}>
        {root.descendants()
          .filter(d => d.depth > 0)
          .map((d, i) => (
            <path
              key={i}
              d={arc(d) || ''}
              fill={colorScale(d.ancestors().map(a => a.data.name).reverse()[1])}
              stroke="#fff"
            >
              <title>
                {d.ancestors().map(a => a.data.name).reverse().slice(1).join(' > ')}
                {': '}
                {d.value?.toLocaleString()}
              </title>
            </path>
          ))}
      </g>
    </svg>
  );
}
```

---

# BONUS: Additional Interview Questions

## More Conceptual Questions

### Q9: How would you create a tooltip that works across different chart types?

```typescript
// Generic tooltip system

interface TooltipData {
  x: number;
  y: number;
  content: React.ReactNode;
}

const TooltipContext = createContext<{
  show: (data: TooltipData) => void;
  hide: () => void;
} | null>(null);

function TooltipProvider({ children }: { children: React.ReactNode }) {
  const [tooltip, setTooltip] = useState<TooltipData | null>(null);
  
  const show = useCallback((data: TooltipData) => {
    setTooltip(data);
  }, []);
  
  const hide = useCallback(() => {
    setTooltip(null);
  }, []);
  
  return (
    <TooltipContext.Provider value={{ show, hide }}>
      {children}
      {tooltip && (
        <div
          className="tooltip"
          style={{
            position: 'fixed',
            left: tooltip.x + 10,
            top: tooltip.y + 10,
            background: 'white',
            border: '1px solid #ccc',
            borderRadius: 4,
            padding: '8px 12px',
            boxShadow: '0 2px 8px rgba(0,0,0,0.15)',
            pointerEvents: 'none',
            zIndex: 1000,
          }}
        >
          {tooltip.content}
        </div>
      )}
    </TooltipContext.Provider>
  );
}

// Hook to use tooltip
function useTooltip() {
  const context = useContext(TooltipContext);
  if (!context) {
    throw new Error('useTooltip must be used within TooltipProvider');
  }
  return context;
}

// Usage in chart component
function ChartWithTooltip({ data }) {
  const { show, hide } = useTooltip();
  
  const handleMouseOver = (e: React.MouseEvent, d: DataPoint) => {
    show({
      x: e.clientX,
      y: e.clientY,
      content: (
        <div>
          <strong>{d.label}</strong>
          <div>Value: {d.value.toLocaleString()}</div>
        </div>
      ),
    });
  };
  
  return (
    <svg>
      {data.map(d => (
        <rect
          key={d.id}
          onMouseOver={(e) => handleMouseOver(e, d)}
          onMouseOut={hide}
          // ... other props
        />
      ))}
    </svg>
  );
}
```

### Q10: How do you handle color accessibility in charts?

```typescript
// Accessible color patterns

// 1. Use color-blind friendly palettes
const COLORBLIND_SAFE_PALETTE = [
  '#0072B2', // Blue
  '#E69F00', // Orange
  '#009E73', // Green
  '#CC79A7', // Pink
  '#F0E442', // Yellow
  '#56B4E9', // Light blue
  '#D55E00', // Red-orange
];

// 2. Combine color with patterns
function PatternDefinitions() {
  return (
    <defs>
      <pattern id="pattern-stripe" patternUnits="userSpaceOnUse" width="8" height="8">
        <line x1="0" y1="0" x2="8" y2="8" stroke="#000" strokeWidth="1" />
      </pattern>
      <pattern id="pattern-dots" patternUnits="userSpaceOnUse" width="8" height="8">
        <circle cx="4" cy="4" r="2" fill="#000" />
      </pattern>
      <pattern id="pattern-crosshatch" patternUnits="userSpaceOnUse" width="8" height="8">
        <line x1="0" y1="0" x2="8" y2="8" stroke="#000" strokeWidth="1" />
        <line x1="8" y1="0" x2="0" y2="8" stroke="#000" strokeWidth="1" />
      </pattern>
    </defs>
  );
}

// 3. High contrast mode
function useHighContrastMode() {
  const [isHighContrast, setIsHighContrast] = useState(false);
  
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-contrast: high)');
    setIsHighContrast(mediaQuery.matches);
    
    const handler = (e: MediaQueryListEvent) => setIsHighContrast(e.matches);
    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, []);
  
  return isHighContrast;
}

// 4. ARIA labels for screen readers
function AccessibleBarChart({ data }) {
  return (
    <svg role="img" aria-label="Bar chart showing monthly sales">
      <title>Monthly Sales Chart</title>
      <desc>
        A bar chart displaying sales data from January to December 2024.
        The highest sales were in December at $45,000.
      </desc>
      
      {data.map((d, i) => (
        <g key={d.month} role="listitem" aria-label={`${d.month}: $${d.value}`}>
          <rect
            x={xScale(d.month)}
            y={yScale(d.value)}
            width={xScale.bandwidth()}
            height={innerHeight - yScale(d.value)}
            fill={colorScale(i)}
            tabIndex={0}
          />
        </g>
      ))}
    </svg>
  );
}
```

### Q11: Explain how to export charts as images

```typescript
// Export SVG to various formats

async function exportChart(
  svgElement: SVGSVGElement,
  format: 'svg' | 'png' | 'pdf',
  filename: string
) {
  const svgString = new XMLSerializer().serializeToString(svgElement);
  const svgBlob = new Blob([svgString], { type: 'image/svg+xml;charset=utf-8' });
  
  if (format === 'svg') {
    // Direct SVG download
    downloadBlob(svgBlob, `${filename}.svg`);
    return;
  }
  
  if (format === 'png') {
    // Convert to PNG via canvas
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d')!;
    const img = new Image();
    
    const scale = 2; // Higher resolution
    canvas.width = svgElement.clientWidth * scale;
    canvas.height = svgElement.clientHeight * scale;
    ctx.scale(scale, scale);
    
    return new Promise<void>((resolve) => {
      img.onload = () => {
        ctx.drawImage(img, 0, 0);
        canvas.toBlob((blob) => {
          if (blob) {
            downloadBlob(blob, `${filename}.png`);
          }
          URL.revokeObjectURL(img.src);
          resolve();
        }, 'image/png');
      };
      
      img.src = URL.createObjectURL(svgBlob);
    });
  }
  
  if (format === 'pdf') {
    // Use library like jsPDF
    // Or server-side rendering
  }
}

function downloadBlob(blob: Blob, filename: string) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

// React hook for export
function useChartExport(svgRef: React.RefObject<SVGSVGElement>) {
  const exportAs = useCallback(async (
    format: 'svg' | 'png',
    filename = 'chart'
  ) => {
    if (!svgRef.current) return;
    await exportChart(svgRef.current, format, filename);
  }, [svgRef]);
  
  return { exportAs };
}
```

### Q12: How do you implement chart annotations?

```typescript
// Flexible annotation system

interface Annotation {
  id: string;
  type: 'point' | 'line' | 'area' | 'text';
  x?: number | string;
  y?: number | string;
  x1?: number | string;
  y1?: number | string;
  x2?: number | string;
  y2?: number | string;
  label?: string;
  color?: string;
}

interface AnnotationLayerProps {
  annotations: Annotation[];
  xScale: any;
  yScale: any;
  width: number;
  height: number;
}

function AnnotationLayer({ 
  annotations, 
  xScale, 
  yScale,
  width,
  height,
}: AnnotationLayerProps) {
  const getX = (value: number | string | undefined) => {
    if (value === undefined) return 0;
    if (typeof value === 'number') return xScale(value);
    return xScale(value);
  };
  
  const getY = (value: number | string | undefined) => {
    if (value === undefined) return 0;
    return yScale(value);
  };
  
  return (
    <g className="annotations">
      {annotations.map(annotation => {
        switch (annotation.type) {
          case 'point':
            return (
              <g key={annotation.id}>
                <circle
                  cx={getX(annotation.x)}
                  cy={getY(annotation.y)}
                  r={6}
                  fill={annotation.color || 'red'}
                />
                {annotation.label && (
                  <text
                    x={getX(annotation.x) + 10}
                    y={getY(annotation.y)}
                    dy="0.35em"
                    fontSize={12}
                  >
                    {annotation.label}
                  </text>
                )}
              </g>
            );
          
          case 'line':
            const isHorizontal = annotation.y !== undefined;
            return (
              <g key={annotation.id}>
                <line
                  x1={isHorizontal ? 0 : getX(annotation.x)}
                  y1={isHorizontal ? getY(annotation.y) : 0}
                  x2={isHorizontal ? width : getX(annotation.x)}
                  y2={isHorizontal ? getY(annotation.y) : height}
                  stroke={annotation.color || 'red'}
                  strokeDasharray="4 4"
                />
                {annotation.label && (
                  <text
                    x={isHorizontal ? width - 5 : getX(annotation.x) + 5}
                    y={isHorizontal ? getY(annotation.y) - 5 : 15}
                    textAnchor={isHorizontal ? 'end' : 'start'}
                    fontSize={11}
                    fill={annotation.color || 'red'}
                  >
                    {annotation.label}
                  </text>
                )}
              </g>
            );
          
          case 'area':
            return (
              <rect
                key={annotation.id}
                x={getX(annotation.x1)}
                y={getY(annotation.y2)}
                width={getX(annotation.x2) - getX(annotation.x1)}
                height={getY(annotation.y1) - getY(annotation.y2)}
                fill={annotation.color || 'yellow'}
                fillOpacity={0.2}
              />
            );
          
          case 'text':
            return (
              <text
                key={annotation.id}
                x={getX(annotation.x)}
                y={getY(annotation.y)}
                fontSize={12}
                fill={annotation.color || '#333'}
              >
                {annotation.label}
              </text>
            );
          
          default:
            return null;
        }
      })}
    </g>
  );
}
```

---

# BONUS: Testing Visualization Components

## Testing with React Testing Library

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { BarChart } from './BarChart';

describe('BarChart', () => {
  const mockData = [
    { label: 'A', value: 100 },
    { label: 'B', value: 200 },
    { label: 'C', value: 150 },
  ];
  
  it('renders correct number of bars', () => {
    render(<BarChart data={mockData} width={400} height={300} />);
    
    const bars = screen.getAllByRole('listitem');
    expect(bars).toHaveLength(3);
  });
  
  it('displays correct labels', () => {
    render(<BarChart data={mockData} width={400} height={300} />);
    
    expect(screen.getByText('A')).toBeInTheDocument();
    expect(screen.getByText('B')).toBeInTheDocument();
    expect(screen.getByText('C')).toBeInTheDocument();
  });
  
  it('shows tooltip on hover', async () => {
    render(<BarChart data={mockData} width={400} height={300} />);
    
    const firstBar = screen.getAllByRole('listitem')[0];
    fireEvent.mouseOver(firstBar);
    
    await screen.findByText('Value: 100');
  });
  
  it('handles empty data', () => {
    render(<BarChart data={[]} width={400} height={300} />);
    
    expect(screen.getByText('No data available')).toBeInTheDocument();
  });
});
```

## Testing D3 Integration

```typescript
import { renderHook, act } from '@testing-library/react';
import { useScale } from './hooks/useScale';

describe('useScale hook', () => {
  it('creates linear scale correctly', () => {
    const { result } = renderHook(() => 
      useScale('linear', [0, 100], [0, 500])
    );
    
    expect(result.current(0)).toBe(0);
    expect(result.current(50)).toBe(250);
    expect(result.current(100)).toBe(500);
  });
  
  it('updates scale when domain changes', () => {
    const { result, rerender } = renderHook(
      ({ domain }) => useScale('linear', domain, [0, 500]),
      { initialProps: { domain: [0, 100] } }
    );
    
    expect(result.current(100)).toBe(500);
    
    rerender({ domain: [0, 200] });
    
    expect(result.current(100)).toBe(250);
  });
});
```

## Snapshot Testing for SVG

```typescript
import { render } from '@testing-library/react';
import { LineChart } from './LineChart';

describe('LineChart snapshots', () => {
  const mockData = [
    { date: new Date('2024-01-01'), value: 100 },
    { date: new Date('2024-02-01'), value: 200 },
    { date: new Date('2024-03-01'), value: 150 },
  ];
  
  it('matches snapshot for basic render', () => {
    const { container } = render(
      <LineChart data={mockData} width={400} height={300} />
    );
    
    expect(container.querySelector('svg')).toMatchSnapshot();
  });
  
  it('matches snapshot with custom options', () => {
    const { container } = render(
      <LineChart 
        data={mockData} 
        width={400} 
        height={300}
        showGrid
        showDots
        curveType="monotone"
      />
    );
    
    expect(container.querySelector('svg')).toMatchSnapshot();
  });
});
```

---

# BONUS: Chart Styling Patterns

## CSS-in-JS Styling

```typescript
// Using styled-components for charts

import styled from 'styled-components';

const ChartContainer = styled.div`
  position: relative;
  width: 100%;
  background: ${props => props.theme.colors.background};
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
`;

const StyledSVG = styled.svg`
  .axis {
    font-size: 12px;
    font-family: ${props => props.theme.fonts.mono};
    
    line,
    path {
      stroke: ${props => props.theme.colors.border};
    }
    
    text {
      fill: ${props => props.theme.colors.textSecondary};
    }
  }
  
  .grid {
    line {
      stroke: ${props => props.theme.colors.border};
      stroke-opacity: 0.5;
      stroke-dasharray: 2, 4;
    }
    
    .domain {
      display: none;
    }
  }
  
  .bar {
    fill: ${props => props.theme.colors.primary};
    transition: fill 0.2s ease;
    
    &:hover {
      fill: ${props => props.theme.colors.primaryDark};
    }
  }
  
  .line {
    fill: none;
    stroke: ${props => props.theme.colors.primary};
    stroke-width: 2;
    stroke-linejoin: round;
    stroke-linecap: round;
  }
`;

const Tooltip = styled.div`
  position: absolute;
  background: ${props => props.theme.colors.tooltipBackground};
  color: ${props => props.theme.colors.tooltipText};
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 13px;
  pointer-events: none;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 10;
  
  &::before {
    content: '';
    position: absolute;
    bottom: -6px;
    left: 50%;
    transform: translateX(-50%);
    border-left: 6px solid transparent;
    border-right: 6px solid transparent;
    border-top: 6px solid ${props => props.theme.colors.tooltipBackground};
  }
`;
```

## Theme Support

```typescript
// Light and dark theme for charts

interface ChartTheme {
  background: string;
  foreground: string;
  grid: string;
  text: string;
  textSecondary: string;
  colors: string[];
}

const lightTheme: ChartTheme = {
  background: '#ffffff',
  foreground: '#000000',
  grid: '#e5e5e5',
  text: '#333333',
  textSecondary: '#666666',
  colors: [
    '#6366f1', '#8b5cf6', '#ec4899', '#f43f5e', 
    '#f97316', '#eab308', '#22c55e', '#14b8a6'
  ],
};

const darkTheme: ChartTheme = {
  background: '#1f2937',
  foreground: '#ffffff',
  grid: '#374151',
  text: '#f3f4f6',
  textSecondary: '#9ca3af',
  colors: [
    '#818cf8', '#a78bfa', '#f472b6', '#fb7185',
    '#fb923c', '#facc15', '#4ade80', '#2dd4bf'
  ],
};

const ChartThemeContext = createContext<ChartTheme>(lightTheme);

function useChartTheme() {
  return useContext(ChartThemeContext);
}

// Usage
function ThemedBarChart({ data }) {
  const theme = useChartTheme();
  
  return (
    <svg style={{ background: theme.background }}>
      {data.map((d, i) => (
        <rect
          key={d.id}
          fill={theme.colors[i % theme.colors.length]}
          // ... other props
        />
      ))}
    </svg>
  );
}
```

---

# BONUS: Production Considerations

## Error Boundaries for Charts

```typescript
import { Component, ErrorInfo, ReactNode } from 'react';

interface ChartErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ChartErrorBoundary extends Component<
  { children: ReactNode; fallback?: ReactNode },
  ChartErrorBoundaryState
> {
  state: ChartErrorBoundaryState = {
    hasError: false,
    error: null,
  };
  
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Chart error:', error, errorInfo);
    // Report to error tracking service
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="chart-error">
          <p>Unable to render chart</p>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Retry
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
<ChartErrorBoundary fallback={<ChartFallback />}>
  <LineChart data={data} />
</ChartErrorBoundary>
```

## Loading States

```typescript
function ChartSkeleton({ width, height }: { width: number; height: number }) {
  return (
    <svg width={width} height={height} className="chart-skeleton">
      <defs>
        <linearGradient id="skeleton-gradient" x1="0%" y1="0%" x2="100%" y2="0%">
          <stop offset="0%" stopColor="#e5e5e5">
            <animate
              attributeName="offset"
              values="-2; 1"
              dur="1.5s"
              repeatCount="indefinite"
            />
          </stop>
          <stop offset="50%" stopColor="#f0f0f0">
            <animate
              attributeName="offset"
              values="-1; 2"
              dur="1.5s"
              repeatCount="indefinite"
            />
          </stop>
          <stop offset="100%" stopColor="#e5e5e5">
            <animate
              attributeName="offset"
              values="0; 3"
              dur="1.5s"
              repeatCount="indefinite"
            />
          </stop>
        </linearGradient>
      </defs>
      
      {/* Skeleton bars */}
      {Array.from({ length: 5 }, (_, i) => (
        <rect
          key={i}
          x={30 + i * 70}
          y={50 + Math.random() * 100}
          width={50}
          height={height - 100 - Math.random() * 100}
          fill="url(#skeleton-gradient)"
          rx={4}
        />
      ))}
    </svg>
  );
}

// Usage with Suspense
function ChartWithSuspense() {
  return (
    <Suspense fallback={<ChartSkeleton width={400} height={300} />}>
      <AsyncChart />
    </Suspense>
  );
}
```

---

**End of Part 10: Data Visualization Complete Guide**

Total Sections: 23 (including bonus)
Key Topics: SVG, D3.js, Recharts, Canvas, Performance, Advanced Patterns
Interview Ready: ✅

---

# BONUS: Geographic Visualization

## Working with Maps

```typescript
// GeoJSON basics
interface GeoJSONFeature {
  type: 'Feature';
  geometry: {
    type: 'Polygon' | 'MultiPolygon' | 'Point' | 'LineString';
    coordinates: number[][] | number[][][] | number[][][][];
  };
  properties: {
    name: string;
    id: string;
    [key: string]: any;
  };
}

interface GeoJSONCollection {
  type: 'FeatureCollection';
  features: GeoJSONFeature[];
}

// D3 Projections
const PROJECTIONS = {
  mercator: d3.geoMercator(),      // Web maps standard
  albers: d3.geoAlbersUsa(),       // Great for USA
  orthographic: d3.geoOrthographic(), // Globe view
  equalEarth: d3.geoEqualEarth(),  // Area-accurate
  natural: d3.geoNaturalEarth1(),  // Good for world maps
};

// Create projection and path generator
function useGeoProjection(geoJSON: GeoJSONCollection, width: number, height: number) {
  return useMemo(() => {
    const projection = d3.geoMercator()
      .fitSize([width, height], geoJSON);
    
    const pathGenerator = d3.geoPath().projection(projection);
    
    return { projection, pathGenerator };
  }, [geoJSON, width, height]);
}
```

## Interactive Map Component

```typescript
interface InteractiveMapProps {
  geoJSON: GeoJSONCollection;
  data: Record<string, number>;
  width: number;
  height: number;
  onRegionClick?: (region: GeoJSONFeature) => void;
  colorScheme?: 'blue' | 'green' | 'red' | 'purple';
}

function InteractiveMap({
  geoJSON,
  data,
  width,
  height,
  onRegionClick,
  colorScheme = 'blue',
}: InteractiveMapProps) {
  const [hoveredRegion, setHoveredRegion] = useState<string | null>(null);
  const [tooltip, setTooltip] = useState<{ x: number; y: number; content: string } | null>(null);
  
  const { projection, pathGenerator } = useGeoProjection(geoJSON, width, height);
  
  // Color scales for different schemes
  const colorScales = {
    blue: d3.interpolateBlues,
    green: d3.interpolateGreens,
    red: d3.interpolateReds,
    purple: d3.interpolatePurples,
  };
  
  const values = Object.values(data);
  const colorScale = d3.scaleSequential(colorScales[colorScheme])
    .domain([Math.min(...values), Math.max(...values)]);
  
  const handleMouseMove = (e: React.MouseEvent, feature: GeoJSONFeature) => {
    const value = data[feature.properties.id];
    setTooltip({
      x: e.clientX,
      y: e.clientY,
      content: `${feature.properties.name}: ${value?.toLocaleString() || 'N/A'}`,
    });
  };
  
  const handleMouseLeave = () => {
    setHoveredRegion(null);
    setTooltip(null);
  };
  
  return (
    <div style={{ position: 'relative' }}>
      <svg width={width} height={height}>
        <g>
          {geoJSON.features.map((feature) => {
            const value = data[feature.properties.id];
            const isHovered = hoveredRegion === feature.properties.id;
            
            return (
              <path
                key={feature.properties.id}
                d={pathGenerator(feature) || ''}
                fill={value !== undefined ? colorScale(value) : '#e5e5e5'}
                stroke={isHovered ? '#000' : '#fff'}
                strokeWidth={isHovered ? 2 : 0.5}
                onMouseEnter={() => setHoveredRegion(feature.properties.id)}
                onMouseMove={(e) => handleMouseMove(e, feature)}
                onMouseLeave={handleMouseLeave}
                onClick={() => onRegionClick?.(feature)}
                style={{ cursor: onRegionClick ? 'pointer' : 'default' }}
              />
            );
          })}
        </g>
      </svg>
      
      {tooltip && (
        <div
          style={{
            position: 'fixed',
            left: tooltip.x + 15,
            top: tooltip.y + 15,
            background: 'white',
            padding: '8px 12px',
            border: '1px solid #ccc',
            borderRadius: 4,
            boxShadow: '0 2px 8px rgba(0,0,0,0.15)',
            pointerEvents: 'none',
            zIndex: 1000,
          }}
        >
          {tooltip.content}
        </div>
      )}
    </div>
  );
}
```

## Zoom and Pan for Maps

```typescript
import { zoom, zoomIdentity } from 'd3-zoom';

function ZoomablePannableMap({ geoJSON, width, height }: MapProps) {
  const svgRef = useRef<SVGSVGElement>(null);
  const gRef = useRef<SVGGElement>(null);
  
  const { pathGenerator } = useGeoProjection(geoJSON, width, height);
  
  useEffect(() => {
    const svg = d3.select(svgRef.current);
    const g = d3.select(gRef.current);
    
    const zoomBehavior = zoom<SVGSVGElement, unknown>()
      .scaleExtent([1, 8])
      .on('zoom', (event) => {
        g.attr('transform', event.transform.toString());
      });
    
    svg.call(zoomBehavior);
    
    // Reset button functionality
    const resetZoom = () => {
      svg.transition()
        .duration(750)
        .call(zoomBehavior.transform, zoomIdentity);
    };
    
    return () => {
      svg.on('.zoom', null);
    };
  }, []);
  
  return (
    <div>
      <button onClick={() => {
        const svg = d3.select(svgRef.current);
        svg.transition().duration(750).call(
          zoom<SVGSVGElement, unknown>().transform as any,
          zoomIdentity
        );
      }}>
        Reset Zoom
      </button>
      
      <svg ref={svgRef} width={width} height={height}>
        <g ref={gRef}>
          {geoJSON.features.map((feature) => (
            <path
              key={feature.properties.id}
              d={pathGenerator(feature) || ''}
              fill="#69b3a2"
              stroke="#fff"
              strokeWidth={0.5}
            />
          ))}
        </g>
      </svg>
    </div>
  );
}
```

---

# BONUS: Study Resources

## Books to Read

```typescript
const RECOMMENDED_BOOKS = {
  fundamentals: [
    {
      title: 'Interactive Data Visualization for the Web',
      author: 'Scott Murray',
      focus: 'D3.js fundamentals',
      level: 'Beginner to Intermediate',
    },
    {
      title: 'D3.js in Action',
      author: 'Elijah Meeks',
      focus: 'Advanced D3 patterns',
      level: 'Intermediate to Advanced',
    },
    {
      title: 'The Visual Display of Quantitative Information',
      author: 'Edward Tufte',
      focus: 'Data visualization principles',
      level: 'All levels',
    },
  ],
  
  design: [
    {
      title: 'Storytelling with Data',
      author: 'Cole Nussbaumer Knaflic',
      focus: 'Effective visualization',
      level: 'All levels',
    },
    {
      title: 'Information Dashboard Design',
      author: 'Stephen Few',
      focus: 'Dashboard best practices',
      level: 'Intermediate',
    },
  ],
};
```

## Online Resources

```typescript
const ONLINE_RESOURCES = {
  documentation: [
    'https://d3js.org/',
    'https://recharts.org/',
    'https://developer.mozilla.org/en-US/docs/Web/SVG',
  ],
  
  tutorials: [
    'https://observablehq.com/@d3/learn-d3',
    'https://www.d3-graph-gallery.com/',
    'https://fullstackopen.com/en/',
  ],
  
  examples: [
    'https://observablehq.com/',
    'https://bl.ocks.org/',
    'https://github.com/d3/d3/wiki/Gallery',
  ],
  
  tools: [
    'RAWGraphs - visual data conversion',
    'Flourish - no-code visualizations',
    'Datawrapper - chart creation',
  ],
};
```

## Interview Preparation Timeline

```typescript
const PREP_TIMELINE = {
  week1: {
    focus: 'SVG and D3 basics',
    topics: [
      'SVG coordinate system',
      'Basic shapes and paths',
      'D3 selections',
      'Data binding basics',
    ],
    practice: 'Build 3 simple charts from scratch',
  },
  
  week2: {
    focus: 'D3 scales and axes',
    topics: [
      'All scale types',
      'Axis generation',
      'Color scales',
      'Time formatting',
    ],
    practice: 'Build time series chart with proper axes',
  },
  
  week3: {
    focus: 'React integration',
    topics: [
      'D3 + React approaches',
      'Custom hooks for D3',
      'Responsive charts',
      'Animation patterns',
    ],
    practice: 'Build interactive dashboard component',
  },
  
  week4: {
    focus: 'Advanced topics',
    topics: [
      'Canvas for performance',
      'Large dataset handling',
      'Accessibility',
      'Testing charts',
    ],
    practice: 'Optimize existing charts, add tests',
  },
};
```

---

# BONUS: Quick Interview Tips

## Common Mistakes to Avoid

```typescript
const COMMON_MISTAKES = [
  {
    mistake: 'Not inverting Y scale',
    correct: `.range([height, 0]) // Y axis goes up, not down`,
  },
  {
    mistake: 'Forgetting key function in data binding',
    correct: `.data(data, d => d.id) // Use unique identifier`,
  },
  {
    mistake: 'Not handling empty data',
    correct: `if (data.length === 0) return <EmptyState />`,
  },
  {
    mistake: 'Using SVG for large datasets',
    correct: '// Switch to Canvas for > 5000 elements',
  },
  {
    mistake: 'Not cleaning up D3 effects',
    correct: `useEffect(() => { ... return () => cleanup(); }, [])`,
  },
  {
    mistake: 'Hardcoded dimensions',
    correct: '// Always use ResponsiveContainer or ResizeObserver',
  },
  {
    mistake: 'Missing accessibility',
    correct: '// Add ARIA labels, title, desc elements',
  },
];
```

## Key Patterns to Remember

```typescript
const KEY_PATTERNS = {
  // Scale pattern
  scale: `
    const scale = d3.scaleLinear()
      .domain([min, max])
      .range([0, size])
      .nice();
  `,
  
  // Axis pattern
  axis: `
    const axis = d3.axisBottom(scale)
      .ticks(5)
      .tickFormat(d => format(d));
    
    svg.append('g')
      .call(axis);
  `,
  
  // Data binding pattern
  binding: `
    svg.selectAll('rect')
      .data(data, d => d.id)
      .join(
        enter => enter.append('rect'),
        update => update.attr('fill', 'green'),
        exit => exit.remove()
      );
  `,
  
  // React integration pattern
  react: `
    // D3 for math, React for rendering
    const scale = useMemo(() => 
      d3.scaleLinear().domain(domain).range(range),
      [domain, range]
    );
    
    return (
      <svg>
        {data.map(d => (
          <rect x={scale(d.x)} ... />
        ))}
      </svg>
    );
  `,
};
```

## One-Liner Tips

```typescript
const ONE_LINERS = [
  // Scale tips
  'Use .nice() on scales for cleaner tick values',
  'Invert Y scale range: [height, 0] not [0, height]',
  'Use scaleBand for categorical axes with padding',
  
  // Axis tips
  'tickSize(-width) creates horizontal grid lines',
  'Remove domain by selecting .domain and calling .remove()',
  
  // Performance tips
  'Canvas for > 5000 elements, SVG for < 1000',
  'Memoize scales and generators',
  'Debounce resize handlers at 100-200ms',
  
  // Animation tips
  'Disable animations during resize',
  'Use requestAnimationFrame for custom animations',
  'transition().duration(500) for smooth updates',
  
  // Accessibility tips
  'Add title and desc elements to SVG',
  'Use role="img" and aria-label',
  'Combine color with patterns for colorblind users',
  
  // Testing tips
  'Snapshot test SVG output',
  'Test scales with explicit values',
  'Mock ResizeObserver in tests',
];
```

---

# BONUS: Complete Chart Library Implementation

## Reusable Chart Components Library

```typescript
// Chart configuration types
interface ChartConfig {
  margin?: { top: number; right: number; bottom: number; left: number };
  colors?: string[];
  animate?: boolean;
  animationDuration?: number;
  showGrid?: boolean;
  showLegend?: boolean;
  showTooltip?: boolean;
}

// Default configuration
const DEFAULT_CONFIG: ChartConfig = {
  margin: { top: 20, right: 20, bottom: 40, left: 50 },
  colors: ['#6366f1', '#8b5cf6', '#ec4899', '#f43f5e', '#f97316'],
  animate: true,
  animationDuration: 500,
  showGrid: true,
  showLegend: true,
  showTooltip: true,
};

// Chart context for shared configuration
const ChartConfigContext = createContext<ChartConfig>(DEFAULT_CONFIG);

function ChartProvider({ 
  config, 
  children 
}: { 
  config?: Partial<ChartConfig>; 
  children: React.ReactNode;
}) {
  const mergedConfig = { ...DEFAULT_CONFIG, ...config };
  
  return (
    <ChartConfigContext.Provider value={mergedConfig}>
      {children}
    </ChartConfigContext.Provider>
  );
}

function useChartConfig() {
  return useContext(ChartConfigContext);
}

// Base chart wrapper
interface BaseChartProps {
  data: any[];
  width?: number;
  height?: number;
  config?: Partial<ChartConfig>;
  className?: string;
}

function BaseChart({
  data,
  width: propWidth,
  height: propHeight,
  config,
  className,
  children,
}: BaseChartProps & { children: (props: any) => React.ReactNode }) {
  const containerRef = useRef<HTMLDivElement>(null);
  const dimensions = useDimensions(containerRef);
  
  const width = propWidth || dimensions.width;
  const height = propHeight || dimensions.height;
  
  const chartConfig = useChartConfig();
  const mergedConfig = { ...chartConfig, ...config };
  
  const { margin } = mergedConfig;
  const innerWidth = width - margin!.left - margin!.right;
  const innerHeight = height - margin!.top - margin!.bottom;
  
  if (!width || !height || innerWidth <= 0 || innerHeight <= 0) {
    return (
      <div ref={containerRef} className={className} style={{ width: '100%', height: 300 }}>
        <ChartSkeleton width={width || 400} height={height || 300} />
      </div>
    );
  }
  
  return (
    <div ref={containerRef} className={className}>
      {children({
        data,
        width,
        height,
        innerWidth,
        innerHeight,
        margin: margin!,
        config: mergedConfig,
      })}
    </div>
  );
}
```

## Usage Example

```typescript
// Complete usage example
function DashboardPage() {
  const salesData = useSalesData();
  const revenueData = useRevenueData();
  const categoryData = useCategoryData();
  
  return (
    <ChartProvider config={{ animate: true, showGrid: true }}>
      <div className="dashboard">
        <div className="chart-row">
          <BaseChart data={salesData} height={300}>
            {(props) => <LineChartContent {...props} />}
          </BaseChart>
          
          <BaseChart data={revenueData} height={300}>
            {(props) => <BarChartContent {...props} />}
          </BaseChart>
        </div>
        
        <div className="chart-row">
          <BaseChart data={categoryData} height={400}>
            {(props) => <PieChartContent {...props} />}
          </BaseChart>
        </div>
      </div>
    </ChartProvider>
  );
}
```

---

**🎉 COMPLETE: Part 10 - Data Visualization Guide**

This comprehensive guide covers everything you need to know about data visualization in frontend development, from fundamental SVG concepts to advanced D3.js patterns, Recharts implementation, performance optimization, and real-world interview preparation.

**Total Coverage:**
- 25+ Sections
- SVG Fundamentals
- D3.js Complete Guide
- Recharts Component Library
- Canvas vs SVG Analysis
- Performance Optimization
- 15+ Interview Questions
- 10+ Coding Challenges
- Advanced Visualization Patterns
- Testing Strategies
- Production Considerations

**Ready for Interview: ✅**



