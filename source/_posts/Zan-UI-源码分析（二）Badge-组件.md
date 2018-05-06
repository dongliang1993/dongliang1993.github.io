---
title: Zan UI 源码分析（二）Badge 组件
date: 2017-12-18 09:59:41
tags:
---
<!-- more -->

```js
import React, { PureComponent, Component } from 'react';
import cx from 'classnames';
import PropTypes from 'prop-types';

// Badge 就是小红点或者未读消息数
export default class Badge extends (PureComponent || Component) {
  static propTypes = {
    count: PropTypes.number, // 消息条数
    maxCount: PropTypes.number, // 最大完全显示消息条数	
    dot: PropTypes.bool, // 是否显示为小红点	
    showZero: PropTypes.bool, // 消息数为0时是否显示	
    children: PropTypes.node, // 自定义额外类名	
    className: PropTypes.string,
    prefix: PropTypes.string // 自定义前缀	
  };

  static defaultProps = {
    count: 0,
    maxCount: 99,
    dot: false,
    showZero: false,
    className: '',
    prefix: 'zent'
  };

  render() {
    const {
      count,
      maxCount,
      dot,
      showZero,
      className,
      prefix,
      children
    } = this.props;
    // 如果 children 的个数 >= 2 那么 是个数组
    // 否则是个对象
    const containerCls = cx({
      [`${prefix}-badge`]: true,
      // 是否是独立徽标的切换
      // 也就是说，如果你传进来一个 children
      // 那么就认为不是 独立徽标
      // 小红点或者消息数要以 children 来绝对定位
      // 如果没有 children ，就是独立徽标
      // 变成一个 inlineblock
      [`${prefix}-badge-none-cont`]: !children,
      [className]: !!className
    });

    // 不过有一点不明白
    // 这函数为什么要写在 render 中？
    // 这样不是每次 render 的时候都要重新生成一下这个函数吗？
    const renderCount = () => {
      let countEle = null;
      // 如果需要显示小红点
      if (dot) {
        countEle = (
          <span className={`${prefix}-badge-count ${prefix}-badge-dot`} />
        );
      } else if (count > 0 || (count === 0 && showZero)) {
        // 如果消息数 > 0 或者 count === 0 但是需要消息数为0时是否显示	
        countEle = (
          <span className={`${prefix}-badge-count`}>
            {/* 如果 count 比 设置的 maxCount 要多，显示 maxCount  否则显示 count 数*/}
            {count > maxCount ? `${maxCount}+` : count}
          </span>
        );
      }
      return countEle;
    };

    return (
      <div className={containerCls}>
        {children ? (
          <div className={`${prefix}-badge-content`}>{children}</div>
        ) : null}
        {/* renderCount 判断是生成小红点还是消息数  */}
        {renderCount()}
      </div>
    );
  }
}

```