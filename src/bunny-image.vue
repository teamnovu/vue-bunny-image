<template>
  <img
    v-if="fileTypeSupported"
    ref="imageRef"
    :src="originalUrl"
    :srcset="imgSrcSet"
    :sizes="imgSizes"
    :width="imgWidth"
    :height="imgHeight"
    @load="onLoaded"
  >
  <img
    v-else
    :src="originalUrl"
    :width="imgWidth"
    :height="imgHeight"
  >
</template>

<script>
function print (level, ...message) {
  if (process.env.NODE_ENV !== 'development') return
  console[level](
    '%cBunnyImage',
    'color: white; background-color: rgba(239, 96, 39, 1); padding: 2px 4px; border-radius: 2px;',
    ...message,
  )
}

const prop = (type, required = false, defaultValue = undefined) => ({
  type,
  required,
  default: defaultValue,
})

export default {
  props: {
    src: prop(String, true),
    sizes: prop(String),
    width: prop(Number),
    height: prop(Number),
    aspectRatio: prop([Number, String]),
    quality: prop(Number),
    sharpen: prop(Boolean),
    blur: prop(Number),
    crop: prop(String),
    cropGravity: prop(String),
    flip: prop(Boolean),
    flop: prop(Boolean),
    brightness: prop(Number),
    saturation: prop(Number),
    hue: prop(Number),
    contrast: prop(Number),
    optimization: prop(String),
    sepia: prop(Number),

    // placeholder
    placeholderQuality: prop(Number, false, 30),
    placeholderWidth: prop(Number, false, 300),
    usePlaceholder: prop(Boolean, false, true),
    placeholderDataUrl: prop(String),
  },
  data () {
    return {
      internalSizes: '1px',
      isLoading: true,
    }
  },
  computed: {
    fileTypeSupported () {
      const regex = /(?:\.([^.]+))?$/
      const fileExtension = regex.exec(this.src)[1]

      return (
        fileExtension &&
        ['jpg', 'png', 'gif', 'webp', 'jpeg'].includes(fileExtension.toLowerCase())
      )
    },

    breakpointSizes () {
      return this.screens.map((screen) => screen.size.replace('px', ''))
    },

    largestBreakpointSize () {
      return this.breakpointSizes[0]
    },

    bunnySrc () {
      if (this.src.startsWith('/')) return this.$bunnyImage.bunnyBaseUrl + this.src
      else return this.$bunnyImage.bunnyBaseUrl + '/' + this.src
    },

    imgSrcSet () {
      const srcSet = this.breakpointSizes.map(
        (breakpointSize) =>
          this.generateSrc({
            ...this.$props,
            width: breakpointSize,
          }) + ` ${breakpointSize}w`,
      )

      if (this.usePlaceholder) {
        srcSet.push(this.placeholderUrl + ' 32w')
      }

      return srcSet.join(',')
    },

    originalUrl () {
      return this.generateSrc({
        ...this.$props,
      })
    },

    placeholderUrl () {
      if (this.placeholderDataUrl && !this.aspectRatio) {
        return this.placeholderDataUrl
      }

      return this.generateSrc({
        ...this.$props,
        quality: this.placeholderQuality,
        width: this.placeholderWidth,
        placeholder: true,
      })
    },

    imgWidth () {
      if ((this.width && this.height) || (this.width && this.aspectRatio)) {
        return this.width
      }

      if (this.aspectRatio && this.height && Number(this.height) > 0) {
        return this.height * this.aspectRatioFloat
      }

      return this.largestBreakpointSize
    },

    imgHeight () {
      if ((this.width && this.height) || (this.height && this.aspectRatio)) {
        return this.height
      }

      if (this.aspectRatio && this.width && Number(this.width) > 0) {
        return this.width / this.aspectRatioFloat
      }

      if (this.aspectRatio) {
        return this.largestBreakpointSize / this.aspectRatioFloat
      }

      return undefined
    },

    aspectRatioFloat () {
      if ((this.aspectRatio + '').includes(':')) return this.aspectRatio.split(':')[0] / this.aspectRatio.split(':')[1]
      else return this.aspectRatio
    },

    imgAspectRatio () {
      if (this.aspectRatio) {
        return this.aspectRatioFloat
      }

      if (this.imgWidth && this.imgHeight) {
        return this.imgWidth / this.imgHeight
      }

      return undefined
    },

    imgSizes () {
      return this.sizes || this.internalSizes
    },
  },

  created () {
    this.screens = Object.entries(this.$bunnyImage.screenSizes)
      .map(([key, value]) => ({
        breakpoint: key,
        media: `min-width: ${value}`,
        size: value,
      }))
      .sort((a, b) => +b.size.replace('px', '') - +a.size.replace('px', ''))
  },

  mounted () {
    window.addEventListener('resize', this.onResize, { passive: true })
  },

  beforeDestroy () {
    window.removeEventListener('resize', this.onResize)
  },

  methods: {

    updateSizes () {
      return new Promise((resolve) => {
        window.requestAnimationFrame(() => {
          if (this.$refs.imageRef) {
            const containerWidth =
              this.$refs.imageRef.getBoundingClientRect().width
            const isCover =
              getComputedStyle(this.$refs.imageRef).objectFit === 'cover'

            if (isCover) {
              const containerHeight =
                this.$refs.imageRef.getBoundingClientRect().height
              const containerAspectRatio = containerWidth / containerHeight

              if (this.imgAspectRatio > containerAspectRatio) {
                const size = Math.ceil(
                  (Math.round(containerHeight) /
                    Math.round(window.innerHeight)) *
                    100,
                )
                this.internalSizes = `${size}vh`
              } else {
                const size = Math.ceil(
                  (Math.round(containerWidth) / Math.round(window.innerWidth)) *
                    100,
                )
                this.internalSizes = `${size}vw`
              }
            } else {
              const size = Math.ceil(
                (Math.round(containerWidth) / Math.round(window.innerWidth)) *
                  100,
              )
              this.internalSizes = `${size}vw`
            }
          }
          resolve()
        })
      })
    },

    onResize () {
      this.updateSizes()
    },

    async onLoaded () {
      await this.updateSizes()
      this.isLoading = false
    },

    generateSrc ({
      placeholder = false,

      width,
      height,
      aspectRatio,
      quality,
      sharpen,
      blur,
      crop,
      cropGravity,
      flip,
      flop,
      brightness,
      saturation,
      hue,
      contrast,
      optimization,
      sepia,

    }) {
      if (!this.fileTypeSupported) return this.bunnySrc

      const transformations = []

      if (placeholder) {
        if (this.$bunnyImage.placeholderTransformation) {
          transformations.push(`${this.$bunnyImage.placeholderTransformation}`)
        } else {
          transformations.push('blur=80', 'width=150', 'quality=20')
        }
      } else {
        if (width) transformations.push(`width=${width}`)
        if (height) transformations.push(`height=${height}`)
        if (aspectRatio) {
        // support either 0.5 or 1:2 formats
          if ((aspectRatio + '').includes(':')) transformations.push(`aspect_ratio=${aspectRatio}`)
          else transformations.push(`aspect_ratio=1:${1 / aspectRatio}`)
        }
        if (quality) transformations.push(`quality=${quality}`)
        if (sharpen) transformations.push(`sharpen=${sharpen}`)
        if (blur) {
          if (blur < 0 || blur > 100) print('warn', 'blur is out of range', { blur })
          transformations.push(`blur=${Math.min(100, blur)}`)
        }
        if (crop && crop !== 'fill') print('warn', 'crop is not supported', { crop })
        if (cropGravity) transformations.push(`crop_gravity=${cropGravity}`)
        if (flip) transformations.push(`flip=${flip}`)
        if (flop) transformations.push(`flop=${flop}`)
        if (brightness) {
          if (brightness < -100 || brightness > 100) print('warn', 'brightness is out of range', { brightness })
          transformations.push(`brightness=${brightness}`)
        }
        if (saturation) transformations.push(`saturation=${saturation}`)
        if (hue) transformations.push(`hue=${hue}`)
        if (contrast) transformations.push(`contrast=${contrast}`)
        if (optimization) transformations.push(`optimization=${optimization}`)
        if (sepia) transformations.push(`sepia=${sepia}`)
      }

      return this.bunnySrc + (transformations.length > 0 ? '?' + transformations.join('&') : '')
    },
  },
}
</script>
