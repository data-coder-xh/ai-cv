<template>
  <div class="section">
    <div class="session-title">
      <img class="icon" src="/icons/fa-star.svg" alt="" /> 其他
    </div>
    
    <!-- 所有内容用一个entry类包裹 -->
    <div v-if="!isEmpty()" class="entry">
      <!-- 技能部分 -->
      <div v-if="otherModule.skills && otherModule.skills.trim()">
        <span class="point-title">技能: </span>
        <span class="point-content">{{ otherModule.skills }}</span>
      </div>
      
      <!-- 证书/执照部分 -->
      <div v-if="otherModule.certificates && otherModule.certificates.trim()">
        <span class="point-title">证书/执照: </span>
        <span class="point-content">{{ otherModule.certificates }}</span>
      </div>
      
      <!-- 语言部分 -->
      <div v-if="otherModule.languages && otherModule.languages.trim()">
        <span class="point-title">语言: </span>
        <span class="point-content">{{ otherModule.languages }}</span>
      </div>
      
      <!-- 兴趣爱好部分 -->
      <div v-if="otherModule.interests && otherModule.interests.trim()">
        <span class="point-title">兴趣爱好: </span>
        <span class="point-content">{{ otherModule.interests }}</span>
      </div>
    </div>
    
    <!-- 如果所有字段都为空，则显示提示信息 -->
    <div v-if="isEmpty()" class="summary empty-message">
      暂无其他信息
    </div>
  </div>
</template>

<script>
export default {
  name: "OtherSectionModern",
  props: {
    otherModule: {
      type: Object,
      required: true,
      default: () => ({})
    }
  },
  methods: {
    // 检查是否所有字段都为空
    isEmpty() {
      const { skills, certificates, languages, interests } = this.otherModule;
      return !skills && !certificates && !languages && !interests;
    }
  }
};
</script>
