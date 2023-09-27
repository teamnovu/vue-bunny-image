# Vue-Bunny-Image

Responsive image component

Uses the image attributes srcSet and sizes to responsively display an an image from bunny.

Generates srcSet with a placeholder for lazyloading based on screen sizes passed in options and adjusts the sizes attribute depending on the screen size.

## Installation

```shell
// yarn
yarn add @teamnovu/vue-bunny-image

// npm
npm i @teamnovu/vue-bunny-image
```

## Setup

```javascript
import bunnyImage from '@teamnovu/vue-bunny-image'

Vue.use(bunnyImage, {
  bunnyBaseUrl: "https://res.bunny.com/<your-cloud-name>/image/upload/",
});
```

## Usage

```html
<bunnyImage src="/assets/image.jpg" />
```

## Plugin Options

| Option                    | Default                                             | Required | Type   | Comment |
| ------------------------- | --------------------------------------------------- | -------- | ------ | ------- |
| bunnyBaseUrl         | null                                                | true     | String |         |
| screenSizes               | [default screen sizes](src/default-screen-sizes.js) | false    | Object |         |
| placeholderTransformation | null                                                | false    | String | Named Transformation used for the placeholder transformation |

## Attributes

| Attribute          | Default | Required | Type    | Description                                                                                                                                                                                                      |
| ------------------ | ------- | -------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| src                | null    | true     | String  | Relative path to image from bunnyAssetUrl                                                                                                                                                                   |
| quality            | 85      | false    | Number  | bunny Option                                                                                                                                                                                                     |
| blur               | null    | false    | Number  | bunny Option                                                                                                                                                                                                     |
| crop               | null    | false    | String  | bunny Option                                                                                                                                                                                                     |
| format             | 'auto'    | false    | String  | bunny Option                                                                                                                                                                                                     |
| fallbackWidth      | null    | false    | Number  | If srcSet and sizes are not supported in the clients browser, this will be the fixed width of the image itself (not css width)                                                                                   |  |
| aspectRatio        | null    | false    | Number  | If you set this attribute the component will ignore the placeholderDataUrl attribute and load the placeholder image from bunny because the aspect ratio would change between placeholder and the real image |
| placeholderDataUrl | null    | false    | String  | If this attribute is set all other placeholder options are ignored and this url is used as the placeholder source                                                                                              |
| placeholderQuality | 30      | false    | Number  | bunny Option                                                                                                                                                                                                     |
| placeholderWidth   | 300     | false    | Number  | bunny Option                                                                                                                                                                                                     |
| usePlaceholder     | true    | false    | Boolean | If this attribute is set to false, no placeholder will be displayed and the original image will be rendered immediately                                                                                          |
| zoom     | undefined    | false    | String, Number |  Zoom level |
| transforms     | undefined    | false    | String, Object |  Additional, raw transformations |

## Example

```html
<bunnyImage
  src="/assets/image.jpg"
/>
```
