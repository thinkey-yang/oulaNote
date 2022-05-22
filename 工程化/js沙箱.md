## js沙箱

- snapshotSandbox：支持单个微应用

- legacyScandbox：支持单个微应用

- proxySandbox：同时激活多个微应用的方案

### snapshotSandbox

```javascript
class SanpshotSanbox {
  widowSnapshot = {}
  modifyPropsMap= {}
	avtive() {
    // 保存window对象上所有的属性的状态
    for(const prop in window) {
      this.widowSnapshot[prop] = winidow[prop]
		}
    // 恢复上一次在运行该微应用时所修该过的window上的属性
   	Object.keys(this.modifyPropsMap).forEach(prop => {
			window[prop] = this.modifyPropsMap[prop]
    })
    
	}
  inActive() {
    for(const prop in window) {
      if(window[prop] !== this.widowSnapshot[prop]) {
        //记录该微前端修改了window上的那些熟悉
        this.modifyPropsMap[prop] = window[prop];
        //将window上的属性状态还原到微应用运行状态之前的状态
        window[prop] = this.widowSnapshot[prop]
      }
		}  
	}
}
```

- 单个微应用
- 性能不好

### legacySandbox

```javascript
class legacySandbox  {
    // 当前微应用的状态
    currentUpdatePropsValueMap = new Map();
    // 被修改的window状态的原始值
    modifyPropsOriginalValuemapInSandbox = new Map();
    addedPropsMapInSandbox = new Map();

    constructor() {
        const fakeWindow = Object.create(null);
        this.ProxyWindow = new Proxy(fakeWindow, {
            set: (target, prop, value, recevier) => {
                const originValue = window[prop];
                if(!window.hasOwnProperty(prop)) {
                    this.addedPropsMapInSandbox.set(prop,value);
                } else if(!this.modifyPropsOriginalValuemapInSandbox.has(prop)) {
                    this.modifyPropsOriginalValuemapInSandbox.set(prop,originValue);
                }
                this.currentUpdatePropsValueMap.set(prop,value);
                window[prop] = value;
            }
        })
    }
    setWindowProp(prop ,value, isToDelete) {
        if(value === undefined && isToDelete) {
            delete window[prop];
        }else{
            window[prop] =  value
        }
    }
    active() {
        //恢复该微前端上一次运行时的全局状态（对window的修改）
        this.currentUpdatePropsValueMap.forEach((value, prop) => {
            this.setWindowProp(prop, value);
        })
    }
    inActive() {
        // 还原window上的属性
        this.modifyPropsOriginalValuemapInSandbox.forEach((value, prop) => {
            this.setWindowProp(prop, value)
        })
        // 删除在微应用运行时，window上新增的属性
        this.addedPropsMapInSandbox.forEach((value, prop) => {
            this.setWindowProp(prop, value, true)
        })
    }
}
```



- 性能良好
- 单个微应用

### proxySandbox

```javascript
class proxySandbox {
	this.isRunning = false;
	cunstructor() {
		cosnt fakeWindows = Object.creact(null);
    const proxyWindows = new Proxy(fakeWindow, {
      set: (target, prop, value, recevier) => {
        if(this.isRunning) {
          target[prop] = value
				}
			},
      get: (target, prop, recevier) => {
        return pops in target ? target[prop]: window[prop]
			}
    })
  }
	active() {
		this.isRunning = true;
  }
	inActive() {
		this.isRunning = false;
  }
}
```

- 性能良好
- 多个微应用

总结：

legacySandbox应该会被淘汰、保留proxyBandbox合snapshotSandbox（兼容性比较好）