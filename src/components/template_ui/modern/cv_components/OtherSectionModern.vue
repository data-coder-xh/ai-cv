<template>
  <div class="section">
    <div class="session-title">
      <img class="icon" src="/icons/fa-star.svg" alt="" /> 其他
    </div>
    
    <!-- 所有内容用一个entry类包裹 -->
    <div 
      v-if="!isEmpty()" 
      class="entry"
      @mouseenter="handleMouseEnter"
      @mouseleave="handleMouseLeave"
      :class="{ 'is-hovered': isHovered }"
      style="position: relative;">
      <!-- 灰色蒙层（hover时出现） -->
      <div class="item-hover-overlay" v-if="isHovered && enableHover">
        <!-- 按钮区域 -->
        <div class="overlay-buttons">
          <button class="overlay-button" @click.stop="onEditClick('otherModule', 'otherModule')">编辑</button>
          <button class="overlay-button delete-button" @click.stop="onDeleteClick">删除</button>
        </div>
      </div>
      <!-- 技能部分 -->
      <div 
        v-if="otherModule.skills && typeof otherModule.skills === 'string' && otherModule.skills.trim()">
        <span class="point-title">技能: </span>
        <span class="point-content">{{ otherModule.skills }}</span>
      </div>
      
      <!-- 证书/执照部分 -->
      <div 
        v-if="otherModule.certificates && typeof otherModule.certificates === 'string' && otherModule.certificates.trim()">
        <span class="point-title">证书/执照: </span>
        <span class="point-content">{{ otherModule.certificates }}</span>
      </div>
      
      <!-- 语言部分 -->
      <div 
        v-if="otherModule.languages && typeof otherModule.languages === 'string' && otherModule.languages.trim()">
        <span class="point-title">语言: </span>
        <span class="point-content">{{ otherModule.languages }}</span>
      </div>
      
      <!-- 兴趣爱好部分 -->
      <div 
        v-if="otherModule.interests && typeof otherModule.interests === 'string' && otherModule.interests.trim()">
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
    },
    enableHover: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      isHovered: false
    }
  },
  methods: {
    // 检查是否所有字段都为空
    isEmpty() {
      const { skills, certificates, languages, interests } = this.otherModule;
      return !skills && !certificates && !languages && !interests;
    },
    onEditClick(type, title) {
      this.$emit('edit-title', type, title);
    },
    onDeleteClick() {
      this.$emit('delete-title', 'otherModule', '');
    },
    handleMouseEnter() {
      if (this.enableHover) {
        this.isHovered = true;
      }
    },
    handleMouseLeave() {
      if (this.enableHover) {
        this.isHovered = false;
      }
    }
  }
};
</script>

<style scoped>
.entry {
  position: relative;
}

.item-hover-overlay {
  position: absolute;
  top: -5px;
  left: -10px;
  width: calc(100% + 20px);
  height: calc(100% + 10px);
  border-radius: 4px;
  z-index: 1;
  background-color: rgba(0, 0, 0, 0.03);
}

.overlay-buttons {
  position: absolute;
  top: 5px;
  right: 10px;
  display: flex;
  gap: 8px;
  z-index: 2;
}

.overlay-button {
  cursor: pointer;
  padding: 4px 8px;
  border: none;
  outline: none;
  background-color: #fff;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  font-size: 12px;
}

.overlay-button:hover {
  background-color: #f5f5f5;
}

.delete-button:hover {
  background-color: #fee;
  color: #d9534f;
}
</style>
