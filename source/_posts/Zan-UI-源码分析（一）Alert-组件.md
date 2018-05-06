---
title: Zan UI 源码分析（一）Alert 组件
date: 2017-12-15 16:32:33
tags:
---

本来想自己实现一套组件，后来想了想，如果一个人开发的话，成本太大，而且有很多工程化的东西不是很懂，所以作罢。

但是还是有很多优秀的组件源码是可以看一下的。有赞的技术团队算是我比较喜欢的，而我目前技术栈正好也是 react , 所以所以正好借此机会把 Zan UI 看一下，顺便水几篇文章:-D~~

接下来可能会写一系列文章来分析常用的组件的源码，当然，中间可能也会穿插一下其他的文章，欢迎大家交流~~

<!-- more -->

首先看一下 Zan UI 的目录结构

![](//dongliang1993.github.io/img/youzan-01.png)

我们向下翻一番，发现所有的组件都在这个文件夹：
![](//dongliang1993.github.io/img/youzan-02.png)

后面我们还会分析一下工程化的东西

```js
// 我们可以看到，基本上 Zan UI 都是尽可能的从 PureComponent
// 继承，这样可以尽可能的提高效率
// 下面是相关文章
// http://www.zcfy.cc/article/why-and-how-to-use-purecomponent-in-react-js-60devs-2344.html
import React, { Component, PureComponent } from 'react';
// 非常实用的一个库，可以用来动态切换类名
import cx from 'classnames';
// 用来检查 props 类型
import PropTypes from 'prop-types';
// 这样从 lodash 中引用函数最好
// 如果你直接 import { has } from 'lodash';
// 其实是把整个 lodash 的库都拿过来了
// 而下面这张写法只是引用了 lodash 文件下对应的函数
// 体积笑了很多
import isFunction from 'lodash/isFunction';

// 忽略不支持的style
// styleClassMap 是用来把 type 转换成对应的类名
const styleClassMap = {
  info: 'alert-style-info',
  warning: 'alert-style-warning',

  // error as an alias to danger
  error: 'alert-style-danger',
  danger: 'alert-style-danger'
};

// 忽略不支持的size
// sizeClassMap 是用来把 size 转换成对应的类名
const sizeClassMap = {
  normal: 'alert-size-normal',
  large: 'alert-size-large'
};

export default class Alert extends (PureComponent || Component) {
  // 定义接受的 props 的类型
  static propTypes = {
    type: PropTypes.oneOf(['info', 'warning', 'danger', 'error']).isRequired,
    size: PropTypes.oneOf(['large', 'normal']),
    rounded: PropTypes.bool,
    closable: PropTypes.bool,
    onClose: PropTypes.func,
    children: PropTypes.node,
    className: PropTypes.string,
    prefix: PropTypes.string
  };
  // 定义接受的 props 默认值
  static defaultProps = {
    type: 'info',
    size: 'normal',
    closable: false,
    rounded: false,
    className: '',
    prefix: 'zent'
  };

  state = {
    // 是否关闭了 Alert
    closed: false
  };

  onClose = () => {
    this.setState(
      {
        closed: true
      },
      () => {
        // onClose是在*关闭以后*被调用的
        const { onClose } = this.props;
        // 判断是否是函数，如果是函数，调用
        // 如果不是函数，忽略
        if (isFunction(onClose)) {
          onClose();
        }
      }
    );
  };

  render() {
    const { closed } = this.state;
    if (closed) {
      // 如果 Alert 是关闭的，直接返回 null
      return null;
    }

    const {
      type,
      prefix,
      rounded,
      className,
      closable,
      size,
      children
    } = this.props;
    // 我们可以看到，文件是没有引用css的
    const containerCls = cx(
      `${prefix}-alert`,
      `${prefix}-${styleClassMap[type]}`,
      `${prefix}-${sizeClassMap[size]}`,
      // 上面几个类名是一定会渲染的
      // 下面几个是根据对应的 true/false 渲染的
      {
        [className]: !!className,
        // 其实我有点想吐槽 rounded 这个 props
        // 因为 Zan UI 默认是 border-radius 是 4px
        // 这个数值有什么特殊么？
        // 还有就是，我也不能定制话的传入 px，如果想变成 npx 只能
        // 在边写样式层叠掉
        [`${prefix}-alert-border-rounded`]: rounded,
        [`${prefix}-alert-closable`]: closable
      }
    );

    return (
      <div className={containerCls}>
        {/* 是否有 X 按钮，是否可以关闭 */}
        {/* 如果 closable 为真，那么渲染 X */}
        {closable && (
          <div className={`${prefix}-alert-close-wrapper`}>
            <span
              className={`${prefix}-alert-close-btn`}
              onClick={this.onClose}
            >
              ×
            </span>
          </div>
        )}
        {/* 下面是内容的主体 */}
        <div className={`${prefix}-alert-content-wrapper`}>
          <div className={`${prefix}-alert-content`}>{children}</div>
        </div>
      </div>
    );
  }
}

```