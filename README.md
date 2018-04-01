## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/SpringLv/springlv.github.io/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

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
`/*`
我们要满足状态只能三种状态：PENDING,FULFILLED,REJECTED三种状态，且状态只能由PENDING=>FULFILLED,或者PENDING=>REJECTED`
*/`
`var PENDING = 0;`
`var FULFILLED = 1;`
`var REJECTED = 2;`
`/*`
`value状态为执行成功事件的入参，deferreds保存着状态改变之后的需要处理的函数以及promise子节点，构造函数里面应该包含这三个属性的初始化`
 */
`function Promise(callback) {`
    `this.status = PENDING;`
    `this.value = null;`
    `this.defferd = [];`
    `setTimeout(callback.bind(this, this.resolve.bind(this), this.reject.bind(this)), 0);`
`}`

`Promise.prototype = {`
   `constructor: Promise,`
   `//触发改变promise状态到FULFILLED`
    `resolve: function (result) {`
        `this.status = FULFILLED;
        `this.value = result;
        `this.done();
    `},
    `//触发改变promise状态到REJECTED
    `reject: function (error) {
        `this.status = REJECTED;
        `this.value = error;
    `},
    `//处理defferd
    `handle: function (fn) {
        `if (!fn) {
            `return;
        `}
        `var value = this.value;
        `var t = this.status;
        `var p;
        `if (t == PENDING) {
             `this.defferd.push(fn);
        `} else {
            `if (t == FULFILLED && typeof fn.onfulfiled == 'function') {
                `p = fn.onfulfiled(value);
            `}
            `if (t == REJECTED && typeof fn.onrejected == 'function') {
                `p = fn.onrejected(value);
            `}
        `var promise = fn.promise;
        `if (promise) {
            `if (p && p.constructor == Promise) {
                `p.defferd = promise.defferd;
            `} else {
                `p = this;
                `p.defferd = promise.defferd;
                `this.done();
            `}
        `}
        `}
    `},
    `//触发promise defferd里面需要执行的函数
    `done: function () {
        `var status = this.status;
        `if (status == PENDING) {
            `return;
        `}
        `var defferd = this.defferd;
        `for (var i = 0; i < defferd.length; i++) {
            `this.handle(defferd[i]);
        `}
    `},
    `/*储存then函数里面的事件
    `返回promise对象
    `defferd函数当前promise对象里面
    `*/
    `then: function (success, fail) {
       `var o = {
            `onfulfiled: success,
            `onrejected: fail
        `};
        `var status = this.status;
        `o.promise = new this.constructor(function () {
        `});`
        `if (status == PENDING) {
           ` this.defferd.push(o);
        `} else if (status == FULFILLED || status == REJECTED) {
            `this.handle(o);
        `}
        `return o.promise;
   `}
`};`
