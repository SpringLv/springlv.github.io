## Welcome to GitHub Pages

```markdown
# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/SpringLv/springlv.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.

### Promise 实现
```markdown
`/*
我们要满足状态只能三种状态：PENDING,FULFILLED,REJECTED三种状态，且状态只能由PENDING=>FULFILLED,或者PENDING=>REJECTED
*/
var PENDING = 0;
var FULFILLED = 1;
var REJECTED = 2;
/*
value状态为执行成功事件的入参，deferreds保存着状态改变之后的需要处理的函数以及promise子节点，构造函数里面应该包含这三个属性的初始化
 */
function Promise(callback) {
    this.status = PENDING;
    this.value = null;
    this.defferd = [];
    setTimeout(callback.bind(this, this.resolve.bind(this), this.reject.bind(this)), 0);
}

Promise.prototype = {
    constructor: Promise,
    //触发改变promise状态到FULFILLED
resolve: function (result) {
    this.status = FULFILLED;
    this.value = result;
    this.done();
},
//触发改变promise状态到REJECTED
reject: function (error) {
    this.status = REJECTED;
    this.value = error;
},
//处理defferd
handle: function (fn) {
    if (!fn) {
        return;
    }
    var value = this.value;
    var t = this.status;
    var p;
    if (t == PENDING) {
         this.defferd.push(fn);
    } else {
        if (t == FULFILLED && typeof fn.onfulfiled == 'function') {
            p = fn.onfulfiled(value);
        }
        if (t == REJECTED && typeof fn.onrejected == 'function') {
            p = fn.onrejected(value);
        }
    var promise = fn.promise;
    if (promise) {
        if (p && p.constructor == Promise) {
            p.defferd = promise.defferd;
        } else {
            p = this;
            p.defferd = promise.defferd;
            this.done();
        }
    }
    }
},
//触发promise defferd里面需要执行的函数
done: function () {
    var status = this.status;
    if (status == PENDING) {
        return;
    }
    var defferd = this.defferd;
    for (var i = 0; i < defferd.length; i++) {
        this.handle(defferd[i]);
    }
},
/*储存then函数里面的事件
返回promise对象
defferd函数当前promise对象里面
*/
then: function (success, fail) {
   var o = {
        onfulfiled: success,
        onrejected: fail
    };
    var status = this.status;
    o.promise = new this.constructor(function () {

    });
    if (status == PENDING) {
        this.defferd.push(o);
    } else if (status == FULFILLED || status == REJECTED) {
        this.handle(o);
    }
    return o.promise;
}
};`
```
### 日历面板 代码片段
```
import React from 'react';
import moment from 'moment';
import styles from '../routes/ApplyApplication/Index.less';

/**
 * @class Calender
 * @extends {React.Component}
 */
class Calender extends React.Component {
  getDaysArray = (days) => {
    const arr = [];
    let day = 1;
    while (day <= days) {
      arr.push(day++);
    }
    return arr;
  }
  getDatePlen = (currentDate) => {
    // currentMomemt
    const currentMomemt = moment(currentDate);
    // 本月天数
    const currentMonthDays = currentMomemt.daysInMonth();
    // 上一月(moment对象)
    const preMonth = moment(currentDate).subtract('months', 1);
    // 年
    const currentYear = currentMomemt.year();
    // 上一月天数
    const preMonthDays = preMonth.daysInMonth();
    // 本月1号周几
    const currentMonthWeekDay = moment(`${currentYear}-${moment(currentDate).month() + 1}-01`).weekday();
    // 本月最后一天周几
    const currentMonthLastDayWeek = moment(`${currentYear}-${moment(currentDate).month() + 1}-${currentMonthDays}`).weekday();
    const currentMonthOfDays = this.getDaysArray(currentMonthDays);
    const preMonthOfDays = this.getDaysArray(preMonthDays);
    const plenArr = preMonthOfDays
    .slice(preMonthDays - currentMonthWeekDay, preMonthDays)
    .concat(currentMonthOfDays)
    .concat(this.getDaysArray(6 - currentMonthLastDayWeek));
    return plenArr;
  }
  DomRender = (plenArr) => {
    // 周排序数组
    const w = ['星期一', '星期二', '星期三', '星期四', '星期五', '星期六', '星期日'];
    const backDomUl = [];
    let backDomli = [];
    backDomUl.push(<ul className={styles.month_week_day_plen} key="9999">
      {w.map((i, key) => {
        return <li key={`${key + 99999}`}>{i}</li>;
      })}
    </ul>);
    plenArr.forEach((item, index) => {
      backDomli.push(<li key={`li-${index}`}>{item}</li>);
      if (!((index + 1) % 7)) {
        backDomUl.push(<ul className={styles.month_week_day_plen} key={`${index}`}>{Object.assign(backDomli)}</ul>);
        backDomli = [];
      } else if (index + 1 === plenArr.length) {
        backDomUl.push(<ul className={styles.month_week_day_plen} key={`${index}`}>{Object.assign(backDomli)}</ul>);
        backDomli = [];
      }
    });
    return backDomUl;
  }
  compose = (f, g) => (x => f(g(x)));
  render() {
    const { getDatePlen, compose, DomRender } = this;
    return (
      <div>
        {compose(DomRender, getDatePlen)('2018-10-10')}
      </div>
    );
  }
}

export default Calender;
```
```
ul.month_week_day_plen {
    display: flex;
    justify-content: center;
    align-items: center;
    list-style: none;
    margin: 0 !important;
    padding: 0 !important;
    li {
        height: 8vw;
        flex-basis: 14.2857%;
        max-width: 14.2857%;
        border: 1px solid #e0e0e0;
    }
  }
```

