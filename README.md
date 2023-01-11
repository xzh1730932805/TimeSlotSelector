# ⏳ 时间槽选择器 Time Slot Selector 🖱

这是一款非常易用的基于Vue 3.0的拖拽式时间槽选择器，让你尽享极致的时间管理体验。

![示例动态图](/demo.gif)

## 主要特性

+ 工作日设置支持
+ 工作时间设置支持
+ 拖拽改变时间段支持
+ 调整事件长度支持
+ 触控设备支持 (依赖 [`DragDropTouch`](https://github.com/Bernardo-Castilho/dragdroptouch) 插件)

## 快速开始

1. 使用 npm 安装

```bash
npm install time-slot-selector --save
```

2. 导入插件到你的项目

```javascript
import time-slot-selector from 'time-slot-selector'
```

3. 使用组件

```html
<time-slot-selector :schedule-data="data" :setting="settingData" />
```

## 配置参数

| 参数 | 类型 | 描述 |
| :--- | :--- | :--- |
| startDate | String | 开始日期（格式为 YYYY/MM/DD） |
| endDate | String | 结束日期（格式为 YYYY/MM/DD） |
| weekdayText | Array | 星期的表示文本 |
| unit | Number | 时间线以多少分钟为一个单元 |
| borderW | Number | 单元格边框宽度（单位为 px） |
| dateDivH | Number | 日期单元格的高度（单位为 px） |
| timeDivH | Number | 时间单元格的高度（单位为 px） |
| unitDivW | Number | 选择单元格的宽度（单位为 px） |
| titleDivW | Number | 标题单元格的宽度（单位为 %） |
| rowH | Number | 行高（单位为 px） |

## 事件回调

| 事件 | 类型 | 描述 |
| :--- | :--- | :--- |
| row-click-event | Function | 标题点击事件 |
| date-click-event | Function | 日期点击事件 |
| click-event | Function | 时间槽点击事件 |
| add-event | Function | 新时间槽添加事件 |
| move-event | Function | 时间槽移动事件 |
| edit-event | Function | 时间槽编辑事件 |
| delete-event | Function | 时间槽删除事件 |

## 时间槽数据

| 参数 | 类型 | 描述 |
| :--- | :--- | :--- |
| title | String | 显示在时间槽上的文字 |
| noBusinessDate | Array | 非工作日列表（一整天） |
| businessHours.start | String | 工作开始时间（格式为 HH:ii） |
| businessHours.end | String | 工作结束时间（格式为 HH:ii） |
| schedule.start | String | 时间槽开始时间（格式为 HH:ii） |
| schedule.end | String | 时间槽结束时间（格式为 HH:ii） |
| schedule.text | String | 在时间槽上显示的文字 |
| schedule.data | Object or Array | 回调函数返回的数据 |

## 贡献

我们真诚地欢迎你参与到本项目的开发中来，一起通过反馈问题或提供修改建议使其更为完美。

感谢你为我们的开源社区注入强大的无尽热情与力量。

如有任何问题或建议，请随时与我们联系。
