<script>
import Icon from '../Icon'
import Button from '../Button'
import Option from './Option'
import OptionGroup from './OptionGroup'
import Overlay from '../Overlay'
import config from '../../managers/config'
import ui from '../../mixins/ui'
import input from '../../mixins/input'
import dropdown from '../../mixins/dropdown'
import keySelect from '../../mixins/key-select'
import warn from '../../utils/warn'
import { walk } from '../../utils/data'
import { uniqueId, cloneDeep, mapValues } from 'lodash'
import '../../common/uiTypes'

config.defaults(
  {
    placeholder: '@@select.placeholder'
  },
  'select'
)

export default {
  name: 'veui-select',
  uiTypes: ['select'],
  mixins: [ui, input, dropdown, keySelect],
  model: {
    event: 'change'
  },
  props: {
    /* eslint-disable vue/require-prop-types */
    value: {},
    /* eslint-ensable vue/require-prop-types */
    placeholder: String,
    clearable: Boolean,
    options: Array,
    filter: Function
  },
  data () {
    return {
      labelId: uniqueId('veui-select-label-'),
      localValue: this.value,
      outsideRefs: ['button'],
      initOptionLabel: ''
    }
  },
  computed: {
    optionMap () {
      return this.extractOptions()
    },
    labelMap () {
      return mapValues(this.optionMap, 'label')
    },
    selectedOption () {
      if (this.localValue == null) {
        return null
      }
      return this.optionMap[this.localValue]
    },
    realPlaceholder () {
      return this.placeholder == null
        ? config.get('select.placeholder')
        : this.placeholder
    },
    label () {
      if (this.localValue == null) {
        return this.realPlaceholder
      }
      if (this.options) {
        return this.labelMap[this.localValue]
      }
      if (this.$refs.options) {
        return this.findOptionsLabel()
      }
      return ''
    },
    realOptions () {
      if (typeof this.filter !== 'function') {
        return this.options
      }
      let filtered = cloneDeep(this.options)
      walk(filtered, option => {
        option.hidden = !this.filter(option)
      })
      return filtered
    }
  },
  watch: {
    value (val) {
      this.localValue = val
    },
    localValue (val) {
      if (this.value !== val) {
        this.$emit('change', val)
      }
    }
  },
  mounted () {
    /**
     * It cann't obtain 'options' refs in computed attribute
     * when the default slot component (OptionGroup) did not mounted.
     * This caused the label data cann't computed by $refs.options.find function.
     * So it recomputed option label on this component mounted.
     */
    if (this.localValue && !this.label) {
      this.initOptionLabel = this.findOptionsLabel()
    }
  },
  methods: {
    findOptionsLabel () {
      let option = this.$refs.options.find(this.localValue)
      return option ? option.label : ''
    },
    handleSelect (value) {
      this.expanded = false
      this.localValue = value
    },
    handleRelocate () {
      this.$refs.options.relocateDeep()
    },
    handleButtonClick () {
      this.expanded = !this.expanded
    },
    handleButtonKeydown (e) {
      if (
        e.key === 'Up' ||
        e.key === 'ArrowUp' ||
        e.key === 'Down' ||
        e.key === 'ArrowDown'
      ) {
        this.expanded = true
        e.stopPropagation()
        e.preventDefault()
      }
    },
    extractOptions () {
      let map = {}
      walk(
        this.options,
        option => {
          let { value } = option
          if (value != null) {
            if (map[value]) {
              warn(
                `[veui-select] Duplicate item value [${value}] for select options.`,
                this
              )
            }
            map[value] = option
          }
        },
        ['options', 'children']
      )
      return map
    },
    focus () {
      this.$refs.button.focus()
    }
  },
  render () {
    return (
      <div
        class={{
          'veui-select': true,
          'veui-select-empty': this.localValue == null,
          'veui-select-expanded': this.expanded,
          'veui-input-invalid': this.realInvalid
        }}
        ui={this.realUi}
        role="listbox"
        aria-owns={this.dropdownId}
        aria-readonly={this.realReadonly}
        aria-expanded={this.expanded}
        aria-disabled={this.realDisabled || this.realReadonly}
        aria-labelledby={this.labelId}
        aria-haspopup="listbox"
      >
        <Button
          ref="button"
          class="veui-select-button"
          disabled={this.realDisabled || this.realReadonly}
          onKeydown={this.handleButtonKeydown}
          onClick={this.handleButtonClick}
        >
          <span class="veui-select-label" id={this.labelId}>
            {this.$scopedSlots.label
              ? this.$scopedSlots.label(
                this.selectedOption || { selected: false }
              )
              : this.label || this.initOptionLabel}
          </span>
          <Icon
            class="veui-select-icon"
            name={this.icons[this.expanded ? 'collapse' : 'expand']}
          />
        </Button>
        {
          <Overlay
            v-show={this.expanded}
            target="button"
            open={this.expanded}
            autofocus
            modal
            options={this.realOverlayOptions}
            overlay-class={this.overlayClass}
            onLocate={this.handleRelocate}
          >
            <div
              id={this.dropdownId}
              ref="box"
              class="veui-select-options"
              {...{
                directives: [
                  {
                    name: 'outside',
                    value: {
                      refs: this.outsideRefs,
                      handler: this.close
                    }
                  }
                ]
              }}
              tabindex="-1"
              ui={this.realUi}
              role="listbox"
              onKeydown={this.handleKeydown}
            >
              {this.$slots.before}
              {this.clearable ? (
                <Option value={null} label={this.realPlaceholder} />
              ) : null}
              <OptionGroup
                ref="options"
                options={this.realOptions}
                ui={this.realUi}
                scopedSlots={{
                  label: this.$scopedSlots['group-label'] || null,
                  option: this.$scopedSlots.option || null,
                  'option-label': this.$scopedSlots['option-label'] || null
                }}
              >
                {this.$slots.default}
              </OptionGroup>
              {this.$slots.after}
            </div>
          </Overlay>
        }
      </div>
    )
  }
}
</script>
