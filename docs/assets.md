# 图像资源
在脚本中调用的图像资源,要使用对应的方式提供,部分类型的图像直接使用即可,本文不作过多说明,请参考以下表格

| 值 | 说明 |
| --- | --- |
| `image` | 直接调用 |
| `sprite` `frame` | <b><a href="#sprite">精灵图 json</a></b> |
| `template` | json |

## sprite <span id="sprite">精灵图</span>
这种资源主要应用在人物上,图像上包含人物的本身以及表情,口型的差分矩阵
具体JSON结构如下

| key | 值 | 说明 | 备注 |
| --- | --- | --- | --- |
| `image` | `"image":{...}` | <b><a href="#sprite:image">图像对象</a></b> |  |
| `body` | `"body":{...}` | <b><a href="#sprite:body">主要人物对象</a></b> |  |
| `@部件名` | `"@部件名":{...}` | <b><a href="#sprite:module">部件对象</a></b> | 一个人物通常包括`眼(eyes)`、`口(mouth)`等部件,这里仅列举一个,键名不包含@ |

### image <span id="sprite:image">图像对象</span>
包含图像的文件名,尺寸等基本信息,图像文件要与JSON文件放在同一个目录

| key | 值 | 说明 | 备注 |
| --- | --- | --- | --- |
| `src` | `字符串` | 图片文件名(包括后缀名) |  |
| `size` | `"size":[w, h]` | 图片的原始宽高 | 单位是px |

### body <span id="sprite:body">主要人物对象</span>
人物主立绘信息，通常包括完整的人物立绘在图像上的位置信息以及可覆盖的部件信息

| key | 值 | 说明 | 备注 |
| --- | --- | --- | --- |
| `position` | `"position":[w, h, x, y]` | 立绘在图像上的宽高以及起点XY坐标 |  |
| `cut` | `"cut":[up, right, bottom, left]` | <b><a href="#sprite:body:cut">立绘裁切边距</a></b> |  |
| `children` | `"children":{...}` | <b><a href="#sprite:body:children">可继承部件对象</a></b> |  |

#### cut <span id="sprite:body:cut">立绘裁切边距</span>
部分人物立绘或部件在图片上可能会有一定的安全边距需要裁切,此参数可以设置立绘的上右下左裁切的像素

#### children <span id="sprite:body:children">可继承部件对象</span>
一个立绘上可能会有需要部分改变的部分,比如眼睛和口型甚至是手等,这些改变不会对立绘的其他地方做更改,此时可以将该区域设定为部件区域,用来允许被另一个部件覆盖
一个区域需要有唯一的名称且名称不能为body,JSON键值正是这个名称,后面的部件名也要使用同样的名称声明,示例如下

```json
"children": {
    "eyes":[w, h, x, y], //区域的宽高以及起始坐标(px)
    "mouth":[w, h, x, y]
}
```

### module <span id="sprite:module">部件对象</span>
部件块信息，跟body很相似,但是键名要和需要覆盖的children中的键名一样
注意:名称不包含@

| key | 值 | 说明 | 备注 |
| --- | --- | --- | --- |
| `position` | `"position":[w, h, x, y]` | <b><a href="#sprite:module:position">部件块大小及位置</a></b> |  |
| `cut` | `"cut":[up, right, bottom, left]` | <b><a href="#sprite:body:cut">立绘裁切边距</a></b> | 与body的cut一样 |
| `map` | `"map":{...}` | <b><a href="#sprite:module:map">部件块映射</a></b> |  |

#### position <span id="sprite:module:position">部件块大小及位置</span>
差分阵列单个块宽高以及第一个块的起点XY坐标,一个部件块内可以有多个帧,多个图块组成,但是此处只需要填写单个块的宽高以及左上角的第一个块的起始坐标即可

#### map <span id="sprite:module:map">部件块映射</span>
一个部件可能是由多个帧组成动画,我们将每一组帧分类命名,当图层类型为frame时可以直接调用对应名称的动画
名称应该作为JSON的键以在脚本文件中调用,示例如下

```json
"map": {
    "smile": [
        [0, 1], //坐标的单位是以块为单位,程序会自动通过position计算对应块的像素位置
        [0, 2],
        [0, 3]
    ],
    "cry": [
        [1, 1],
        [1, 2],
        [1, 3]
    ]
}
```