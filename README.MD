# A Mapbox GL Storybook (made with react)
The basic idea of this repo is to illustration how one could wrap a third-party lib (like mapbox-gl) with react.

## Preface
What's unique about this approach is that instead of following the feature separation (defined by mapbox) of the lib where there is a Layer nested within a Source, etc... I define an *Plugin* abstraction layer to handle both that is defined by an object.

## Rational
The rational of having a *Plugin* is to reduce the implementation detail of composing a Source and Layer components; as well as various other classes that are available within the lib. The benefit of this is to also keep the component compositions more flat which will add to the better understanding of the product one is building. For example (ES6/JSX syntax):
```
<Mapbox
  style="mapbox://styles/mapbox/..."
  ...options
>
  <GeojsonLayer
    id="special-identifier"
    data={geojsonData}
    layerStyle={{
      "source-layer": "water",
      "type": "fill",
      "paint": {
        "fill-color": "#00ffff"
      }
    }}
  >
  <ScaleControl>
  <FullscreenControl>
  ...etc
</Mapbox>
```

If you notice the `layerStyle` prop of the _GeojsonLayer_ *Plugin*; it is used to define what a source could look like. This, of course, is one of many ways one could define how a component is layed out but on a high level this (to me) seems less opinionated and gives you the freedom to build it as you see fit instead of relaying on a react lib design.

## Design choices
Internally there are some design choices that where made to take advantage of some new react concepts to simplify the parent/child relationship and provide the map instance.

### Context Provider
The way plugins get what they need to be *mounted* to the mapbox instance is via the [React Context](https://reactjs.org/docs/context.html) and [HOCs](https://reactjs.org/docs/higher-order-components.html) (if you haven't read about these React features you may want to brush up before continuing).

In a nut shell the parent component needs to be build with a context provider that is then exported for use in the plugins (when imported):

```
// Create the Context object
const MapContext = React.createContext({ map: null });

// Create the HOC to wrap components with the context and provide the child with the instance of the map that is passed from the parent once loaded.
export const withMap = ( NewComponwnt ) => () => (
  <MapContext.Consumer>
    {
      ( props ) => ( <NewComponwnt map={ props.map } { ...props } /> )
    }
  </MapContext.Consumer>
);

// skipping implementation details

// wrap the returning/render function with provider
render() {
  const { children } = this.props;
  const { mapInst } = this.state;
  return (
    <MapContext.Provider value={ { map: mapInst } }>
      { mapInst && children }
    </MapContext.Provider>
  );
}
```

Then within a (child) plugin component:
```
import { withMap } from '../Map';

export default withMap(( props ) => {
  console.log( props );
  return (<div>plugin with the map);
});
```

When everything loads you should see an object printed within the console that will have a shape similar to this: `{ map: { ...mapbox instance methods } }`

### Final Notes:
When a (child plugin) component mounts successfully and there's some jsx returned, it is added/appended to the map `div.mapboxgl-map` container. if nothing is returned _ie._ `return null;` then nothing is rendered to the DOM but it will still give the plugin a chance to load.

## Install steps:
First, create a `process.env` in the root of the project and add a single line `MAPBOX_TOKEN="mapbox-gl.token.goes.here"` to the file. This will be needed when you run the project using your own environment (protected) variable. Then follow the install and start process:

- `yarn` - install project deps
- `yarn bootstrap` - install package deps
- `yarn start` - start storybook project

or all in one shot: `yarn && yarn bootstrap && yarn start`. After which the server will auto launch to `http://localhost:6006/`