<!-- src/components/CreativeModernOtherSection.vue -->
<template>
  <section v-if="!isEmpty()" class="other-section session">
    <div class="session-title">其他</div>
    
    <!-- 所有内容用一个entry类包裹 -->
    <div class="entry">
      <!-- 技能部分 -->
      <div v-if="otherModule.skills && typeof otherModule.skills === 'string' && otherModule.skills.trim()" class="item-content-item">
        <span class="point-title">技能: </span>
        <span class="point-content">{{ otherModule.skills }}</span>
      </div>
      
      <!-- 证书/执照部分 -->
      <div v-if="otherModule.certificates && typeof otherModule.certificates === 'string' && otherModule.certificates.trim()" class="item-content-item">
        <span class="point-title">证书/执照: </span>
        <span class="point-content">{{ otherModule.certificates }}</span>
      </div>
      
      <!-- 语言部分 -->
      <div v-if="otherModule.languages && typeof otherModule.languages === 'string' && otherModule.languages.trim()" class="item-content-item">
        <span class="point-title">语言: </span>
        <span class="point-content">{{ otherModule.languages }}</span>
      </div>
      
      <!-- 兴趣爱好部分 -->
      <div v-if="otherModule.interests && typeof otherModule.interests === 'string' && otherModule.interests.trim()" class="item-content-item">
        <span class="point-title">兴趣爱好: </span>
        <span class="point-content">{{ otherModule.interests }}</span>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  name: "CreativeModernOtherSection",
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

<style scoped>
.other-section {
  display: flow-root;
}

.entry {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.item-content-item {
  display: flex;
  align-items: baseline;
  gap: 8px;
}

.point-title {
  font-weight: bold;
  color: var(--custom-color);
  min-width: 80px;
}

.point-content {
  flex: 1;
}
</style>