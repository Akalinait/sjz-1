<template>
  <div class="guns-list-root">
    <!-- 顶部栏 -->
    <header class="guns-list-header">
      <h1 class="guns-list-title">枪械列表</h1>
      <div class="guns-list-search">
        <input
          type="text"
          v-model="searchName"
          placeholder="搜索枪械名称…"
          class="guns-search-input"
        />
      </div>
    </header>

    <!-- 分类 Tabs -->
    <nav class="guns-list-tabs" v-if="!selectedGun">
      <button
        v-for="c in categories"
        :key="c"
        @click="currentCategory = c"
        :class="['guns-tab', { active: currentCategory === c }]"
      >
        {{ c }}
      </button>
      <button
        @click="currentCategory = '全部'"
        :class="['guns-tab', { active: currentCategory === '全部' }]"
      >全部</button>
    </nav>

    <!-- 列表卡片区 -->
    <section class="guns-list-cards" v-if="!selectedGun">
      <div
        v-for="gun in filteredGuns"
        :key="gun.id"
        class="gun-card"
        @click="selectGun(gun)"
        tabindex="0"
        @keyup.enter="selectGun(gun)"
        role="button"
        :aria-label="`查看${gun.name}详情`"
      >
        <div class="gun-card-header">
          <span class="gun-card-title">{{ gun.name }}</span>
        </div>
        <div class="gun-card-meta">
          <span class="gun-card-category">{{ gun.category }}</span>
        </div>
      </div>
      <div v-if="filteredGuns.length === 0" class="guns-list-empty">
        暂无符合条件的枪械
      </div>
    </section>

    <!-- 枪械详情视图 -->
    <section class="gun-detail-view" v-if="selectedGun">
      <div class="gun-detail-topbar">
        <div class="gun-detail-back-container">
          <button class="gun-detail-back-btn-modern" @click="selectedGun = null" aria-label="返回枪械列表">
            <svg class="back-arrow-icon" viewBox="0 0 24 24">
              <path d="M15 18l-6-6 6-6" stroke="white" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
            <span class="back-text">返回列表</span>
          </button>
        </div>
        <div class="gun-detail-title-wrap-modern">
          <div class="breadcrumb-modern">枪械 / {{ selectedGun.category }}</div>
          <h2 class="gun-detail-title-modern">{{ selectedGun.name }}</h2>
        </div>
      </div>
      <div class="gun-detail-category">分类：{{ selectedGun.category }}</div>
      <div class="gun-detail-codes">
        <div v-if="copyMessage" class="copy-message">{{ copyMessage }}</div>
        <h3>改枪码</h3>
        <ul>
          <li v-for="(item, index) in selectedGun.codes" :key="index" class="gun-code-item">
            <div class="code-card">
              <span class="code-text">{{ item.code }}</span>
              <span class="code-remark" v-if="item.remark">{{ item.remark }}</span>
              <button class="copy-btn" @click.stop="copyCode(item.code)">复制</button>
            </div>
          </li>
        </ul>
      </div>
      <div class="gun-detail-desc" v-if="selectedGun.desc">
        <h3>描述</h3>
        <p>{{ selectedGun.desc }}</p>
      </div>
    </section>
  </div>
</template>

<script>
import { GUNS } from '../data/guns'

