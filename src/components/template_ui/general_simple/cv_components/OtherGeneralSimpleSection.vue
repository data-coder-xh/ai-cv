<!-- src/components/OtherGeneralSimpleSection.vue -->
<template>
  <section class="other-section session">
    <div class="session-title-and-background">
      <h2 class="session-title" @mouseenter="titleHover = true" @mouseleave="titleHover = false">其他
        <span v-if="titleHover" class="session-title-add-icon" @click="onAddTitleClick">
        <svg enable-background="new 0 0 512 512" fill="var(--color-primary)" height="10px" id="Layer_1" version="1.1"
          viewBox="0 0 512 512" width="10px" xml:space="preserve" xmlns="http://www.w3.org/2000/svg">
          <path d="M256,512C114.625,512,0,397.391,0,256C0,114.609,114.625,0,256,0c141.391,0,256,114.609,256,256  
        C512,397.391,397.391,512,256,512z M256,64C149.969,64,64,149.969,64,256s85.969,192,192,192c106.047,0,192-85.969,192-192  
        S362.047,64,256,64z M288,384h-64v-96h-96v-64h96v-96h64v96h96v64h-96V384z" />
        </svg>
      </span>
      </h2>
    </div>
    <div class="session-title-underline"></div>

    <div 
      class="session-item"
      v-if="!isEmpty()"
      @mouseenter="handleMouseEnter"
      @mouseleave="handleMouseLeave"
      :class="{ 'is-hovered': isHovered }">
      <!-- 灰色蒙层（hover时出现） -->
      <div class="item-hover-overlay" v-if="isHovered && enableHover">
        <!-- 按钮区域 -->
        <div class="overlay-buttons">
          <button class="overlay-button" @click.stop="onEditClick('otherModule', 'otherModule')">编辑</button>
          <button class="overlay-button delete-button" @click.stop="onDeleteClick">删除</button>
        </div>
      </div>
      
      <!-- 所有内容用一个session-item容器包裹 -->
      <div class="item-content">
        <!-- 技能部分 -->
        <div 
          v-if="otherModule.skills && typeof otherModule.skills === 'string' && otherModule.skills.trim()" 
          class="item-content-item">
          <div class="bullet-point-prefix">·</div>
          <div class="bullet-point-content">
            <span class="bullet-point">技能:</span>
            <span class="bullet-content">{{ otherModule.skills }}</span>
          </div>
        </div>
        
        <!-- 证书/执照部分 -->
        <div 
          v-if="otherModule.certificates && typeof otherModule.certificates === 'string' && otherModule.certificates.trim()" 
          class="item-content-item">
          <div class="bullet-point-prefix">·</div>
          <div class="bullet-point-content">
            <span class="bullet-point">证书/执照:</span>
            <span class="bullet-content">{{ otherModule.certificates }}</span>
          </div>
        </div>
        
        <!-- 语言部分 -->
        <div 
          v-if="otherModule.languages && typeof otherModule.languages === 'string' && otherModule.languages.trim()" 
          class="item-content-item">
          <div class="bullet-point-prefix">·</div>
          <div class="bullet-point-content">
            <span class="bullet-point">语言:</span>
            <span class="bullet-content">{{ otherModule.languages }}</span>
          </div>
        </div>
        
        <!-- 兴趣爱好部分 -->
        <div 
          v-if="otherModule.interests && typeof otherModule.interests === 'string' && otherModule.interests.trim()" 
          class="item-content-item">
          <div class="bullet-point-prefix">·</div>
          <div class="bullet-point-content">
            <span class="bullet-point">兴趣爱好:</span>
            <span class="bullet-content">{{ otherModule.interests }}</span>
          </div>
        </div>
        
        <!-- 空状态提示 -->
        <div v-if="isEmpty()" class="item-content-item empty-message">
          <div class="bullet-point-prefix">·</div>
          <div class="bullet-point-content">
            <span class="bullet-point">其他信息:</span>
            <span class="bullet-content">暂无其他信息</span>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  name: "OtherGeneralSimpleSection",
  props: {
    otherModule: {
      type: Object,
      required: true,
      default: () => ({})
    },
    highlightTitle: {
      type: String,
      default: ''
    },
    enableHover: {  
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      titleHover: false,
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

    onAddTitleClick() {
      this.$emit('add-title', 'otherModule');
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
.other-section {
  display: flow-root;
}

.empty-message {
  opacity: 0.5;
}

.session-item {
  position: relative;
}
</style>