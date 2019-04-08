瞎折腾记录

## ver 3.1.4
`sample`  
![我的头像](./sample.png)

* 调用：`import { L2Dwidget as Live2DWidget } from 'live2d-widget/src/index'`
* `src/config/configMgr.js` `28`行
    `defaultsDeep(currConfig, userConfig, defaultConfig);`

* `src/config/defaultsDeep.js` `16-20`行
    ```
    if (typeof val === 'object' && val !== null && typeof value === 'object' && value !== null) {
        defaultsDeep(val, value);
    } else {
    ﻿   target[key] = value;
    }
    ```
* `src/lib/live2d.core.js`注释`4871-4906`行