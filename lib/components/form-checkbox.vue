<template>
<div :role="multiple ? 'group' : null"
     :id="id || null"
     :class="buttons ? btnGroupClasses : checkGroupClasses"
>
    <label v-for="(option, idx) in checkOptions"
           :class="(button || buttons) ? btnLabelClasses : labelClasses"
           :aria-pressed="(button || buttons) ? (isChecked(option.value) ? 'true' : 'false') : null"
    >
        <input type="checkbox"
               :id="id ? `__BV_checkbox_${id}_${idx}_` :  null"
               :name="name"
               :value="option.value"
               :disabled="disabled || option.disabled"
               :required="multiple ? null : required"
               autocomplete="off"
               :aria-required="multiple ? null : (required ? 'true' : null)"
               :class="(custom && !isButtons) ? 'custom-control-input' : null"
               :checked="isChecked(option.value)"
               @focusin.native="handleFocus"
               @focusout.native="handleFocus"
               @change="handleChange">
        <span v-if="custom && !isButtons"
              class="custom-control-indicator"
              aria-hidden="true"
        ></span>
        <span :class="(custom && !isButtons) ? 'custom-control-description' : null">
            <template v-if="!multiple">
                <slot></slot>
            </templat>
        </span>
    </label>
</div>
</template>

<script>
import { formMixin, formCustomMixin, formCheckBoxMixin } from '../mixins';
import { arrayIncludes, isArray } from '../utils/array';

export default {
    mixins: [formMixin, formCustomMixin, formCheckBoxMixin],
    model: {
        prop: 'checked',
        event: 'change' // chagned from `change`
    },
    data() {
        return {
            localValue: this.multiple ? (this.checked || []) : this.cheked;
        };
    },
    props: {
        value: {
            default: true
        },
        uncheckedValue: {
            default: false
        },
        checked: {
            default: true
        },
        indeterminate: {
            type: Boolean,
            default: false,
        },
        size: {
            type: String,
            default: null
        },
        inline: {
            type: Boolean,
            default: false
        },
        button: {
            type: Boolean,
            default: false,
        },
        buttonVariant: {
            type: String,
            default: 'secondary',
        }
    },
    computed: {
        multiple() {
            return this.options ? true : false;
        },
        isButtons() {
            return this.button || this.buttons;
        },
        checkGroupClasses() {
            if (this.multiple) {
                return [
                    this.size ? `form-control-${this.size}` : null,
                    this.state ? `has-${this.state}` : '',
                    this.stacked ? 'custom-controls-stacked' : ''
               ];
            }
            return [];
        },
        btnGroupClasses() {
            if (this.multiple) {
                return [
                    this.size ? `button-group-${this.size}` : null,
                    this.stacked ? 'btn-group-vertical' : ''
                 ];
               ];
            }
            return [];
        },
        checkOptions() {
            return this.multiple ? this.formOptions : [ { value: this.value } ];
        },
        labelClasses() {
            return [
                this.size ? `form-control-${this.size}` : '',
                this.custom ? 'custom-checkbox' : '',
                this.checkboxClass
            ];
        },
        btnLabelClasses() {
            return [
                'btn',
                `btn-${this.buttonVariant}`,
                (this.size && !this.multiple ) ? `btn-${this.size}` : '',
                this.isChecked ? 'active' : '',
                this.disabled ? 'disabled' : ''
            ];
        }
    },
    watch: {
        indeterminate(newVal, oldVal) {
            this.setIndeterminate(newVal);
        }
    },
    methods: {
        isChecked(val) {
            if (isArray(this.checked)) {
                return arrayIncludes(this.checked, val);
            } else {
                return this.checked === val;
            }
        },
        handleChange({ target: { checked } }) {
            if (isArray(this.checked)) {
                if (this.isChecked) {
                    this.$emit('change', this.checked.filter(x => x !== this.value));
                } else {
                    this.$emit('change', [...this.checked, this.value]);
                }
            } else {
                this.$emit('change', checked ? this.value : this.uncheckedValue)
            }
            this.$emit('update:indeterminate', this.$refs.check.indeterminate);
        },
        setIndeterminate(state) {
            this.$refs.check.indeterminate = state;
            // Emit update event to prop
            this.$emit('update:indeterminate', this.$refs.check.indeterminate);
        },
        handleFocus(evt) {
            // Add or remove 'focus' class on label in button mode
            if (this.button || this.buttons) {
                if (evt.type === 'focusin') {
                    evt.target.classList.add('focus');
                } else if (evt.type === 'focusout') {
                    evt.target.classList.remove('focus');
                }
            }
        }
    },
    mounted() {
        // Set initial indeterminate state
        this.setIndeterminate(this.indeterminate);
    }
};

</script>
