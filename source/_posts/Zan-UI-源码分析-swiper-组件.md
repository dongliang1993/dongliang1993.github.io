---
title: Zan UI 源码分析 swiper 组件
date: 2017-12-19 10:56:03
tags:
---

<!-- more -->

```js
// swiper.js
import React, { PureComponent, Component, Children, cloneElement } from 'react';
import WindowResizeHandler from 'utils/component/WindowResizeHandler';
import Icon from 'icon';
import PropTypes from 'prop-types';
import cx from 'classnames';

import forEach from 'lodash/forEach';
import throttle from 'lodash/throttle';

import SwiperDots from './SwiperDots';

function setStyle(target, styles) {
  const { style } = target;

  Object.keys(styles).forEach(attribute => {
    style[attribute] = styles[attribute];
  });
}

export default class Swiper extends (PureComponent || Component) {
  static PropTypes = {
    className: PropTypes.string, // 自定义额外类名	
    prefix: PropTypes.string, // 自定义前缀	
    transitionDuration: PropTypes.number, // transitionDuration
    autoplay: PropTypes.bool, // 是否自动切换	
    autoplayInterval: PropTypes.number, // 自动切换间隔时间(ms)	
    dots: PropTypes.bool, // 是否显示下方翻页按钮	
    dotsColor: PropTypes.string, // 下方翻页按钮颜色	
    dotsSize: PropTypes.oneOf(['normal', 'small', 'large']), // 下方翻页按钮大小	
    arrows: PropTypes.bool, // 是否显示两侧翻页按钮	
    arrowsType: PropTypes.oneOf(['dark', 'light']), // 两侧箭头颜色	
    onChange: PropTypes.func // 切换时回调函数	
  };

  static defaultProps = {
    className: '',
    prefix: 'zent',
    transitionDuration: 300,
    autoplay: false,
    autoplayInterval: 3000,
    dots: true,
    dotsColor: 'black',
    dotsSize: 'normal',
    arrows: false,
    arrowsType: 'dark'
  };

  state = {
    // 当前的切换到第几张图片
    currentIndex: 0
  };
  // 初始化
  init = () => {
    const { currentIndex } = this.state;
    // 获取所有的slider， 是个 collection
    const innerElements = this.swiperContainer.children;

    this.setSwiperWidth();
    // 拿到根节点的宽度，然后 swiperContainer 的宽度就是 根节点 * innerElements.length
    setStyle(this.swiperContainer, {
      width: `${this.swiperWidth * innerElements.length}px`
    });

    forEach(innerElements, item => {
      // 每个 slider 的宽度是百分比
      setStyle(item, {
        width: `${100 / innerElements.length}%`
      });
    });

    innerElements.length > 1 && this.translate(currentIndex, true);
  };
  // 获取整个 swiper 根节点的真实 dom
  getSwiper = swiper => {
    this.swiper = swiper;
  };
  // 获取所有 slider 的父节点，不包括 arrow
  getSwiperContainer = swiperContainer => {
    this.swiperContainer = swiperContainer;
  };
  // 设置 Swiper 的宽度
  setSwiperWidth = () => {
    // 获取的是整个根节点的宽度
    this.swiperWidth = this.swiper.getBoundingClientRect().width;
  };
  // 自动轮播
  startAutoplay = () => {
    const { autoplayInterval } = this.props;
    this.autoplayTimer = setInterval(this.next, Number(autoplayInterval));
  };
  // 清除轮播
  clearAutoplay = () => {
    clearInterval(this.autoplayTimer);
  };
  // 下一张
  next = () => {
    const { currentIndex } = this.state;
    this.swipeTo(currentIndex + 1);
  };
  // 上一张
  prev = () => {
    const { currentIndex } = this.state;
    this.swipeTo(currentIndex - 1);
  };

  swipeTo = index => {
    let currentIndex = index;
    this.setState({ currentIndex });
  };
  // 设置移动的距离
  translate = (currentIndex, isSilent) => {
    const { transitionDuration } = this.props;
    const initIndex = -1;
    const itemWidth = this.swiperWidth;
    const translateDistance = itemWidth * (initIndex - currentIndex);

    setStyle(this.swiperContainer, {
      transform: `translateX(${translateDistance}px)`,
      'transition-duration': isSilent ? '0ms' : `${transitionDuration}ms`
    });
  };
  // 重置位置
  resetPosition = currentIndex => {
    const { transitionDuration, children: { length } } = this.props;
    if (currentIndex < 0) {
      setTimeout(
        () =>
          this.setState({
            currentIndex: length - 1
          }),
        transitionDuration
      );
    } else {
      setTimeout(
        () =>
          this.setState({
            currentIndex: 0
          }),
        transitionDuration
      );
    }
  };

  getRealPrevIndex = index => {
    const { children: { length } } = this.props;

    if (index > length - 1) {
      return length - 1;
    } else if (index < 0) {
      return 0;
    }

    return index;
  };
  // 克隆轮播图
  cloneChildren = children => {
    // 获取轮播图的数量
    const length = Children.count(children);
    // 如果只有一张轮播图，直接返回
    if (length <= 1) {
      return children;
    }
    // 这个是说，要在原来的轮播图的基础上
    // 头部和尾部分别 push 进一个节点
    // 感觉这个写法有点麻烦
    const clonedChildren = new Array(length + 2);
    Children.forEach(children, (child, index) => {
      clonedChildren[index + 1] = child;
      if (index === 0) {
        clonedChildren[length + 1] = child;
      } else if (index === length - 1) {
        clonedChildren[0] = child;
      }
    });

    return clonedChildren;
  };
  // 鼠标移入的时候，清除定时器
  handleMouseEnter = () => {
    const { autoplay } = this.props;
    autoplay && this.clearAutoplay();
  };
  // 鼠标移出的时候，恢复定时器
  handleMouseLeave = () => {
    const { autoplay } = this.props;
    autoplay && this.startAutoplay();
  };
  // 设置currentIndex
  handleDotsClick = index => {
    this.setState({ currentIndex: index });
  };

  componentDidMount() {
    const { autoplay } = this.props;
    // 如果启用了自动播放，调用 startAutoplay 函数
    autoplay && this.startAutoplay();
    this.init();
  }

  componentDidUpdate(prevProps, prevState) {
    const { onChange, children: { length } } = this.props;
    const { currentIndex } = this.state;
    const prevIndex = prevState.currentIndex;

    if (prevIndex === currentIndex) {
      return;
    }

    const isSilent = prevIndex > length - 1 || prevIndex < 0;
    this.translate(currentIndex, isSilent);

    if (currentIndex > length - 1 || currentIndex < 0) {
      return this.resetPosition(currentIndex);
    }

    onChange && onChange(currentIndex, this.getRealPrevIndex(prevIndex));
  }

  componentWillUnmount() {
    this.clearAutoplay();
  }

  render() {
    const {
      className,
      prefix,
      dots,
      dotsColor,
      dotsSize,
      arrows,
      arrowsType,
      children
    } = this.props;
    // 获取到当前轮播到第几页
    const { currentIndex } = this.state;
    // 返回的是一个字符串，不是一个对象
    const classString = cx(`${prefix}-swiper`, className, {
      // 如果显示箭头，并且箭头的类型是 ‘light’ 那么就加上这个类名
      [`${prefix}-swiper-light`]: arrows && arrowsType === 'light'
    });
    // 轮播图的数量
    const childrenCount = Children.count(children);
    // 克隆所有的轮播图，首尾各加上一张图片
    const clonedChildren = this.cloneChildren(children);

    return (
      <div
        ref={this.getSwiper}
        className={classString}
        onMouseEnter={this.handleMouseEnter}
        onMouseLeave={this.handleMouseLeave}
      >
        {/* 如果需要显示左右 arrows，并且轮播的数量大于 0 的时候就渲染 */}
        {/* 左箭头 */}
        {arrows &&
          childrenCount && (
            <div
              className={`${prefix}-swiper__arrow ${prefix}-swiper__arrow-left`}
              onClick={this.prev}
            >
              <Icon
                type="right-circle"
                className={`${prefix}-swiper__arrow-icon`}
              />
            </div>
          )}
        {/* 右箭头 */}
        {arrows &&
          childrenCount > 1 && (
            <div
              className={`${prefix}-swiper__arrow ${prefix}-swiper__arrow-right`}
              onClick={this.next}
            >
              <Icon
                type="right-circle"
                className={`${prefix}-swiper__arrow-icon`}
              />
            </div>
          )}
        <div
          ref={this.getSwiperContainer}
          className={`${prefix}-swiper__container`}
        >
          {/* 渲染每一个 slider  */}
          {Children.map(clonedChildren, (child, index) => {
            return cloneElement(child, {
              key: index - 1,
              style: { float: 'left', height: '100%' }
            });
          })}
        </div>
        {/* 如果需要显示左右 arrows，并且轮播的数量大于 0 的时候就渲染 */}
        {dots &&
          childrenCount > 1 && (
            <SwiperDots
              prefix={prefix}
              dotsColor={dotsColor}
              dotsSize={dotsSize}
              items={children}
              currentIndex={currentIndex}
              onDotsClick={this.handleDotsClick}
            />
          )}
        <WindowResizeHandler onResize={throttle(this.init, 1000 / 60)} />
      </div>
    );
  }
}

```

