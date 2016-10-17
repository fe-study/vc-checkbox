<template>
    <div class="vc-checkbox-component" :class="{ 'group-item': group }">
        <label 
            v-show="buttonStyle"
            :class="['btn', 'btn-' + typeColor, { 'active': checked, 'disabled': disabled, 'readonly': readonly }]"
            @click.prevent="toggle"
            :title="title"
        >
            <input type="checkbox" autocomplete="off"
                v-el:input
                v-show="!readonly"
                :name="name"
                :value="value"
                :checked="active"
                :disabled="disabled"
                :readonly="readonly"
            />
            <slot>{{ label }}</slot>
        </label>
        <div v-else
            :class="['checkbox', typetypeColor, { 'active': checked, 'disabled': disabled, 'readonly': readonly }]"
            @click.prevent="toggle"
        >
            <label class="open" :title="title">
                <input type="checkbox" autocomplete="off"
                    v-el:input
                    :name="name"
                    :value="value"
                    :checked="active"
                    :disabled="disabled"
                    :readonly="readonly"
                />
                <span class="icon" :class="[active ? 'btn-' + typeColor : '', { 'bg': typeColor === 'default' }]"></span>
                <span v-if="active && typeColor === 'default'" class="icon"></span>
                <span class="label-content"><slot>{{ label }}</slot></span>
            </label>
        </div>
    </div>
</template>

<style lang="less">
.vc-checkbox-component {
    display: inline-block;

    &.group-item {
        float: left;
    }

    // 非group模式的重置margin
    .open {
        margin-right: 5px;
    }

    span.label-content {
        display: inline-block;
        position: relative;
        top: -1px;
        left: -3px;
        height: 20px;
        line-height: 20px;
        font-size: 14px;
        vertical-align: baseline;
    }
}
.checkbox { 
    position: relative;
}
.checkbox > label > input {
    box-sizing: border-box;
    position: absolute;
    z-index: -1;
    padding: 0;
    opacity: 0;
    margin: 0;
}
.checkbox > label > .icon {
    position: absolute;
    top: 2px;
    left: 0;
    display: block;
    width: 14px;
    height: 14px;
    line-height: 10px;
    text-align: center;
    user-select: none;
    border-radius: 4px;
    background-repeat: no-repeat;
    background-position: center center;
    background-size: 50% 50%;
}
.checkbox:not(.active) > label > .icon {
    background-color: #ddd;
    border: 1px solid #bbb;
}
.checkbox > label > input:focus ~ .icon {
    outline: 0;
    // border: 1px solid #66afe9;
    // box-shadow: inset 0 1px 1px rgba(0,0,0,.075),0 0 8px rgba(102,175,233,.6);
}
.checkbox.active > label > .icon {
    background-size: 10px 10px;
    background-image: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4NCjxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB3aWR0aD0iNyIgaGVpZ2h0PSI3Ij48cGF0aCBmaWxsPSIjZmZmIiBkPSJtNS43MywwLjUybC0zLjEyNDIyLDMuMzQxNjFsLTEuMzM4OTUsLTEuNDMyMTJsLTEuMjQ5NjksMS4zMzY2NWwyLjU4ODYzLDIuNzY4NzZsNC4zNzM5LC00LjY3ODI2bC0xLjI0OTY5LC0xLjMzNjY1bDAsMGwwLjAwMDAyLDAuMDAwMDF6Ii8+PC9zdmc+);
}
.checkbox.active .btn-default { filter: brightness(75%); }
.checkbox.disabled > label > .icon,
.checkbox.readonly > label > .icon,
.btn.readonly {
    filter: alpha(opacity=65);
    box-shadow: none;
    opacity: .65;
}
label.btn > input[type=checkbox] {
    position: absolute;
    clip: rect(0,0,0,0);
    pointer-events: none;
}
</style>

<script>
export default {
    name: 'vc-checkbox',
    props: {
        name: {
            type: String,
            default: null
        },
        label: [String, Number],
        value: {
            default: true
        },
        checked: {
            twoWay: true
        },
        button: {
            type: Boolean,
            default: false
        },
        disabled: {
            type: Boolean,
            default: false
        },
        readonly: {
            type: Boolean,
            default: false
        },
        type: {
            type: String,
            default: null
        }
    },
    data () {
        return {
            title: ''
        }
    },
    computed: {
        active () {
            return typeof this.value !== 'boolean' && this.group ? ~this.$parent.value.indexOf(this.value) : this.checked === this.value
        },
        buttonStyle () {
            return this.button || (this.group && this.$parent.buttons)
        },
        group () {
            return this.$parent && this.$parent._checkboxGroup
        },
        typeColor () {
            return (this.type || (this.$parent && this.$parent.type)) || 'default'
        }
    },
    watch: {
        checked (val) {
            if (typeof this.value !== 'boolean' && this.group) {
                if (this.checked && !~this.$parent.value.indexOf(this.value)) this.$parent.value.push(this.value)
                if (!this.checked && ~this.$parent.value.indexOf(this.value)) this.$parent.value.$remove(this.value)
            }
        }
    },
    created () {
        if (typeof this.value === 'boolean') { return }
        const parent = this.$parent
        if (parent && parent._btnGroup && !parent._radioGroup) {
            parent._checkboxGroup = true
            if (!(parent.value instanceof Array)) { parent.value = [] }
        }
    },
    ready () {
        this.setTitleAttr()
        if (!this.$parent._checkboxGroup || typeof this.value === 'boolean') { return }
        if (this.$parent.value.length) {
            this.checked = ~this.$parent.value.indexOf(this.value)
        } else if (this.checked) {
            this.$parent.value.push(this.value)
        }
    },
    methods: {
        setTitleAttr () {
            try {
                this.title = this._slotContents.default.textContent.trim() || this.label
            } catch (e) {
                // console.error('vc-checkbox title getter error', e)
                if (this.label && this.label !== '') {
                    this.title = this.label
                }
            }
        },
        eval () {
            if (typeof this.value !== 'boolean' && this.group) {
                this.checked = ~this.$parent.value.indexOf(this.value)
            }
        },
        focus () {
            this.$els.input.focus()
        },
        toggle () {
            if (!this.disabled) {
                this.focus()
                if (!this.readonly) {
                    this.checked = this.checked ? null : this.value
                    if (this.group && typeof this.value !== 'boolean') {
                        const index = this.$parent.value.indexOf(this.value)
                        this.$parent.value[~index ? '$remove' : 'push'](this.value)
                    }
                }
            }
            return false
        }
    }
}
</script>

