---
title: Zan UI 源码分析（三）BlockHeader 组件
date: 2017-12-18 14:39:17
tags:
---
<!-- more -->

```js
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import cx from 'classnames';
import Pop from 'pop';
import Icon from 'icon';

export default class BlockHeader extends Component {
  static propTypes = {
    className: PropTypes.string, // 自定义类名	
    title: PropTypes.string.isRequired, // 标题
    tooltip: PropTypes.node, // pop显示内容	
    content: PropTypes.node, // 自定义content	
    position: PropTypes.string, // pop posotion	
    prefix: PropTypes.string // 自定义前缀	
  };

  static defaultProps = {
    prefix: 'zent',
    className: '',
    position: 'top-right',
    tooltip: '',
    content: ''
  };

  render() {
    const {
      prefix,
      content,
      title,
      tooltip,
      position,
      className,
      children
    } = this.props;
    return (
      <div className={cx(`${prefix}-block-header`, className)}>
        {title && (
          <div className={`${prefix}-block-header__left`}>
            <h3>{title}</h3>
          </div>
        )}
        <div className={`${prefix}-block-header__content`}>
          {/* 把传入的 content 和 children 都渲染出来 */}
          {content && content}
          {children && children}
        </div>
        <div className={`${prefix}-block-header__pop`}>
          {tooltip && (
            <Pop
              trigger="hover"
              centerArrow
              position={position}
              content={
                <p className={`${prefix}-block-header__tooltip`}>{tooltip}</p>
              }
              wrapperClassName={`${prefix}-block-header__tooltip-trigger`}
            >
              <Icon type="help-circle" />
            </Pop>
          )}
        </div>
      </div>
    );
  }
}

```