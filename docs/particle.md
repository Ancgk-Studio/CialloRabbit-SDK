# 粒子动画
粒子动画用于在画面上生成大量随机的小元素,实现下雨、下雪、烟花、气泡上浮等自然或特效动画
粒子配置是一个独立的json文件,存放在`assets/particles/`目录内,文件名不包含后缀名,在脚本中通过图层类型为`particle`的`res`字段引用(参考script.md的图层资源名)

## 粒子文件与图层字段的分工
凡是作用于整个粒子层的表现,一律使用图层的字段,粒子文件内不再提供重复配置,图层字段请参考script.md的图层对象
| 效果 | 使用图层的字段 | 使用粒子文件 |
| --- | --- | --- |
| 混合模式(发光使用`lighter`) | `cover` | 不提供 |
| 整体透明度与淡入淡出 | `alpha` `transition` | 不提供 |
| 延迟开始发射 | `sleep` | 不提供 |
| 在画面上的摆放位置与对齐 | `position` `align` | `emit`的区域内部坐标 |
| 生成频率与物理行为 | 不提供 | `emit` `particle` |

## 通用取值规则
本文件内的每个数值字段(包括`start`与`end`外观对象内的字段)都支持两种写法
- 单个数字: 所有粒子固定为该值
- `[min, max]`数组: 每个粒子在生成时于区间内独立随机取值,保证效果自然错落
本文件所有数值字段默认遵循此规则,下文不再重复说明

## 顶层结构
| key | 值 | 说明 |
| --- | --- | --- |
| `image` | `字符串` `"image":{...}` | <b><a href="#particle:image">粒子贴图</a></b> |
| `emit` | `"emit":{...}` | <b><a href="#particle:emit">发射器对象</a></b> |
| `particle` | `"particle":{...}` | <b><a href="#particle:particle">粒子行为对象</a></b> |

## image <span id="particle:image">粒子贴图</span>
粒子渲染时使用的图像文件名,不包含后缀名,文件位于`assets/textures/particles/`目录内
例如填写`raindrop`则使用`assets/textures/particles/raindrop.png`
省略时程序使用内置的纯白圆形贴图,适合配合图层`cover: "lighter"`制作发光粒子
此字段支持两种写法:
- 字符串写法`"image": "raindrop"`: 直接使用基础目录下的贴图
- 对象写法`"image": {...}`: 需要多语言贴图时使用,包含`name`与`lang`两个字段,参考<b><a href="#particle:image:lang">多语言贴图</a></b>

默认值:`null`

### image.lang <span id="particle:image:lang">设定语言列表</span>
与script.md图层图像的`lang`字段规则相同:将贴图放入不同的语言子文件夹内,并在`lang`数组中声明可用语言
| key | 值 | 说明 |
| --- | --- | --- |
| `name` | `字符串` | 贴图文件名,不包含后缀名 |
| `lang` | `"lang":[...]` | 可用语言列表,第一项同时作为回退语言 |
示例:
```json
"image": {
  "name": "raindrop",
  "lang": ["zh_CN", "ja_JP", "en_US"]
}
```
(在用户使用简体中文的情况下)贴图位置为`assets/textures/particles/zh_CN/raindrop.png`
当用户选择的语言没有对应的资源时,使用`lang`数组第一项声明的语言目录,如上例的`zh_CN`

## emit <span id="particle:emit">发射器对象</span>
控制粒子的生成位置、区域形状与生成频率
| key | 值 | 说明 | 备注 |
| --- | --- | --- | --- |
| `shape` | `point` `line` `box` `circle` | <b><a href="#particle:emit:shape">生成区域形状</a></b> |  |
| `position` | `"position":[x, y]` | <b><a href="#particle:emit:position">区域中心</a></b>,相对图层原点,单位px | 非必须 |
| `size` | `"size":[w, h]` | <b><a href="#particle:emit:size">区域尺寸</a></b>,含义随`shape` | 非必须 |
| `rate` | `整数` | <b><a href="#particle:emit:rate">每秒生成数量</a></b> | 非必须 |
| `burst` | `整数` | <b><a href="#particle:emit:burst">开始瞬间一次生成的数量</a></b> | 非必须 |
| `max` | `整数` | <b><a href="#particle:emit:max">同屏最大粒子数</a></b> | 非必须 |

### shape <span id="particle:emit:shape">生成区域形状</span>
决定粒子生成的空间分布,粒子会在该形状定义的区域内随机取坐标生成
| 值 | 说明 | 适用场景 |
| --- | --- | --- |
| `point` | 单点发射,所有粒子从`position`坐标精确生成 | 烟花爆炸中心、喷泉口、魔法阵中心 |
| `line` | 线形发射,粒子在长度为`size[0]`的线段上随机生成 | 屏幕顶部下雨、下雪、瀑布顶端 |
| `box` | 矩形区域发射,粒子在`size`定义的宽高矩形范围内随机生成 | 环境烟雾、全屏落叶、沙尘暴 |
| `circle` | 圆形区域发射,粒子在半径`size[0]`的圆形范围内随机生成 | 爆炸扩散范围、环形魔法阵 |

