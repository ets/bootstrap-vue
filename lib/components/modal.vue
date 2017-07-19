<template>
    <div>
        <transition enter-class=""
                    enter-active-class=""
                    enter-to-class=""
                    @before-enter="onBeforeEnter"
                    @enter="onEnter"
                    @after-enter="onAfterEnter"
                    leave-class=""
                    leave-active-class=""
                    leave-to-class=""
                    @before-leave="onBeforeLeave"
                    @leave="onLeave"
                    @after-leave="onAfterLeave"
        >
            <div :class="['modal',{fade: !noFade, show: isShowing}]"
                 :id="id || null"
                 role="dialog"
                 ref="modal"
                 key="modal"
                 v-show="isShowing"
                 :aria-hidden="isShowing ? 'true' : 'false'"
                 @click="onClickOut()"
                 @keyup.esc="onEsc()"
            >

                <div :class="['modal-dialog','modal-'+size]">
                    <div class="modal-content"
                         tabindex="-1"
                         role="document"
                         ref="content"
                         :aria-labelledby="(hideHeader || !id) ? null : (id + '__BV_header_')"
                         :aria-describedby="id ? (id + '__BV_body_') : null"
                         @click.stop
                    >

                        <header class="modal-header"
                                ref="header"
                                :id="id ? (id + '__BV_header_') : null"
                                v-if="!hideHeader"
                        >
                            <slot name="modal-header">
                                <h5 :is="titleTag" class="modal-title">
                                    <slot name="modal-title">{{title}}</slot>
                                </h5>
                                <button type="button"
                                        v-if="!hideHeaderClose"
                                        class="close"
                                        :aria-label="headerCloseLabel"
                                        @click="hide"
                                >
                                    <span aria-hidden="true">&times;</span>
                                </button>
                            </slot>
                        </header>

                        <div class="modal-body" ref="body" :id="id ? (id + '__BV_body_') : null">
                            <slot></slot>
                        </div>

                        <footer class="modal-footer" ref="footer" v-if="!hideFooter">
                            <slot name="modal-footer">
                                <b-btn v-if="!okOnly"
                                       variant="secondary"
                                       :size="buttonSize"
                                       @click="hide(false)"
                                ><slot name="modal-cancel">{{ closeTitle }}</slot></b-btn>
                                <b-btn variant="primary"
                                       :size="buttonSize"
                                       :disabled="okDisabled"
                                       @click="hide(true)"
                                ><slot name="modal-ok">{{ okTitle }}</slot></b-btn>
                            </slot>
                        </footer>

                    </div>
                </div>
            </div>
        </transition>
        <div key="modal-backdrop"
             :class="['modal-backdrop',{fade: !noFade, show: is_visible || isTransitioning}]"
             v-if="isShowing"
        ></div>
    </div>
</template>

