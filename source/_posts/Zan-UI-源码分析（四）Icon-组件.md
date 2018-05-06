---
title: Zan UI 源码分析（四）Icon 组件
date: 2017-12-18 15:09:57
tags:
---
<!-- more -->

```js
import React, { Component, PureComponent } from 'react';
import PropTypes from 'prop-types';
import cx from 'classnames';

export default class Icon extends (PureComponent || Component) {
  static propTypes = {
    type: PropTypes.string.isRequired, // 图标类型	
    className: PropTypes.string, // 自定义额外类名	
    spin: PropTypes.bool
  };

  static defaultProps = {
    className: '',
    spin: false
  };

  render() {
    const { type, className, spin, ...otherProps } = this.props;
    const cls = cx('zenticon', `zenticon-${type}`, className, {
      'zenticon-spin': spin
    });

    return <i className={cls} {...otherProps} />;
  }
}

```