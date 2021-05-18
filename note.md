## Layout 布局 [Row & Col]

#### 屏幕尺寸断点

```scss
// var.scss
/* Break-point
--------------------------*/
$--sm: 768px !default;
$--md: 992px !default;
$--lg: 1200px !default;
$--xl: 1920px !default;

$--breakpoints: (
  'xs': (
    max-width: $--sm - 1,
  ),
  'sm': (
    min-width: $--sm,
  ),
  'md': (
    min-width: $--md,
  ),
  'lg': (
    min-width: $--lg,
  ),
  'xl': (
    min-width: $--xl,
  ),
);
```

```scss
// mixins.scss
/* Break-points
 -------------------------- */
@mixin res($key, $map: $--breakpoints) {
  // 循环断点Map，如果存在则返回
  @if map-has-key($map, $key) {
    @media only screen and #{inspect(map-get($map, $key))} {
      @content;
    }
  } @else {
    @warn "Undefeined points: `#{$map}`";
  }
}
```

```scss
// col.scss
@include res(md) {
  .el-col-md-0 {
    display: none;
  }
  @for $i from 0 through 24 {
    .el-col-md-#{$i} {
      width: (1 / 24 * $i * 100) * 1%;
    }
  }
}
```

生成的 css 代码如下：

```css
@media only screen and (min-width: 992px) {
  .el-col-md-0 {
    display: none;
  }
  .el-col-md-0 {
    width: 0%;
  }
  .el-col-md-1 {
    width: 4.1666666667%;
  }
  .el-col-md-2 {
    width: 8.3333333333%;
  }
  /* 3~23 省略了 */
  .el-col-md-24 {
    width: 100%;
  }
}
```

## Container 布局容器【Container, Main, Header, Footer】

### @at-root

```scss
// config.scss
$namespace: 'el';
$state-prefix: 'is-';

// mixins.scss
@mixin b($block) {
  $B: $namespace + '-' + $block !global;

  .#{$B} {
    @content;
  }
}

@mixin when($state) {
  @at-root {
    &.#{$state-prefix + $state} {
      @content;
    }
  }
}

@include b(container) {
  display: flex;
  flex-direction: row;
  flex: 1;
  flex-basis: auto;
  box-sizing: border-box;
  min-width: 0;

  @include when(vertical) {
    flex-direction: column;
  }
}
```

=>

```css
.el-container {
  display: flex;
  flex-direction: row;
  flex: 1;
  flex-basis: auto;
  box-sizing: border-box;
  min-width: 0;
}
.el-container.is-vertical {
  flex-direction: column;
}
```

## Button 按钮

### 动态拓展字符串

```scss
$modifier-separator: '--';
@mixin m($modifier) {
  $selector: &;
  $currentSelector: '';
  @each $unit in $modifier {
    $currentSelector: #{$currentSelector + & + $modifier-separator + $unit + ','};
  }

  @at-root {
    #{$currentSelector} {
      @content;
    }
  }
}

button {
  @include m([ 'test', 'by' ]) {
    color: 'green';
  }
}
```

=> CSS

```css
button--test,
button--by {
  color: 'green';
}
```

## Radio 单选框

### BEM

```scss
// config.scss
$namespace: 'el';
$element-separator: '__';
$modifier-separator: '--';
$state-prefix: 'is-';

// function.scss
/* BEM support Func
 -------------------------- */
// selectorToString($namespace) => el
@function selectorToString($selector) {
  $selector: inspect($selector); // 获取变量的字符串形式(含"") => "el"
  $selector: str-slice($selector, 2, -2); // 去除前后2个" => el
  @return $selector;
}

// 判断 $selector 是否包含 $modifier-separator (即--)
@function containsModifier($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, $modifier-separator) {
    @return true;
  } @else {
    @return false;
  }
}

// 判断 $selector 是否包含 '.'+ $state-prefix (即 .is-)
@function containWhenFlag($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, '.' + $state-prefix) {
    @return true;
  } @else {
    @return false;
  }
}

// 判断 $selector 是否包含 ':'
@function containPseudoClass($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, ':') {
    @return true;
  } @else {
    @return false;
  }
}

// 包含 -- 或 .is- 或 :
@function hitAllSpecialNestRule($selector) {
  @return containsModifier($selector) or containWhenFlag($selector) or containPseudoClass($selector);
}

/* BEM eg. .el-radio__input--small
 -------------------------- */
@mixin b($block) {
  $B: $namespace + '-' + $block !global;

  .#{$B} {
    @content;
  }
}

