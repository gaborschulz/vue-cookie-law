<template>
  <transition appear :name="transitionName">
    <div class="Cookie" :class="[containerPosition, cookieTheme]" v-if="isOpen">
      <slot :accept="accept" :close="close" :decline="decline" :open="open">
        <div class="Cookie__content">
          <slot name="message">
            {{ message }}
          </slot>
          <slot name="cookieTypes">
            <div class="cl-mt-6" v-if="cookieTypes">
              <span v-for="cookieType in cookieTypes" :key="cookieType.id">
                <input type="checkbox" 
                       :id="'ct-' + cookieType.id" 
                       :ref="'ct-' + cookieType.id" 
                       :value="cookieType.label" 
                       v-model="cookieTypesSelected[cookieType.id]" 
                       :disabled="cookieType.required"
                       :class="cookieTypeCheckboxClass">
                <label :for="'ct-' + cookieType.id">{{ cookieType.label }}</label>
              </span>
            </div>
          </slot>
        </div>
        <div class="Cookie__buttons">
          <a :target="target" :href="buttonLink" v-if="externalButtonLink" :class="buttonClass">{{ buttonLinkText }}</a>
          <router-link :to="buttonLink" v-if="internalButtonLink" :class="buttonClass">{{ buttonLinkText }}</router-link>
          <button v-if="buttonDecline" :class="buttonDeclineClass" @click="decline">{{ buttonDeclineText }}</button>
          <button :class="buttonClass" @click="accept">{{ buttonText }}</button>
        </div>
      </slot>
    </div>
  </transition>
</template>

