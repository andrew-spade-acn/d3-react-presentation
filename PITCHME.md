## `D3 <> React`
#### Optimizing for Developer Experience (DX) and Performance

---

### What is DX?

DX is about what developers _think_ and _feel_ about their coding environments.

An attempted definition: 
> A metric for a tool that provides some __functionality__, emphasizing __stability__, __usability__, and __clarity__

+++

#### Functionality
* What does it **do**? Does it do that thing **well**?
  * Tiny: Cross-platform console colors with **`Chalk`**
  * Big: A new approach to writing JS with **`TypeScript`**

+++

#### Stability
* How **reliable** are the features? How often do **unpredictable** issues surface?
  * **`AWS EC2`**'s 99.99% uptime
  * **`Lodash`** test coverage => 5382.3 hits per line

+++

#### Usability
* Does it play nice with the rest of my workflow? How hard is it to set up? Will I be locked in to anything?
  * Time to make a Todo App in **`Angular`** vs **`React`**.
  * Time spent reading documentation vs developing. _`*`cough`*`, webpack..._
 
+++

#### Clarity
* Does it make sense? Do I understand why it exists? Does it follow common conventions?
  * Difficulty of learning **`Redux`** vs **`RxJS`**
  * Predictability of transpiled **`CoffeeScript`** vs **`TypeScript`**

---

### D3 

* A library for creating **data-driven visualizations**
* Often used for making charts, but not explicitly a charting library
* **Declarative abstractions** for clunky SVG syntax + DOM methods
* Mediocre DX

+++

#### Raw HTML
```html
  <line x1="0" y1="0" x2="200" y2="200"/>
```

#### D3
```javascript
d3.svg.line()
  .x( (d) => (d.x) )
  .y( (d) => (d.y) );
```

---

### React 

* A declarative framework for building components, composed together for complex UIs
* Easy to learn, supported by **JSX** and **component lifecycle** methods
* Super fast, leveraging the **virtual DOM** and a powerful **reconciliation** algorithm
* High quality DX

+++
#### React: 
```jsx
const MyDiv = styled.div`
  font-size: 16px;
  color: blue;
`;

const greet = (name) => {
  return `Hello, ${name}!`;
};

export default Component = () => (
  <MyDiv>
    {greet('friends')}
  </MyDiv>
);
```
#### HTML: 
```html
<div style='font-size:16px;color:blue;'>
  Hello, friends!
</div>
```
---
#### Why would we want to combine them?

* D3 examples tend to be standalone apps (or stuck in `<script>` tags), limiting its scope
* Vanilla D3 struggles when handling multiple complex, interconnected visualizations
* If combined elegantly, D3 and React becomes a stateless, declarative, component-driven, data viz toolkit!

---
#### What's the problem?

* React and D3 each want to render their own DOM elements
* D3 is stateful and relies on DOM mutation
* React's best assets require handling rendering inside components
* Similarly, many great developer utilities work best inside React, like Hot Module Replacement, Hot Reloading, Time Travel Debugging

---
#### How can we combine them?

1. The D3 Purist
2. The React Runaround
3. The DOM Emulator
---
#### The D3 Purist
```jsx
export class CircleD3 extends Component {
  componentDidMount() {
    // We can hook this up to Redux/Promises/RxJS/etc
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
    // We can have these props passed in externally
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
    const data = fetchData();
    const svg = makeSVG('renderedD3', 100, 100); 
    // We can break the D3 segments into separate 'elements' or 'partials'
    makeCircles(svg, data);
  }
  render() {
    return (<div className='renderedD3' />)
  }
}
```
+++

Pros:
* It's fast and simple
* It allows writing normal D3 code, making it easy to code from example

Cons:
* Where's the React?
* It results in two separate development worlds
* Not simple to set up HMR / TTD
* D3 parts are detached from the Redux store, component props, and app router
* Clunky integration with lifecycle hooks
---
#### The React Runaround
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
#### The React Runaround
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
#### The React Runaround
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
#### The React Runaround
```jsx
export class CircleReact extends Component {
  render() {
    /* MakeGrid is a HOC to abstract away the nested d3.range parts */
    const CircleGrid = MakeGrid(Dot);
     
    return (
      <SVG width={100} height={100}>
        <Group>
          <CircleGrid x={20} y={20} r={2} />
        </Group>
      </SVG>
    )
  }
}
```
+++

Pros:
* This looks like React!
* HOCs are a great pattern for enhancing components
* Can break everything into smaller components, leading to modular, reusable code
* HMR, TTD, and lifecycle hooks!

Cons:
* Performance degrades at scale. Can result in 100+ (or even 1000+) nested components _per chart_
* No clear, clean way to handle D3's `enter`/`exit`/`update`
* Syntax deviates from D3, causing extra work and awkward syntax
---
#### The DOM Emulator

What if we could trick D3 into thinking it's rendering DOM elements?

+++
#### The DOM Emulator

We'll take the first example:
```jsx
export class CircleD3 extends Component {
  componentDidMount() {
    const data = fetchData();
    const svg = makeSVG('renderedD3', 100, 100); 
    makeCircles(svg, data);
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
    const faux = this.props.connectFauxDOM('div', 'chart'); // 1
    const data = fetchData();
    const svg = makeSVG(faux, 100, 100); 
    makeCircles(svg, data);
  }
  render() {
    return (
      <div className='renderedD3'>
        {this.props.chart} // 2
      </div>
    )
  }
}
return withFauxDOM(DotChartTwo); // 3
```
+++

Pros:
* We get to keep vanilla D3!
* We're rendering in React!
* We get to keep HMR, TTD, component lifecycle hooks!
  * We can even have stateless componenets
* Stress testing 100k circles, this renders 5x faster than approach #2
  * Further, the heavy lifting happens outside of `render`, meaning the rest of the app is not blocked

+++ 

Cons:
* Slight performance implication due to overhead of DOM emulation
* Animations are functional, but timings need work
---
#### Show performance stats?
---
#### What's the best approach?

A hybrid solution:
1. For demanding animations or extremely complex visualizations: **D3 in Canvas**
  * 1000+ animated or interactive elements on screen
2. For visualizations with low complexity: **React**
  * Static charts, with simple click states
3. For feature rich visualizations with complex requiremnts: **DOM Emulation**
  * Animated charts, connected to live data stores, with user interaction
---
#### Future Work

* Offload heavier processes to web workers to free up main thread
* Improve animations
* Identify a consistent, scalable pattern for `enter`/`exit`/`update`
---
#### Any questions?
