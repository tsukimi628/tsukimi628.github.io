---
title: TypeScript在Vue3 CompositionAPI组件中的运用
date: 2022-09-01 12:06:43
tags:
- TypeScript
- ts
- Vue3
---
### 为 props 标注类型
#### 使用defineProps宏函数
1. 结合泛型参数作类型声明
```ts
<script setup lang="ts">
const props = defineProps<{
    foo: string,
    bar?: number
}>()

// 或
interface Props {
    foo: string,
    bar?: number
}
const props = defineProps<Props>()
</script>
```
基于类型的声明，编译器会尽可能地尝试根据类型参数推导出等价的运行时选项。为了能定义props默认值，可以使用withDefaults编译器宏：
```ts
interface Props {
    msg?: string,
    labels?: string[]
}
const props = withDefaults(defineProps<Props>(), {
    msg: 'hello',
    labels: () => ['world','js']
})
```
以上代码会被编译为等价的运行时props的default选项

2. 运行时声明
```ts
<script setup lang="ts">
const props = defineProps({
    foo: {type: String, required: true},
    bar: Number
})
</script>
```
这里传给defineProps()的参数会作为运行时的props选项使用

### 为 emits 标注类型
```ts
<script setup lang="ts">
// 基于类型的声明
const emit = defineEmits<{
    (e: 'change', id: number): void
    (e: 'update', value: string): void
}>()

// 运行时
const emit = defineEmits(['change','update'])
</script>
```

### 为 ref() 标注类型
ref会根据初始化时的值自动推导其类型
```ts
<script setup lang="ts">
// 将被自动推导为number类型 Ref<number>
const num = ref(0)
</script>
```
有时可能需要为ref内的值指定一个更复杂的类型，可以通过`Ref接口`实现：
```ts
<script setup lang="ts">
import { ref } from 'vue'
import type { Ref } from 'vue'
const input: Ref<string | number> = ref('123')
input.value = 123 // 再赋值为数字123也是可行的
</script>
```
又如，调用ref()时传入一个泛型参数，来覆盖默认的推导行为
```ts
<script setup lang="ts">
// 推导类型为Ref<string | number>
const input = ref<string | number>('123')
input.value = 123

//推导类型为Ref<string | undefined>
const params = ref<string>()
</script>
```

### 为 reactive() 标注类型
reactive() 也会隐式地从它的参数中推导类型
```ts
<script setup lang="ts">
import { reactive } from 'vue'
// 推导得到的类型: {title: string}
const book = reactive({title:'权威指南'})
</script>
```
通过接口显示指定一个reactive变量的类型
```ts
<script setup lang="ts">
import { reactive } from 'vue'
interface Book {
    title: string,
    price?: number
}
const book:Book = reactive({title: '权威指南', price: 123.00})
</script>
```
### 为 computed() 标注类型
computed()会自动从其计算函数的返回值上推导出类型：
```ts
<script setup lang="ts">
const { ref, computed } from 'vue'
const price = ref(123)
// 推导得到的类型：ComputedRef<string>
const priceShow = computed(() => `¥${price.value.toFix(2)}元`)
// string
console.log(typeof priceShow.value)
</script>
```
通过泛型参数显示指定类型：
```ts
<script setup lang="ts">
const priceShow = computed<string>(() => {
    // 如返回值不是string类型会报错
})
</script>
```

### 为事件处理函数标注类型
在处理原生DOM事件时，应该给事件处理函数的参数正确地标注类型
```HTML
<template>
    <input @change="handleChange">
</template>
<script setup lang="ts">
function handleChange(e){
    // e 被隐式标注为any类型
    console.log(e.target.value)
}
</script>
```
如果`tsconfig.json`中配置了 "strict":true 或 "noImplicitAny":true(不允许隐式的any类型推导)，将会报TS错误。因此建议显示声明其类型：
```ts
function handleChange(e:Event){
    // 类型断言---as: 把一个大的范围断言成小的、精确的范围
    console.log((e.target as HTMLInputElement).value)
}
```

### 为 provide/inject 标注类型
provide和inject通常会在不同的组件中运行。要正确地为注入的值标记类型，Vue提供了一个InjectionKey接口，它是一个继承自Symobl的泛型类型，可以用来在提供者和消费者间同步注入值的类型：
```ts
// 组件1
<script setup lang="ts">
import { provide } from 'vue'
import type { InjectionKey } from 'vue'
const key = Symbol() as InjectionKey<string>
const title = ref('权威指南')
// 提供的响应式状态使后代组件可以由此和提供者建立响应式的联系
provide(key, title) // 若提供的非字符串值会导致错误
</script>
// 组件2
<script setup lang="ts">
import { inject } from 'vue'
const foo = inject(key) // foo 的类型：string|undefined
</script>
```
优化：将注入的key的类型放在一个单独的文件中，这样可以被多个组件导入

### 为dom模板引用标注类型
模板ref需要通过一个显式指定的泛型参数和一个初始值null来创建：
```ts
<template>
    <input ref="el" />
</template>
<script setup lang="ts">
import { ref, onMounted } from 'vue'
const el = ref<HTMLInputElement | null>(null)
onMounted(() => {
    el.value?.focus()
})
</script>
```
注意为了严格的类型安全，有必要在访问 el.value 时使用可选链或类型守卫(in 操作符)。这是因为直到组件被挂载前，这个 ref 的值都是初始的 null，并且在由于 v-if 的行为将引用的元素卸载时也可以被设置为 null。

### 为组件模板引用标注类型
例如，某些场景下需要调用子组件公开的方法
```ts
// 子组件
<script setup lang="ts">
import { ref } from 'vue'

const isShow = ref(false)
const open = () => (isShow.value = true)

// 通过 defineExpose 宏显式暴露 open
defineExpose({
    open
})
</script>
```

```ts
// 父组件
<template>
    <child-comp ref="child"></child-comp>
</template>
<script setup lang="ts">
// 
import { ref } from 'vue'
import ChildComp from '@/components/xxx'

// 获取组件实例
// 为获取 ChildComp 的类型，首先通过 typeof 得到其类型，再使用TypeScript内置的InstanceType工具类型来获取其实例类型：
const child = ref<InstanceType<typeof ChildComp> | null>(null)
const openChildComp = () => {
    child.value?.open()
}
</script>
```


[参考：https://cn.vuejs.org/guide/typescript/composition-api.html](https://cn.vuejs.org/guide/typescript/composition-api.html)