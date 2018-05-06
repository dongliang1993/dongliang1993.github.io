---
title: Zan UI 源码分析 Layout  组件
date: 2017-12-18 15:27:17
tags:
---

<!-- more -->

```js
// index.js
import Row from './Row';
import Col from './Col';

// 简单粗暴，Layout 就导出一个 Row，一个 Col
// 问题来了，为啥需要这个鬼东西在外面包一层？
const Layout = {
  Row,
  Col
};

export default Layout;

```

```js
// Row.js
import React, { Component, PureComponent } from 'react';
import PropTypes from 'prop-types';
import cx from 'classnames';

export default class Row extends (PureComponent || Component) {
  static propTypes = {
    className: PropTypes.string, // 额外的样式名	
    prefix: PropTypes.string // UI 前缀	
  };

  static defaultProps = {
    prefix: 'zent'
  };

  render() {
    const { className, prefix, ...others } = this.props;

    const classes = cx({
      [`${prefix}-row`]: true,
      // className 加上 [] 是为了防止有些不规范的命名不能设置成对象的属性名
      [className]: className
    });

    return (
      // 所以 Row 这个组件就是单纯的包了一层div。。。
      <div {...others} className={classes}>
        {this.props.children}
      </div>
    );
  }
}

```

```js
// Col.js
import React, { Component, PureComponent } from 'react';
import PropTypes from 'prop-types';
import cx from 'classnames';

export default class Col extends (PureComponent || Component) {
  static propTypes = {
    span: PropTypes.number, // col所占的栅格数	
    offset: PropTypes.number, // col左偏移的栅格数	
    className: PropTypes.string, // 额外的样式名	
    prefix: PropTypes.string // UI 前缀	
  };

  static defaultProps = {
    prefix: 'zent'
  };

  render() {
    const { span, offset, className, prefix, ...others } = this.props;
    // 看来有时间还要看看 css 那写文件
    const classes = cx({
      [`${prefix}-col`]: true,
      [`${prefix}-col-${span}`]: span,
      [`${prefix}-col-offset-${offset}`]: offset,
      [className]: className
    });

    return (
      <div {...others} className={classes}>
        {this.props.children}
      </div>
    );
  }
}

```