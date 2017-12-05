## `D3 <> React`
#### Optimizing for Developer Experience (DX) and Performance

---

### What is DX?

DX is defined by _robust functionality_ that delivers _stability_, _usability_, and _clarity_

+++

Functionality
* What does the thing do?
  * Cross-platform console colors with `Chalk`
  * A new approach to writing JS with `TypeScript`

Stability
* How reliable are the features? How often do unpredictable bugs surface?
  * `AWS EC2`'s 99.99% uptime
  * `Lodash` test coverage => 5382.3 hits per line

+++

Usability
* Does it play nice with other tools? How long does it take to set up?
  * Time to make a Todo App in `Angular` vs `React`.
  * Time spent reading documentation vs developing.
 
Clarity
* Does it make sense? Do I understand why it exists? Does it follow conventions?
  * Difficulty of learning `Redux` vs `RxJS`
  * Predictability of compiled `CoffeeScript`

---

### D3 

* Visualization toolkit for creating data-driven docs
* Often used for making charts, but not explicitly a charting library
* Declarative abstractions for clunky SVG syntax + DOM methods

+++

Raw HTML
```html
  <line x1="0" y1="0" x2="200" y2="200"/>
```

D3
```javascript
d3.svg.line()
  .x((d) => (d.x))
  .y((d) => (d.y));
```
```jsx
d3.svg.line()
  .x((d) => (d.x))
  .y((d) => (d.y));
```

---

### React 

* A declarative framework for building components, composed together for complex UIs
* Easy to learn, supported by JSX and component lifecycle methods
* Super fast, via the virtual DOM and a powerful reconciliation algorithm

+++
In React: 
```jsx
const greet = (name) => {
  return `Hello, ${name}!`
}

const Component = (
  <h1>
    {greet('friends')}
  </h1>
);

ReactDOM.render(
  Component,
  document.getElementById('root')
);
```
In HTML: 
```html
<h1>
  Hello, friends!
</h1>
```
---
#### What's the best way to combine them?

1. The D3 Purist
2. The React Holdout
3. The DOM Emulator
---
#### The D3 Purist
```jsx
export class CircleD3 extends Component {
  componentDidMount() {
    const data = fetchData();  
  }  
  render() {
    return (<div className='renderedD3' />)
  }
}
```
+++
#### The D3 Purist
```jsx
export class CircleD3 extends Component {
  componentDidMount() {
    const data = fetchData();  
    const svg = d3.select('.renderedD3')
      .append('svg')
      .attr('width', 100)
      .attr('height', 100)
      .append('g');
  }  
  render() {
    return (<div className='renderedD3' />)
  }
}
```
+++
#### The D3 Purist
```jsx
export class CircleD3 extends Component {
  componentDidMount() {  
    const data = fetchData();  
    const svg = makeSVG('renderedD3', 100, 100);   
  }
  render() {
    return (<div className='renderedD3' />)
  }
}
```
+++ 
#### The D3 Purist
```jsx
export class CircleD3 extends Component {
  componentDidMount() {  
    const data = fetchData();  
    const svg = makeSVG('renderedD3', 100, 100);  
    svg.selectAll('circle')
      .data(data)
      .enter()
      .append('circle')
      .attr('cx', (d) => (d.x))
      .attr('cy', (d) => (d.y))
      .attr('r', 2)
      .attr('fill', () => ( getColor ))   
  }
  render() {
    return (<div className='renderedD3' />)
  }
}
```
+++ 
#### The D3 Purist
```jsx
export class CircleD3 extends Component {
  componentDidMount() {
    // We can hook this up to Redux/Promises/RxJS/etc
    const data = fetchData();
    // We can have these props passed in externally
    const svg = makeSVG('renderedD3', 100, 100); 
    makeCircles(svg);
  }
  render() {
    return (<div className='renderedD3' />)
  }
}
```
+++
Pros:
* It's fast and simple
* It allows writing normal D3 code

Cons:
* Is this even React?
* It results in two separate development worlds
* Not simple to set up with Hot-Module Replacement (HMR) + Time Travel Debugging (TTD)
* D3 is detached from the Redux store, component props, and app router
* Clunky integration with lifecycle hooks
---
#### The React Way
```jsx
export class CircleReact extends Component {
  render() {
    return (
      <svg width={100} height={100}>
        <g>
        </g>
      </svg>
    )
  }
}
```
+++
#### The React Way
```jsx
export class CircleReact extends Component {
  render() {
    return (
      <svg width={100} height={100}>
        <g>
          {d3.range(20).map(x =>
            d3.range(20).map(y =>
              <circle cx={x} cy={y} r={2}
                      ref="dot"
                      style={{fill: getColor}} />
            )
          )}
        </g>
      </svg>
    )
  }
}
```
+++
#### The React Way
```jsx
export class CircleReact extends Component {
  render() {
    return (
      /* Pure React Components to wrap svg, circle, and g */
      <SVG width={100} height={100}>
        <G>
          {d3.range(20).map(x =>
            d3.range(20).map(y =>
              <Circle cx={x} cy={y} r={2}
                      ref="dot"
                      style={{fill: getColor}} />
            )
          )}
        </G>
      </SVG>
    )
  }
}
```
+++
#### The React Way
```jsx
export class CircleReact extends Component {
  render() {
    /* RangeGrid is a HOC to abstract away the nested d3.range parts */
    const RangeDot = RangeGrid(Dot);
     
    return (
      <SVG width={100} height={100}>
        <Group>
          <RangeDot x={20} y={20} r={2} />
        </Group>
      </SVG>
    )
  }
}
```
+++
Pros:
* This looks like React!
* Can break everything into its own components, leading to more modular code
* HMR, TTD, and lifecycle hooks!

Cons:
* Performance degrades at scale. Can easily result in 100+ (or even 1000+) nested React Components per chart
* No clear, clean way to handle D3's `enter`/`exit`/`update`
* Syntax deviates from D3, causing frustration and awkward syntax (d3.axis is extra rough)
---
+++
#### The DOM Emulator

We'll the first example:
```jsx
export class CircleD3 extends Component {
  componentDidMount() {
    const data = fetchData();
    const svg = makeSVG('renderedD3', 100, 100); 
    makeCircles(svg);
  }
  render() {
    return (<div className='renderedD3' />)
  }
}
```
+++
#### The DOM Emulator
```jsx
export class CircleD3 extends Component {
  componentDidMount() {
    const faux = this.props.connectFauxDOM('div', 'chart');
    const data = fetchData();
    const svg = makeSVG(faux, 100, 100); 
    makeCircles(svg);
  }
  render() {
    return (
      <div className='renderedD3'>
        {this.props.chart}
      </div>
    )
  }
}
return withFauxDOM(DotChartTwo);
```
+++
Pros:
* We get to keep vanilla D3!
* We get to keep HMR, component lifecycle hooks, time-travel debugging, etc
* Stress testing 100k circles, this rendered 5x faster than approach #2
  * Further, the heavy lifting happens outside of `render`, meaning the app is not blocked

Cons:
* Slight performance implication due to overhead of DOM emulation
* Animations are functional, but timings need work
---
Future Work:
* Offload heavier processes to web workers to free up main thread
* Fix animations
* Identify consistent pattern (through lifecycle hooks) for `enter`/`exit`/`update`
---
#### Any questions?
