---
title: Zan UI 源码分析（四）Button 组件
date: 2017-12-18 15:04:03
tags:
---
<!-- more -->

```js
// 这部分代码写的很乱
import React, { Component, PureComponent } from 'react';
// 这个山炮怎么又换了一种命名？
import setClass from 'classnames';
import PropTypes from 'prop-types';
// 这个函数会创建一个新的对象，然后把你出想去除的属性从对象中删除
// var object = { 'a': 1, 'b': '2', 'c': 3 };
// _.omit(object, ['a', 'c']);
// => { 'b': '2' }
import omit from 'lodash/omit';

const BLACK_LIST = [
  'type',
  'size',
  'htmlType',
  'block',
  'component',
  'disabled',
  'loading',
  'outline',
  'bordered',
  'className',
  'prefix'
];

// 为啥要这么写？有毒吧？
const BTN_BLACK_LIST = ['href', 'target'].concat(BLACK_LIST);

const A_BLACK_LIST = ['href', 'target'].concat(BLACK_LIST);

export default class Button extends (PureComponent || Component) {
  static propTypes = {
    type: PropTypes.oneOf(['default', 'primary', 'success', 'danger', 'link']),
    size: PropTypes.oneOf(['large', 'medium', 'small']),
    htmlType: PropTypes.oneOf(['button', 'submit', 'reset']),
    className: PropTypes.string,
    block: PropTypes.bool,
    component: PropTypes.oneOfType([PropTypes.string, PropTypes.func]),
    disabled: PropTypes.bool,
    loading: PropTypes.bool,
    outline: PropTypes.bool,
    bordered: PropTypes.bool,
    prefix: PropTypes.string
  };

  static defaultProps = {
    type: 'default', // 风格 'default'	'primary' 、 'danger' 、 'success'
    size: 'medium', // 尺寸 'medium'	'large' 、 'small'
    htmlType: 'button', // button标签原生type属性	'button'	submit 、 reset 、 button
    className: '', // 自定义类名	
    block: false,
    disabled: false, // 状态控制	false
    loading: false, // 状态控制	
    outline: false, // 边框有颜色，内部没有颜色	
    bordered: true, // 边框透明控制	
    prefix: 'zent'
  };

  constructor(props) {
    super(props);
  }

  // 处理点击事件
  handleClick = (event) => {
    if (this.props.disabled || this.props.loading) return;

    if (this.props.onClick) {
      this.props.onClick(event);
    }
  }

  // render a 标签
  renderLink = (classNames) => {
    // 如果没有 自定义组件标签类型 就是 a 标签
    const Node = this.props.component || 'a';
    // loading 和 disabled 都会禁用掉按钮
    const disabled = this.props.disabled || this.props.loading;
    const { href = '', target } = this.props;
    const nodeProps = omit(this.props, A_BLACK_LIST);

    return (
      <Node
        {...(disabled ? {} : { href, target })}
        {...nodeProps}
        className={classNames}
        onClick={this.handleClick}
      >
        {this.props.children}
      </Node>
    );
  }

  // render button 标签
  renderButton = (classNames) => {
    const Node = this.props.component || 'button';
    const disabled = this.props.disabled || this.props.loading;
    const htmlType = this.props.htmlType;
    const nodeProps = omit(this.props, BTN_BLACK_LIST);

    return (
      <Node
        {...nodeProps}
        {...(htmlType ? { type: htmlType } : {})}
        className={classNames}
        disabled={disabled}
        onClick={this.handleClick}
      >
        {this.props.children}
      </Node>
    );
  }

  render() {
    // 如果传进来了 href/target ，按钮就变成了链接
    let renderer =
      (this.props.href || this.props.target) ? 'renderLink' : 'renderButton';

    let {
      className,
      type,
      size,
      block,
      disabled,
      loading,
      outline,
      bordered,
      prefix
    } = this.props;
    let classNames = setClass(
      {
        [`${prefix}-btn-${type}${outline ? '-outline' : ''}`]:
          type !== 'default',
        [`${prefix}-btn-${size}`]: size !== 'medium',
        [`${prefix}-btn-block`]: block,
        [`${prefix}-btn-loading`]: loading,
        [`${prefix}-btn-disabled`]: disabled,
        [`${prefix}-btn-border-transparent`]: !bordered
      },
      `${prefix}-btn`,
      className
    );

    return this[renderer](classNames);
  }
}

```