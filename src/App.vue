<template>
  <div class="cart-container">
    <!-- 头部 -->
    <div class="cart-header">
      <h1>购物车 ({{ cartItems.length }})</h1>
      <div class="actions">
        <span>对比</span>
        <span>管理</span>
      </div>
    </div>

    <!-- 优惠券提示 -->
    <div class="coupon-bar">
      <span>💴</span>
      <span>您有2张共168元消费券待使用，仅剩23:33:44</span>
    </div>

    <!-- 标签栏 -->
    <div class="tabs">
      <div
        v-for="tab in tabs"
        :key="tab"
        class="tab-item"
        :class="{ active: activeTab === tab }"
        @click="activeTab = tab"
      >
        {{ tab }}
      </div>
    </div>

    <!-- 按店铺分组的商品列表 -->
    <div v-for="shop in shops" :key="shop.id" class="shop-section">
      <div class="shop-header">
        <div
          class="checkbox"
          :class="{ checked: isShopAllChecked(shop.id) }"
          @click="toggleShopCheck(shop.id)"
        ></div>
        <span class="shop-name">{{ shop.name }}</span>
        <span v-if="shop.tag" class="shop-tag">{{ shop.tag }}</span>
      </div>

      <div
        v-for="item in shop.items"
        :key="item.id"
        class="cart-item"
      >
        <div
          class="checkbox"
          :class="{ checked: item.checked, disabled: item.stock === 0 }"
          :style="{ opacity: item.stock === 0 ? 0.5 : 1 }"
          @click="toggleItemCheck(item)"
        ></div>
        <img :src="item.image" :alt="item.title" class="item-img" />
        <div class="item-info">
          <div class="item-title">{{ item.title }}</div>
          <div class="item-spec" v-if="item.spec">{{ item.spec }}</div>
          <div class="item-tags">
            <span
              v-for="tag in item.tags"
              :key="tag.text"
              class="tag"
              :class="{ gray: tag.gray }"
            >
              {{ tag.text }}
            </span>
          </div>

          <!-- 修复核心：把v-if和v-else放在同一层级且直接相邻 -->
          <div v-if="item.stock > 0">
            <div class="item-price-row">
              <div>
                <span class="item-price">¥{{ item.price }}</span>
                <span v-if="item.originalPrice" class="item-original-price">¥{{ item.originalPrice }}</span>
              </div>
              <span class="item-count">×{{ item.count }}</span>
            </div>
            <div v-if="item.tax" class="item-tax">{{ item.tax }}</div>
            <div class="item-detail">明细 ></div>
          </div>
          <div v-else>
            <div class="out-of-stock">所选款式缺货，请重新选择</div>
            <div class="reselect">重选 ></div>
          </div>
        </div>
      </div>
    </div>

    <!-- 底部结算栏 -->
    <div class="cart-footer">
      <div class="select-all" @click="toggleAllCheck">
        <div
          class="checkbox"
          :class="{ checked: isAllChecked }"
        ></div>
        <span>全选</span>
      </div>
      <div class="total-price">合计: <span>¥{{ totalPrice.toFixed(2) }}</span></div>
      <button class="checkout-btn">结算</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// 标签栏数据
const tabs = ref(['消费券', '官方立减', '降价', '分组', '筛选'])
const activeTab = ref('消费券')

// 购物车数据
const shops = ref([
  {
    id: 1,
    name: '国际 天猫国际自营全球超级店',
    tag: '9.9起换购',
    items: [
      {
        id: 101,
        title: '38焕新周 【自营】Lamer海蓝150ml',
        image: 'https://via.placeholder.com/80',
        price: 719,
        originalPrice: 1000,
        count: 1,
        tax: '进口税:价格已含税',
        stock: 1,
        checked: false,
        tags: [
          { text: '官方立减120元', gray: false },
          { text: '直降161元', gray: false },
          { text: '退货宝', gray: true }
        ]
      }
    ]
  },
  {
    id: 2,
    name: '国际 全球超值购',
    tag: '',
    items: [
      {
        id: 201,
        title: 'SK-II神仙水乳液清莹露套装',
        spec: '1盒;基本款',
        image: 'https://via.placeholder.com/80',
        price: 1659,
        count: 1,
        tax: '进口税:价格已含税',
        stock: 1,
        checked: false,
        tags: [
          { text: '7天价保', gray: true },
          { text: '退货宝', gray: true },
          { text: '超级爆款', gray: true },
          { text: '假一赔十', gray: true }
        ]
      }
    ]
  },
  {
    id: 3,
    name: '国际 DonCoo海外旗舰店',
    tag: '',
    items: [
      {
        id: 301,
        title: '三浦制药DonCoo美体丸',
        image: 'https://via.placeholder.com/80',
        price: 0,
        count: 1,
        stock: 0,
        checked: false,
        tags: [
          { text: '不支持7天无理由退货', gray: true },
          { text: '正品保障', gray: true }
        ]
      }
    ]
  },
  {
    id: 4,
    name: '淘工厂 天天特卖工厂',
    tag: '',
    items: [
      {
        id: 401,
        title: '超级快充自带线充电宝',
        image: 'https://via.placeholder.com/80',
        price: 0,
        count: 1,
        stock: 1,
        checked: false,
        tags: []
      }
    ]
  }
])

// 计算所有商品列表，用于全选和总价计算
const cartItems = computed(() => shops.value.flatMap(shop => shop.items))

