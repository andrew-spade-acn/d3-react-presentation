## `D3 <> React`
#### Optimizing for Developer Experience (DX) and Performance

---

### What is DX?

DX is defined by _robust functionality_ that delivers _stability_, _usability_, and _clarity_

+++

Functionality
* The feature layer. "What does the thing do?". Functionality can be anywhere from a tiny helper function to an enormous framework. 
* Ex: 
  * Cross-platform console colors with `Chalk`
  * A new approach to writing front-end code through `TypeScript`

+++

Stability
* How reliable are the features? How often do unpredictable bugs surface?
* Ex: 
  * `AWS EC2`'s 99.99% uptime
  * `Lodash` with 100% predictable functionality.

+++

Usability
* Does it play nice with other tools? What is the performance impact of using it? How long does it take to set up?
* Ex: 
  * Time to make a simple `TodoApp` in `Angular` vs `React` vs `JQuery`.
  * The amount of time spent looking up documentation compared to the amount of time spent developing.
 
+++ 
 
Clarity
* Does it make sense? Do I understand why it exists, and can I explain it? Does it follow common conventions? What are the costs associated with using this tool?
* Ex: 
  * How difficult it is to describe `Redux` compared to `RxJS`
  * The predictability of compiled `CoffeeScript` syntax 

---

#### D3 

* A *visualization* toolkit for creating _data-driven documents_
* Most often used for making charts, but is not explicitly a charting library
* Declarative, providing reliable abstractions for clunky SVG syntax and DOM methods
* "Efficient manipulation of documents based on data"

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

---

#### React 

* A framework for creating modular and encapsulated components, composed together for complex UIs
* Declarative components supported by JSX and component lifecycle methods
* Built for performance, leveraging the Virtual DOM and a reconciliation engine to keep render loops fast

+++

In React: 
```javascript
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

What's the best way to combine them?

1. The D3 Purist
2. The React Holdout
3. The DOM Emulator
---
#### The D3 Purist

```javascript
export class CircleD3 extends Component {

  componentDidMount() {
  
    const data = _.flatten(
      d3.range(20).map(x =>
        d3.range(20).map(y =>
          [x, y]
        )
      )
    );
  
    const svg = d3.select('.renderedD3')
      .append('svg')
      .attr('width', 100)
      .attr('height', 100)
      .append('g');
  
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
    return (
      <div className='renderedD3' />
    )
  }

}
```
+++
#### Consequences

Pros:
* It's fast.
* It's lightweight.
* It allows writing normal D3 code. This is great for onboarding new devs, accessing the wealth of online resources, and getting external help help.

Cons:
* Is this even React?
* It results in two entirely separate development 'worlds'
* No way to set it up with the conveniences React(+Redux) provides -- Hot Module Reloading, Time Travel Debugging.
* Only way to handle state on the D3 side is by manually importing the Redux store.

+++ 
#### The D3 Purist 2.0

```javascript
export class CircleD3 extends Component {
  componentDidMount() {
    /* This allows abstracting logic away into 
       utility functions and 'pure D3 elements', 
       emphasizing our declarative approach */
    const data = makeGrid(20, 20);
    const svg = makeSVG(100, 100);
    makeCircles(svg);
  }
  render() {
    return (
      <div className='renderedD3' />
    )
  }
}
```
---
#### The React Way

```javascript
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
#### Consequences?

Pros:
* This looks like React!
* We can put everything in our own components, making it easy to write modular code

Cons:
* Performance degrades heavily at scale. Forcing everything inside of SVG into React Components pollutes the DOM and results in a major memory hog. Does it matter? Unclear.
* No clear, clean way to handle D3's `enter`/`exit`/`update`*
* The syntax is a massive deviation from D3, making it difficult to learn

+++
#### The React Way, Improved

```javascript
export class CircleReact extends Component {
  render() {
    /* <RangeGrid> is a HOC to abstract away the nested d3.range parts */
    const RangeDot = RangeGrid(Dot);
      
    return (
      /* SVG and G are both pure React Components that wrap the native HTML elements */
      <SVG width={100} height={100}>
        <G>
          <RangeDot x={20} y={20} r={2} />
        </G>
      </SVG>
    )
  }
}
```
---