<script>
  import * as Cookie from 'tiny-cookie'

  const STORAGE_TYPES = {
    local: 'localStorage',
    cookies: 'cookies'
  }

  const CONSENT_AGE_MULTIPLIER = {
    's': 1000,
    'm': 60 * 1000,
    'h': 60 * 60 * 1000,
    'D': 24 * 60 * 60 * 1000,
    'M': 30 * 24 * 60 * 60 * 1000,
    'Y': 365 * 24 * 60 * 60 * 1000
  }

  export default {
    name: 'VueCookieLaw',
    props: {
      buttonText: {
        type: String,
        default: 'Got it!'
      },
      buttonDecline: {
        type: Boolean,
        default: false
      },
      buttonDeclineText: {
        type: String,
        default: 'Decline'
      },
      buttonLink: {
        type: [String, Object],
        required: false
      },
      buttonLinkText: {
        type: String,
        default: 'More info'
      },
      buttonLinkNewTab: {
        type: Boolean,
        default: false
      },
      message: {
        type: String,
        default: 'This website uses cookies to ensure you get the best experience on our website.'
      },
      theme: {
        type: String,
        default: 'base'
      },
      cookieTypes: {
        type: Array,
        default: [
          { id: 'necessary', required: true, label: 'Strictly necessary' }
        ]
      },
      cookieTypeCheckboxClass: {
        type: String
      },
      /**
       * Cookie Container position
       * bottom, top
       * @type {Object}
       */
      position: {
        type: String,
        default: 'bottom'
      },
      /**
       * Transition name has following possibilities
       * slideFromBottom
       * slideFromTop
       * fade
       * @type {Object}
       */
      transitionName: {
        type: String,
        default: 'slideFromBottom'
      },
      buttonClass: {
        type: String,
        default: 'Cookie__button'
      },
      buttonDeclineClass: {
        type: String,
        default: 'Cookie__declineButton'
      },
      storageName: {
        type: String,
        default: 'cookie:visited'
      },
      storageType: {
        type: String,
        default: STORAGE_TYPES.local
      },
      cookieOptions: {
        type: Object,
        default: () => {},
        required: false
      },
      timeout: {
        type: Number,
        default: 0
      },
      maxConsentAge: {
        type: String,
        default: '1M'
      }
    },
    data () {
      return {
        supportsLocalStorage: true,
        isOpen: false,
        cookieTypesSelected: {}
      }
    },
    computed: {
      containerPosition () {
        return `Cookie--${this.position}`
      },
      cookieTheme () {
        return `Cookie--${this.theme}`
      },
      externalButtonLink () {
        return typeof this.buttonLink === 'string' && this.buttonLink.length
      },
      internalButtonLink () {
        return typeof this.buttonLink === 'object' && this.buttonLink != null && Object.keys(this.buttonLink).length
      },
      target () {
        return this.buttonLinkNewTab ? '_blank' : '_self'
      },
      canUseLocalStorage () {
        return this.storageType === STORAGE_TYPES.local && this.supportsLocalStorage
      }
    },
    created () {
      if (this.storageType === STORAGE_TYPES.local) {
        // Check for availability of localStorage
        try {
          const test = '__vue-cookielaw-check-localStorage'

          window.localStorage.setItem(test, test)
          window.localStorage.removeItem(test)
        } catch (e) {
          console.info('Local storage is not supported, falling back to cookie use')
          this.supportsLocalStorage = false
        }
      }

      const consentAge = new Date() - this.getVisited()
      const maxConsentAge = this.getMaxConsentAgeInMs()
      if (consentAge > maxConsentAge) {
        this.isOpen = true
      }

      if (this.cookieTypes) {
        this.cookieTypes.forEach((cookieType) => {
          this.cookieTypesSelected[cookieType.id] = cookieType.required
        })
      }

      if (this.timeout > 0) {
        setTimeout(() => this.decline(), this.timeout)
      }
    },
    methods: {
      getMaxConsentAgeInMs () {
        const maxConsentAgeDimension = this.maxConsentAge.charAt(this.maxConsentAge.length - 1)
        const maxConsentAgeMultiplier = CONSENT_AGE_MULTIPLIER[maxConsentAgeDimension]
        const maxConsentAgeValue = Number(this.maxConsentAge.substring(0, this.maxConsentAge.length - 1))

        return maxConsentAgeValue * maxConsentAgeMultiplier
      },
      setVisited () {
        if (this.canUseLocalStorage) {
          localStorage.setItem(this.storageName, (new Date()).toISOString())
        } else {
          Cookie.set(this.storageName, (new Date()).toISOString(), { ...this.cookieOptions, expires: this.maxConsentAge })
        }
      },
      setAccepted () {
        const acceptedCookieTypes = this.cookieTypes.map((cookieType) => `cookie:${cookieType.id}`)
        if (this.canUseLocalStorage) {
          acceptedCookieTypes.forEach((cookieType) => localStorage.setItem(cookieType, true))
        } else {
          acceptedCookieTypes.forEach((cookieType) => Cookie.set(cookieType, true, { ...this.cookieOptions, expires: this.maxConsentAge }))
        }
      },
      setDeclined () {
        const cookieTypeSettings = Object.keys(this.cookieTypesSelected).map((cookieTypeKey) => {
          return { name: `cookie:${cookieTypeKey}`, status: this.cookieTypesSelected[cookieTypeKey] }
        })

        if (this.canUseLocalStorage) {
          cookieTypeSettings.forEach((cookieTypeSetting) => localStorage.setItem(cookieTypeSetting.name, cookieTypeSetting.status))
        } else {
          cookieTypeSettings.forEach((cookieTypeSetting) => Cookie.set(cookieTypeSetting.name, cookieTypeSetting.status, { ...this.cookieOptions, expires: this.maxConsentAge }))
        }
      },
      getVisited () {
        if (this.canUseLocalStorage) {
          return new Date(localStorage.getItem(this.storageName))
        } else {
          return new Date(Cookie.get(this.storageName))
        }
      },
      accept () {
        this.setVisited()
        this.setAccepted()
        this.isOpen = false
        this.$emit('accept')
      },
      close () {
        this.isOpen = false
        this.$emit('close')
      },
      decline () {
        this.setVisited()
        this.setDeclined()
        this.isOpen = false
        this.$emit('decline')
      },
      open () {
        if (!this.getVisited() === true) {
          this.isOpen = true
        }
      }
    }
  }
