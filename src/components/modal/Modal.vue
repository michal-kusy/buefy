<template>
    <transition :name="animation">
        <div class="modal is-active" v-if="isActive">
            <div class="modal-background" @click="cancel('outside')"></div>
            <div class="animation-content"
                :class="{ 'modal-content': !hasModalCard }"
                :style="{ maxWidth: newWidth }">
                <component
                    v-if="component"
                    v-bind="props"
                    v-on="events"
                    :is="component"
                    @close="close">
                </component>
                <div
                    v-else-if="content"
                    v-html="content">
                </div>
                <slot v-else></slot>
            </div>
            <button
                v-if="showX"
                class="modal-close is-large"
                @click="cancel('x')">
            </button>
        </div>
    </transition>
</template>

<script>
    import { removeElement } from '../../utils/helpers'
    import config from '../../utils/config'

    export default {
        name: 'bModal',
        props: {
            active: Boolean,
            component: [Object, Function],
            content: String,
            programmatic: Boolean,
            props: Object,
            events: Object,
            width: {
                type: [String, Number],
                default: 960
            },
            hasModalCard: Boolean,
            animation: {
                type: String,
                default: 'zoom-out'
            },
            canCancel: {
                type: [Array, Boolean],
                default: () => ['escape', 'x', 'outside', 'button']
            },
            onCancel: {
                type: Function,
                default: () => {}
            },
            scroll: {
                type: String,
                default: () => {
                    return config.defaultModalScroll
                        ? config.defaultModalScroll
                        : 'clip'
                },
                validator: (value) => {
                    return [
                        'clip',
                        'keep'
                    ].indexOf(value) >= 0
                }
            }
        },
        data() {
            return {
                isActive: this.active || false,
                savedScrollTop: null,
                newWidth: typeof this.width === 'number'
                    ? this.width + 'px'
                    : this.width
            }
        },
        computed: {
            cancelOptions() {
                return typeof this.canCancel === 'boolean'
                    ? this.canCancel
                        ? ['escape', 'x', 'outside', 'button']
                        : []
                    : this.canCancel
            },
            showX() {
                return this.cancelOptions.indexOf('x') >= 0
            }
        },
        watch: {
            active(value) {
                this.isActive = value
            },
            isActive() {
                this.handleScroll()
            }
        },
        methods: {
            handleScroll() {
                if (typeof window === 'undefined') return

                if (this.scroll === 'clip') {
                    document.documentElement.classList.toggle('is-clipped', this.isActive)
                    return
                }

                this.savedScrollTop = !this.savedScrollTop
                    ? document.documentElement.scrollTop
                    : this.savedScrollTop

                document.body.classList.toggle('is-noscroll', this.isActive)

                if (this.isActive) {
                    document.body.style.top = `-${this.savedScrollTop}px`
                    return
                }

                document.documentElement.scrollTop = this.savedScrollTop
                document.body.style.top = null
                this.savedScrollTop = null
            },

            /**
             * Close the Modal if canCancel.
             */
            cancel(method) {
                if (this.cancelOptions.indexOf(method) < 0) return

                this.close()
            },

            /**
             * Call the onCancel prop (function).
             * Emit events, and destroy modal if it's programmatic.
             */
            close() {
                this.onCancel.apply(null, arguments)
                this.$emit('close')
                this.$emit('update:active', false)

                // Timeout for the animation complete before destroying
                if (this.programmatic) {
                    this.isActive = false
                    setTimeout(() => {
                        this.$destroy()
                        removeElement(this.$el)
                    }, 150)
                }
            },

            /**
             * Keypress event that is bound to the document.
             */
            keyPress(event) {
                // Esc key
                if (event.keyCode === 27) this.cancel('escape')
            }
        },
        created() {
            if (typeof window !== 'undefined') {
                document.addEventListener('keyup', this.keyPress)
            }
        },
        beforeMount() {
            // Insert the Modal component in body tag
            // only if it's programmatic
            this.programmatic && document.body.appendChild(this.$el)
        },
        mounted() {
            if (this.programmatic) this.isActive = true
        },
        beforeDestroy() {
            if (typeof window !== 'undefined') {
                document.removeEventListener('keyup', this.keyPress)
            }
        }
    }
</script>