@mixin e($element) {
  $E: $element !global;
  $selector: &;
  $currentSelector: '';
  @each $unit in $element {
    $currentSelector: #{$currentSelector + '.' + $B + $element-separator + $unit + ','};
  }

  @if hitAllSpecialNestRule($selector) {
    @at-root {
      #{$selector} {
        #{$currentSelector} {
          @content;
        }
      }
    }
  } @else {
    @at-root {
      #{$currentSelector} {
        @content;
      }
    }
  }
}

@mixin m($modifier) {
  $selector: &;
  $currentSelector: '';
  @each $unit in $modifier {
    $currentSelector: #{$currentSelector + & + $modifier-separator + $unit + ','};
  }

  @at-root {
    #{$currentSelector} {
      @content;
    }
  }
}

@include b(radio) {
  line-height: 1.5;
  @include e(input) {
    outline: none;
    @include m(small) {
      line-height: 1;
    }
  }
}

/* Test  hitAllSpecialNestRule 其实就是多嵌套一层*/
@include b('radio.is-disable') {
  @include e(input) {
    outline: none;
  }
}
```

=> CSS

```css
.el-radio {
  line-height: 1.5;
}
.el-radio__input {
  outline: none;
}
.el-radio__input--small {
  line-height: 1;
}

/* Test  hitAllSpecialNestRule 其实就是多嵌套一层*/
.el-radio.is-disable .el-radio.is-disable__input {
  outline: none;
}
```

### dispatch & broadcast

触发特定组件的特定事件

```js
// src/mixins/emitter.js

function broadcast(componentName, eventName, params) {
  this.$children.forEach((child) => {
    var name = child.$options.componentName;

    if (name === componentName) {
      child.$emit.apply(child, [eventName].concat(params));
    } else {
      broadcast.apply(child, [componentName, eventName].concat([params]));
    }
  });
}

export default {
  methods: {
    // 找到离自己最近的componentName祖先元素，触发其 eventName 事件，参数是 ...params
    dispatch(componentName, eventName, params) {
      var parent = this.$parent || this.$root;
      var name = parent.$options.componentName;

      while (parent && (!name || name !== componentName)) {
        parent = parent.$parent;

        if (parent) {
          name = parent.$options.componentName;
        }
      }
      if (parent) {
        parent.$emit.apply(parent, [eventName].concat(params));
      }
    },
    // 找到自己包含的所有componentName子元素，触发其 eventName 事件，参数是 ...params
    broadcast(componentName, eventName, params) {
      broadcast.call(this, componentName, eventName, params);
    },
  },
};
```

> https://www.sassmeister.com/

## Checkbox 多选框

### v-model 使用 computed 属性，拦截 set 操作

```vue
<template>
  <input type="checkbox" :value="label" :name="name" v-model="model" />
</template>

<script>
export default {
  computed: {
    model: {
      get() {
        return this.isGroup ? this.store : this.value !== undefined ? this.value : this.selfModel;
      },

      set(val) {
        this.isLimitExceeded = false;
        this._checkboxGroup.min !== undefined &&
          val.length < this._checkboxGroup.min &&
          (this.isLimitExceeded = true);

        this._checkboxGroup.max !== undefined &&
          val.length > this._checkboxGroup.max &&
          (this.isLimitExceeded = true);

        // 条件满足后才触发实际修改
        this.isLimitExceeded === false && this.dispatch('ElCheckboxGroup', 'input', [val]);
      },
    },
  },
};
</script>
```

## input 输入框

### 文本合成事件

使用输入法编辑器 (IME) 输入中文/日文等，会触发文本合成事件。事件顺序：

`compositionstart` => `compositionstart` [value] => `input` [value] => `compositionstart` [value] => `input` [value] =>... => `compositionend` [value] => `input` [value]

<iframe height="325" style="width: 100%;" scrolling="no" title="composition event" src="https://codepen.io/sluggishpj/embed/poebXYX?height=325&theme-id=dark&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/sluggishpj/pen/poebXYX'>composition event</a> by pj
  (<a href='https://codepen.io/sluggishpj'>@sluggishpj</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### box-sizing

- `content-box`: `width` = 内容的宽度, `height` = 内容的高度
- `border-box`: `width` = `border` + `padding` + 内容的宽度, `height` = `border` + `padding` + 内容的高度

> https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-sizing

## el-scroll-bar

### Element.getBoundingClientRect()

方法返回元素的大小及其相对于视口的位置

