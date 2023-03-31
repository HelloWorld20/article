---
title: Vue3 vs React
date: 2022-10-31 16:56:00
tags: [总结,Vue,Vue3]
---

问：为什么template可以直接访问setup里的变量

答：`<script setup>` 形式书写的组件模板被编译为了一个内联函数，和 `<script setup>` 中的代码位于同一作用域

vue需要学习额外的知识，很多很多语法糖。

如 scoped 无法修改子组件，如要修改，需要添加:deep()

在\<script\>中需要defineEmit的返回值中拿到emit对象，为啥在模板中又不需要？

## 什么是宏？

## 组件v-model
相对于vue2，vue3 组件v-model默认对应的字段是modelValue与update:modelValue。也可以自定义v-model字段。如v-model:userName对应的字段是userName与update:userName，都是v-model:xxx => xxx => update:xxx的组合。

多层的组件v-model(待验证)

假设有多层组件 a,b,c

```vue
//Man.vue
<template>
	<Skill v-model="form" />
</template>

<script>
	const form = ref({
		name: 'wei',
		age: 5,
		skill: {
			program: {
				Rust: true,
				React: true,
				Java: false
			}
		}
	})
</script>

// Skill.vue
<template>
	<Detail v-model="skill" />
</template>

<script>
	const props = defineProps(['modelValue']);
	const skill = computed(props.modelValue.skill)
</script>

// Program.vue
<template>
	<input v-model="program.Rust" />
	<input v-model="program.React" />
	<input v-model="program.Java" />
</template>

<script>
	const props = defineProps(['modelValue']);
	const program = computed(props.modelValue.program)
</script>

```

Vue3的语法糖略多，需要背。比如definedProps就有几种写法，ref与reactive，存在功能重合。

Vue3的自动解包有时候会让人迷惑，虽然有了TS，以及熟练度的提升，这个问题可以忽略

Vue3的PascalCase命名方法会容易导致代码难以追踪

自定义指令在某些场景实在是非常的优雅。如element-plus的v-loading。

终于理解为什么React推崇imutable js