export default {
  name: 'GunsList',
  data() {
    return {
      guns: GUNS.map(gun => {
        // 保持 codes 为 [{code, remark}] 格式
        let codes = [];
        if (Array.isArray(gun.codes)) {
          codes = gun.codes.map(item => {
            if (typeof item === 'string') {
              return { code: item, remark: '' }
            } else if (typeof item === 'object' && item !== null) {
              return { code: item.code, remark: item.remark || '' }
            }
            return { code: '', remark: '' }
          });
        } else if (gun.code) {
          codes = [{ code: gun.code, remark: '' }];
        }
        return { ...gun, codes }
      }),
      currentCategory: '全部',
      searchName: '',
      selectedGun: null,
      copyMessage: ''
    }
  },
  computed: {
    categories() {
      return ['突击步枪', '冲锋枪', '通用机枪', '射手步枪', '狙击步枪']
    },
    filteredGuns() {
      let filtered = this.guns
      if (this.currentCategory !== '全部') {
        filtered = filtered.filter(g => g.category === this.currentCategory)
      }
      if (this.searchName.trim() !== '') {
        const keyword = this.searchName.trim().toLowerCase()
        filtered = filtered.filter(g => g.name.toLowerCase().includes(keyword))
      }
      return filtered
    }
  },
  methods: {
    selectGun(gun) {
      this.selectedGun = gun
    },
    copyCode(code) {
      navigator.clipboard.writeText(code)
        .then(() => {
          this.copyMessage = '改枪码已复制'
          setTimeout(() => {
            this.copyMessage = ''
          }, 3000)
        })
        .catch(err => console.error('复制失败', err))
    }
  }
}
</script>

<style scoped>
.guns-list-root {
  max-width: 900px;
  margin: 40px auto 0 auto;
  padding: 24px 18px 40px 18px;
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 2px 16px 0 rgba(0,0,0,0.07), 0 0.5px 2px 0 rgba(0,0,0,0.04);
  min-height: 80vh;
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
}

.guns-list-header {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 18px;
}
.guns-list-title {
  font-size: 2.1rem;
  font-weight: 700;
  color: #23272e;
  margin: 0;
  letter-spacing: 2px;
}
.guns-list-search {
  flex: 1 1 240px;
  max-width: 340px;
  display: flex;
  justify-content: flex-end;
}
.guns-search-input {
  padding: 8px 14px;
  font-size: 1rem;
  border: 1px solid #d6d7db;
  border-radius: 7px;
  width: 100%;
  background: #f7f8fa;
  transition: border 0.2s;
  outline: none;
}
.guns-search-input:focus {
  border-color: #409eff;
  background: #fff;
}

.guns-list-tabs {
  display: flex;
  gap: 10px;
  margin: 0 0 24px 0;
  border-bottom: 1.5px solid #e8e8e8;
  padding-bottom: 10px;
  overflow-x: auto;
}
.guns-tab {
  background: none;
  border: none;
  padding: 8px 22px;
  font-size: 1rem;
  color: #595f6b;
  border-radius: 22px 22px 0 0;
  cursor: pointer;
  transition: color 0.18s, background 0.18s;
  font-weight: 500;
  margin-bottom: -1.5px;
  outline: none;
}
.guns-tab.active {
  background: #f5f8ff;
  color: #206cd6;
  font-weight: 700;
  border-bottom: 2.5px solid #206cd6;
}
.guns-tab:not(.active):hover {
  color: #206cd6;
  background: #f7faff;
}

.guns-list-cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 24px;
  margin-top: 8px;
}
.gun-card {
  background: #f9fafb;
  border-radius: 13px;
  box-shadow: 0 1px 5px 0 rgba(0,0,0,0.04);
  padding: 22px 18px 16px 18px;
  display: flex;
  flex-direction: column;
  transition: box-shadow 0.18s, transform 0.13s;
  border: 1px solid #e7eaf0;
  min-height: 120px;
  position: relative;
  cursor: pointer;
  user-select: none;
}
.gun-card:hover,
.gun-card:focus {
  box-shadow: 0 4px 18px 0 rgba(32,108,214,0.10), 0 1.5px 6px 0 rgba(0,0,0,0.06);
  transform: translateY(-2px) scale(1.015);
  border-color: #b7d2fa;
  outline: none;
}
.gun-card-header {
  display: flex;
  align-items: baseline;
  justify-content: flex-start;
  margin-bottom: 7px;
}
.gun-card-title {
  font-size: 1.18rem;
  font-weight: 600;
  color: #22304a;
  letter-spacing: .5px;
}
.gun-card-meta {
  font-size: 0.98rem;
  color: #206cd6;
  margin-bottom: 5px;
  font-weight: 500;
}
.guns-list-empty {
  grid-column: 1/-1;
  color: #a0a4b0;
  font-size: 1.14rem;
  text-align: center;
  margin: 36px 0 0 0;
  letter-spacing: 1px;
}

