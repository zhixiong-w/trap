nuxt 作为一个SEO优化工具，使用的人较少，容易遇见一部分问题，虽然网络上有大量的解决方案，但对于框架的选型和后期项目功能的预判会明显感觉到不足，希望此文档在对以后开发过程中遇到的坑和问题有所帮助

### 适配问题

常用适配的屏幕尺寸，对于卡屏翻页问题，高度尺寸不准确 
手机 750*1334
pc端 2k屏 1920*937 1280*720 
pad 1024*1366

### 适配方案

安装 `postcss-px2rem` 插件 

在nuxt.config.js build下添加以下代码
```javaScript
postcss: [
  require("postcss-px2rem")({
    remUnit: 75 // 相当于 1rem = 75px
  })
],
```

在 static/js 文件夹下添加 flexible.js 文件
``` JavaScript
(function flexible(window, document) {
  const docEl = document.documentElement // 返回文档的root元素
  const dpr = window.devicePixelRatio || 1
  // 获取设备的dpr，即当前设置下物理像素与虚拟像素的比值



  const browser = {
    isMobile() {
      const u = navigator.userAgent
      return !!u.match(/AppleWebKit.*Mobile.*/)
    }
  }

  function setRemUnit() {

    const dpr = window.devicePixelRatio
    let width = window.innerWidth || docEl.clientWidth
    let rem
    if (!browser.isMobile()) {
      width = width > 1200 ? width : 1200
    }
    if (width / dpr > 540) {
      width = width * dpr
    }
    if (browser.isMobile()) {
      rem = width / 7.5
    } else {
      rem = width / 19.2
    }
    docEl.style.fontSize = rem + 'px'
    if (window.getComputedStyle(document.getElementsByTagName("html")[0]).fontSize) {  // 这段代码是针对微信显示异常问题
      const size = window.getComputedStyle(document.getElementsByTagName("html")[0]).fontSize.split('p')[0]
      if (size * 1.2 < 100 * (width / 750)) {
        // 如果当前html的font-size 的1.2倍仍然小于 之前想设置的值，就说明是问题机型，给之前想附的值乘1.25倍，这样他会被系统再次除1.25得到的才是我们想附的值
        rem = 125 * (width / 750);
        docEl.style.fontSize = rem + 'px'
      }
    }
  }


  setRemUnit()
  // 当我们页面尺寸大小发生变化的时候，要重新设置下rem 的大小
  window.addEventListener('resize', setRemUnit)
  // pageshow 是我们重新加载页面触发的事件
  window.addEventListener('pageshow', function (e) {
    // e.persisted 返回的是true 就是说如果这个页面是从缓存取过来的页面，也需要从新计算一下rem 的大小
    if (e.persisted) {
      setRemUnit()
    }
  })

  // 检测0.5px的支持，支持则root元素的class中有hairlines
  if (dpr >= 2) {
    const fakeBody = document.createElement('body')
    const testElement = document.createElement('div')
    testElement.style.border = '.5px solid transparent'
    fakeBody.appendChild(testElement)
    docEl.appendChild(fakeBody)
    if (testElement.offsetHeight === 1) {
      docEl.classList.add('hairlines')
    }
    docEl.removeChild(fakeBody)
  }
}(window, document))
```

### 浏览器厂商挖的坑

#### 移动端火狐浏览器 

对于相同屏幕，火狐移动端浏览器获取的屏幕宽度要比系统自带浏览器像素宽， rem适配方案中，需要乘以0.9比例会显示正常

移动端360，QQ，UC浏览器有劫持video标签问题，会遇见样式错乱，无法关闭视屏等问题。占定解决方案为在手机屏幕旋转后，重新渲染dom树，或点开视频后强制用户刷新

PC端搜狗浏览器  会遇见三位渲染功能错乱问题，三维效果请谨慎使用

PC端火狐浏览器会出现图片模糊，文字模糊等问题ß
