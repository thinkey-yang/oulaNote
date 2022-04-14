# saltejs

 ##  渲染机制

### 文档结构

使用react的渲染机制，直接写jsx

渲染过程： slateValue --> reactNode --> html

更新过程： 通过eventHandler（onKeydown 、onCompositionEnd等）监听用户输入，通过tansform更改全局editor。

```javascript
// element,将节点数据渲染为 html，可传入 renderElement 自定义渲染
interface Element {
  children: Node[]
} 

// text，渲染文本内容，可通过 renderLeaf 来控制文本内容的样式，会自动合并
// 纯函数
interface Text {
  text: string
}
```



![img](https://docimg1.docs.qq.com/image/xkNB-IbYeG7uITi715wNGA.png?w=1122&h=942)        

### Selection

slate selection 的数据模型模仿 dom selection 的数据结构，由 anchor 和 focus 两个`Point`组成，

```javascript
/**
* 文档中某一块内容，可以用来表示光标选中的区域，如果光标未选中内容，则 anchor 等于 focus
* selection value发生改变时，在 useEffect 中更新 domSelection，监听 selectionchange 事件，用于更新 * * * slate Selection value
*/
interface Range {
    anchor: Point;
    focus: Point;
    [key: string]: unknown;
}

/**
* 文档中的某一个位置，path 代表节点在文档中的位置，offset 代表在节点中的位置
*/

interface Point {
    path: Path;
    offset: number;
    [key: string]: unknown;
}
```

## 序列化

可以根据自己定义的customElemt，借助'escape-html'工具实现序列化

```javascript
import escapeHtml from 'escape-html'
import { Node, Text } from 'slate'

const serialize = node => {
  if (Text.isText(node)) {
    return escapeHtml(node.text)
  }

  // 后序遍历
  const children = node.children.map(n => serialize(n)).join('')
	// 根据node的type定义自己的序列化规则
  switch (node.type) {
    case 'quote':
      return `<blockquote><p>${children}</p></blockquote>`
    case 'paragraph':
      return `<p>${children}</p>`
    case 'link':
      return `<a href="${escapeHtml(node.url)}">${children}</a>`
    default:
      return children
  }
```

## 反序列化

```javascript
import { jsx } from 'slate-hyperscript'

const deserialize = el => {
  if (el.nodeType === 3) {
    return el.textContent
  } else if (el.nodeType !== 1) {
    return null
  }

  const children = Array.from(el.childNodes).map(deserialize)

  switch (el.nodeName) {
    case 'BODY':
      return jsx('fragment', {}, children)
    case 'BR':
      return '\n'
    case 'BLOCKQUOTE':
      return jsx('element', { type: 'quote' }, children)
    case 'P':
      return jsx('element', { type: 'paragraph' }, children)
    case 'A':
      return jsx(
        'element',
        { type: 'link', url: el.getAttribute('href') },
        children
      )
    default:
      return el.textContent
  }
}
```

## 插件机制

插件本身接受全局的editor上下文作为参数，通过覆盖和扩展其方法实现插件功能，然后将其返回，使用方式类似koa洋葱皮的方式：

```javascript
const withLinks = editor => {
  const { normalizeNode } = editor
  editor.normalizeNode = entry => {
    const [node, path] = entry
    // 校验数据
    if (
      Element.isElement(node) &&
      node.type === 'link' &&
      typeof node.url !== 'string'
    ) {
      // 通过 apply(operation) 完成数据的纠正
      Transforms.setNodes(editor, { url: null }, { at: path })
      return
    }
    normalizeNode(entry)
  }
  return editor
}

//使用插件
const editor = withLinks(createEditor())
```

## History （撤销和重做）

官方提供了个一个**Slate History**的子库，提供了类似功能。

## vue

社区有部分实现