/* 枪械详情视图 */
.gun-detail-view {
  padding: 16px 12px;
}
.gun-detail-title {
  font-size: 2rem;
  font-weight: 700;
  color: #23272e;
  margin: 0 0 12px 0;
  letter-spacing: 1.5px;
}
.gun-detail-category {
  font-size: 1.1rem;
  color: #206cd6;
  font-weight: 600;
  margin-bottom: 20px;
}
.gun-detail-codes h3,
.gun-detail-desc h3 {
  font-size: 1.2rem;
  margin-bottom: 8px;
  color: #22304a;
}
.gun-detail-codes ul {
  list-style: disc inside;
  margin: 0 0 20px 0;
  padding-left: 18px;
  color: #3e4653;
  font-size: 1rem;
}
.gun-code-item {
  padding: 2px 0;
  font-family: 'Consolas', 'Menlo', monospace;
}
.gun-detail-desc p {
  font-size: 1rem;
  color: #3e4653;
  line-height: 1.5;
  margin: 0;
}

@media (max-width: 600px) {
  .guns-list-root {
    padding: 10px 2vw 24px 2vw;
    min-height: 60vh;
  }
  .guns-list-header {
    flex-direction: column;
    align-items: stretch;
    gap: 10px;
  }
  .guns-list-title {
    font-size: 1.35rem;
    text-align: left;
  }
  .guns-list-search {
    max-width: 100%;
  }
  .guns-list-cards {
    grid-template-columns: 1fr;
    gap: 14px;
  }
}

.gun-detail-topbar {
  display: flex;
  align-items: center;
  gap: 18px;
  padding: 12px 8px 20px 8px;
  border-bottom: 1px solid #eef0f4;
  margin-bottom: 24px;
}

.gun-detail-back {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.95rem;
  color: #206cd6;
  cursor: pointer;
  user-select: none;
  padding: 6px 10px;
  border-radius: 8px;
  transition: background 0.2s, transform 0.15s;
}

.gun-detail-back:hover {
  background: #f5f8ff;
  transform: translateX(-2px);
}

.back-arrow {
  font-size: 1.2rem;
  font-weight: 600;
}

.back-text {
  font-weight: 500;
}

.gun-detail-title-wrap {
  display: flex;
  flex-direction: column;
}

.gun-detail-title {
  font-size: 1.9rem;
  font-weight: 700;
  margin: 0;
  color: #23272e;
}

.breadcrumb {
  font-size: 0.85rem;
  color: #8a90a0;
  margin-bottom: 2px;
}
  .gun-detail-back-btn-advanced .back-arrow svg {
    display: block;
    width: 20px;
    height: 20px;
  }

  .gun-detail-back-btn-advanced:hover .back-arrow svg path {
    stroke: #fff;
    stroke-width: 2.5;
  }
/* 改枪码卡片样式 */
.code-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #f5f7fa;
  padding: 6px 12px;
  border-radius: 8px;
  margin-bottom: 6px;
}
.code-text {
  font-family: 'Consolas', monospace;
  font-weight: 600;
}
.code-remark {
  font-size: 0.85rem;
  color: #8a90a0;
  margin-left: 10px;
}
.copy-btn {
  background: #206cd6;
  color: #fff;
  border: none;
  border-radius: 6px;
  padding: 4px 8px;
  cursor: pointer;
  font-size: 0.9rem;
}
.copy-btn:hover {
  background: #1a5fcc;
}
  .copy-message {
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(32,108,214,0.95);
    color: #fff;
    padding: 8px 16px;
    border-radius: 8px;
    font-size: 0.95rem;
    z-index: 999;
    box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    transition: opacity 0.3s ease;
  }
</style>

