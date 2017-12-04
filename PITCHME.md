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

D3 is: 

* A *visualization* toolkit for creating _data-driven documents_
* Most often used for making charts, but is not explicitly a charting library
* Declarative, providing reliable wrappers for clunky SVG syntax and DOM methods
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

`React` is "a JavaScript library for building user interfaces", often referred to as the 'V' in MVC.

More specifically, it is: 
* a framework for creating modular and encapsulated components, composed together for complex UIs
* declarative, with components supported by thin render loops and component lifecycle methods
* fast, leveraging the Virtual DOM and a reconciliation engine to determine what components need to be updated

+++

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

```html
<h1>
  Hello, friends!
</h1>
```

---

What's the best way to combine them?

1. The React way
2. The D3 way
3. DOM emulation

+++

Let's test these, conceptually, with a minimal concept: a colored dot.

```javascript
export class Circle extends React.Component {
  render() {
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