// 计算总价
const totalPrice = computed(() => {
  return cartItems.value
    .filter(item => item.checked && item.stock > 0)
    .reduce((sum, item) => sum + item.price * item.count, 0)
})

// 判断是否全选
const isAllChecked = computed(() => {
  const availableItems = cartItems.value.filter(item => item.stock > 0)
  return availableItems.length > 0 && availableItems.every(item => item.checked)
})

// 切换单个商品选中状态
const toggleItemCheck = (item) => {
  if (item.stock === 0) return
  item.checked = !item.checked
}

// 切换店铺全选
const toggleShopCheck = (shopId) => {
  const shop = shops.value.find(s => s.id === shopId)
  if (!shop) return
  const availableItems = shop.items.filter(item => item.stock > 0)
  const allChecked = availableItems.every(item => item.checked)
  availableItems.forEach(item => {
    item.checked = !allChecked
  })
}

// 判断店铺是否全选
const isShopAllChecked = (shopId) => {
  const shop = shops.value.find(s => s.id === shopId)
  if (!shop) return false
  const availableItems = shop.items.filter(item => item.stock > 0)
  return availableItems.length > 0 && availableItems.every(item => item.checked)
}

// 全选/取消全选
const toggleAllCheck = () => {
  const availableItems = cartItems.value.filter(item => item.stock > 0)
  const allChecked = isAllChecked.value
  availableItems.forEach(item => {
    item.checked = !allChecked
  })
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}
.cart-container {
  max-width: 428px;
  margin: 0 auto;
  background-color: #fff;
  min-height: 100vh;
}
.cart-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 12px;
  background-color: #fff;
  border-bottom: 1px solid #eee;
  position: sticky;
  top: 0;
  z-index: 10;
}
.cart-header h1 {
  font-size: 18px;
  font-weight: 600;
}
.cart-header .actions {
  display: flex;
  gap: 16px;
  font-size: 15px;
  color: #333;
}
.coupon-bar {
  background-color: #fff3f3;
  color: #ff4444;
  padding: 8px 12px;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 4px;
}
.tabs {
  display: flex;
  background-color: #fff;
  padding: 12px 0;
  gap: 16px;
  overflow-x: auto;
  border-bottom: 1px solid #eee;
}
.tab-item {
  white-space: nowrap;
  padding: 4px 12px;
  font-size: 14px;
  color: #666;
  border-radius: 16px;
  background-color: #f5f5f5;
  cursor: pointer;
}
.tab-item.active {
  background-color: #ff4444;
  color: #fff;
}
.shop-section {
  margin-top: 8px;
  background-color: #fff;
}
.shop-header {
  display: flex;
  align-items: center;
  padding: 12px;
  gap: 8px;
  border-bottom: 1px solid #f5f5f5;
}
.shop-name {
  font-size: 16px;
  font-weight: 500;
  color: #666;
  flex: 1;
}
.shop-tag {
  font-size: 12px;
  color: #999;
  padding: 2px 6px;
  border: 1px solid #eee;
  border-radius: 4px;
}
.cart-item {
  display: flex;
  padding: 12px;
  gap: 12px;
  border-bottom: 1px solid #f5f5f5;
}
.checkbox {
  width: 20px;
  height: 20px;
  border: 1px solid #ddd;
  border-radius: 50%;
  margin-top: 20px;
  flex-shrink: 0;
  cursor: pointer;
}
.checkbox.checked {
  background-color: #ff4444;
  border-color: #ff4444;
  position: relative;
}
.checkbox.checked::after {
  content: "✓";
  color: #fff;
  font-size: 12px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.checkbox.disabled {
  cursor: not-allowed;
}
.item-img {
  width: 80px;
  height: 80px;
  background-color: #f5f5f5;
  border-radius: 8px;
  object-fit: cover;
}
.item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.item-title {
  font-size: 15px;
  color: #333;
  line-height: 1.4;
}
.item-spec {
  font-size: 12px;
  color: #999;
}
.item-tags {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}
.tag {
  font-size: 10px;
  color: #ff4444;
  border: 1px solid #ff4444;
  padding: 1px 4px;
  border-radius: 2px;
}
.tag.gray {
  color: #999;
  border-color: #eee;
}
.item-price-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.item-price {
  font-size: 16px;
  font-weight: 600;
  color: #ff4444;
}
.item-original-price {
  font-size: 12px;
  color: #999;
  text-decoration: line-through;
  margin-left: 4px;
}
.item-count {
  font-size: 14px;
  color: #666;
}
.item-tax {
  font-size: 12px;
  color: #999;
}
.item-detail {
  font-size: 13px;
  color: #ff6600;
  text-align: right;
  cursor: pointer;
}
.out-of-stock {
  color: #ff4444;
  font-size: 14px;
  margin-top: 4px;
}
.reselect {
  font-size: 13px;
  color: #ff6600;
  text-align: right;
  cursor: pointer;
}
.cart-footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  max-width: 428px;
  margin: 0 auto;
  background-color: #fff;
  border-top: 1px solid #eee;
  display: flex;
  align-items: center;
  padding: 8px 12px;
}
.select-all {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  color: #333;
  cursor: pointer;
}
.total-price {
  flex: 1;
  text-align: right;
  font-size: 16px;
  font-weight: 600;
  color: #333;
}
.checkout-btn {
  background-color: #ff4444;
  color: #fff;
  border: none;
  padding: 10px 24px;
  border-radius: 20px;
  font-size: 15px;
  font-weight: 500;
  margin-left: 12px;
  cursor: pointer;
}
</style>