默认值:`point`

### position <span id="particle:emit:position">发射器位置</span>
发射器区域中心相对图层原点的偏移,图层原点由图层的`position`与`align`决定

默认值:`[0, 0]`

### size <span id="particle:emit:size">区域尺寸</span>
区域尺寸,含义随`shape`变化
| shape | `size[0]` | `size[1]` |
| --- | --- | --- |
| `point` | 忽略 | 忽略 |
| `line` | 线段长度(px) | 无效 |
| `box` | 矩形宽(px) | 矩形高(px) |
| `circle` | 圆半径(px) | 无效 |

默认值:`[0, 0]`

### rate <span id="particle:emit:rate">生成频率</span>
发射器每秒持续生成的粒子数量,例如`60`表示每秒均匀生成60个

默认值:`0`

### burst <span id="particle:emit:burst">瞬间生成</span>
在图层开始渲染的第一帧瞬间一次性生成的粒子数量,通常用于爆炸、烟花绽放等瞬间出现大量粒子的效果
可与`rate`同时使用,例如烟花爆炸后残留的火星继续以`rate`生成

默认值:`0`

### max <span id="particle:emit:max">同屏上限</span>
场景内同时存在的最大粒子数,达到上限后暂停生成,直到有粒子销毁
建议不小于`rate × life(秒) + burst`,参考性能建议

默认值:`500`

## particle <span id="particle:particle">粒子行为对象</span>
控制单个粒子从生成到销毁的运动与外观
| key | 值 | 说明 | 备注 |
| --- | --- | --- | --- |
| `life` | `毫秒` | <b><a href="#particle:particle:life">存活时间,到时销毁</a></b> |  |
| `speed` | `px/秒` | <b><a href="#particle:particle:speed">初始移动速度</a></b> |  |
| `angle` | `角度` | <b><a href="#particle:particle:angle">初始移动方向</a></b> |  |
| `gravity` | `"gravity":[x, y]` | <b><a href="#particle:particle:gravity">加速度</a></b> | 非必须 |
| `friction` | `0-1` | <b><a href="#particle:particle:friction">速度衰减比例</a></b> | 非必须 |
| `start` | `"start":{...}` | <b><a href="#particle:particle:appearance">初始外观对象</a></b>,包含`size` `alpha` `rotate` `color` |  |
| `end` | `"end":{...}` | <b><a href="#particle:particle:appearance">结束外观对象</a></b>,字段与`start`相同 | 非必须 |

### life <span id="particle:particle:life">存活时间</span>
粒子从生成到销毁的存活时间,单位为毫秒(与脚本时间单位一致)
例如`[1000, 2000]`表示每个粒子的存活时间在1秒到2秒之间随机
`life`耗尽时粒子立即销毁并回收

默认值:`1000`

### speed <span id="particle:particle:speed">初始移动速度</span>
粒子生成时的初始移动速度,单位为px/秒,沿`angle`决定的方向运动
速度会受`gravity`的加速度与`friction`的衰减影响而变化

默认值:`0`

### angle <span id="particle:particle:angle">初始移动方向</span>
粒子生成时的初始方向,采用屏幕坐标系(Y轴向下,X轴向右)
| 值 | 方向 |
| --- | --- |
| `0` | 向右 |
| `90` | 向下 |
| `180` | 向左 |
| `270` | 向上 |
例如`[75, 105]`表示主要向下,在左右各15度的扇形范围内随机偏移,`[0, 360]`表示全向发射

默认值:`90`

### gravity <span id="particle:particle:gravity">加速度</span>
格式`[x, y]`,水平与垂直方向的加速度,单位px/秒²
- 下雨、烟花下坠: `[0, 200]`
- 气泡上浮、热气球: `[0, -50]`
- 横向风吹: `[50, 0]`

默认值:`[0, 0]`

### friction <span id="particle:particle:friction">速度衰减比例</span>
速度衰减比例,取值0到1,每秒速度变为原来的`(1 - friction)`倍,与帧率无关
- 烟雾、灰尘缓慢飘散: `0.05` ~ `0.1`
- 雨雪保持匀速: `0`

默认值:`0`