</script>

<style lang="scss">
  @import "~@nextindex/next-scss/next-scss.scss";

  .cl-mt-6 {
    margin-top: 6px;
  }

  .Cookie {
    position: fixed;
    overflow: hidden;
    box-sizing: border-box;
    z-index: 9999;
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    flex-direction: column;

    > * {
      margin: rem(15) 0;
      align-self: center;
    }

    @include media($sm-up) {
      flex-flow: row;

      > * {
        margin: 0;
      }
    }
  }

  .Cookie--top {
    top: 0;
    left: 0;
    right: 0;
  }

  .Cookie--bottom {
    bottom: 0;
    left: 0;
    right: 0;
  }
  .Cookie__buttons {
    display: flex;
    flex-direction: column;

    > * {
      margin: rem(5) 0;
    }

    @include media($sm-up) {
      flex-direction: row;
      > * {
        margin: 0 rem(15);
      }
    }
  }
  .Cookie__button {
    cursor: pointer;
    align-self: center;
    white-space: nowrap;
  }

  .Cookie__declineButton {
    cursor: pointer;
    align-self: center;
    white-space: nowrap;
  }

  @mixin generateTheme($theme, $backgroundColor, $fontColor, $buttonBackgroundColor, $buttonFontColor: #fff, $buttonRadius: 0) {
    .Cookie--#{$theme} {
      background: $backgroundColor;
      color: $fontColor;
      padding: 1.250em;

      .Cookie__button {
          background: $buttonBackgroundColor;
          padding: 0.625em 3.125em;
          color: $buttonFontColor;
          border-radius: $buttonRadius;
          border: 0;
          font-size: 1em;

          &:hover {
            background: darken($buttonBackgroundColor, 10%);
          }
      }
      .Cookie__declineButton {
          background: darken($backgroundColor, 25%);
          padding: 0.625em 3.125em;
          color: $buttonFontColor;
          border-radius: $buttonRadius;
          border: 0;
          font-size: 1em;

          &:hover {
            background: darken($backgroundColor, 15%);
          }
      }
    }
  }

  @include generateTheme('base', #F1F1F1, #232323, #97D058);
  @include generateTheme('base--rounded', #F1F1F1, #232323, #97D058, #fff, 20px);
  @include generateTheme('blood-orange', #424851, #fff, #E76A68);
  @include generateTheme('blood-orange--rounded', #424851, #fff, #E76A68, #fff, 20px);
  @include generateTheme('dark-lime', #424851, #fff, #97D058);
  @include generateTheme('dark-lime--rounded', #424851, #fff, #97D058, #fff, 20px);
  @include generateTheme('royal', #FBC227, #232323, #726CEA, #fff);
  @include generateTheme('royal--rounded', #FBC227, #232323, #726CEA, #fff, 20px);
  @include generateTheme('website--rounded', #FFAA00, #232323, #181818, #fff, 20px);

  .slideFromTop-enter, .slideFromTop-leave-to {
    transform: translate(0px, -12.500em);
  }

  .slideFromTop-enter-to, .slideFromTop-leave {
    transform: translate(0px, 0px);
  }

  .slideFromBottom-enter, .slideFromBottom-leave-to {
    transform: translate(0px, 12.500em);
  }

  .slideFromBottom-enter-to, .slideFromBottom-leave {
    transform: translate(0px, 0px);
  }

  .slideFromBottom-enter-active,
  .slideFromBottom-leave-active,
  .slideFromTop-enter-active,
  .slideFromTop-leave-active, {
    transition: transform .4s ease-in;
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s
  }
  .fade-enter, .fade-leave-to {
    opacity: 0
  }
</style>
