---
title: Vue3 vs React
date: 2022-10-31 16:56:00
tags: [总结,Vue,Vue3]
---

问：为什么template可以直接访问setup里的变量

答：`<script setup>` 形式书写的组件模板被编译为了一个内联函数，和 `<script setup>` 中的代码位于同一作用域

vue需要学习额外的知识

如 scoped 无法修改子组件，如要修改，需要添加:deep()

在\<script\>中需要defineEmit的返回值中拿到emit对象，为啥在模板中又不需要？

## 什么是宏？

组件v-model
相对于vue2，vue3 组件v-model默认对应的字段是modelValue与update:modelValue。也可以自定义v-model字段。如v-model:userName对应的字段是userName与update:userName，都是v-model:xxx => xxx => update:xxx的组合。