<script>
    import bBtn from './button.vue';
    import { listenOnRootMixin } from '../mixins';
    import { from as arrayFrom } from '../utils/array'

    const ClassName = {
        SCROLLBAR_MEASURER : 'modal-scrollbar-measure',
        BACKDROP           : 'modal-backdrop',
        OPEN               : 'modal-open',
        FADE               : 'fade',
        SHOW               : 'show'
    }

    const Selector = {
        DIALOG             : '.modal-dialog',
        FIXED_CONTENT      : '.fixed-top, .fixed-bottom, .is-fixed, .sticky-top',
        NAVBAR_TOGGLER     : '.navbar-toggler'
    }

    const ON = true;
    const OFF = false;

    const FOCUS_SELECTOR = [
        'button:not([disabled])',
        'input:not([disabled])',
        'select:not([disabled])',
        'textarea:not([disabled])',
        'a:not([disabled]):not(.disabled)',
        '[tabindex]:not([disabled]):not(.disabled)'
    ].join(',');

    // Determine if an HTML element is visible - Faster than CSS check
    function isVisible(el) {
        return el && (el.offsetWidth > 0 || el.offsetHeight > 0);
    }

    // Find the first visible element contained in a given root element
    function findFirstVisible(root, selector) {
        if (!root || !root.querySelectorAll || !selector) {
            return null;
        }
        let els = arrayFrom(root.querySelectorAll(selector));

        // IE 10 & 11 do not support native array.find()
        // So we try native find first, then fall back to a loop
        let el = els.find ? els.find(el => isVisible(el)) : null;
        for (let i = 0; !el && i < els.length; i++) {
            if (isVisible(els[i])) {
                el = els[i];
            }
        }
        return el;
    }

    export default {
        mixins: [listenOnRootMixin],
        components: {bBtn},
        data() {
            return {
                is_visible: false,
                return_focus: this.returnFocus || null,
                scrollbarWidth: 0,
                isEntering: false,
                isLeaving: false,
                isShown: false
            };
        },
        model: {
            prop: 'visible',
            event: 'change'
        },
        computed: {
            body() {
                if (typeof document !== 'undefined') {
                    return document.querySelector('body');
                }
            },
            isTransitioning() {
                return this.isLeaving || this.isEntering;
            },
            isShowing() {
                return !this.isTransitioning && (this.isShown || this.is_visible);
            }
        },
        watch: {
            visible(new_val, old_val) {
                if (new_val === old_val || this.isTransitioning) {
                    return;
                }

                if (new_val) {
                    this.show();
                } else {
                    this.hide();
                }
            }
        },
        props: {
            id: {
                type: String,
                default: null
            },
            visible: {
                type: Boolean,
                default: false
            },
            title: {
                type: String,
                default: ''
            },
            titleTag: {
                type: String,
                default: 'h5'
            },
            size: {
                type: String,
                default: 'md'
            },
            buttonSize: {
                type: String,
                default: 'md'
            },
            noFade: {
                type: Boolean,
                default: false
            },
            noCloseOnBackdrop: {
                type: Boolean,
                default: false
            },
            noCloseOnEsc: {
                type: Boolean,
                default: false
            },
            noAutoFocus: {
                type: Boolean,
                default: false
            },
            noEnforceFocus: {
                type: Boolean,
                default: false
            },
            hideHeader: {
                type: Boolean,
                default: false
            },
            hideFooter: {
                type: Boolean,
                default: false
            },
            hideHeaderClose: {
                type: Boolean,
                default: false
            },
            okOnly: {
                type: Boolean,
                default: false
            },
            okDisabled: {
                type: Boolean,
                default: false
            },
            returnFocus: {
                default: null
            },
            headerCloseLabel: {
                type: String,
                default: 'Close'
            },
            closeTitle: {
                type: String,
                default: 'Close'
            },
            okTitle: {
                type: String,
                default: 'OK'
            }
        },
        methods: {
            show() {
                if (this.is_visible) {
                    return;
                }
                this.is_visible = true;
                if (this.noFade) {
                    // If no fade animation, tehn triger the start ourselves
                    this.onBeforeEnter();
                }
                this.body.classList.add('modal-open');
                this.$emit('change', true);
                if (this.noFade) {
                    // If no fade animation, tehn triger the end ourselves
                    this.onAfterEnter()
                }
                this.setListeners(ON);
            },
            hide(isOK) {
                if (!this.is_visible) {
                    return;
                }

                // Create event object
                let canceled = false;
                const e = {
                    isOK,
                    cancel() {
                        canceled = true;
                    }
                };

                // Emit events
                this.$emit('change', false);
                this.$emit('hide', e);

                if (isOK === true) {
                    this.$emit('ok', e);
                } else if (isOK === false) {
                    this.$emit('cancel', e);
                }

                // Hide if not canceled
                if (!canceled) {
                    this.setListeners(OFF);
                    this.is_visible = false;
                    this.$root.$emit('hidden::modal', this.id);
                    this.$emit('hidden', e);
                    this.body.classList.remove('modal-open');
                    this.returnFocusTo();
                }
            },
            onClickOut() {
                // If backdrop clicked, hide modal
                if (this.is_visible && !this.noCloseOnBackdrop) {
                    this.hide();
                }
            },
            onEsc() {
                // If ESC pressed, hide modal
                if (this.is_visible && !this.noCloseOnEsc) {
                    this.hide();
                }
            },
            onBeforeEnter(el) {
                this.$emit('show');
                this.$root.$emit('shown::modal', this.id);
                this.isEntering = true;
            },
            onEnter(el) {
                // Add display:block after Vue v-show rmoves display:none
                // Change this to a CSS data prop and apply as :style=
                this.$refs.modal.style.display='block';
            },
            onAfterEnter() {
                this.isEntering = false;
                this.isShown = true;
                this.$emit('shown');
                this.focusFirst();
            },
            onBeforeLeave() {
                this.isLeaving = true;
                this.isShown = false;
            },
            onLeave(el) {
            },
            onAfterLeave() {
                this.isLeaving = false;
                this.$emit('hidden');
            },
            getScrollbarWidth() {
                if (typeof document === 'undefined') {
                    return 0;
                }
                const scrollDiv = document.createElement('div');
                scrollDiv.className = ClassName.SCROLLBAR_MEASURER;
                document.body.appendChild(scrollDiv);
                this.scrollbarWidth = scrollDiv.getBoundingClientRect().width - scrollDiv.clientWidth;
                document.body.removeChild(scrollDiv);
                return this.scrollbarWidth;
            },
            focusFirst() {
                // Don't try and focus if we are SSR
                if (typeof document === 'undefined') {
                    return;
                }
                this.$nextTick(() => {
                    // If activeElement is child of content, no need to change focus
                    if (document.activeElement && this.$refs.content.contains(document.activeElement)) {
                        return;
                    }

                    let el;
                    if (!this.noAutoFocus) {
                        // Focus the modal's first focusable item, searching body, footer, then header
                        if (this.$refs.body) {
                            el = findFirstVisible(this.$refs.body, FOCUS_SELECTOR);
                        }
                        if (!el && this.$refs.footer) {
                            el = findFirstVisible(this.$refs.footer, FOCUS_SELECTOR);
                        }
                        if (!el && this.$refs.header) {
                            el = findFirstVisible(this.$refs.header, FOCUS_SELECTOR);
                        }
                    }
                    if (!el) {
                        // Focus the modal content wrapper
                        el = this.$refs.content;
                    }
                    if (el && el.focus) {
                        el.focus();
                    }
                });
            },
            returnFocusTo() {
                if (typeof document === 'undefined') {
                    return;
                }

                // Prrefer returnFocus prop over event specified value
                let el = this.returnFocus || this.return_focus || null;

                if (el) {
                    if (typeof el === 'string') {
                        // CSS Selector
                        el = document.querySelector(el);
                    }
                    if (el && el.$el && typeof el.$el.focus === 'function') {
                        // Component vm reference
                        el.$el.focus();
                    } else if (el && typeof el.focus === 'function') {
                        // Plain element
                        el.focus();
                    }
                }
            },
            setListeners(on) {
                if (typeof document === 'undefined') {
                    return;
                }
                if (on) {
                    // Guard against infinite focus loop
                    document.removeEventListener('focusin', this.enforceFocus, false);
                    // Handle constrained focus
                    document.addEventListener('focusin', this.enforceFocus, false);
                    // Add resize listeners
                    window.addEventListener('resize', this.handeSize, false);
                    window.addEventListener('orientationchange', this.handeSize, false);
                } else {
                    // Remove focus handler
                    document.removeEventListener('focusin', this.enforceFocus, false);
                    // Remove resize listeners
                    window.removeEventListener('resize', this.handleSize, false);
                    window.removeEventListener('orientationchange', this.handleSize, false);
                }
            },
            enforceFocus(e) {
                // If focus leaves modal, bring it back
                // Event Listener bound on document
                if (!this.noEnforceFocus &&
                    this.is_visible &&
                    document !== e.target &&
                    this.$refs.content &&
                    this.$refs.content !== e.target &&
                    !this.$refs.content.contains(e.target)) {
                    this.$refs.content.focus();
                }
            },
            showHandler(id, triggerEl) {
                if (id === this.id) {
                    this.return_focus = triggerEl || null;
                    this.show();
                }
            },
            hideHandler(id) {
                if (id === this.id) {
                    this.hide();
                }
            }
        },
        created() {
            this.listenOnRoot('show::modal', this.showHandler);
            this.listenOnRoot('hide::modal', this.hideHandler);
        },
        mounted() {
            // calculate hte scrollbar width once.
            this.getScrollbarWidth();
            // Initially show modal?
            if (this.visible === true) {
                this.show();
            }
        },
        destroyed() {
            // Make sure global event listenerers are rmoved
            this.setListeners(OFF);
        }
    };

</script>
