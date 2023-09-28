# Vue-Bunny-Image

Responsive image component

Uses the image attributes srcSet and sizes to responsively display an an image from bunny.

Generates srcSet with a placeholder for lazyloading based on screen sizes passed in options and adjusts the sizes attribute depending on the screen size.

## Installation

```shell
// npm
npm i @teamnovu/vue-bunny-image
```

## Setup

```javascript
import BunnyImage from '@teamnovu/vue-bunny-image'

Vue.use(BunnyImage, {
  bunnyBaseUrl: "https://<your-cloud-name>.b-cdn.net",
});
```

## Usage

```html
<BunnyImage src="/assets/image.jpg" />
```

## Plugin Options

| Option                    | Default                                             | Required | Type   | Comment |
| ------------------------- | --------------------------------------------------- | -------- | ------ | ------- |
| bunnyBaseUrl              | null                                                | true     | String |         |
| screenSizes               | [default screen sizes](src/default-screen-sizes.js) | false    | Object |         |
| placeholderTransformation | null                                                | false    | String | Named Transformation used for the placeholder transformation |

## Attributes
See [Bunny Image Processing](https://docs.bunny.net/docs/stream-image-processing) for details about the options.

| Attribute          | Default | Required | Type    | Description                                                                                                                                                                                                      |
| ------------------ | ------- | -------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| src                | null    | true     | String  | Relative path to image on bunny                                                                                                                                                                  |
| width              | auto    | false    | Number  | Bunny Optimizer Option                                                                                                                                                                                                     |
| height             | auto    | false    | Number  | Bunny Optimizer Option                                                                                                                                                                                                     |
| aspect-ratio       | auto    | false    | String  | Bunny Optimizer Option                                                                                                                                                                                                     |
| quality            | 85      | false    | Number  | Bunny Optimizer Option                                                                                                                                                                                                   |
| sharpen            | false   | false    | Boolean | Bunny Optimizer Option                                                                                                                                                                                                 |
| blur               | 0       | false    | Number  | Bunny Optimizer Option                                                                                                                                                                                                 |
| crop               | null    | false    | String  | Bunny Optimizer Option                                                                                                                                                                                               |
| crop-gravity       | center  | false    | String  | Bunny Optimizer Option                                                                                                                                                                                               |
| flip               | false   | false    | Boolean | Bunny Optimizer Option                                                                                                                                                                                               |
| flop               | false   | false    | Boolean | Bunny Optimizer Option                                                                                                                                                                                               |
| brightness         | 0       | false    | String  | Bunny Optimizer Option                                                                                                                                                                                               |
| saturation         | 0       | false    | String  | Bunny Optimizer Option                                                                                                                                                                                               |
| hue                | 0       | false    | String  | Bunny Optimizer Option                                                                                                                                                                                               |
| contrast           | 0       | false    | String  | Bunny Optimizer Option                                                                                                                                                                                               |
| optimization       | none    | false    | String  | Bunny Optimizer Option                                                                                                                                                                                               |
| sepia              | 0       | false    | String  | Bunny Optimizer Option                                                                                                                                                                                               |
| usePlaceholder     | true    | false    | Boolean | If this attribute is set to false, no placeholder will be displayed and the original image will be rendered immediately                                                                                          |
| fallbackWidth      | null    | false    | Number  | If srcSet and sizes are not supported in the clients browser, this will be the fixed width of the image itself (not css width)                                                                                   |  |
| aspectRatio        | null    | false    | Number  | If you set this attribute the component will ignore the placeholderDataUrl attribute and load the placeholder image from bunny because the aspect ratio would change between placeholder and the real image |
| placeholderDataUrl | null    | false    | String  | If this attribute is set all other placeholder options are ignored and this url is used as the placeholder source                                                                                              |
| placeholderQuality | 30      | false    | Number  | bunny Option                                                                                                                                                                                                     |
| placeholderWidth   | 300     | false    | Number  | bunny Option                                                                                                                                                                                                     |

## Example

```html
<BunnyImage
  src="/assets/image.jpg"
  :aspect-ratio="1.5"
  :flip="true"
/>
```