.gun-detail-back-btn-modern {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 12px 20px;
  border-radius: 50px;
  border: none;
  background: linear-gradient(135deg, #4f95f5, #1c64f2);
  color: #fff;
  font-weight: 700;
  font-size: 1rem;
  cursor: pointer;
  box-shadow: 0 10px 28px rgba(32,108,214,0.35);
  transition: transform 0.2s, box-shadow 0.25s, background 0.25s;
  position: relative;
  overflow: hidden;
}
.gun-detail-back-btn-modern:hover {
  transform: translateY(-3px) scale(1.07);
  box-shadow: 0 14px 40px rgba(32,108,214,0.45);
  background: linear-gradient(135deg, #1c64f2, #4f95f5);
}
.back-arrow-icon {
  width: 24px;
  height: 24px;
}
.back-text {
  font-weight: 700;
  font-size: 1rem;
  letter-spacing: 0.5px;
}

/* 二级标题增强 */
.gun-detail-title-wrap-modern {
  display: flex;
  flex-direction: column;
}
.breadcrumb-modern {
  font-size: 0.9rem;
  color: #8a90a0;
  margin-bottom: 4px;
}
.gun-detail-title-modern {
  font-size: 2.4rem;
  font-weight: 800;
  color: #1f2a49;
  margin: 0;
  letter-spacing: 1.5px;
}

.gun-detail-back-btn-super {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 20px;
  border-radius: 60px;
  border: none;
  background: linear-gradient(135deg, #4f95f5, #1c64f2);
  color: #fff;
  font-weight: 700;
  font-size: 1.05rem;
  cursor: pointer;
  box-shadow: 0 8px 24px rgba(32,108,214,0.35);
  transition: transform 0.2s, box-shadow 0.25s, background 0.25s;
  position: relative;
  overflow: hidden;
}
.gun-detail-back-btn-super::after {
  content: '';
  position: absolute;
  top: 0; left: -50%;
  width: 200%;
  height: 100%;
  background: linear-gradient(120deg, rgba(255,255,255,0.15) 0%, rgba(255,255,255,0.0) 50%, rgba(255,255,255,0.15) 100%);
  transform: skewX(-20deg) translateX(-100%);
  transition: transform 0.6s;
}
.gun-detail-back-btn-super:hover::after {
  transform: skewX(-20deg) translateX(100%);
}
.gun-detail-back-btn-super:hover {
  transform: translateY(-3px) scale(1.07);
  box-shadow: 0 12px 36px rgba(32,108,214,0.45);
  background: linear-gradient(135deg, #1c64f2, #4f95f5);
}
.gun-detail-back-btn-super .back-arrow svg {
  width: 24px;
  height: 24px;
}
.gun-detail-back-btn-super .back-text {
  font-weight: 700;
  letter-spacing: 0.5px;
}

.gun-detail-back-btn-advanced {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 16px;
  border-radius: 50px;
  border: none;
  background: linear-gradient(135deg, #4f95f5, #1c64f2);
  color: #fff;
  font-weight: 600;
  font-size: 1rem;
  cursor: pointer;
  box-shadow: 0 6px 20px rgba(32,108,214,0.35);
  transition: transform 0.15s, box-shadow 0.2s, background 0.2s;
}
.gun-detail-back-btn-advanced:hover {
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 8px 24px rgba(32,108,214,0.45);
  background: linear-gradient(135deg, #1c64f2, #4f95f5);
}
.back-arrow {
  font-size: 1.3rem;
}
.back-text {
  font-weight: 600;
}

.gun-detail-back-container {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 16px;
}

.gun-detail-back-btn {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  border: 1px solid #e1e6ef;
  background: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.2s, box-shadow 0.2s, transform 0.15s;
}

.gun-detail-back-btn:hover {
  background: #f5f8ff;
  box-shadow: 0 4px 12px rgba(32,108,214,0.15);
  transform: translateY(-1px);
}

.back-arrow {
  font-size: 1.2rem;
  color: #206cd6;
  font-weight: 600;
}

.back-text {
  font-size: 1rem;
  font-weight: 500;
  color: #206cd6;
}