---
title: "Component name should always be multi-word 에러 조치 방법"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- Vue
- error
- component name
- multi-word
---

오늘은 뷰 컴포넌트 명칭 에러 관련하여, 해당 현상이 발생하는 원인과 조치 방법에 대해 알아보도록 하겠습니다.

<!--more-->

### 에러 발생

동영상 강의의 예제 코드를 따라 하던 중 빌드 시 아래와 같은 오류가 발생 하였습니다.

```shell
You may use special comments to disable some warnings.
Use // eslint-disable-next-line to ignore the next line.
Use /* eslint-disable */ to ignore all warnings in a file.
ERROR in [eslint] 
/Users/2carrot/Documents/workspace/vue-advanced/design/slots/src/Item.vue
  11:9  error  Component name "Item" should always be multi-word  vue/multi-word-component-names
```
에러가 발생하는 소스 코드
```vue
<template>
  <li>
    <slot></slot>
  </li>
</template>
<script>
export default {
  name: "Item"
}
</script>
<style scoped>
</style>
```

### 원인
친절한 에러 문구에 나와 있듯이 에러의 원인은 `컴포넌트 명칭이 하나의 단어`로 되어 있기 때문에 발생하고 있었습니다.

기본적으로 뷰에서는 컴포넌트 명칭을 `2가지 이상의 단어 조합을 권장`하고 있다고 합니다.

### 조치 방법
1. 가장 간단한 조치 방법은 단일 단어를 2가지 이상의 단어 조합으로 변경 하면 됩니다.
   > Item -> MyItem

2. ESLint 설정 변경 (lintOnSave:false)
   > vue.config.js 파일에 아래와 같은 설정값을 추가
```javascript
const { defineConfig } = require('@vue/cli-service')
module.exports = defineConfig({
  lintOnSave:false,
})
```

### 마무리
프런트에 대해 얇게 공부를 하고 있지만, 어려운것 같네요. ESLint.. TypeScript... 

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://jeongkyun-it.tistory.com/145](https://jeongkyun-it.tistory.com/145)