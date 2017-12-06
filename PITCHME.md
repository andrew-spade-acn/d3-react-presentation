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
#### Objective

Determine a method for building complex data visualizations with an emphasis on scalability and maintainability (and a great developer experience!)

Enter: D3 and React

---

### D3 

* A library for creating **data-driven visualizations**
* Often used for making charts, but not explicitly a charting library
* **Declarative abstractions** for clunky SVG syntax + DOM methods
* Mediocre DX

+++

#### D3
```javascript
const data = _.flatten(d3.range(10).map(x =>
  d3.range(10).map(y => [x, y])
));

const svg = d3.select('body')
  .append('svg')
  .attr('width', 200)
  .attr('height', 200)
  .append('g');

svg.selectAll('circle')
  .data(data)
  .enter()
  .append('circle')
  .attr('cx', (d) => (d[0]))
  .attr('cy', (d) => (d[1]))
  .attr('r', 2)
  .attr('fill', 'blue')
```

+++
#### Raw HTML
```html
<svg width='200' height='200'>
  <g>
    <circle cx='0' cy=0' r='2' fill='blue' />
    <circle cx='20' cy=0' r='2' fill='blue' />
    ...
  </g>
</svg
```
![Dots](https://i.imgur.com/7M48Amm.png)

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
    // Pulling this functionality out lets us have
    // D3 sections in separate 'elements' or 'partials'
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
      // SVG, G, Circle are simple wrapper components
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
    // MakeGrid is a HOC* to abstract away 
    // the nested d3.range calls
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
*
```jsx
// HOCs are functions that take a component and return an
// enhanced component, usually with some new functionality
const HOC = (component) => (newComponent);

const enhancedComponent =  HOC(myComponent);
```
+++

Pros:
* This looks like React!
* HOCs are a great pattern for enhancing components
* Can break into smaller components
* HMR, TTD, and lifecycle hooks!

Cons:
* Performance degrades at scale. Can result in 1000+ components _per chart_
* No clear way to handle `enter`/`exit`/`update`
* Deviates from D3 => extra work and awkward syntax
+++
#### 1000+ components? How?!

This is what a _super simple_ chart looks like, with axes and labels. 
It has 66 unique visual elements!

![wow](https://raw.githubusercontent.com/liufly/Dual-scale-D3-Bar-Chart/master/preview/thumbnail.png)

+++

#### Okay, but how does that lead to 1000, in a _single chart_?

+++
![muchwow](https://i.imgur.com/FFRNhsa.png)
---
#### The DOM Emulator

What if we could trick D3 into thinking that it's rendering DOM elements? 

We could then hand off the true rendering responsibilities to React.

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
  componentDidMount() {
    const data = fetchData();
    
    // Here's our problem!
    const svg = makeSVG('renderedD3', 100, 100); 
    
    makeCircles(svg, data);
  }
```
+++
#### The DOM Emulator

```jsx
componentDidMount() {
    
  // We'll make a fake <div> element
  // And assign it to this.props.chart
  const faux = this.props.connectFauxDOM('div', 'chart');
  
  const data = fetchData();
    
  // And pass that into makeSVG
  const svg = makeSVG(faux, 100, 100); 
  
  makeCircles(svg, data);
}
```
+++
#### The DOM Emulator
```jsx
render() {
  return (
    <div className='renderedD3'>
      // Then we'll render that prop
      {this.props.chart}
    </div>
  )
}
```
+++
#### The DOM Emulator
```jsx
// And wrap CircleD3 in a HOC, to attach the faux DOM methods
return withFauxDOM(CircleD3);
```
+++
#### The DOM Emulator
```jsx
export class CircleD3 extends Component {
  componentDidMount() {
    const faux = this.props.connectFauxDOM('div', 'chart'); // 1
    const data = fetchData();
    const svg = makeSVG(faux, 100, 100); // 2
    makeCircles(svg, data);
  }
  render() {
    return (
      <div className='renderedD3'>
        {this.props.chart} // 3
      </div>
    )
  }
}
return withFauxDOM(CircleD3); // 4
```
+++

Pros:
* We get to keep vanilla D3!
* We're rendering in React!
* We get to keep HMR, TTD, component lifecycle hooks!
  * We can even have stateless componenets
* Stress testing 100k circles, this renders ~5x faster than approach #2
  * The heavy lifting happens outside of `render`, so the rest of the app is not blocked

+++ 

Cons:
* Slight performance implication due to overhead of DOM emulation
* Animations are functional, but timings need work
---
#### Performance Stats

Using `Chrome 62.0.3202.94`

| 10k Circles     | FMP    | Ready  | % of React |
|-----------------|--------|--------|------------|
| D3 Purist       | 660ms  | 717ms  | 15.04%     |
| React Runaround | 4765ms | 4765ms | N/A        |
| DOM Emulator    | 600ms  | 1225ms | 25.7%      |


| 100k Circles    | FMP     | Ready   | % of React |
|-----------------|---------|---------|------------|
| D3 Purist       | 3806ms  | 3950ms  | 6.7%       |
| React Runaround | 58200ms | 58200ms | N/A        |
| DOM Emulator    | 4266ms  | 9124ms  | 15.67%     |

* FMP   -> First Meaningful Paint, excluding the circle grid
* Ready -> Circle grid rendered

---
#### What's the best approach?

A hybrid solution, depending on the requirements:
1. For demanding animations or extremely complex visualizations: **The D3 Purist, in Canvas**
  * 1000+ animated or interactive elements on screen
2. For visualizations with low complexity: **The React Runaround**
  * Static charts, with simple click states
3. For feature rich visualizations with complex requirements: **The DOM Emulator**
  * Animated charts, connected to live data stores, with user interaction
---
#### Future Work

* Offload heavier processes to web workers to free up main thread
* Improve animations
* Identify a consistent, scalable pattern for `enter`/`exit`/`update`
---
#### Any questions?
