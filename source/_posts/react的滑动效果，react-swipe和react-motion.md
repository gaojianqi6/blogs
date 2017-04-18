---
title: react的滑动效果，react-swipe和react-motion
date: 2017-04-18 17:18:44
tags: [javascript]
categories: [技术]
---

这次在项目中需要用到焦点图滑动效果和在页面一个浮层中增加一个滑动的动画效果，之前已经写过很多次滑动效果的页面了，但是都没有留下记录。
现在记载下这次在项目中使用react-swipe和react-motion的收获。

### react-swipe
安装需要同时安装swipe-js-jso包：

``` shell
npm install react swipe-js-iso react-swipe
```

常用属性介绍：
continuous: 循环到最后一个时继续滑动是否从头开始，默认为true
transitionEnd：滑动动画结束时调用，callback(index, item), index为当前滑动的下标，item为当前被滑动的元素
因为swipe没有提供轮滑图中显示的提示圆点之类，自己可以根据下标在上面加上提示用户的圆点等。

示例代码:
```javascript
// import ReactSwipe from 'react-swipe';
<div className="welfare-focus-wrap">
  <ReactSwipe className="carousel" swipeOptions={{continuous: true, transitionEnd: (index) => {
    this.setState({
      focusIndex: index,
    });
  }}}>
    {
      this.state.activityDetails.map((item, index) =>
        <div className={`welfareFocusWrap focusIndex${index}`} key={`welfareFocus${index}`} onClick={() => { location.href = item.activityUrl;}}>
          <img src={item.pictureUrl} className="welfareFocusImg" />
        </div>)
    }
  </ReactSwipe>
  <div className="welfare-focus-circle-wrap">
    {
      this.state.activityDetails.map((item, index) =>
        <div
          className={`welfareFocusCircle ${this.state.focusIndex == index ? 'checked' : ''}`}
          key={`welfareFocusCircle${index}`} />)
    }
  </div>
</div>
```

github地址：[react-swipe](https://github.com/voronianski/react-swipe) 、[swipe-js-iso](https://github.com/voronianski/swipe-js-iso)

### react-motion
在没实际写上代码测试时候，对motion的概念感觉似懂非懂，翻了很多介绍react-motion的资料，在写下测试代码之后，自己理解react-motion做了对
动画过程中从A点到B点平滑的变化数值，比自己实现的定时滑动会更加的平滑自然。

因为要兼容手机不同分辨率滑动的单位距离不一致，所以在Motion的外层div的onLoad事件中，每次通过元素的clientWidth来获得滑动的距离，在react中不得已使用了
dom，谁有更好的办法欢迎告诉我。

```javascript
const stepbarLeft = this.state.stepbarLeft;
const oldStepbarLeft = curQuestionIndex > 0 ? (curQuestionIndex -1)*stepbarLeft : 0;
const newStepbarLeft = stepbarLeft*curQuestionIndex;
const currentNode = (
  <Motion defaultStyle={{left: oldStepbarLeft }} style={{left: spring(newStepbarLeft)}}>
    {
      // 在动画执行过程中，react-motion会不断的会掉此方法，返回动画进行过程中相应的数值，可以自己cosole输出interpolatingStyle看
      interpolatingStyle => {
        return (
          <div className="questionPageCurNode" style={interpolatingStyle}>
            <img src={questionPageCurrentImage} className="questionPageCurDot" />
            <span className="questionPageCurSequence">{ curQuestionIndex + 1 }</span>
          </div>
        );
      }
    }
  </Motion>
);

return (
  <div className="questionPageHeader" onLoad={() => {
    const left = document.querySelectorAll('.questionPageNode')[3].clientWidth + document.querySelectorAll('.questionPageLine')[3].clientWidth;
    this.setState({
      stepbarLeft: left,
      conentLeft: document.querySelector('.questionPageContentsArea').clientWidth,
    });
  }}>
    <div className="questionPageStepbar">
      {stepbar}
      {currentNode}
    </div>
  </div>
);
```

Motion参考资料：[React Motion 缓动函数剖析](https://segmentfault.com/a/1190000004224778)
