title: Creating Responsive D3 Chart using ReactJS
date: 2020-04-04 16:02:00
tags:

- JavaScript
- React
- D3

---

This post shows how to create a responsive chart using [D3](https://d3js.org/) and React.

Here is demo URL https://i5u7r.csb.app/ to play with and it looks like this

{% youtube fefdzyL6UzY %}

Building D3 chart requires `width` and `height` values upfront, this helps D3 to map data points to an `x`, `y` coordinate on the SVG canvas. In this post we're creating a simple line chart with x-axis and y-axis and it resizes when the browser window is resized and axes ticks are updated based on the available width and height.

First we need a way to get `width` and `height` values and it should update when the browser window is resized. For this, I'm creating a custom resize hook called `useResize`, which returns the size of HTML element passed to it.

```js
function useResize(ref) {
  const [state, setState] = useState();
  useEffect(() => {
    const getSize = debounce(() => {
      if (!ref || !ref.current) {
        return;
      }
      const width = ref.current.offsetWidth;
      const height = ref.current.offsetHeight;
      setState({
        width,
        height,
      });
    }, 1000);

    window.addEventListener("resize", getSize);
    getSize();
    return () => window.removeEventListener("resize", getSize);
  }, [ref]);

  return state;
}
```

`useResize` hook subscribes to `resize` event and uses `offsetWidth`, `offsetHeight` properties on the HTML node to get `width` and `height` respectively.

> Make sure `getSize` method is not called frequently to reduce the number of updates. Here, I'm using `debounce` helper function from `lodash`.

In my chart component, `ref` of my root element is passed to `useResize` hook to get it's size.

```jsx
const LineChart = props => {
  ....
  const rootRef = useRef(null);
  const size = useResize(rootRef);
  .....
  return (
    <div className="chart-area" ref={rootRef}>
      .....
    </div>
  );
};
```

Once we've this in place, implementing a line chart is straight forward. However to make the axes responsive, i.e. change it's tick based on the available size. Pass a ratio based on `width` or `height` to the `ticks` method of both axes.

```js
const { width, height } = size;

const xAxis = d3
  .axisBottom()
  .scale(xScale)
  .ticks(width / 100); // I've hard coded 100 here, you can change it based on the no:of ticks required
const yAxis = d3
  .axisLeft()
  .scale(yScale)
  .ticks(height / 50); // I've hard coded 50 here, you can change it based on the no:of ticks required
```

Here is the full working demo running in [CodeSandbox](https://codesandbox.io/s/responsive-d3-chart-i5u7r)

{% raw %}

<iframe
     src="https://codesandbox.io/embed/responsive-d3-chart-i5u7r?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="responsive-d3-chart"
     allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb"
     sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"
   ></iframe>
{% endraw %}

Hope you enjoyed this, happy coding.
