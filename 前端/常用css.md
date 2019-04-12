#### 多行文本溢出，超出两行（n 行）溢出省略号

```scss
@mixin textOverflow($line: 1) {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  /*! autoprefixer: off */
  -webkit-box-orient: vertical;
  -webkit-line-clamp: $line;
}
```

#### 文字两端居中（司徒正美方案）

```css
.test1 {
  text-align: justify;
  text-justify: distribute-all-lines; /*ie6-8*/
  text-align-last: justify; /* ie9*/
  -moz-text-align-last: justify; /*ff*/
  -webkit-text-align-last: justify; /*chrome 20+*/
}

@media screen and (-webkit-min-device-pixel-ratio: 0) {
  /* chrome*/
  .test1:after {
    content: ".";
    display: inline-block;
    width: 100%;
    overflow: hidden;
    height: 0;
  }
}
```

#### 平滑滚动
```css
scroll-behavior:smooth;
```
