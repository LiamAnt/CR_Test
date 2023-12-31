#題目3

<template>
  <div
    class="gdb-input"
    :class="{
      [`gdb-input--${size}`]: true,
      'gdb-input--error': state.isError,
      'gdb-input--password': state.inputType === 'password',
      'gdb-input--disabled': state.isDisabled,
      'gdb-input--full': fullWidth,
    }"
  >
    <div class="gdb-input__inner" :class="{ 'gdb-input__inner--focus': isFocus }">
      <div v-if="state.hasPrefix" class="gdb-input__prefix">{{ prefix }}</div>
      <div class="gdb-input__input-wrapper">
        <input
          v-model="value"
          :type="state.inputType"
          class="gdb-input__input"
          :placeholder="placeholder"
          :disabled="state.isDisabled"
          @focus="focusEvent"
          @blur="blurEvent"
          @change="changeEvent"
        />
        <div v-if="type === 'password'" class="gdb-input__password" @click="isShowPassword = !isShowPassword">
          <span v-if="isShowPassword" class="gdb-input__password-icon material-icons-round">visibility</span>
          <span v-else class="gdb-input__password-icon material-icons-round">visibility_off</span>
        </div>
      </div>
      <div v-if="state.hasSuffix" class="gdb-input__suffix">{{ suffix }}</div>
    </div>
  </div>
</template>

<script>

// 對每個區塊加註解增加易讀性
import { isEmpty } from 'lodash-es'
import { isFieldError } from '@/components/form/GDBFormItem.vue'
import type { PropType } from 'vue'

export default defineComponent({
  name: 'GDBInput',
  props: {
    modelValue: {
      type: [String, Number] as PropType<string | number | null>,
      default: null,
    },
    type: {
      type: String,
      default: 'text', // text | password | number
    },
    placeholder: {
      type: String,
      default: '',
    },
    prefix: {
      type: String,
      default: '',
    },
    suffix: {
      type: String,
      default: '',
    },
    max: {
      type: Number,
      default: Infinity,
    },
    min: {
      type: Number,
      default: -Infinity,
    },
    hasDecimalPoint: {
      type: Boolean,
      default: false,
    },
    size: {
      type: String as PropType<'small' | 'medium'>,
      default: 'medium',
    },
    fullWidth: {
      type: Boolean,
      default: false,
    },
    emptyToNull: {
      type: Boolean,
      default: true,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
  },
  emits: ['update:modelValue', 'change'],
  setup(props, { emit }) {
    // 僅命名 value 容易混淆，可在前面加個形容詞，例如 inputValue
    const value = computed({
      // getter 把 .value 寫出來，後面使用比較方便
      get: () => props.modelValue,
      set: (val) => {
        switch (true) {
          case props.disabled:
            break
          case props.type === 'number':
            if (val === '') {
              emit('update:modelValue', val)
              return
            }
            // Number 轉型失敗的例外處理
            emit('update:modelValue', Number(val))
            break
          case props.emptyToNull && val === '':
            emit('update:modelValue', null)
            break
          default:
            emit('update:modelValue', val)
            break
        }
      },
    })

    const isShowPassword = ref(false)
    const isFocus = ref(false)

    const isError = inject(isFieldError, readonly(computed(() => false)))

    const state = computed(() => {
      return {
        // 如果需要兩層三元表達式，改用 if else 寫可讀性較佳
        inputType: props.type === 'password' ? (isShowPassword.value ? 'text' : 'password') : props.type,
        isError: isError.value,
        isDisabled: props.disabled,
        hasPrefix: !isEmpty(props.prefix),
        hasSuffix: !isEmpty(props.suffix),
      }
    })

    const changeEvent = () => {
      // 改用 if 判斷較易讀
      !props.disabled && emit('change', value.value)
    }

    const focusEvent = () => {
      isFocus.value = true
    }

    const blurEvent = () => {
      const { max, min, hasDecimalPoint, type } = props
      // 使用有括號包覆的 if、Number 轉型失敗的例外處理
      if (type !== 'number') return
      if (value.value === '') value.value = 0
      if (!hasDecimalPoint) value.value = Math.floor(Number(value.value))
      if (Number(value.value) < min) value.value = min
      if (Number(value.value) > max) value.value = max

      isFocus.value = false
    }

    return {
      value,
      isShowPassword,
      isFocus,
      state,
      changeEvent,
      focusEvent,
      blurEvent,
    }
  },
})
</script>

<style>

.gdb-input {
  $self: &;

  position: relative;
  display: block;
  width: 100%;
  max-width: 400px;

  &--full {
    max-width: unset;
  }

  &--medium {
    #{$self}__inner {
      height: 50px;
    }
  }

  &--small {
    #{$self}__inner {
      height: 40px;
    }
  }

  &__inner {
    display: flex;
    width: 100%;
    overflow: hidden;
    border: 1px map-get($theme-colors, gray-3) solid;
    border-radius: map-get($border-radius, s);

    &--focus {
      border-color: map-get($theme-colors, green-1);
    }
  }

  &__prefix,
  &__suffix {
    @extend %font-s-default-regular;
    @extend %px-l;

    display: flex;
    flex: 1 0 max-content;
    align-items: center;
    height: 100%;
    color: map-get($theme-colors, white);
    background-color: map-get($theme-colors, gray-3);
  }

  &__input-wrapper {
    width: 100%;
    height: 100%;
    background-color: map-get($theme-colors, white);
  }

  &__input {
    @extend %font-m-default-regular;
    @extend %px-l;

    width: 100%;
    height: 100%;
    color: map-get($theme-colors, gray-1);
    border: unset;

    &:focus {
      outline: unset;
    }

    &:disabled {
      color: map-get($theme-colors, gray-1);
      background-color: unset;
    }

    &::placeholder {
      color: map-get($theme-colors, gray-3);
    }
  }

  &__password {
    position: absolute;
    top: 0;
    right: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 40px;
    height: 100%;
    cursor: pointer;
  }

  &__password-icon {
    @extend %icon-s;

    color: map-get($theme-colors, gray-3);
  }

  &--password {
    #{$self}__input-wrapper {
      padding-right: 40px;
    }
  }

  &--error {
    #{$self}__inner {
      border-color: map-get($theme-colors, red-1);
    }
  }

  &--disabled {
    #{$self}__input-wrapper {
      background-color: map-get($theme-colors, gray-5);
    }
  }
}
    
</style>