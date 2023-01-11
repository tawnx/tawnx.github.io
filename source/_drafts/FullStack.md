## React

### Key-attribute

列表项，即由*map*方法生成的元素，必须有一个唯一的键值：一个叫做*key*的属性。

```jsx
const App = (props) => {
  const { notes } = props

  return (
    <div>
      <h1>Notes</h1>
      <ul>
        {notes.map(note =>
          <li key={note.id}>
            {note.content}
          </li>
        )}
      </ul>
    </div>
  )
}
```


不知道为什么，写入数据只能保存两个字段。
![image-20230102134208735](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2023-01/image-20230102134208735.png)

`save()` 只会写入model中定义的属性，而我 `PersonScheme` 中是这样定义的，所以 `number` 属性不会被写入。

![image-20230102135252286](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2023-01/image-20230102135252286.png)