### start与end <span id="particle:particle:appearance">外观对象</span>
`start`定义粒子生成瞬间的初始外观,`end`定义`life`结束时的外观,两个对象包含相同的四个字段
| key | 值 | 说明 | 备注 |
| --- | --- | --- | --- |
| `size` | `px` | 大小 |  |
| `alpha` | `0-1` | 透明度 |  |
| `rotate` | `角度` | 旋转角度 |  |
| `color` | `hex颜色`或`hex颜色数组` | 颜色,仅`start`支持数组写法 |  |
规则:
- `start`整体省略时,使用默认外观(size `8`, alpha `1`, rotate `0`, color `"#ffffff"`)
- `end`整体省略,或`end`内省略某个字段时,该属性全程保持`start`的状态
- `start`与`end`都填写某字段时,程序根据粒子已存活的百分比(0%到100%),在两个值之间线性过渡
示例:
- 淡出: `start.alpha`设为`1`,`end.alpha`设为`0`
- 膨胀: `start.size`设为`[10, 10]`,`end.size`设为`[50, 80]`
- 火焰冷却变色: `start.color`设为`"#ffff00"`,`end.color`设为`"#ff0000"`
`color`会与贴图进行正片叠底(相乘)混合,`"#ffffff"`表示保留贴图原色;`start.color`填写颜色数组时,每个粒子随机挑选其中一个作为自己的颜色

默认值:`start`与`end`均为不设置,外观保持默认值(size `8`, alpha `1`, rotate `0`, color `"#ffffff"`)

## 默认值汇总
| 字段 | 默认值 |
| --- | --- |
| <a href="#particle:image">`image`</a> | `null`(内置纯白圆) |
| <a href="#particle:emit:shape">`emit.shape`</a> | `point` |
| <a href="#particle:emit:position">`emit.position`</a> | `[0, 0]` |
| <a href="#particle:emit:size">`emit.size`</a> | `[0, 0]` |
| <a href="#particle:emit:rate">`emit.rate`</a> | `0` |
| <a href="#particle:emit:burst">`emit.burst`</a> | `0` |
| <a href="#particle:emit:max">`emit.max`</a> | `500` |
| <a href="#particle:particle:life">`particle.life`</a> | `1000` |
| <a href="#particle:particle:speed">`particle.speed`</a> | `0` |
| <a href="#particle:particle:angle">`particle.angle`</a> | `90` |
| <a href="#particle:particle:gravity">`particle.gravity`</a> | `[0, 0]` |
| <a href="#particle:particle:friction">`particle.friction`</a> | `0` |
| <a href="#particle:particle:appearance">`particle.start`</a> | 不设置(使用默认外观) |
| <a href="#particle:particle:appearance">`particle.start.size`</a> | `8` |
| <a href="#particle:particle:appearance">`particle.start.alpha`</a> | `1` |
| <a href="#particle:particle:appearance">`particle.start.rotate`</a> | `0` |
| <a href="#particle:particle:appearance">`particle.start.color`</a> | `"#ffffff"` |
| <a href="#particle:particle:appearance">`particle.end`</a> | 不设置(保持初始状态) |

## 典型效果示例

### 下雨
核心逻辑: 屏幕顶部线形发射,高速向下,无重力匀速,半透明,结束前淡出
```json
{
  "image": "raindrop",
  "emit": {
    "shape": "line",
    "position": [640, -50],
    "size": [1400, 0],
    "rate": 300,
    "max": 1000
  },
  "particle": {
    "life": [800, 1200],
    "speed": [800, 1000],
    "angle": [95, 100],
    "start": {
      "size": [2, 4],
      "alpha": [0.4, 0.7]
    },
    "end": {
      "alpha": 0
    }
  }
}
```

### 烟花爆炸
核心逻辑: 单点瞬间爆发,全向发射,受重力下坠,逐渐变小变暗并变色,建议图层设置`cover: "lighter"`实现发光
```json
{
  "image": "spark",
  "emit": {
    "shape": "point",
    "position": [640, 360],
    "burst": 200
  },
  "particle": {
    "life": [1000, 1500],
    "speed": [200, 400],
    "angle": [0, 360],
    "gravity": [0, 150],
    "friction": 0.02,
    "start": {
      "size": [8, 12],
      "color": ["#ffff00", "#ffaa00"]
    },
    "end": {
      "size": [0, 2],
      "alpha": 0,
      "color": "#ff0000"
    }
  }
}
```

### 气泡上浮
核心逻辑: 底部线形发射,缓慢上浮,逐渐变大并淡出
```json
{
  "image": "bubble",
  "emit": {
    "shape": "line",
    "position": [640, 750],
    "size": [1000, 0],
    "rate": 20,
    "max": 100
  },
  "particle": {
    "life": [3000, 5000],
    "speed": [40, 80],
    "angle": [260, 280],
    "gravity": [0, -20],
    "friction": 0.01,
    "start": {
      "size": [10, 20],
      "alpha": [0.3, 0.6]
    },
    "end": {
      "size": [30, 50],
      "alpha": 0
    }
  }
}
```

## 性能建议
同屏粒子数约等于`rate × life(秒) + burst`,应始终不超过`max`
- `rate`过大(如超过500)且`life`较长时,同屏粒子数会大量堆积,引起卡顿
- 引擎在达到`max`后暂停生成而不是丢弃效果,请通过`max`控制上限
- 贴图尺寸尽量小;纯色光点可直接省略`image`,使用内置白圆配合图层`cover: "lighter"`
