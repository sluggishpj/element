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

> https://www.sassmeister.com/
