<template>
  <img
    v-if="fileTypeSupported"
    ref="imageRef"
    :src="originalUrl"
    :srcset="imgSrcSet"
    :sizes="imgSizes"
    :width="imgWidth"
    :height="imgHeight"
    :aspect-ratio="imgAspectRatio"
    @load="onLoaded"
  >
  <img
    v-else
    :src="originalUrl"
    :width="imgWidth"
    :height="imgHeight"
    :aspect-ratio="imgAspectRatio"
  >
</template>

<script>
import { joinURL, encodePath } from 'ufo'

export default {
  props: {
    src: {
      required: true,
      type: String,
    },
    quality: {
      required: false,
      type: [Number, String],
      default: 'auto',
    },
    blur: {
      required: false,
      type: Number,
      default: undefined,
    },
    crop: {
      required: false,
      type: String,
      default: 'lfill',
    },
    format: {
      required: false,
      type: String,
      default: 'auto',
    },
    aspectRatio: {
      required: false,
      type: Number,
      default: undefined,
    },
    placeholderQuality: {
      required: false,
      type: Number,
      default: 30,
    },
    placeholderWidth: {
      required: false,
      type: Number,
      default: 300,
    },
    usePlaceholder: {
      required: false,
      type: Boolean,
      default: true,
    },
    placeholderDataUrl: {
      required: false,
      type: String,
      default: undefined,
    },
    focal: {
      required: false,
      type: [Array, String],
      default: 'auto',
    },
    fallbackWidth: {
      required: false,
      type: Number,
      default: 2000,
    },
    width: {
      type: [String, Number],
      default: undefined,
    },
    height: {
      type: [String, Number],
      default: undefined,
    },
    zoom: {
      required: false,
      type: [String, Number],
      default: undefined,
    },
    transforms: {
      required: false,
      type: [String, Object],
      default: undefined,
    },
    sizes: {
      required: false,
      type: String,
      default: undefined,
    },
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
        ['jpg', 'png', 'gif', 'webp', 'jpeg', 'avif'].includes(fileExtension.toLowerCase())
      )
    },
    breakpointSizes () {
      return this.screens.map((screen) => screen.size.replace('px', ''))
    },
    largestBreakpointSize () {
      return this.breakpointSizes[0]
    },
    imgSrcSet () {
      const srcSet = this.breakpointSizes.map(
        (breakpointSize) =>
          this.generateSrc({
            quality: this.quality,
            width: breakpointSize,
            format: this.format,
            aspectRatio: this.aspectRatio,
            crop: this.crop,
            focal: this.focal,
            zoom: this.zoom,
          }) + ` ${breakpointSize}w`,
      )

      if (this.usePlaceholder) {
        srcSet.push(this.placeholderUrl + ' 32w')
      }

      return srcSet.join(',')
    },
    originalUrl () {
      return this.generateSrc({
        quality: this.quality,
        blur: this.blur,
        width: this.fallbackWidth,
        format: this.format,
        aspectRatio: this.aspectRatio,
        crop: this.crop,
        focal: this.focal,
        zoom: this.zoom,
        transforms: this.transforms,
      })
    },
    placeholderUrl () {
      if (this.placeholderDataUrl && !this.aspectRatio) {
        return this.placeholderDataUrl
      }

      return this.generateSrc({
        quality: this.placeholderQuality,
        width: this.placeholderWidth,
        format: this.format,
        aspectRatio: this.aspectRatio,
        crop: this.crop,
        focal: this.focal,
        placeholder: true,
        zoom: this.zoom,
        transforms: this.transforms,
      })
    },
    imgWidth () {
      if ((this.width && this.height) || (this.width && this.aspectRatio)) {
        return this.width
      }

      if (this.aspectRatio && this.height && Number(this.height) > 0) {
        return this.height * this.aspectRatio
      }

      return this.largestBreakpointSize
    },
    imgHeight () {
      if ((this.width && this.height) || (this.height && this.aspectRatio)) {
        return this.height
      }

      if (this.aspectRatio && this.width && Number(this.width) > 0) {
        return this.width / this.aspectRatio
      }

      if (this.aspectRatio) {
        return this.largestBreakpointSize / this.aspectRatio
      }

      return undefined
    },
    imgAspectRatio () {
      if (this.aspectRatio) {
        return this.aspectRatio
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
    const screens = Object.entries(this.$bunnyImage.screenSizes)
      .map(([key, value]) => ({
        breakpoint: key,
        media: `min-width: ${value}`,
        size: value,
      }))
      .sort((a, b) => +b.size.replace('px', '') - +a.size.replace('px', ''))

    this.screens = screens
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
      quality,
      width,
      aspectRatio,
      blur,
      crop,
      format,
      focal,
      placeholder = false,
      zoom,
      transforms: additionalTransforms,
    }) {
      if (!this.fileTypeSupported) {
        return joinURL(this.$bunnyImage.bunnyBaseUrl, encodePath(this.src))
      }

      const transformations = []

      if (!placeholder) {
        if (width) transformations.push(`w_${width}`)
        if (quality) transformations.push(`q_${quality}`)
        if (blur) transformations.push(`e_blur:${blur}`)
        if (format) transformations.push(`f_${format}`)
        if (zoom) transformations.push(`z_${zoom}`)
        if (width && aspectRatio) {
          transformations.push(`h_${Math.round(width / aspectRatio)}`)
        } else if (!width && aspectRatio) {
          transformations.push(`ar_${aspectRatio}`)
        }
      } else {
        if (this.$bunnyImage.placeholderTransformation) {
          transformations.push(`t_${this.$bunnyImage.placeholderTransformation}`)
        } else {
          transformations.push('e_blur:2000,f_auto,q_auto:eco,w_300,z_1.1')
        }
        if (aspectRatio) transformations.push(`ar_${aspectRatio}`)
      }
      if (crop) transformations.push(`c_${crop}`)

      if (focal && ['crop', 'fill', 'lfill', 'lpad', 'mpad', 'pad', 'thumb'].includes(this.crop)) {
        if (Array.isArray(focal)) {
          transformations.push(`x_${focal[0]},y_${focal[1]},g_xy_center`)
        } else {
          transformations.push(`g_${focal}`)
        }
      }

      if (additionalTransforms) {
        if (typeof additionalTransforms === 'object') {
          Object.entries(additionalTransforms).forEach(([key, value]) => {
            transformations.push(`${key}_${value}`)
          })
        } else {
          transformations.push(additionalTransforms)
        }
      }

      const remoteFolderMapping = this.$bunnyImage.bunnyBaseUrl.match(/\/image\/upload\/(.*)/)

      // Handle delivery remote media file URLs
      // Note: Non-remote images will pass into this function if the baseURL is not using a sub directory
      if (remoteFolderMapping?.length >= 1) {
        // need to do some weird logic to get the remote folder after image/upload after the operations and before the src
        const remoteFolder = remoteFolderMapping[1]
        const baseURLWithoutRemoteFolder = this.$bunnyImage.bunnyBaseUrl.replace(new RegExp(`${remoteFolder}$`), '')

        return joinURL(baseURLWithoutRemoteFolder, transformations.join(','), remoteFolder, encodePath(this.src))
      }

      return joinURL(this.$bunnyImage.bunnyBaseUrl, transformations.join(','), encodePath(this.src))
    },
  },
}
</script>