```js
// SwiperDots.js
import React, { PureComponent, Component } from 'react';
import cx from 'classnames';

const buildInDotsColors = ['black', 'blue', 'red', 'green'];

export default class SwiperDots extends (PureComponent || Component) {
  // 根据 currentIndex（父组件） 和自己在数组中的index
  // 判断是不是 active 的
  isDotActive = (index, currentIndex, length) => {
    return (
      index === currentIndex ||
      (index === 0 && currentIndex > length - 1) ||
      (index === length - 1 && currentIndex < 0)
    );
  };
  // 判断是不是内置颜色
  isBuildInColor = dotsColor => {
    return buildInDotsColors.indexOf(dotsColor) !== -1;
  };

  render() {
    const {
      prefix,
      dotsColor,
      dotsSize,
      items,
      currentIndex,
      onDotsClick
    } = this.props;
    const classString = cx(
      `${prefix}-swiper__dots`,
      `${prefix}-swiper__dots-${dotsSize}`,
      {
        [`${prefix}-swiper__dots-${dotsColor}`]: this.isBuildInColor(dotsColor)
      }
    );

    return (
      <ul className={classString}>
        {items.map((item, index) => {
          const isActive = this.isDotActive(index, currentIndex, items.length);
          if (isActive && !this.isBuildInColor(dotsColor)) {
            return (
              <li
                key={index}
                style={{ background: dotsColor }}
                className={`${prefix}-swiper__dots-item ${prefix}-swiper__dots-item-active`}
              />
            );
          }
          return (
            <li
              key={index}
              className={cx(`${prefix}-swiper__dots-item`, {
                [`${prefix}-swiper__dots-item-active`]: isActive
              })}
              onClick={() => onDotsClick(index)}
            />
          );
        })}
      </ul>
    );
  }
}

```